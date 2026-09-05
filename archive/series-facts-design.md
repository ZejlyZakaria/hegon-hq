# Design — `watching.series_facts` (table de référence partagée)

> Opus · 2026-07-15 · **Design, non appliqué.** Fait suite à l'audit `db-schema-audit.md` (point B).
> ✅ **Revu par Fable (2ᵉ avis) — corrections intégrées ci-dessous.** Verdict Fable : design ~85 % → solide
> après 6 corrections ; **différer l'implémentation** jusqu'au déclencheur (voir §8).

## ⚠️ Corrections post-revue Fable (intégrées dans le doc)
1. **🔴 Ordre des phases INVERSÉ** — dual-**write** doit précéder la bascule **lecture** (sinon
   `series_facts` = snapshot figé pendant que le sync rafraîchit encore `media_items` → app lit du
   périmé). §7 corrigé (expand → migrate → contract).
2. **🔴 Garde `type <> 'film'` au merge** — ids TMDB movie/tv peuvent coïncider numériquement ; la Library
   mêle les types → un merge naïf par `tmdb_id` collerait des saisons sur un film. §5 corrigé.
3. **🟠 Split de types TS** (`MediaRow` sans facts vs `WatchingMedia = MediaRow & SeriesFacts`) → le
   compilateur force l'hydratation, la discipline ne suffit pas. §5.
4. **🟠 DDL à compléter** : index de sync partiel SUR `series_facts` (les 2 colonnes du
   `media_items_sync_idx` déménagent → il devient orphelin, à dropper en phase 4 + amender la baseline) ;
   `updated_at` explicite dans l'upsert ; `GRANT` sans `anon` si aucun chemin anonyme. §3.
5. **🟠 `status` n'est PAS série-only** — le flux d'ajout écrit un status brut pour les FILMS
   (`"released"`…). Auditer son usage AVANT la phase 4 (sinon drop = perte de donnée film). §2.
6. **🟡 Orphelins + backfill** : le sync ne doit cibler que les séries **encore possédées**
   (`WHERE EXISTS … media_items`) sinon croissance non bornée « forever » ; backfill = préférer les facts
   **non-nuls** (pas juste `max(last_synced_at)`) + `CHECK (type = 'film' OR tmdb_id IS NOT NULL)`. §6/§7.
- **Non-problèmes confirmés par Fable** : poster custom per-user (reste inline, intact), unique
  `(user_id,tmdb_id,type)` (indifférent au split), `season_years`/`caught_up_at` per-user vs `season_aired`
  mondial (*plus* cohérent après split), RLS façon `trending_cache` (bon modèle).
