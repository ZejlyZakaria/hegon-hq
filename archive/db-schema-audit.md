# Audit schéma `watching` — structure, scalabilité, tenue à 10 000 users

> Opus · 2026-07-15 · **Read-only.** Reconstruit depuis TOUTES les migrations `supabase/migrations/*.sql`
> + croisé avec `src/modules/watching/service.ts` (patterns de requêtes réels) + données empiriques
> (831 lignes, 427 `tmdb_id` distincts, 3 comptes / 3 orgs).
>
> Question de l'owner (verbatim) : pas « est-ce que ça marche », mais **« est-ce le MEILLEUR, scalable
> à 10 000 users, maintenable, solide dès maintenant ».** HEGON doit vivre pour toujours.

---

## Verdict en une page

**La structure est fondamentalement saine et honnête.** RLS cohérent (org_isolation partout + garde-fous
démo), colonnes curées et documentées sur les listes chaudes, hygiène FK correcte (tout casse en cascade
depuis `media_items`), et des commentaires de colonnes d'une qualité rare.

> **Précision ajoutée le 21/07 (mesurée, pas déduite).** Cette ligne disait « aucun `select(*)` sur les
> chemins chauds ». C'est vrai des **listes** — le point qui compte — mais l'inventaire était
> incomplet : il y en a **5** au chargement d'une fiche (`media_items` par id, `theme_favorites`,
> `episode_highlights`, `media_lists`, `user_settings`). **Mesuré : 17,8 KB à eux tous**, moins que le
> seul bundle TMDB de la même page (22 KB). Le verdict « légitime » du §Bien-fait est confirmé : la
> ligne de fiche fait **4,1 KB**, dont **46 % de `cast_members` — la colonne qu'on utilise**. Nommer
> les colonnes y économiserait ~2 KB contre le risque d'en oublier une : **délibérément pas fait.**
> Le seul poids réel est `theme_favorites` (**8 KB**, deux fois la fiche) : le `ThemePlayer` vit dans
> le layout, donc la playlist complète part sur *chaque* page Watching alors que seul l'état des ♥ est
> nécessaire tant qu'on ne l'ouvre pas. Restructuration, gain modeste → **backlog, pas urgent.**

> **⚠️ CORRECTION post-introspection live (2026-07-15).** La 1ʳᵉ version de cet audit inférait les index
> depuis les migrations et concluait « index manquants = le gros frein ». **L'introspection live
> (`pg_indexes`) l'a démentie** : `media_items` porte en réalité **8 index** (dont un UNIQUE
> `(user_id, tmdb_id, type)`), mais **6 d'entre eux + la contrainte n'existent QUE dans la DB live** —
> créés à la main, jamais réécrits en migration. Les points A et C sont donc **déjà résolus** ; le vrai
> enseignement est le **point D** (le repo a divergé de la DB), et le seul chantier lourd reste **B**.

| # | Sujet | Gravité | Coût | Verdict |
|---|-------|---------|------|---------|
| **B** | **Faits-du-monde dupliqués par user (×1.95)** → `series_facts(tmdb_id)` | 🟠 OK à 3 users, bloquant à l'échelle | Migration + refactor service (à froid) | **Prérequis avant multi-tenant réel — LE chantier** |
| **D** | **Schéma réel ≠ repo** : 6/8 index de `media_items` + l'unique n'existent que dans la DB live | 🟠 (requalifié) Maintenabilité « forever app » | Petit (1 migration baseline) | **À faire — capture l'état réel** |
| **A** | Index sur `user_id`/`type`/`tmdb_id` | ✅ **Déjà en place** (dont composites `(user_id,type,…)` et `(user_id,tmdb_id,type)`) | — | **RÉSOLU** |
| **C** | Unicité `(user_id, type, tmdb_id)` | ✅ **Déjà en place** (index UNIQUE `media_items_unique_user_tmdb`) | — | **RÉSOLU** |
| **E** | `media_list_items.media_item_id` sans index dédié (seulement derrière `list_id`) | 🟡 Mineur (table 166 lignes) | Trivial (1 index) | Optionnel |

