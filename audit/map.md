# HEGON — LA CARTE (phase 0)

> **Constat pur, aucun jugement.** Établi le **2026-09-05**. Chiffres regénérables par
> `node scripts/map.mjs`. Phase 0 du chantier — voir [`plan.md`](plan.md).
>
> **Test de réussite** : l'owner doit pouvoir redessiner la §1 de mémoire. Si non, c'est un dépotoir.
> **Les écarts de la §3 ne se corrigent PAS ici** — ils alimentent la phase 1 (purge + cliquets).

---

## 1. LA CARTE — une page

```
        navigateur
            |
            v
   +--------------------------------------------------------+
   | middleware.ts   getClaims() en process, ~18ms, 0 reseau |  <- seule garde d'acces
   +----------------------------+---------------------------+
                                v
   +--------------------------------------------------------+
   | app/(main)/layout.tsx      +   Dock  (LA nav, 1 fichier)|
   +----------------------------+---------------------------+
                                v
   +--------- 9 MODULES PRODUIT + 5 MODULES DE PLOMBERIE --------------+
   |                                                                   |
   |  Life    Goals . Habits . Journal . Books                         |
   |  Perso   Sport (football | tennis | f1) . Watching                |
   |  Pro     Tasks                                                    |
   |  Home    Dashboard-OS (ecran d'accueil : widgets + tuiles)        |
   |                                                                   |
   |  plomberie : dashboard (donnees) . command-center (Cmd-K)         |
   |              settings . orgs (page /invite) . styleguide          |
   |              onboarding                                           |
   +-------------------------------------------------------------------+

   TOUT module suit LA MEME chaine, et elle est CLIENT a 96 % :

     components/*.tsx   ("use client")
            |   useQuery / useMutation
            v
     hooks/use*.ts   +   hooks/query-keys.ts
            |
            v
     service.ts    <- LE seul endroit qui parle a Supabase
            |   createClient()  (navigateur)
            v
     Postgres  --  3 schemas :  public (vie + travail) . watching . sport

   Le SERVEUR ne sert que 3 choses (rien d'autre n'est rendu cote serveur) :
     - middleware.ts     la garde
     - 9 pages sur 34    Watching (3) . Football team+competition (2) . / . /onboarding
                         . /perso/sports (redirect) . /settings/styleguide
     - 13 routes /api/*  proxys vers les API externes, seul endroit qui tient les secrets
                         tmdb . omdb . books . weather . proxy-image . football x5 . tennis x2

   A COTE, hors de l'app, tourne un DEUXIEME systeme :

     14 crons (pg_cron, DANS la base)  -->  internal.call_edge('<nom>')
                                                |
                                                v
                                    20 edge functions Deno  -->  API externes
                                    (football x6 . f1 x6 . tennis x3
                                     . watching x3 . signup . for-you)
                                                |
                                                v
                                        ecrit dans Postgres

   L'app ne les appelle jamais (1 exception : `for-you-refresh`).
   Elle LIT le resultat dans des tables-cache. Si un cron meurt, l'app ne le sait pas.
```

**En une phrase :** *une SPA client sur Supabase, plus une flotte de 20 robots qui remplissent la base
par derrière ; les deux ne se parlent que par des tables.*

---

## 2. LES CHIFFRES

> `node scripts/map.mjs` recalcule tout ce tableau. Un écart = la carte a bougé.

| | |
|---|---|
| Code applicatif | **80 030 lignes · 565 fichiers** (`src/`) |
| Modules | 14 (`src/modules/`) — Watching 22 941 l. · Sport 11 477 · Tasks 8 398 · Habits 7 015 · Goals 6 035 |
| Routes | **34 pages · 13 routes API** · 1 middleware |
| Rendu | **281 fichiers `"use client"` sur 327 `.tsx`** — **9 pages serveur sur 34** |
| Base | 3 schémas · **64 tables référencées** · 2 buckets storage (`posters`, `avatars`) |
| Async | **20 edge functions · 14 crons** |
| Migrations | 105 fichiers · **appliquées à la main**, `schema_migrations` vide |
| Tests | **14 fichiers** — Watching 9 · Books/Habits/Journal/Tasks/shared 1 chacun · **Sport 0** |
| Gates | CI = lint + tsc + build · pre-commit = `eslint --max-warnings 0` |
| Design system | 26 primitifs · **475 hits hors-système** (`npm run ds:coverage`) |
| Docs | `hq/` = 52 fichiers / **14 425 lignes** · `memory/` ≈ 50 fichiers |

Les 9 pages serveur, la liste complète — tout le reste de l'app est client :
`/` · `/onboarding` · `/perso/sports` (redirect) · `/perso/sports/football/team/[externalId]` ·
`/perso/sports/football/competition/[id]` · `/perso/watching` · `/perso/watching/library` ·
`/perso/watching/lists` · `/settings/styleguide`

**Le design system, par module** (hits hors-système = type / radius / scrim / glass arbitraires)

