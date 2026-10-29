# HEGON — NOW (état courant)

> Réécrit le 2026-09-17. Ce fichier dit **où on en est et quoi faire ensuite** — rien d'autre.
> Un jalon fini sort d'ici (il vit dans `git log` et dans `modules/*/decisions.md`). Le plan de
> travail ordonné est dans `hq/roadmap.md` ; le pilotage du chantier audit dans `hq/audit/plan.md`.

## 🛑 Le cadre en vigueur — CHANTIER AUDIT & FONDATION (depuis le 2026-09-05)

- **Gel par module** (amendement 6, 12/09) : un module reste gelé jusqu'à la fin de SON audit de
  phase 3. **Watching = audité → dégelé** (`modules/watching/audit-2026-09.md`). **Sport, Books,
  Journal, dashboard-os, Habits, Goals, Tasks = gelés** : on n'y touche pas, même pour une ligne.
- Phases 0-2 ✅ (carte du repo, purge + cliquets CI, doctrine R1-R11, watchdog, sauvegarde).
- Phase 3 = un audit par module, « petit et jugé raté » d'abord : **prochain = dashboard-os**
  (2 693 l.) — il débloque les deux tuiles que Watching attend. Brief dans `audit/plan.md` §7 bis.

## ✅ Watching v4 — FINI le 2026-09-17

§1→§8 (panneau saison, watchlist, dismiss persistant, priorité éditable, ruban TOP 10, le Musée
Oscars/Emmys, achievements « pourquoi », fondu des affiches), **§11 Follow people** (suivi, Cast &
Crew paginé, Library › People avec rails de 20 et panneaux paginés par 50 en base, robot
`watching-people-sync` hebdo + à chaud), **§12 le « + »** d'ajout direct, la grammaire des affiches
(bookmark à 5 états, cercles de statut, cœur à gauche, « + » à droite). Bloc « autonomie » du 17/09 :
les robots répondent 500 quand tout échoue, fraîcheur awards / people dans le watchdog, R7 sur tous
les appels tiers, titres pas encore sortis rafraîchis chaque semaine, Dock peint côté serveur,
squelettes homogènes (un `Pulse`, cellules de rail partagées `RAIL_CELLS`, plus de doublon More Like
This). Deux irrégularités de chargement (`library` / `movies`, notées le 22/07) restent **non
mesurées en prod** — en dev la compilation masque tout ; à mesurer sur hegon.fr avant de toucher.

**Reste (avec déclencheurs) → `modules/watching/backlog.md`** : §9 For You v2 (à cadrer), §10
thèmes (attend un mp3), Universes (non tranché), dettes mesurées.

## 🤖 Ce qui tourne tout seul (20 crons, watchdog toutes les 6 h, sauvegarde hebdo)

Watching : `series-sync` 4 h · `anime-cours` 6 h · `trending` quotidien · `for-you` 5 j ·
`awards` Oscars/Emmys (2×/jour en saison, mensuel sinon) · `people` mardi 05:35 (+ titres non
sortis). Un robot mort se voit dans `Watchdog` (GitHub Actions) et sur healthchecks.io — pas
besoin d'ouvrir l'app. Clés : `AGENTS.md` §Keys.

## 📏 Mesuré le 17/09 (dev, 2ᵉ chargement, octets réels)

Hubs 51-92 KB · Library 0 KB (semée serveur) · Lists 25 KB · Stats 399 KB (a besoin de tout) ·
People **102 KB** (index lean) · **Awards 980 KB** — le poids est le canon (`award_entries` 730 KB
pour 20 cartes), levier = requête dédiée, chiffré dans `modules/watching/backlog.md`.

## ▶️ PROCHAINE SESSION

0. **Squelette de la fiche** (`DetailSkeleton`) : l'owner le voit « hero vide, My Take et Details
   saturés, Top 10 / In list / Accolades absents » — impossible à capturer ici (la ligne arrive
   avant l'interception). **Partir de SA capture** au reload d'une fiche (film + série), puis
   corriger en une passe. Ensuite, si voulu, la requête dédiée d'Awards (~1 h).

1. Lire `AGENTS.md` (Codex) ou `CLAUDE.md` (Claude) → ce fichier → `hq/roadmap.md`.
2. **2ᵉ audit de phase 3 : dashboard-os** (brief `audit/plan.md` §7 bis). Ne pas construire de
   feature dessus avant la fin de l'audit — c'est la règle qui a rendu Watching fiable.
3. Après l'audit : les deux tuiles Watching → Dashboard (« nouvelle saison / épisode », « Oscars
   dans N jours, N nommés à voir ») — la moitié Watching est prête.