**Le seul vrai chantier est B.** Le reste est soit déjà fait (A, C), soit une mise en cohérence
repo↔DB (D), soit un micro-gain optionnel (E).

---

## 1. Inventaire des tables (schéma `watching`)

| Table | Rôle | Clé de tenant | RLS | Garde démo | Index (hors PK) |
|-------|------|---------------|-----|-----------|-----------------|
| **`media_items`** | Le hub. ~50 colonnes : identité TMDB + ta position/stance/notes/années | `org_id` | `org_isolation` | ✅ | **8 index (live)** : `org_id` · `(user_id)` · `(type)` · `(user_id,type,watched,current_episode)` · `(watched_at DESC)` · `sync_idx` (partiel) · **UNIQUE `(user_id,tmdb_id,type)`** · pkey |
| **`media_lists`** | Listes custom (soft-delete) | `org_id` | `org_isolation` | ✅ | `media_lists_live_idx (user_id, created_at) WHERE deleted_at IS NULL` |
| **`media_list_items`** | Appartenance média↔liste | `org_id` | `org_isolation` | ✅ | `UNIQUE(list_id, media_item_id)` — **rien sur `media_item_id` seul** |
| **`episode_highlights`** | Marques par épisode (best-ep + note 0-10) | `org_id` (migré depuis `user_id`) | `org_isolation` | ✅ | `episode_highlights_media_item_id_idx` · `UNIQUE(user_id, media_item_id, season, episode)` |
| **`rewatches`** | 1 ligne / re-visionnage | `org_id` | `org_isolation` | ✅ | `idx_rewatches_media (media_item_id, watched_on DESC)` |
| **`theme_favorites`** | OP/ED animes favoris | `org_id` | `org_isolation` | ✅ | `idx_theme_favorites_org (org_id, created_at DESC)` · `UNIQUE(org_id, track_key)` |
| **`trending_cache`** | Réf. globale « Don't Miss » (1 ligne / type) | — (global) | `select using(true)` | n/a | PK `type` |
| **`for_you_cache`** | Recos pré-calculées / user | `user_id` | `auth.uid()=user_id` (SELECT) | n/a | PK `(user_id, type)` |

**Colonnes `media_items`** (regroupées par sens) :
- *Identité TMDB (le monde)* : `tmdb_id`, `type`, `title`, `original_title`, `description`, `poster_url`,
  `backdrop_url`, `year`, `runtime`, `episode_runtime`, `rating`, `studio`, `directors` (jsonb),
  `cast_members` (jsonb), `tags` (text[]).
- *Faits-du-monde volatils (→ candidats `series_facts`)* : `status`, `seasons`, `episodes`,
  `season_episodes`, `season_aired`, `season_end_dates`, `season_air_dates`, `season_posters`,
  `season_air_years` (⚠️ voir §8), `last_synced_at`.
- *À TOI (per-user)* : `watched`, `in_progress`, `want_to_watch`, `paused`, `dropped`, `drop_reason`,
  `is_reference`, `favorite`, `priority`, `priority_level`, `user_rating`, `notes`, `note_updated_at`,
  `watched_at`, `last_watched_at`, `caught_up_at`, `current_season`, `current_episode`, `season_years`
  (jsonb), `season_ratings` (jsonb).
- *Système* : `id`, `org_id`, `user_id`, `created_at`, `updated_at`.

---

## 2. ✅ Index — en fait déjà bien couvert (points A + C RÉSOLUS)

**Introspection live (`pg_indexes`, 2026-07-15).** Contrairement à ce que suggéraient les migrations,
`media_items` est bien indexé — mais **hors migrations** (créé au dashboard). État réel :

