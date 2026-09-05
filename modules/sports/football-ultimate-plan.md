# Sport → niveau Watching — Plan (démarré 2026-08-02)

> État de la réflexion "passer Sport de widget à monde". On garde **3 sports** (foot/tennis/f1),
> chacun poussé à sa version ultimate. **Priorité = Football** (choix owner). Séquence data-driven
> globale : Football → (F1) → Tennis. Tennis en dernier car sa data est le mur.

---

## 1. Diagnostic — pourquoi Sport paraît minuscule

Les 3 sports font la même chose aujourd'hui : suivre des entités + agenda + résultats + classements.
= un **tableau de bord en lecture-survol**. Il manque les **5 piliers** de Watching :
1. **Fiches** (rien ne s'ouvre : ni équipe, ni joueur, ni match).
2. **Couche PERSO** (tu *suis*, tu ne *vis* rien : aucune note, aucun "j'y étais", aucun pronostic).
3. **Stats / Wrapped** (rien sur toi comme fan).
4. **Musée / histoire** (aucun H2H, palmarès, record).
5. **Connexions cross-module** (rien ne nourrit Goals/Journal).

**La colonne vertébrale commune aux 3** (à coder une fois) : fiches + **Fan Log** (j'ai regardé/où/note/mot)
+ **Wrapped fan** + connexions Goals/Journal. C'est ça qui hisse Sport au niveau Watching.

---

## 2. Football — feature set ultimate (UI/UX validés)

- **Fiche Match** (pièce maîtresse) : hero (2 écussons, score/compte à rebours, dégradé aux couleurs
  des clubs), tabs `Résumé · Compos · Stats · Face-à-face · Mon avis`. Résumé = fil d'événements
  vertical (minute au centre, dom. gauche / ext. droite). Compos = terrain (réutilise le renderer
  Best XI). Mon avis = Fan Log inline.
- **Fiche Équipe** : hero (couleur club, position, forme 5 pastilles, prochain match), tabs
  `Aperçu · Effectif · Calendrier · Résultats · Palmarès · Rivalités`.
- **Fiche Joueur** (transpose la page Person de Watching) : photo détourée, stats saison, carrière.
  Rend le Best XI + les compos cliquables.
- **Fan Log** (le cœur) : inline sur chaque match (Vu 📺/🏟️/📱 + note + mot + pronostic) **+** surface
  "Mon Football" (journal des matchs vus, bilan de témoin V/N/D, 🏟️ "j'y étais" = collection).
- **Rivalités / Derby** (musée) : H2H dédié, bilan all-time, timeline.
- **Trophy Cabinet** (musée, par équipe) : étagère à trophées + années.
- **Wrapped / Ta Saison** : matchs vus, bilan de témoin, match de l'année, buts vus, visites au stade,
  bonus "porte-bonheur ?" (bilan quand tu regardes).
- **Cross-module** : Goals `metric_module='football'` (matches_watched / stadium) nourri par le Fan Log ;
  Journal (match noté → entrée). **Pas de nouvelle table sport pour ça.**

**La landing devient un COCKPIT** (de haut en bas) : ① bandeau LIVE/imminent (si équipe joue) → ②
Mes équipes (heroes) → ③ À venir → ④ Récents + "ton avis" (hameçon Fan Log) → ⑤ Classements → ⑥ Ta
saison (tuile compacte) → ⑦ Best XI + Légendes. Chaque section se masque si vide.

---

## 3. Architecture data — les 3 régimes

- **CRON (eager, borné à tes favoris/compètes)** : ce qui existe déjà + write-through vers
  `football_matches` (retention).
- **LAZY cache-aside (rempli à l'ouverture d'une fiche)** : détail match/joueur/effectif/H2H. Check
  Supabase → hit&frais ? sert : fetch API 1× → upsert → sert. TTL par nature : **match fini = immuable
  → caché à vie** ; bio joueur = TTL long ; stats saison = TTL court. **Le client ne tape jamais l'API
  direct** (tout serveur, comme /api/tmdb).
- **PERSO (écritures user)** : Fan Log, pronostics.

**Insight quota** : les fiches ne sont PAS un problème de quota (match fini immuable, bio TTL long →
après chauffe, tout est hit cache). **Seul le live est coûteux.** On ne précharge JAMAIS l'univers.

---

## 4. Existant DB/edge (lu le 2026-08-02) — aucune migration SQL ne définit `sport` (créé à la main)

**Tables `sport` (football)** — reconstruites depuis les crons :
- Référence (pas d'`org_id`) : `football_competitions` (whitelist **PD/CL/PL**), `football_teams`
  (api_external_id, name, short_name, tla, crest_url, founded, venue), `football_team_competitions`
  (jonction), `football_next_matches` (team_id suivi, competition_id, external_match_id, home/away_name
  + external_id, match_date — **plafonné 3/équipe**), `football_past_matches` (idem + home/away_score,
  **3/équipe**), `football_standings`.
- User : `football_user_settings` (main_team_id), `user_favorites` (entity_type partagé 3 sports),
  `football_best_xi` + `football_best_xi_players` (**saisis à la main**).
- ⚠️ **Le pipeline DÉTRUIT l'historique** (>3 derniers = supprimés) → `football_matches` first-class
  no-purge est obligatoire pour le Fan Log/Wrapped.
- Contrainte unique = `(user_id, tmdb_id, type)` côté watching ; côté sport, matchs dénormalisés
  (seul le team suivi est un uuid, adversaire = name + external_id).

**Edge functions** : 5 crons football via **football-data.org v4** (`X-Auth-Token: FOOTBALL_DATA_KEY`,
écrit avec `HEGON_SECRET_KEY`, `Content-Profile: sport`, sleeps anti rate-limit).

---

## 5. ⭐ VERDICT DATA (le gros de la session 2026-08-02) — voir aussi [[reference_football_data_sources]]

- **football-data.org GRATUIT** = scores (différés), fixtures, calendriers, **classements**, 12 compètes,
  10 appels/min. **PAS d'événements, PAS de compos, PAS de buteurs/cartons.** (Confirmé sur la page
  pricing, PAS sur le JSON que l'owner avait partagé — ses arrays vides = **matchs non joués**, pas le
  tier. Leçon : ne pas conclure d'un échantillon non joué.)
- **football-data €29/mo "Deep Data"** = Line-ups & Subs + Goal scorers + Bookings/Cards + **Squads**
  (couvre aussi la phase 1b joueurs/effectif). **€12/mo "Livescores"** = le bandeau LIVE.
- **API-Football GRATUIT = TESTÉ le 2026-08-02** → `"Free plans do not have access to this season, try
  from 2022 to 2024."` Donc events/compos riches MAIS **saisons 2022-2024 seulement** → **inutile pour
  la saison en cours**. Current season = payant (~€19+/mo).
- **Pas de source gratuite + officielle + toutes compètes + saison en cours** pour compos/événements.
  Forks restants pour la couche "diffusion" du current season :
  - **Sofascore (non officiel)** : gratuit, riche, current — **ToS-gris/fragile → perso only, coupé
    sur la démo publique**.
  - **FPL API** : gratuit, officiel, current — **Premier League UNIQUEMENT**.
  - **Payant** : €29 football-data Deep Data, ou API-Football payant.
- **Photos joueurs** : football-data n'en a pas ; **TheSportsDB** (gratuit) en a. Stats fines (xG…) =
  API-Football, hors scope gratuit.

**DÉCISION : on DÉCOUPLE.** Phase 1a = fiche (**score + contexte**) + **Fan Log**, sur football-data
**gratuit** — aucune dépendance à ces forks, et c'est déjà le saut niveau Watching (la couche perso que
personne d'autre ne fait). Le fil événements/compos = **fork différé** (perso-gris Sofascore / PL-gratuit
FPL / payant €29), à trancher **plus tard**, sans refonte (mêmes tables `football_match_events` /
`football_match_lineup_players`, qui s'allument quand la source arrive).

---

## 6. Plan par phases

- **Phase 1a (PILOTE, verrouillé)** : `football_matches` (+ write-through retention) +
  `football_watched_matches` → **la fiche match (score+contexte) + le Fan Log**. 100% gratuit.
- **Phase 1b** : `football_players` + `football_team_squad` + `football_scorers` (fiches joueur/équipe).
  ⚠️ dépend du fork data (squads = €29 ou Sofascore/FPL).
- **Phase 1c** : `football_h2h` + `football_predictions` + Wrapped + connexion Goals.
- **Phase musée** : trophées via **Wikidata** (mutualisé avec le musée Watching).

---

## 7. ⏭️ REPRISE DEMAIN — exactement ici

Finir le **schéma SQL des 2 tables pilotes** (j'ai été coupé en plein `football_matches`). Design arrêté :

```sql
-- RÉFÉRENCE (pas d'org_id, ne purge jamais). Une ligne par match RÉEL (dédup via external_match_id).
create table if not exists sport.football_matches (
  id                     uuid primary key default gen_random_uuid(),
  external_match_id      integer not null unique,        -- id football-data du match
  competition_id         uuid references sport.football_competitions(id),
  season                 integer,                        -- année de début de saison
  utc_date               timestamptz not null,
  status                 text not null,                  -- SCHEDULED/TIMED/IN_PLAY/PAUSED/FINISHED
  matchday               integer,
  stage                  text,
  "group"                text,
  venue                  text,
  attendance             integer,
  home_team_external_id  text not null,
  away_team_external_id  text not null,
  home_team_name         text not null,
  away_team_name         text not null,
  home_team_id           uuid references sport.football_teams(id),
  away_team_id           uuid references sport.football_teams(id),
  home_score             integer,
  away_score             integer,
  home_score_ht          integer,
  away_score_ht          integer,
  winner                 text,                           -- HOME_TEAM/AWAY_TEAM/DRAW
  last_updated           timestamptz,
  fetched_at             timestamptz not null default now(),
  created_at             timestamptz not null default now()
);
create index on sport.football_matches (competition_id, utc_date);
create index on sport.football_matches (home_team_external_id);
create index on sport.football_matches (away_team_external_id);
create index on sport.football_matches (status);

-- PERSO (user_id + RLS). Le Fan Log. Existence ≈ vu.
create table if not exists sport.football_watched_matches (
  id                 uuid primary key default gen_random_uuid(),
  user_id            uuid not null references auth.users(id) on delete cascade,
  external_match_id  integer not null references sport.football_matches(external_match_id),
  watched            boolean not null default true,
  watched_where      text check (watched_where in ('tv','stadium','live')),
  rating             numeric(3,1) check (rating >= 0 and rating <= 10),
  note               text,
  watched_at         timestamptz not null default now(),
  created_at         timestamptz not null default now(),
  updated_at         timestamptz not null default now(),
  unique (user_id, external_match_id)
);
create index on sport.football_watched_matches (user_id);
alter table sport.football_watched_matches enable row level security;
create policy fwm_select on sport.football_watched_matches for select using (user_id = auth.uid());
create policy fwm_insert on sport.football_watched_matches for insert with check (user_id = auth.uid());
create policy fwm_update on sport.football_watched_matches for update using (user_id = auth.uid());
create policy fwm_delete on sport.football_watched_matches for delete using (user_id = auth.uid());
```

À décider/vérifier demain avant de coder :
- Écrire la migration versionnée (rappel : SQL **appliqué à la main**, jamais `db push`).
- Le write-path : upsert `football_matches` AVANT `football_watched_matches` (l'FK l'exige).
- Brancher les crons next/past en **write-through** vers `football_matches` (retention).
- 3 curls à confirmer sur la clé owner : `/competitions/{id}/scorers` (buteurs), mi-temps/affluence
  dans la fiche, H2H — pour savoir ce que le gratuit football-data donne vraiment au-delà du score.

---

## 8. SESSION 2026-08-03 — Phase 1a EN COURS (la §7 est FAITE)

**Décisions figées :**
- **App PERSO → JAMAIS de tier payant.** Free only. Contexte (score/fixtures/classements/bracket UCL)
  = football-data gratuit. Couche riche (compos/événements) = **Sofascore** (API non officielle) en
  **bonus gris OPTIONNEL, dégradation gracieuse** si l'envie vient — pas une priorité, peut-être jamais.
  (L'owner : « je vais pas concurrencer FlashScore ».)
- **Surface d'un match = SLIDING PANEL** (coup d'œil + Fan Log), très probablement la surface FINALE.
  On garde la **route** `/perso/sports/football/match/[id]` comme échappatoire "page pleine" si la
  couche riche arrive un jour.
- **Méthodo** : MVP d'abord pour verrouiller data+flux+archi ; les décisions de CONTENEUR (panel vs
  page) se prennent MAINTENANT (structurel) ; **UNE passe polish "niveau Watching" plus tard, menée par
  Fable**. Le UI actuel de la fiche = **MVP brut assumé**, à polir en passe groupée.
- **Best XI + Légendes** → descendent bas de landing / onglet dédié (hors chemin principal).
- **Compétitions auto** (à faire) : tuer la whitelist `PD/CL/PL` → dériver des équipes suivies, borné
  aux 12 compètes gratuites, rendues par TYPE (ligue → table, coupe → **bracket** via `stage`).

**FAIT aujourd'hui :**
- Migration `20260802_football_matches_and_fan_log.sql` **appliquée** (2 tables + RLS + **grants** — le
  grant manquait → "permission denied", corrigé).
- `FOOTBALL_DATA_KEY` ajoutée au **`.env.local`**.
- **Route cache-aside** `/api/football/match/[id]` — **testée OK** (Real Madrid 4-2 Athletic, retention
  confirmée). FINISHED = immuable/à vie ; écriture via service-role.
- **Fan Log** : service (`getFootballFanLog`/`upsert`/`delete`, client RLS) + hooks
  (`useFootballFanLog` + mutations).
- **Fiche match MVP** + formulaire Fan Log (où TV/Stade/Live + note /10 + mot). tsc/eslint verts.

**Fichiers créés :**
`supabase/migrations/20260802_football_matches_and_fan_log.sql` ·
`src/app/api/football/match/[id]/route.ts` ·
`src/modules/sports/football/hooks/{useFootballMatch,useFootballFanLog}.ts` (+ query-keys match/fanLog) ·
`src/modules/sports/football/components/match/FootballMatchClient.tsx` ·
`src/app/(main)/perso/sports/football/match/[id]/page.tsx` ·
types `FootballMatchRow`/`FootballFanLogEntry`/`FanLogInput`/`WatchedWhere` + service `getFootballMatch` + 3 fan-log.

**⏭️ REPRISE (exactement ici) :**
1. **Basculer la fiche en `SlidingPanel`** (primitif `@/shared/components/ui/sliding-panel`, cf.
   QuickAddPanel). Le contenu `FootballMatchClient` ne bouge pas — on le monte dans le panel.
2. **Câbler les cartes résultats/à-venir** → ouvrir le panel : **remonter `external_match_id`** dans
   `getFootballPageData` (il mappe `id`=row id, PAS l'external — les tables `football_next/past_matches`
   L'ONT) + le trigger.
3. Plus tard : garde **démo read-only** sur écritures sport (avant publi) · ~~Goals
   `metric_module='football'`~~ ✅ fait · Wrapped · write-through crons → `football_matches` (retention complète).

## 9. SESSION 2026-08-03 (suite) — Goals↔Football FAIT + vision page-match

**Connexion Goals ✅** (testée) : RPC `recalc_goal_progress` branche football (compte
`football_watched_matches`, gère `stadium`) [migration `20260803_football_goal_metric.sql`] · pickers
create+edit (source Football, keys `matches`/`stadium`) · fiche goal adaptée · `syncFootballGoals` sur
chaque log → barre bouge · **ripple toast** (copie du GoalRippleToast Watching, accent emerald, icône
`Goal`, déclenché sur les deltas du RPC → ne sort que si la progression bouge vraiment). Placeholder
target contextuel (football→30). Cibles conseillées : matches 30/38/50 · stadium 3/5/10.

**⭐ VISION à garder — la PAGE match comme HUB cross-module** (owner, 03/08) : on garde la route
`/perso/sports/football/match/[id]` (pas juste le panel). Le panel = coup d'œil rapide ; la **page** =
le hub riche vers lequel pointent d'autres modules. Déjà : la fiche goal « Counting toward this goal »
→ ouvre cette page. À venir : un match au **stade** devient une **entrée Travel** (le déplacement) +
un souvenir **Journal** + une contribution **Goal** — la même entité vue par 3 modules. C'est *le* rêve
HEGON (profondeur cross-module que personne d'autre ne fait). Quand Travel/Journal seront prêts, la
page match les tisse.

**⚠️⚠️ PERF — SPORT DOIT ÊTRE PLUS RAPIDE QUE WATCHING (owner, SÉVÈRE, 03/08).** Anomalie : Sport
(~10% des ressources de Watching, **quasi aucune image**, tout vient de Supabase) est **pourtant plus
LENT** que Watching. Ça doit être l'INVERSE. Suspicion : `getFootballPageData` = **une grosse requête
client-side avec waterfalls internes** (getUserId → settings → favorites → détails équipes → matchs →
crests → bestXI, en partie séquentiel), **pas de server-render** (CLAUDE.md dit "sports = server" mais
le service utilise `createClient` browser). Watching = CSR-first + requêtes granulaires **cachées** +
server components → peint plus vite. → **Passe MESURÉE (A/B chiffré, Opus ; aide Fable OK)** : server-
render la page / granulariser+cacher, **tuer les waterfalls**, images `sizes`/optimisées, skeletons.
**Priorité #1 non-feature du chemin Sport→niveau Watching.**

## 10. SESSION 2026-08-03 (fin) — "Counting toward this goal" : 2 UI + ⚠️ NON COMMITÉ

**Fait :** dans `GoalDetailPage.tsx`, la section "Counting toward this goal" branche sur le module :
- **sport (football)** → **cartes horizontales** (équipes · date), clic → route match (un match n'a pas de poster).
- **watching + books** → grille poster, avec le **hover corrigé** = **toute la tuile scale** (`hover:scale-[1.04] hover:z-10` sur le `<button>`, image en `object-cover` sans scale) — comme la Watching library (avant : l'image scalait DANS le cadre, faux).

**⚠️ TOUT CE LOT (football module + connexion Goals + ce UI) N'EST PAS COMMITÉ** (owner le fera).

**⏭️ REPRISE — finitions UI restantes :**
1. Watching/Books : appliquer un **rung de taille défini** (`--poster-*` globals.css) à la grille du "Counting toward" (au lieu des `grid-cols-4/6/9` fluides) + confirmer que `scale-[1.04]` = exactement la valeur library.
2. Football : enrichir la carte horizontale avec le **SCORE** → ajouter `home_score`/`away_score` dans `getGoalContributingMatches` (goals/service.ts) + étendre le shape retourné (aujourd'hui `ContributingMedia` = id/title/poster_url/watched_at).
3. Puis LE gros morceau : **passe PERF Sport** (voir §9, priorité #1).