## Active workstream ? Watching Wave 2A remote schema ready (2026-10-12)

Wave 1 is locked. The owner explicitly authorized the controlled CLI push after the read-only migration-state audit. Migrations `20261011000000` and `20261012000000` are applied and recorded on linked HEGON project `femvhonlpafdajyamvcu`; registry now has 23 matching versions.

Remote schema counts match the reference (76 tables, 800 columns, 220 policies, 39 functions, 266 indexes, 20 crons). All 924 media rows and personal-field/identity hashes are unchanged; non-demo Movies 375, Series 74, Anime 69. Existing rows have revision/version 0; historical tables are empty. Watching/Library localhost reads and canonical selectors return 200 with no missing-column errors. Evidence: `modules/watching/wave2a-history.md` (controlled remote migration section).

No application/function deployment, personal write tests, manual SQL application, repair, destructive cleanup, commit or push. Next: owner-authorized coordinated application rollout/review; Wave 2B+ remains excluded. Prior dashboard audit/backlog sequencing remains in `roadmap.md`.


## Targeted Watching availability fix (2026-10-13)

Wave 1 and Wave 2A are committed; Wave 2A remote schema is applied. The targeted pre-Wave-2B fix gates consumption actions through `isWatchableNow`, including unowned discover titles, detail cards, poster/list menus, quick-add and action handlers. Want to Watch/Waiting For and Library status semantics remain unchanged. 118 targeted tests, TypeScript, affected-file lint (zero warnings), design-system coverage regeneration and git diff --check pass. No production personal writes, schema changes, deployment or Git staging/commit. Next: owner review/commit of this fix. Wave 2B remains excluded.

## Active Watching workstream: Wave 2B ready for review (2026-10-13)

Resumed from current tree; provider evidence/scoped opinions and cour stale-context guard are locally validated. Kuroko/Bleach, cour identity/legacy retention, revision/snapshot races,162 targeted tests, TypeScript, affected lint, Deno and local migration rehearsal pass. Final production build rerun failed only fetching Google Fonts Inter. No remote2B migration/deployment, production writes, staging/commit/push or2C work. Next: owner review; coordinated rollout requires explicit authorization. Availability/badge dirty work remains separate. Manifest and validation: `modules/watching/wave2b-provider-ratings.md`.

## Watching Wave 2B remote ready and locked (2026-10-13)

Owner-authorized finalization complete: two wording fixes,5 receipt tests, TypeScript, affected lint, coverage and diff checks pass. Linked HEGON project confirmed; clean23-version registry/dry run established only20261013000000 pending. Normal CLI push applied Wave2B;24 local/remote versions now match. All new schema objects/grants/RLS/indexes and global reference counts verified through read-only checks. Fresh925-row/type counts plus identity/personal hashes unchanged; all provider/scoped-rating/history tables remain empty. No production personal writes, app/function deployment or Git commit/push. Evidence: `modules/watching/wave2b-provider-ratings.md`, finalization section. Wave2C is deferred to the next session.

## Watching Wave 2C remote ready and locked (2026-10-20)

Final re-review is lockable with no blockers/minor fixes. Owner-authorized controlled migration20261018000000 applied to linked HEGON femvhonlpafdajyamvcu after a clean25-local/24-remote audit and single-migration dry run. All25 versions now match; Wave2C recorded once. Remote function body hash equals local reviewed SQL, signature/security/ACL preserved. Fresh before/after full-row hashes match for925 media,1 viewing record,1 range,9 rewatches and empty provider/scoped-rating tables; type counts662films/125series/138anime unchanged. Read-only compatibility checks pass. No application code change/deployment, production personal writes, reconciliation, Git commit/push or2D. Evidence: `modules/watching/wave2c-history-consumption.md`, finalization section. Wave2D is deferred to the next session.

## Watching Wave2D local code lockable; remote migration pending (2026-10-21)