| module | fichiers | primitifs utilisés | hors-système |
|---|---|---|---|
| sports | 78 | 8 | **146** |
| dashboard-os | 20 | 2 | **78** |
| tasks | 53 | 13 | 52 |
| habits | 29 | 14 | 48 |
| books | 29 | 8 | 39 |
| **watching** | **136** | **15** | **38** ← le plus gros module, la plus petite dette |
| goals | 30 | 14 | 37 |
| journal | 25 | 9 | 16 |

---

## 3. CE QUE LA CARTE A TROUVÉ

> Constat, pas correction. Chaque ligne est vérifiée par exécution. Matière de la phase 1.

### 3.1 — Un bug vivant, silencieux

**Le dashboard interroge une table supprimée depuis le 2026-08-09.**
`getTodayFootballEvents` et `getNextFootballEvents` lisent `sport.football_next_matches`
([dashboard/service.ts:152](../../src/modules/dashboard/service.ts#L152), [:302](../../src/modules/dashboard/service.ts#L302)),
table droppée par [`20260809_football_drop_widget_tables.sql`](../../supabase/migrations/20260809_football_drop_widget_tables.sql).
Le `Promise.allSettled` de `getDashboardData` **avale l'erreur** → le football du dashboard est vide
depuis 27 jours, sans une seule alerte.
*Même schéma exact que la panne de 40 h : l'échec est structurellement silencieux.*

### 3.2 — Le référentiel de la base n'est pas dans le repo

- **28 tables sur 64 n'ont AUCUNE migration qui les crée** — dont **tout Tasks** (`tasks`, `projects`,
  `statuses`, `workspaces`, `tags`, `task_tags`, `task_activities`), **tout F1** (7 tables), **tout
  Tennis** (5), et l'essentiel de Football (`football_teams`, `football_competitions`,
  `football_standings`, `football_best_xi`, `football_legends`, `football_user_settings`).
  `supabase/migrations/` décrit **moins de la moitié** du schéma réel.