- **À NE PAS faire (sur-engineering, Fable d'accord)** : vue SQL/matérialisée maintenant, FK+embed
  PostgREST, déménager l'identité statique, `person_credits`.

---

## 1. Le problème, en une phrase

Les **faits du monde** d'une série (ce que TMDB sait : statut, épisodes diffusés, dates…) sont stockés
**une fois par utilisateur qui possède le titre** (×1.95 aujourd'hui). À 10 000 users c'est des millions
de copies, un job de sync qui réécrit N fois la même vérité, et une **fenêtre de divergence** (une copie
fraîche, une autre périmée) entre deux syncs.

## 2. La décision qui structure tout : QUOI on déplace

C'est là que se joue le sur-mesure vs le sur-dimensionné. Toutes les colonnes de `media_items` ne se
valent pas.

**✅ CE QUI DÉMÉNAGE (le vrai mal — volatil, synchronisé, sujet à divergence) — 9 colonnes :**
`status`, `seasons`, `episodes`, `season_episodes`, `season_aired`, `season_air_dates`,
`season_end_dates`, `season_posters`, `last_synced_at`.
→ Ce sont **exactement** les colonnes que le job de sync réécrit, identiques pour tous les possesseurs,
et la source de toute divergence. C'est aussi la surface d'accroche de l'overlay AniList (anime v2).

**❌ CE QUI RESTE inline dans `media_items` (phase 1) — l'identité STATIQUE :**
`title`, `original_title`, `year`, `description`, `backdrop_url`, `poster_url`, `rating`, `studio`,
`runtime`, `directors`, `cast_members`, `tags`.
→ Raisons : (a) ça ne **change jamais** → aucune divergence à corriger, aucun coût de sync ; (b)
`poster_url` peut être **surchargé par l'utilisateur** (upload custom) → ce n'est PAS purement du monde ;
(c) le déplacer n'apporte que de l'économie de stockage (bon marché) contre une **jointure sur CHAQUE
affichage**. Mauvais ratio en phase 1.

**❌ HORS PÉRIMÈTRE — les films.** Un film n'a ni saisons, ni sync, ni statut qui bouge. Sa « duplication »
est du stockage statique pur (poster + titre) = zéro douleur à l'échelle. On n'y touche pas.

> **Principe :** on normalise ce qui **change et se duplique dangereusement**, pas ce qui est simplement
> répété mais figé. Déplacer un `poster_url` immuable = de l'over-engineering.

## 3. La table

```sql
create table watching.series_facts (
  tmdb_id           integer     primary key,     -- id TMDB dans le namespace « tv » (série ET anime le partagent)

  status            varchar(20),                 -- ended | ongoing | canceled
  seasons           integer,
  episodes          integer,
  season_episodes   jsonb       not null default '[]'::jsonb,  -- ANNONCÉ
  season_aired      integer[],                                 -- DIFFUSÉ (seule source du progrès)
  season_air_dates  jsonb       not null default '[]'::jsonb,
  season_end_dates  jsonb       not null default '[]'::jsonb,
  season_posters    jsonb       not null default '[]'::jsonb,

  last_synced_at    timestamptz,
  updated_at        timestamptz not null default now()   -- à poser explicitement dans le DO UPDATE de l'upsert
);

-- Index de sync (remplace media_items_sync_idx qui devient orphelin en phase 4) :
-- les séries pas terminées, refresh le plus vieux d'abord.
create index series_facts_sync_idx on watching.series_facts (last_synced_at nulls first)
  where status is distinct from 'ended' and status is distinct from 'canceled' and status is distinct from 'cancelled';
```

**Clé = `tmdb_id` seul.** Les ids de films (namespace « movie ») ne rentrent jamais ici → pas de
collision. Une série et un anime qui partagent le même `tmdb_id` (même show, classé différemment par 2
users) partagent **la même ligne de faits** — ce qui est exactement voulu.

## 4. RLS — c'est de la RÉFÉRENCE GLOBALE, pas du per-tenant

Modèle **identique à `trending_cache`** (déjà en place et éprouvé) : donnée publique TMDB, la même pour
tout le monde.

```sql
alter table watching.series_facts enable row level security;
create policy "series_facts_read" on watching.series_facts for select using (true);
grant select on watching.series_facts to anon, authenticated;
-- Écritures : UNIQUEMENT via le service role (job de sync + flux d'ajout) → bypass RLS.
-- Pas de policy insert/update pour les users : ils ne peuvent pas altérer les faits du monde.
```

→ **Pas d'`org_id`, pas de duplication par tenant.** Une série = une ligne, point.

## 5. Le chemin de LECTURE change (le vrai coût du refactor)

Aujourd'hui `getMediaItems` lit `season_aired`, `status`, etc. **dans la même ligne**. Après le split,
ces champs viennent de `series_facts`. Options :

- **(A — recommandé phase 1) Merge applicatif.** On récupère les lignes `media_items` (per-user), on
  collecte leurs `tmdb_id`, on fait **UN** `series_facts.select().in('tmdb_id', ids)`, on fusionne en JS.
  = 2 requêtes au lieu d'1, pas de N+1 (les ids sont batchés). C'est **déjà le pattern** de
  `getListsWithThumbnails`. Simple, explicite, zéro magie.
- **(B — plus tard, si besoin) Vue SQL** `media_items_full` = `media_items LEFT JOIN series_facts`.
  ⚠️ **Footgun Supabase (Fable) :** une vue exposée à PostgREST s'exécute par défaut avec les droits de
  son *owner* → elle **bypasse la RLS de `media_items`** (fuite cross-org silencieuse). Obligatoire :
  `security_invoker = on` (PG15+). Et le JOIN doit porter `AND type <> 'film'`. À garder en réserve, avec
  ce warning.

On part sur **(A)**. Deux garde-fous imposés par Fable :
- **Garde `type <> 'film'`** dans `hydrateSeriesFacts` : les ids TMDB movie/tv peuvent coïncider
  numériquement, la Library mêle les types → n'attacher les facts **que** sur les lignes non-film.
- **Split de types TS** : `MediaRow` (colonnes per-user, sans facts) vs `WatchingMedia = MediaRow &
  SeriesFacts` (hydraté). Le service ne retourne QUE de l'hydraté → le compilateur force l'hydratation sur
  les ~6 chemins de lecture (`deriveWatchStatus` lit `season_aired`/`status`/`caught_up_at`). La discipline
  seule laisserait passer un call-site non hydraté qui dérive un statut faux en silence.
- Bonus TanStack : `series_facts` peut avoir sa propre query-key par `tmdb_id`, cachée et **partagée entre
  sections** (impossible avec une vue).

## 6. Le chemin d'ÉCRITURE

- **Job de sync** : écrit désormais **1 ligne dans `series_facts` par `tmdb_id`** au lieu de N lignes
  `media_items`. → simplification nette + la divergence disparaît **par construction** (il n'y a plus
  qu'une copie).
- **Ajout d'un titre** (modale + ajout-à-liste) : `insert media_items` (à toi) **+ `upsert series_facts`**
  (le monde, `on conflict (tmdb_id) do update` si la ligne est plus fraîche). 2 écritures, découplées.
- **Progression/notes/stance** (le +1, les stances, les notes) : **inchangé** — 100 % dans `media_items`.

## 7. Déploiement PHASÉ (sans big-bang, réversible à chaque étape)

**Ordre corrigé (Fable) : expand → migrate → contract. Le dual-WRITE précède la bascule LECTURE.**

1. **Phase 1 — additive.** Créer `series_facts` (+ son index de sync partiel, + RLS) + **backfill**
   (pour chaque `tmdb_id` série/anime encore possédé, prendre la copie aux **facts non-nuls** la plus
   fraîche). Poser `CHECK (type = 'film' OR tmdb_id IS NOT NULL)` sur `media_items`. *Rien dans l'app ne
   change encore.*
2. **Phase 2 — dual-WRITE.** Sync + flux d'ajout écrivent **`series_facts` ET encore `media_items`**
   (les deux). `series_facts` devient et reste frais. Sélection des candidats du sync = `WHERE EXISTS
   (… media_items WHERE tmdb_id = series_facts.tmdb_id AND type <> 'film')` → pas de sync d'orphelins.
3. **Phase 3 — bascule LECTURE.** `hydrateSeriesFacts` lit `series_facts` (avec garde `type <> 'film'`),
   fallback sur `media_items` si absent. Comme la phase 2 garde `series_facts` frais, la lecture ne voit
   jamais de périmé.
4. **Phase 4 — contract.** Une fois stable : le sync/ajout cessent d'écrire les colonnes dans
   `media_items`, on **audite l'usage du `status` FILM** (le garder inline si lu), puis on `drop` les 9
   colonnes série de `media_items` + on **droppe `media_items_sync_idx` (orphelin) et on amende la
   baseline**.

Chaque phase est **commitable et réversible** seule. Aucun moment où l'app lit du périmé ou est cassée.

## 8. ⚖️ Mon jugement honnête sur le TIMING (rôle anti-over-engineering)

Le design est bon et c'est le bon **end-state**. Mais soyons droits sur le **ROI aujourd'hui** :

- Le job de sync **groupe déjà par `tmdb_id`** et écrit toutes les copies → **la divergence est déjà
  largement mitigée**. `series_facts` la supprime *totalement*, mais le gain marginal est modeste à 3 users.
- Le vrai gain « scale » (millions de lignes, N× appels TMDB) ne se matérialise **qu'à vrai multi-user**.
- Le coût est réel **maintenant** : une jointure/merge sur chaque lecture de série, un refactor en 4 phases.

**Donc le déclencheur qui rend `series_facts` rentable TOUT DE SUITE, c'est l'overlay AniList** (anime v2 :
découpage en cours + posters par cour). Lui a **besoin** d'un foyer par `tmdb_id` — et `series_facts` est
exactement ça. Sans AniList imminent, `series_facts` est de la belle archi à ROI différé.

**Ma reco :** deux scénarios défendables, et c'est LA grande question pour Fable —
- **Faire maintenant** si AniList v2 est le prochain gros chantier (les deux se font ensemble, series_facts
  devient l'ossature d'AniList → ROI immédiat).
- **Différer** (juste garder ce design au chaud) si le prochain focus est ailleurs (Waiting for, craft,
  autres modules) → on évite d'ajouter de la complexité de lecture pour un gain qui n'arrive qu'à l'échelle.

Ce n'est PAS un « il faut le faire ». C'est un « voici le design solide **et** voici quand il devient
rentable ». La décision timing dépend de la roadmap, pas de la technique.

**Précision Fable (validée) :** le déclencheur exact = **AniList v2 OU l'ouverture réelle des inscriptions
(signups aujourd'hui OFF), le premier des deux.** Et « couplé à AniList » ≠ « même commit » : livrer
`series_facts` (phases 1-3) et le STABILISER, PUIS poser l'overlay dessus. Argument pour différer sans
coût : le backfill est trivial à 831 lignes et le restera tant qu'on est ≤3 users → **différer ne coûte
rien**, la migration ne devient pas plus dure d'ici le déclencheur.

## 9. Questions ouvertes pour Fable (2ᵉ avis)

1. **Périmètre** : les 9 colonnes volatiles seulement (ma reco) — ou pousser jusqu'à l'identité statique /
   les films ? Où est le vrai point d'équilibre normalisation vs simplicité à 10k ?
2. **Lecture** : merge applicatif (A) vs vue SQL (B) — quel est le choix qui vieillit le mieux ?
3. **Timing** : faire maintenant (couplé à AniList) vs différer — sur-ingénierie ou investissement juste ?
4. **Angle mort** : qu'est-ce que ce design casse ou complique qu'on n'a pas vu ?