Permanent achievement receipts and Goals-owned completion receipts are implemented locally. Fresh authenticated server observations use canonical history, raw thresholds and immutable owner/org receipts; Marathoner shares canonical Stats hours including accepted legacy estimates; other receipt metrics keep explicit evidence.10 consumption milestones are sticky;2 current-opinion badges stay live. Goals remain completed with lower current progress/date retained.195 targeted tests, PostgreSQL concurrency/RLS/data-safety rehearsal, TypeScript/lint/build and four-width isolated UI checks pass. Local migration20261021000000 and measured schema reference pending review/owner approval; no remote application, production mutation, deployment, Git commit/push or2E. Exact manifest/policies/evidence: `modules/watching/wave2d-milestone-receipts.md`. Next: targeted2D review; controlled migration only after explicit owner approval.

Targeted pre-review fixes (2026-10-21): legacy full dates now populate monthly/daily analytics at all years, without suppressing pre-2024 dates. Fresh read-only real-library reproduction preserves3331.0333333333333 hours and restores119 calendar months. Marathoner server eligibility now matches that canonical mixed Stats total, rather than0.8 strict-only hours. Remote2D tables/functions remain absent; migration not applied. Final2D review has NOT started. Details/evidence: modules/watching/wave2d-milestone-receipts.md.

Monthly/chart follow-up2026-10-21: user's2021/2022/2023/2026 reproduced; zero-month positions restored, Bleach legacy2023 date retained, Dexter/Presumed Innocent stored2026 calendar events restored without moving annual hours. Rating points/ticks share numeric coordinates and exact decimal scores no longer round maxima upward.150 targeted tests, TypeScript/lint/coverage and offline actual-card4-width checks pass. No new migration or production mutation. Final2D review still NOT started;2E not started. Evidence in wave2d-milestone-receipts.md.

Final targeted2D review verdict: WAVE 2D NOT LOCKABLE.225 tests/12files, TypeScript, affected lint, schema/diff checks and fresh local PostgreSQL RLS/concurrency/migration rehearsal pass, but three uncovered cases reproduce: same-year partial history still moves unrelated legacy calendar dates; retained local (noTMDB) titles lose consumption after deletion; Stats user-only scope/cache can diverge from org-scoped receipts for a multi-org owner. No application source correction or production operation during review. Next: correct these cases with regression tests, then targeted re-review; remote migration remains pending. Evidence: modules/watching/wave2d-milestone-receipts.md final-review section.

Three final-review findings corrected and targeted re-review passes2026-10-21: same-year legacy calendar retained under scoped history; deleted local title identity shared across analytics/service/receipts; Stats/history/rewatch reads and cache now user+organization scoped, with captured org for Stats/receipt queries.235 targeted tests/13files, TypeScript, affected lint and schema/diff/CRLF checks pass. Verdict: WAVE 2D LOCKABLE (local code); no known remaining local blocker. Remote2D migration remains UNAPPLIED, so receipt_snapshot errors remain expected until controlled owner-authorized migration. No production writes, deployment, Git mutation or2E. Evidence: modules/watching/wave2d-milestone-receipts.md correction/re-review section. Next: explicitly authorized controlled remote migration and post-migration verification.

Pre-migration verification2026-10-21 complete: linked HEGON femvhonlpafdajyamvcu is ACTIVE_HEALTHY;25 historical versions match, only20261021000000 pending. db push --linked --dry-run succeeds and plans only watching_milestone_receipts.sql (no seeds/roles). Schema reference/diff checks pass. No actual push or receipt observation; remote migration remains unapplied. Next: explicitly authorized normal migration and post-checks.

Read-only owner-requested legacy-date diagnostic2026-10-21: remote registry now includes20261021000000 (owner applied; full post-migration verification not performed here). Screenshot gap is a conditional paragraph occupying a chart-grid column.2022 unknown event is legacy Game of Thrones rewatch2022-03-01, scope/duration not frozen. Old-date audit:291 personal consumed dated titles through2024;127December,101December31,89December31noon. Old year-projection scripts found; exact personal months require exports/backups/owner evidence, not random redistribution. No date/data/UI writes or2E. Evidence: modules/watching/legacy-date-diagnostic-2026-10-21.md; ignored291-row review CSV prepared with no replacement dates.