- **13 crons sur 14 n'existent que dans la base.** Un seul `cron.schedule` est versionné
  ([`20260807_football_season_rollover_cron.sql:18`](../../supabase/migrations/20260807_football_season_rollover_cron.sql#L18)) ;
  un second est en commentaire. **15 edge functions sur 20 n'ont aucun appelant dans le repo**
  (tout F1, tout Tennis, tout Watching, `notify-new-signup`, `football_sync_competitions/_teams`) —
  leur déclencheur est invisible depuis le code.
- Conséquence : **le repo ne peut pas reconstruire HEGON.** Cloner + appliquer les migrations donne
  une base incomplète et sans crons.

### 3.3 — Quatre catalogues de modules qui divergent

| clé | `shared/constants/modules.ts` | `Dock.tsx` | `dashboard-os/config.ts` |
|---|---|---|---|
| sport | `#10b981` emerald | **`#B6FF2E` lime** | **`#B6FF2E` lime** |
| watching | `#8b5cf6` violet | `--color-accent-watching-vivid` | `#2dd4bf` teal |
| tasks | `#71717a` | `#71717a` | `#a1a1aa` |
| tech | `#06b6d4` | `#06b6d4` | `#a855f7` |
| travel | `#0ea5e9` | `#0ea5e9` | `#38bdf8` |
| fitness | présent | **absent** | **absent** |

`modules.ts` s'auto-décrit *« Canonical module catalog — the single source »* et cite la **Sidebar**,
morte depuis le Dock. C'est le plus faux des trois — et c'est celui que consomment
[`(main)/layout.tsx`](../../src/app/%28main%29/layout.tsx), [`settings/page.tsx`](../../src/app/%28main%29/settings/page.tsx)
et [`app/page.tsx`](../../src/app/page.tsx) (le résolveur de home).
**Quatrième vérité** : les `--section-*` de [`globals.css:352`](../../src/app/globals.css#L352)
(football emerald, books orange, tasks indigo…).

### 3.4 — Ce que `CLAUDE.md` décrit et qui n'existe pas

- §3 *« Server pages/components → `createServerClient()` + `cache()` from React »* → **`cache()` de
  React n'est utilisé nulle part** dans `src/`. `createServerClient` n'apparaît que dans **1 fichier**
  de `src/modules/` (`tasks/actions/onboarding.ts`).
- §3 *« Sports modules (football, tennis, f1) use server-side `createServerClient()` + `cache()` — no
  client-side queries »* → les 3 pages Sport sont **`"use client"`**.
- §4 F1 : les 5 fonctions documentées (`getUpcomingRaces`, `getRecentRaces`, `getDriverStandings`,
  `getConstructorStandings`, `getUserFavoriteTeams`) sont **toutes orphelines** (`knip`). Le vrai F1
  passe par `getF1PageData` → `useF1`.
- §5 *« `SECTION_COLORS` … exportée du Dock — c'est le seul endroit où ces valeurs vivent »* →
  l'export existe ([Dock.tsx:71](../../src/shared/components/navigation/Dock.tsx#L71)) et **personne
  ne l'importe**.
- §9 *« `src/lib/` cleanup »* → **`src/lib/` n'existe plus.** Tâche finie, jamais rayée.

### 3.5 — Code mort prouvé (`knip`)

- **10 fichiers UI orphelins** dans Football : toute la chaîne `squad/` (`BestXI`, `FootballBestXI`,
  `FootballLegends`, `FootballLegendsServer`, `FootballXISection`), `fanlog/FootballFanLog`,
  `hooks/useFanLog`, `hooks/useFootball` — plus `getFanLog` / `getBestXI` dans le service.
- **84 exports + 55 types exportés** non consommés (dont ~30 ré-exports de primitifs `shared/ui`,
  attendus pour un design system ; les 5 de F1 ci-dessus ne le sont pas).
- **2 dépendances mortes** : `@radix-ui/react-dialog`, `playwright` (devDep).
- ⚠️ **`knip` n'est pas exploitable en l'état** : sans `knip.json` il déclare *unused* les 20 edge
  functions Deno et les 12 scripts `.mjs` (points d'entrée, pas des imports), et il rate
  `orgs/service.ts` (importé en `import * as`). **Le configurer est le préalable au cliquet CI.**

### 3.6 — Ce que rien ne surveille

- **La CI ne lance pas les tests.** [`ci.yml`](../../.github/workflows/ci.yml) = lint + tsc + build.
  Les 14 fichiers de test ne tournent que si quelqu'un tape `npm run test`.
- **`ds:coverage` n'est pas non plus dans la CI**, et son résultat commité datait du **2026-07-12** :
  il annonçait 28 primitifs (dont `command` et `tabs`, **supprimés depuis**) et 517 hits. Le vrai
  chiffre est 26 / 475. Le fichier qui promet en tête *« GENERATED FROM THE CODE, never hand-written…
  it cannot lie »* **avait deux mois de retard** — la mesure existe, personne ne la déclenche.
- **⛔ `hq/` et `memory/` sont dans `.gitignore`** ([lignes 48-49](../../.gitignore#L48)). Le plan
  d'audit, cette carte, les décisions par module et toute la mémoire **ne sont pas versionnés** : ils
  vivent sur une seule machine, sans historique, sans sauvegarde. La règle ⭐ *« rien n'atterrit en
  prose seule »* est donc plus dure qu'écrite — ici, la prose n'atterrit nulle part du tout.
  *(Arbitrage owner, pas une correction à faire d'office : ces fichiers contiennent du contexte
  personnel. Voir §5.)*
- ~~**Route morte livrée en prod** : `/icon-lab`, accessible à qui connaît l'URL.~~
  ⚠️ **FAUX — corrigé le 2026-09-07.** Vérifié : `git ls-files` ne le connaît pas et `git log` n'en a
  aucune trace. `src/app/(main)/icon-lab/` et `src/shared/components/app-icon/` sont **gitignorés**
  (lignes 66-67), donc Vercel — qui construit depuis GitHub — ne les a jamais vus. C'est un labo
  **strictement local**, jamais déployé. Rien à supprimer côté repo.
- **`hq/log/` s'arrête en juin.** Un seul fichier (`2026-04-a-06.md`) alors que le protocole impose un
  fichier par mois — juillet / août / septembre (Watching v2, toute la refonte Football) ne sont pas
  journalisés.

---

## 4. LES 20 % QUI PORTENT LE RISQUE

Les fichiers à connaître à fond, parce que tout passe par eux :

| fichier | lignes | pourquoi |
|---|---|---|
| [`watching/service.ts`](../../src/modules/watching/service.ts) | 1 787 | le plus gros fichier du repo |
| [`onboarding/OnboardingFlow.tsx`](../../src/modules/onboarding/components/OnboardingFlow.tsx) | 1 383 | le module entier tient dans un fichier |
| [`goals/service.ts`](../../src/modules/goals/service.ts) | 1 168 | Goals est le hub : 5 modules l'importent |
| [`middleware.ts`](../../src/middleware.ts) | — | la seule garde d'accès de toute l'app |
| [`Dock.tsx`](../../src/shared/components/navigation/Dock.tsx) | — | LA nav, rendue une fois pour tout |
| [`dashboard-os/layout-engine.ts`](../../src/modules/dashboard-os/layout-engine.ts) | — | placement du home screen ; référence drag & motion |

**Le graphe de dépendances est sain** : aucun cycle. `goals`, `settings` et `command-center` sont les
trois seuls hubs, et ils n'exposent qu'un service et des hooks.

---

## 5. CE QUE LA PHASE 1 DOIT TRAITER

1. Le bug vivant §3.1 (dashboard → table droppée).
2. `knip.json` + `knip` en CI · `npm run test` en CI · `ds:coverage` en CI.
3. Supprimer les 10 fichiers Football orphelins, les 2 dépendances mortes, `/icon-lab`.
4. Réduire les 4 catalogues de couleurs à 1.
5. Rapatrier dans le repo le schéma réel et les 14 crons (§3.2) — **le seul point qui ne se règle pas
   par une suppression.**
6. **Trancher `hq/` + `memory/` gitignorés** (§3.6). Trois issues : versionner tel quel · versionner
   en scindant ce qui est personnel · assumer le non-versionnement et poser une sauvegarde.
   **Arbitrage owner — rien ne bouge sans son go.**