| Index (live) | Colonnes | Ce qu'il sert |
|---|---|---|
| `idx_media_user_id` | `(user_id)` | filtre tenant applicatif |
| `idx_media_type` | `(type)` | (faible sélectivité — voir §8) |
| `idx_media_items_in_progress` | `(user_id, type, watched, current_episode)` | **son préfixe `(user_id, type)` sert TOUTES les sections** + la requête In Progress au complet |
| `idx_media_watched_at` | `(watched_at DESC)` | tri Last Watched |
| **`media_items_unique_user_tmdb`** | **UNIQUE `(user_id, tmdb_id, type)`** | **unicité (= point C) + lookup par identité TMDB** (préfixe `(user_id, tmdb_id)`) |
| `idx_media_items_org_id` | `(org_id)` | prédicat RLS |
| `media_items_sync_idx` | `(last_synced_at)` partiel | le job de sync |

Croisé aux requêtes chaudes, tout est servi par un préfixe d'index :

| Fonction service | Filtre | Index qui le sert |
|---|---|---|
| `getMediaItems` (4 sections) | `user_id, type, {flag}` + order | `idx_media_items_in_progress` (préfixe `user_id,type`) |
| `getOwnedTmdbIds` / `getRatingsForType` / `getTakenPriorities` | `user_id, type` | idem |
| `getExistingMediaItem` / `addTmdbItemToList` | `user_id, type, tmdb_id` | `media_items_unique_user_tmdb` |
| `getTitlesByPerson` | `user_id, in(tmdb_id[])` | `media_items_unique_user_tmdb` (préfixe `user_id,tmdb_id`) |
| `getLibraryMedia` / `getWatchingStatsData` | `user_id` + `or(...)` | `idx_media_user_id` (puis filtre `or` sur le tas — acceptable) |

**Conclusion : les points A et C sont déjà résolus en production.** Le seul « manque » réel est mineur
(point E) :

```sql
-- media_list_items : le FK media_item_id n'est indexé QUE derrière list_id
-- (UNIQUE(list_id, media_item_id)) → un lookup par media_item_id seul (getListsForMedia,
-- cleanup de stub, addTmdbItemToList) ne peut pas l'utiliser. Table petite → gain modeste.
CREATE INDEX IF NOT EXISTS idx_media_list_items_media
  ON watching.media_list_items (media_item_id);
```

> **Leçon (→ point D §8).** Ces 6 index + l'unique **n'existent dans AUCUN fichier de migration** : ils
> ne vivent que dans la DB live. Une recréation du schéma depuis le repo les perdrait tous, en silence.
> C'est la vraie action à mener.

---

## 3. RLS — isolation multi-tenant

**Cohérent et correct.** `org_isolation` (`org_id IN (SELECT * FROM my_orgs())`) sur les 6 tables
per-user, `my_orgs()` étant `SECURITY DEFINER STABLE` → **évaluée une seule fois par requête** (pas par
ligne) et lisant `memberships` via `idx_memberships_user_id`. Le prédicat `org_id` est index-friendly
grâce à `idx_media_items_org_id`. C'est le bon pattern, il tient à l'échelle.

- `for_you_cache` (auth.uid()=user_id, SELECT only, écritures service-role) et `trending_cache` (lecture
  publique) sont des **exceptions justifiées** (cache par-user / réf. globale) — pas des incohérences.
- Garde-fou démo `demo_readonly_{insert,update,delete}` RESTRICTIVE appliqué sur les 6 tables. ✅
- **Nuance cosmétique** : deux idiomes syntaxiques de la même policy coexistent
  (`SELECT * FROM public.my_orgs()` vs `SELECT my_orgs.my_orgs FROM my_orgs() my_orgs(my_orgs)`).
  Strictement équivalents, mais à unifier un jour (lisibilité).

**Rien à corriger côté sécurité.** La seule remarque perf est indirecte : la RLS s'appuie sur
`idx_media_items_org_id`, qui existe — mais le filtre *applicatif* `user_id` qui suit, lui, n'a pas
d'index (→ point A).

---

## 4. Relations / FK / intégrité