Targeted estimate proposal2026-10-21: owner states no personal evidence exists and explicitly requests plausible synthetic dates; authorized offline proposal only, not a DB write or general2E work.89 strong31Dec/noon candidates reviewed with49 public TMDB release reads and stored TV dates.85 estimates prepared (2TV metadata-gap flags),4ambiguous rows left blank;202other dated rows unchanged. Dates are explicitly ESTIMATED_WITHOUT_PERSONAL_EVIDENCE, never recovered/confirmed history. Original291-row CSV preserved. Review files: scripts/.shots/watching-dates-through-2024-proposal.csv and watching-date-proposal-review.md. No mutation/migration/receipt observation, deploy or Git write. Application requires approval of this concrete mapping and preservation of estimated provenance; years unchanged.

Targeted owner-approved estimated legacy-date repair prepared2026-10-21: migration20261021010000 contains exactly85 approved mappings, same years, explicit INFERRED/NONE evidence and original-value provenance;4 review cases+202 non-candidates excluded. Fresh85-row backup/preflight clean. Local PostgreSQL application/stale/repeat/unchanged-data checks and all-time+2013-2026 Stats invariants pass; affected lint/schema/diff pass. No production write or2E. Next: owner runs npx supabase db push --linked, then read-only post-application verification. Details: modules/watching/legacy-date-diagnostic-2026-10-21.md.

Targeted rewatches correction2026-10-21: full-title default + multi-select display seasons, date-bounded episode coverage and immutable membership/runtime; no episode-grid UX. Legacy Complete action replaces one old event atomically after explicit confirmation. Duration notice grouped under Hours Watched.214 targeted tests and local PostgreSQL replacement/RLS/retry/concurrency tests pass; four-width actual offline React/browser fixture passes. New backward-compatible validator migration20261021020000 permits optional episode runtime; owner migration must precede watching-series-sync deployment. No production writes, deploy, Git mutation, general2E or Black Clover poster fix. Evidence/manifest/limits: modules/watching/rewatch-hardening-2026-10-21.md.

Rewatch initialization follow-up2026-10-21: remote registry now20261021020000, but fresh read-only checks find no accepted episode heads for the blocked personal TV/anime titles. Series robot now bootstraps missing catalogues regardless of fresh/ended cadence, prioritizes legacy rewatches, preserves normal work capacity and failed-check cooldown.82 targeted tests and four-width offline browser checks pass. UI explains missing sync instead of a futile refresh. Next: owner deploys and invokes watching-series-sync; read-only verify heads/runtimes afterwards. No production writes or new migration.

2026-10-22 Hunter rewatch follow-up: owner robot run accepted12catalogues and rejected Hunter x Hunter. Fresh TMDB replay proves absolute season numbering63..136 and137..148; helper normalizes only complete offset-proven blocks, preserving IDs/dates/runtime. Explicit ProviderHead/Map tuples address IDE unknown inference; actual edge Deno check passes.69 tests and raw148episode/3503minute replay pass. No new migration or production writes. Next: owner redeploy/invoke watching-series-sync and verify Hunter acceptance. Details in rewatch-hardening-2026-10-21.md.

2026-10-22 business discussion audit: fresh read-only Watching snapshot reproduces Black Clover2022 rating8.5/68hours but null season poster due >=2-stamped-years guard;2026 no annual rating leak. Long Hauler500live vs0eligibility, Ever-Present132live vs9eligibility (Activity133months), Marathoner already has permanent receipt but UI awaits full observer pipeline.7.37MB raw snapshot, warm pure receipt evaluation~22ms; posterw92?w154 costs+4.5?19kB/sample. Legacy first-watch hours remain runtime-sensitive; frozen canonical rewatches do not. No app/DB edits; discuss accepted-legacy policy and scope/date semantics before implementation. Evidence: modules/watching/business-discussion-2026-10-22.md.


2026-10-22 owner-approved Watching corrections locally validated: shared accepted consumption/calendar/achievement evaluator; stable legacy scope/runtime migration, annual posters (Black Clover2022 S1/2026 S2), receipt-first Marathoner and adaptive compact posters.218 tests/11files, TypeScript, zero-warning affected lint, Knip, schema/coverage/diff checks, PostgreSQL RLS/concurrency rehearsal and actual-component4-width browser checks pass.513-row local audit replay retains every annual/all-time count/hour/activity/rating distribution; Long Hauler500, Ever-Present133, Marathoner3729.9 match UI/server. Fresh read-only linked registry:28remote match28historical local; only20261022000000 pending. No production mutation/deployment/Git writes or2E/final2D review. Next: owner dry-run/push single migration, read-only post-check, localhost refresh. Manifest/proof: modules/watching/accepted-consumption-2026-10-22.md.

2026-10-28 Bleach completion diagnostic: read-only remote state confirms today's S2E50 FIRST_WATCH with CONFIRMED DAY precision, but global watched_at remains2023 and FINISHED_TITLE completion date is UNKNOWN. Actual offline status-writer replay reproduces preserved S2=2023 -> max-year finish projection; available4TYBW parts are not overlaid because raw TMDB has2seasons. No application/production/Git mutation. Targeted business correction remains to discuss; do not move all earlier S2 consumption to2026. Evidence: modules/watching/bleach-completion-diagnostic-2026-10-28.md.


2026-10-28 Bleach correction ready locally: canonical terminal-viewing completion date, consistent completed-period label, retained legacy calendar before sorting-date changes, no redundant empty first-watch on Finish. Exact backup replay repairs watched_at to2026-10-28 and label2020-2026 while preserving166.4h, annual/monthly activity, S1=2020/S2=2023 and4ratings.310 targeted tests/11files, TypeScript, zero-warning affected lint, PostgreSQL stale/rollback/idempotency/immutability checks and4-width real StatusCard fixture pass. Linked HEGON registry through20261022000000; normal cachedCLI2.121 dry-run succeeds and lists only20261028000000, no seeds/roles. Full owner repair migration and unchanged-count watchdog reference prepared. No remote repair, deploy, commit/push or2E/final2D review. Next: owner npx supabase db push --linked, then read-only remote post-check. Evidence: modules/watching/bleach-completion-diagnostic-2026-10-28.md.


## Watching point 1 — viewing periods and parts (2026-10-28)

Owner authorized closing the discussed season/part/date/rating rules before final2D validation. Read-only remote verification confirms Bleach repair migration20261028000000 is already applied, revision8, finish2026-10-28, old calendar/year maps retained. Local implementation adds nested verified AniList parts to ordinary multi-season anime, S2 period2023–2026, precise canonical EP labels, hidden historical scores after growth and no overlapping season/part annual votes. December/January hours stay in their viewing years; real Bleach retains166.4h and all annual/monthly totals. No new migration, production write, commit/push or2E/final2D review. Owner worker deployment remains pending before closing activation: npx supabase@2.121.0 functions deploy watching-anime-cours-sync. Exact manifest/evidence/limits: modules/watching/season-periods-and-parts-2026-10-28.md. Local validation complete:378 targeted tests, TypeScript, lint, Deno, Knip and schema/diff checks pass. Next: owner deployment/verification, then point2 final2D validation.


## Watching season-panel regression correction (2026-10-28)

Owner requested discussion then authorized restoring hidden season ratings and correcting the panel. Legacy saved opinions are visible again without promoting their coverage to verified; real growth and remappings remain guarded. Poster is fixed2:3 in normal flow; personal controls/history live below it, with one period row and plain optional explanation. Real JJK cour scores8.5/9/9.5 confirmed read-only.169 targeted tests/7files, TypeScript, zero-warning affected lint, Knip, coverage and main/HQ diff checks pass.12 isolated browser cases across4widths verify visible poster/panel scores, fixed image dimensions, disclosures and previous9 confirmation callback. No migration, production write, deploy or Git mutation. Report: modules/watching/season-panel-polish-2026-10-28.md. Next: owner discusses the visual result; final2D review/2E remain unstarted. Prior worker deployment remains a separate owner activation step.


## Watching final Wave2D validation (2026-10-28)

Local code validated; remote schema ready; full operational closure awaits confirmation of updated watching-anime-cours-sync deployment. Found and fixed shared history/rating cache invalidation after progress/finish/year edits plus history_date-only analytical invalidation, at the existing mutation boundary.430 tests/21files, TypeScript, affected lint, Deno, Knip, production build, real local PostgreSQL safety/concurrency rehearsals and four-width actual-component browser checks pass. Fresh owner-authorized read-only snapshot:3730.7h, Long Hauler500, Ever-Present133; server/UI agree and existing receipts remain permanent. Bleach S2=2023–2026 with only48min in2026; Hannibal8.75; Black Clover2026 no legacy-rating leak.30local/remote migrations match; watchdog81/857/225/49/277/20 matches; five receipt SQL bodies/ACLs verified. No production write/deploy/Git mutation or2E. Report: modules/watching/final-wave2d-review-2026-10-28.md. Next owner step: npx supabase@2.121.0 functions deploy watching-anime-cours-sync, then activation verification before full2D lock and2E discussion. Earlier pending-migration notes above are historical and superseded.