- **Cascades saines** : supprimer un `media_items` nettoie ses `episode_highlights`, `rewatches` et
  `media_list_items` (tous `ON DELETE CASCADE`). `org_id → organizations ON DELETE CASCADE` partout.
  Aucun orphelin possible par suppression normale.
- **Soft-delete `media_lists`** : applicatif (`deleted_at`), volontaire — le hard-cascade sur
  `media_list_items` ne se déclenche que sur une purge délibérée. Cohérent avec « purger = décision
  séparée ».
- **Nettoyage des stubs `is_reference`** : géré applicativement dans `removeMediaFromList` (supprime le
  média si plus aucune liste ne le référence). Correct, mais dépend d'un `count` sur `media_item_id` —
  qui bénéficierait de l'index optionnel du point E.
- ✅ **Intégrité `(user_id, type, tmdb_id)` — DÉJÀ garantie (point C RÉSOLU).** L'introspection live
  montre un index **UNIQUE `media_items_unique_user_tmdb (user_id, tmdb_id, type)`**. L'hypothèse
  d'unicité du code (`maybeSingle()`) est donc **réellement protégée par la base** : un insert dupliqué
  concurrent est rejeté par Postgres, pas laissé passer. Rien à ajouter. (Comme les index du §2, cette
  contrainte n'est capturée dans **aucune migration** → point D.)

---

## 5. 🟠 Normalisation — la duplication des faits-du-monde (point B)

**Le sujet structurel de fond.** 831 lignes pour 427 `tmdb_id` → chaque **fait du monde** est stocké
~2 fois aujourd'hui (une copie par user possédant le titre). Colonnes concernées : `status`,
`season_episodes`, `season_aired`, `season_end_dates`, `season_air_dates`, `season_posters`, `seasons`,
`episodes`, `last_synced_at` (~9 colonnes).

**Pourquoi c'est OK maintenant et bloquant à l'échelle :**
- À 3 users : 2× de rien, invisible.
- À 10 000 users possédant ~400 titres avec un fort recouvrement sur les titres populaires : **des
  millions de lignes de faits dupliqués**, un job de sync qui paie N× le même appel TMDB, et surtout un
  **risque de divergence** (deux copies de la même série synchronisées à des heures différentes = deux
  vérités dans la base). Le job groupe déjà par `tmdb_id` pour amortir ça — mais c'est un pansement sur
  un modèle qui stocke la donnée au mauvais endroit.

**La cible : `series_facts(tmdb_id)`** — table de référence **partagée**, une ligne par titre du monde,
`media_items` ne gardant que ce qui est **à toi** (position, stance, années, notes, ratings, favorite,
priority, watched_at…). Bénéfices : ~2× moins d'appels TMDB, **zéro divergence possible**, et c'est
l'ancre naturelle de l'**overlay AniList** (anime v2 : découpage en cours + posters par cour s'accrochent
à `series_facts.tmdb_id`). `seasons`/`episodes` (dérivables) s'y dissolvent aussi.

> C'est déjà au backlog (`hq/now.md` §Différé + mémoire `reference_series_aired_vs_announced`). Cet audit
> le **requalifie** : ce n'est pas un « nice-to-have », c'est le **prérequis** avant tout scale
> multi-tenant réel. = migration + refactor service, à froid.

**Usage jsonb/array** — sain dans l'ensemble : `season_years`/`season_ratings` (maps par saison),
`season_*` (arrays par saison), `cast_members`/`directors` (arrays d'objets), `tags` (text[]). **On ne
requête jamais À L'INTÉRIEUR** de ces colonnes en SQL → jsonb est le bon choix (pas de besoin d'index
GIN). Seule exception : `getPeopleCounts` tire `cast_members`/`directors` de toutes tes lignes vues et
agrège **en JS** — acceptable à ton échelle, mais c'est le premier calcul qui souffrira à 10k (voir §6).

---

## 6. Requêtes — chemins chauds

**Bien fait (à garder comme modèle) :** `SECTION_COLUMNS`, `LIBRARY_COLUMNS`, `LIST_SEARCH_COLUMNS`,
`PERSON_TITLE_COLUMNS` — des sélections de colonnes explicites et **commentées** sur les chemins chauds,
au lieu de `select(*)`. À quelques centaines de titres, ça divise le payload par plusieurs. `select(*)`
n'est utilisé que sur `getMediaItemById` (une seule ligne) — légitime.

**À surveiller (pas urgent) :**
- **Agrégations en JS** (`getPeopleCounts`, stats, tri Library) : lisent la tranche du tenant et calculent
  côté client. Parfait aujourd'hui ; à l'échelle, `getPeopleCounts` (lecture de 2 colonnes jsonb sur
  toutes tes lignes) est le candidat n°1 à une table `person_credits` normalisée ou un compteur
  matérialisé. **Prématuré maintenant.**
- **`searchMediaForList`** : `ilike '%query%'` (wildcard en tête) → aucun btree utilisable, seq-scan dans
  la tranche du tenant. Négligeable à quelques centaines de lignes ; si un jour la recherche traîne, un
  index **GIN trigram** (`pg_trgm`) sur `title` règle ça. Basse priorité.
- **`addMediaToList`** calcule `max(position)+1` puis insère (2 requêtes, race possible). Risque nul en
  mono-user ; à noter si l'édition de listes devient collaborative.
- **Limites** : `getMediaItems` plafonne à 100 (bon garde-fou). Les autres lectures per-user ne paginent
  pas — acceptable tant qu'un tenant reste sous quelques milliers de titres.

---

## 7. Croissance — tables non bornées

Aucune table pathologiquement non bornée :
- `media_items` grandit avec ta collection (la table principale) — déjà couverte par ses index (§2).
- `episode_highlights` / `rewatches` : bornées par ce que tu marques/re-regardes, croissance lente.
- `for_you_cache` (par user×type), `trending_cache` (par type) : bornées par construction.

Rien à faire ici.

---

## 8. Maintenabilité — dette structurelle

- 🟠 **LE POINT D — le schéma réel a divergé du repo.** L'introspection live l'a prouvé : `media_items`
  porte **8 index + une contrainte UNIQUE**, dont **6 index + l'unique n'existent dans AUCUN fichier de
  migration** (créés à la main dans le dashboard). Idem, la table elle-même n'a aucun `CREATE TABLE` : sa
  vérité est éclatée entre la DB live et 20+ `ALTER`. Pour une app censée **vivre pour toujours** (et
  potentiellement accueillir un 2ᵉ dev ou un autre agent), c'est l'angle mort qui coûte cher le jour d'une
  recréation/restauration : on **perd silencieusement** l'unicité anti-doublon et les index de perf.
  **Correctif** : une migration *baseline* idempotente qui capture l'état RÉEL — `CREATE TABLE IF NOT
  EXISTS media_items (…)` avec toutes les colonnes + commentaires, **plus les 8 `CREATE INDEX/UNIQUE IF
  NOT EXISTS`** listés au §2 — pour que le repo redevienne la source de vérité.
- ⚠️ **`supabase/` est gitignoré** (décision owner connue) → les migrations sont appliquées à la main,
  hors historique git normal. Même racine que le point D : à terme, au minimum versionner
  `supabase/migrations/` (le SQL ne contient pas de secret).
- ✅ **Baseline créée** : `supabase/migrations/20260715_baseline_media_items.sql` capture la table réelle
  (colonnes/types/défauts relevés dans `information_schema`) + les **8 index** (relevés dans `pg_indexes`),
  tout en `IF NOT EXISTS` → inerte sur la DB actuelle, source de vérité pour toute recréation. Point D
  traité. Colonnes réordonnées (important→secondaire, liées côte à côte) — s'applique aux recréations
  futures uniquement (Postgres ne réordonne pas une table existante ; sans impact fonctionnel).
- **`idx_media_type (type)` peu utile seul** (3 valeurs → faible sélectivité, le planner le choisira
  rarement) et **`idx_media_user_id (user_id)` en partie redondant** avec le préfixe des composites. Sans
  danger, candidats à un nettoyage futur (allège les écritures). Non urgent.
- 🟡 **Incohérence de type temporel** : `watched_at` / `created_at` / `updated_at` sont en `timestamp`
  **sans fuseau**, alors que toutes les colonnes de date récentes sont en `timestamptz`. Dette classique
  (décalage possible du même instant selon le fuseau serveur). Nettoyage futur = migration dédiée avec
  conversion de données. Non urgent, mais à corriger avant tout déploiement multi-région.
- ~~`season_air_years`~~ : **n'existe pas** dans la DB réelle (l'introspection le confirme) — faux
  problème, rien à droper.
- **Excellent** : la densité de commentaires de colonnes (`season_aired`, `caught_up_at`,
  `last_watched_at`, `note_updated_at`…) est un modèle d'auto-documentation. Le seul trigger
  (`touch_note_updated_at`) est ciblé et correct. **À garder tel quel.**

---

## 9. Plan d'action priorisé (révisé post-introspection)

**Déjà résolu en prod (aucune action) :** points **A** (index sur `user_id`/`type`/`tmdb_id` en place) et
**C** (unicité `(user_id, tmdb_id, type)` garantie par index UNIQUE). L'audit initial les surestimait —
l'introspection live a tranché.

**P1 — les vrais chantiers :**
1. **Migration baseline `media_items` (point D)** — capturer l'état RÉEL dans le repo : `CREATE TABLE IF
   NOT EXISTS` complet + les **8 index/unique** du §2 en `IF NOT EXISTS`. Petit, sans risque (idempotent),
   et c'est l'assurance-vie du schéma le jour d'une restauration. **À faire tôt.**
2. **`series_facts(tmdb_id)` (point B)** — sort les 9 colonnes de faits-du-monde de `media_items`. Absorbe
   la duplication 1.95×, tue la divergence, ancre l'overlay AniList. → §5. *Le seul gros chantier
   structurel — candidat au 2ᵉ avis Fable.*

**P2 — hygiène (optionnel) :**
3. Index `(media_item_id)` sur `media_list_items` (point E) — micro-gain, table 166 lignes.
4. Vérifier + droper `season_air_years` si morte ; au passage, envisager de droper `idx_media_type` /
   `idx_media_user_id` redondants (lors de la baseline).
5. Unifier les deux idiomes de policy RLS ; garder `pg_trgm` en réserve si la recherche Library ralentit
   un jour à l'échelle.

---

## Annexe — requêtes de vérification (read-only, à lancer par l'owner)

```sql
-- (1) Index réels sur les tables watching. ✅ DÉJÀ LANCÉE (2026-07-15) → a corrigé l'audit :
--     media_items a 8 index dont UNIQUE (user_id, tmdb_id, type). Points A + C résolus.
SELECT tablename, indexname, indexdef
FROM pg_indexes WHERE schemaname = 'watching' ORDER BY tablename, indexname;

-- (2) Doublons intra-user. Devrait retourner 0 lignes (l'index UNIQUE les interdit) —
--     à lancer surtout pour CONFIRMER avant de bâtir series_facts.
SELECT user_id, type, tmdb_id, count(*)
FROM watching.media_items
GROUP BY user_id, type, tmdb_id HAVING count(*) > 1;

-- (3) Facteur de duplication des faits-du-monde (mesure le gain de series_facts, point B).
SELECT count(*) AS lignes, count(DISTINCT tmdb_id) AS titres_distincts,
       round(count(*)::numeric / nullif(count(DISTINCT tmdb_id),0), 2) AS facteur
FROM watching.media_items;

-- (4) season_air_years est-elle morte ? (0 non-vide = droppable)
SELECT count(*) FILTER (WHERE season_air_years IS NOT NULL
                          AND season_air_years <> '[]'::jsonb) AS non_vides
FROM watching.media_items;
```