2026-10-28 owner-found Top Picks follow-up corrected: annual eligibility requires first-view consumption, not rewatch-only activity, even for favorites. Real Interstellar/Fight Club/Identity removed from2026 picks (33→30); original-year/all-time eligibility and every other Stats output unchanged.205 tests/7files, TypeScript, affected lint and four-width real card/panel fixtures pass. No migration/production/Git write or2E. Report: modules/watching/top-picks-rewatch-fix-2026-10-28.md. Prior worker activation confirmation remains before full2D closure.


2026-10-28 rated-scope/Part4 amendment locally validated: Top Picks labels follow the exact score votes (Hannibal8.75 S1–S2, JJK9.5 S3, actual Bleach9 S2). Owner confirms all Bleach Part4 E41–50 watched in2026. New controlled data-only migration20261028010000 adds missing E41–48 with confirmed YEAR precision, retaining original E49/E50 days, frozen192legacy minutes and all ratings. Projected2023=16h/2026=4h, all-time166.4h unchanged.202tests/8files, TypeScript/lint/Knip, schema/diff, disposable PostgreSQL safety tests and4width UI pass. Only this migration is pending; remote remains revision8/30versions. No production/Git write/deploy/2E. Next owner: npx supabase@2.121.0 db push --linked, then read-only post-check; prior AniList worker activation also unconfirmed. Report: modules/watching/rated-scope-and-bleach-part4-2026-10-28.md.


2026-10-28 owner defers Bleach part-year correction and requests commit commands. Keep production data as currently stored; unapplied20261028010000 and its dedicated rehearsal archived outside deployable paths in ignored scripts/.shots. Removed its runner hook and restored watchdog watermark20261028000000; no pending local migration. Existing completion repair (already remote) and generic Top Picks scope fixes retained.2D application code validated and ready to commit; full operational lock still needs confirmation of previously requested AniList worker deployment.2E not started. No Git staging/commit/push or production write executed.


2026-10-28 owner deployment verified read-only: watching-anime-cours-sync v11 ACTIVE, deployed22:16 Europe/Paris. Downloaded remote code and all4shared dependencies exactly match committed local source (normalized SHA256). Prior deployment-unconfirmed notes are superseded. Active6h cron's last retained response19:10Paris is200/ok/failed0, before v11; do not claim a post-deploy execution test. No robot invocation/deploy/production/Git write. App commitd2a041a is present, app tree only unrelated debug.log. Bleach part-year data repair remains deferred;2E not started. Next: observe normal v11 cron for operational runtime evidence; frame2E separately with explicit owner scope.


## Current Watching milestone - 2026-10-28

Wave2D CLOSED for the validated scope. Commit d2a041a present; schema/watchdog and deployed anime worker v11 source verified. Next normal v11 cron is routine monitoring, not a closure blocker; no post-deploy execution is claimed. This supersedes older pending activation/closure notes above. Bleach part-year repair stays deferred. Next: owner review of modules/watching/wave2e-reconciliation-scope.md, then a read-only legacy inventory. No2E implementation, production write, deployment or Git mutation in this framing step.


## Current Watching Wave2E read-only audit - 2026-10-28

Owner approved reconciliation scope but explicitly requires audit/report before any correction. Fresh515-item snapshot and current-code replay complete;174tests pass. Report: modules/watching/wave2e-reconciliation-audit-2026-10-28.md. Important new evidence supersedes the prior no-pending/never-applied Part4 statements: remote registry already contains20261028010000, its repair body matches the archived migration and Bleach is revision9. Local deployable registry/reference stop at20261028000000. No repair, rollback or production write performed; Bleach reconciliation stays excluded. Four stale-looking year stamps on three anime have no effect on current tested Stats; true viewing scope remains UNKNOWN. Next: owner reviews findings before any local/remote correction;2E NOT LOCKED.


## Current Watching migration alignment - 2026-10-29

Exact archived20261028010000 restored locally, all3SQLstatements match remote,31/31versions align; watchdog watermark20261028010000 with unchanged counts. No production write, reapplication, migration repair, deploy or Git mutation. Targeted local SQL safety/idempotency rehearsal passes. Full raw fresh replay fails native baseline pg_cron availability; independent SQL probes also prove historical data-repair prerequisites block an empty-data replay. Do NOT claim full replayability. Owner decision2026-10-29: full empty-database reconstruction is separate technical debt, NOT a2E blocker; applied migrations must remain unchanged. Next: owner confirms actual viewing for the three anime from wave2e-three-anime-clarification-2026-10-29.md.2E findings stay audit-only, Bleach and three anime data untouched. Report: modules/watching/migration-alignment-2026-10-29.md. This supersedes earlier remote-only/never-applied/current-pending statements above.


## Current Watching Wave2E generic correction - 2026-10-29

Owner approved the generic scope-aware backwards cleanup, explicitly excluding three-row production edits. Existing personal_history_command now owns invalidation/audit; duplicate client clipping removed.251targeted tests and real local PostgreSQL safety/RLS/scope rehearsals pass, plus TypeScript/lint/Knip/schema/diff. New20261029000000 remains LOCAL, no backfill; apply owner migration before updated client correction path. Tokyo/SDS/FireForce before/after prepared, unchanged remotely. Next: owner review of modules/watching/wave2e-backward-correction-2026-10-29.md, migration rollout separately and explicit data-correction approval. Bleach excluded; empty-DB bootstrap debt does not block2E. No Git staging/commit/push/deployment/production write.


## Current Watching pre-migration review - 2026-10-29

NOT LOCKABLE / NOT SAFE FOR REMOTE APPLICATION: targeted counterexampleR1 proves invalid old cour identity can erase a different retained identity's compatibility slot when ordinal/score coincide. Expected cour_ratings[3]=7, actual empty, retained canonical opinion still1. Candidate unchanged; no push/application. Pending set clean: only20261029000000 (31remote/32local), dry-run single migration/no seeds/roles. Seven before/after personal-data fingerprints and full three-anime rows unchanged. Next: correctR1 locally, add regression, re-review; only then owner migration and unchanged-data verification before the separate three-anime repair. Report: modules/watching/wave2e-backward-review-2026-10-29.md. Prior locally-validated deployment readiness is superseded;2E remains open.


## Current Watching R1 fix - 2026-10-29

READY FOR RE-REVIEW: compatibility slot resolution now checks stable leaf subject identities/frozen membership; removed duplicate ordinal/score-based deletion. Exact {"3":7} regression passes with both timestamp orders, plus different-score controls, retained audit/lineage and no unconditional preservation. Mutation control fails under original rule as expected. Real local PostgreSQL suite,21targeted UI/provider tests, lint/schema/diff PASS. No remote calls/writes, migration application, three-anime correction, deploy or Git mutation in this block. Candidate changed locally; previous remote preflight/hash are historical and must be refreshed before rollout. Report: modules/watching/wave2e-r1-fix-2026-10-29.md.


## Current Watching Wave2E rollout - 2026-10-29

WAVE 2E LOCKED / COMPLETED by explicit owner approval2026-10-29. Owner explicitly authorized agent-run normal CLI migration and three-anime corrections.20261029000000 generic R1-corrected migration applied with all seven personal fingerprints unchanged. Separate data-only20261029010000 removes only the approved Tokyo/SDS/FireForce residuals with exact full-row audit; positions/status/global scores and9.6h/19.2h/7.2h retained.33local/remote versions aligned, no pending; watchdog82/867/226/50/279/20 verified. Full real-code Stats replay retains3730.7h and all annual hours/counts/rating/TopPicks; only false Tokyo2019-12 event removed. Bleach/all other rows and history/rewatches/receipts/highlights untouched. Local PostgreSQL atomicity/idempotence/R1 checks pass. No Git mutation/deploy/new workstream. Empty-DB bootstrap debt remains separate/nonblocking. Next: owner commit/push this locked block; no next workstream started. Report: modules/watching/wave2e-final-reconciliation-2026-10-29.md. Supersedes earlier pending/re-review/no-three-anime-correction statements.
