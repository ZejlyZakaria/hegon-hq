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


## Kuroko follow-up completed - 2026-10-29

Owner confirmed all three main seasons and accidental edits. Live TMDB API verifies25/25/25,75total. Applied guarded data migrations20261029020000 and20261029030000: accepted corrected catalogue, restored audited2018years/8.5scores/estimated date and completedS3E25, then explicit S2/S3YEAR2018 history replacing accidental tombstone suppression without reactivating2026records. Fresh actual Stats replay75episodes/30h in2018,0h2026, Top Picks8.5S1–S3. All other personal rows/history/rewatches/receipts unchanged; two full audits retained.35local/remote versions align, no pending; watchdog counts unchanged/watermark20261029030000. Local SQL atomicity/idempotence plus actual Stats/retracted-history fixtures pass. No application/worker refactor/deploy or Git mutation. Wave2E remains LOCKED. Report: modules/watching/kuroko-reconciliation-2026-10-29.md.


## Watching image / Top Picks implementation - 2026-10-29

Owner-approved local implementation complete. Full-work annual rated labels hidden only with complete first-view/rating coverage; partial/multi-year scopes retained. Film release year removed in both Top Picks surfaces; Kuroko/BreakingBad concise, Hannibal8.75S1–S2unchanged. Shared native image helper now selects density/width candidates for season header/strip, carousels, ForYou and expandedDontMiss; compact TopPicks density and blur preserved. Eager pictures avoid fallback double-preloads.192targeted tests, TypeScript/lint/Knip/schema/diff pass; coverage regenerated unchanged. Real-data15period replay preserves every non-presentation result. Eight isolated JSX/CSS/NextImage browser cases plus eager-request test pass, screenshots read. No production mutation/new migration/deploy/Git mutation; prior Kuroko migration/reference changes remain separate. Wave2E remainsLOCKED. Next: owner reviews local UI and chooses commit/deployment. Report: modules/watching/image-quality-and-top-picks-implementation-2026-10-29.md.


## Watching scoped ratings / rewatch follow-up - 2026-10-30

Owner-approved local follow-up implemented after locked2E: proven single-season/cour opinions synchronize global/scoped projections transactionally; scope growth restores independence; nested parts use stable identities. Bleach remote read confirms no accepted AniList head despite cached parts: robot bootstrap now prioritizes missing accepted evidence without bypassing scope guards. Rewatch has one entry, editable date and compact shared controls.212targeted tests and disposable PostgreSQL rehearsal pass, including no backfill/+1function, original provider/RLS/backwards contracts and four nested parts; browser fixtures at4widths checked. No remote data change, migration application, deploy or Git mutation. New20261029040000 and planned watchdog51functions remain LOCAL/PENDING. Next: owner-controlled migration/robot rollout after exact pending/project verification; then real Bleach UI check. Python not started. Report: modules/watching/single-scope-ratings-and-rewatch-2026-10-30.md.


## Viewing history correction - 2026-10-30

Owner rejects raw assertion dump and explicitly chooses today as the default film watched date, editable afterward. Replaced public audit rows with a dedicated timeline: one movie viewing, three evidenced Bleach consumption rows with terminal finish, four distinct concise GOT rewatches. Old component/duplicate rewatch popover removed; edit/remove retained in the timeline. Scoped rating milestones and tombstones remain internal, not more consumption. Shared service defaults new film insert/update dates atomically, preserving explicit past/unknown values and historical corrections; no existing film redated.216targeted tests/7files, TS/lint/Knip/schema/diff and four browser widths pass; captures read. No new migration or production mutation. Existing Boogie Nights UNKNOWN canonical date remains owner-confirmable via Watched. Prior single-scope migration/robot rollout still pending; no commit/push/deploy. Report: modules/watching/viewing-timeline-and-film-date-2026-10-30.md.

## Best Episodes image quality - 2026-10-30

Owner explicitly approves only this image improvement. Other recent local UI/business follow-ups remain REJECTED / UNVALIDATED; earlier technical-green entries do not constitute owner acceptance. Best Episodes now uses w300 at1x and native original at2x/higher, preserving264pxframe/hover/lazy/fallback and existing formatter.45targeted tests, TS/lint, coverage/diff and12native-browser source-selection cases pass. Original HD payload may increase; no fabricated source detail or latency claim. No production mutation/deploy/Git mutation. Next: owner review of this isolated change, then discuss rejected changes before further implementation. Report: modules/watching/best-episode-image-quality-2026-10-30.md.


## Personal card / viewing panel redesign - 2026-10-30

Owner approved the revised mockup for implementation. Local replacement of rejected per-event timeline: compact state/action card, one contextual SlidingPanel, grouped first viewing by season/actual nested part, independent rewatches, separate provider block, explicit correction vs viewing. Uses the existing Stats consumption ledger; no alternate hours calculation or invented part dates. Saved516-item replay preserves first-view episode/minute totals. Actual-component browser fixture passes390/768/1024/1440 with no overflow/errors and correct navigation/correction callbacks; screenshots read.196targeted tests, TypeScript, affected lint, Knip CI, regenerated coverage and whitespace checks verified. No authenticated live-page end-to-end claim, fresh remote query, production mutation, new migration, deploy or Git mutation. Pending20261029040000 remains separate/unapplied. Next: owner reviews actual localhost UX; this implementation is NOT yet owner-accepted. Report: modules/watching/personal-card-redesign-2026-10-30.md. This supersedes the rejected event-row UI, not its internal audit records.


## Watching activity classification ? 2026-11-01

Owner-requested generic rail fix implemented locally. Knight S1E6/[6] no longer stays In Progress because of stale posture; shared activityBucket classifies both rails and Stats status counts, before display limits, with paginated candidate reads. Duplicate component filtering/merging removed.122 targeted tests pass, preserving204min and original date. No production/schema/Git mutation or deployment. Localhost connectivity probe timed out; live authenticated UI unverified. Report: modules/watching/activity-classification-2026-11-01.md. Right-sidebar redesign remains discussion-only; prior rejected UI work and scoped-rating pending rollout are unchanged.


## Personal sidebar implementation - 2026-11-01

Owner-approved local follow-up implemented: contextual viewing panel, menu-only Log rewatch, inline position correction, provider logo footer and revised sidebar order. Quick Stats now uses the existing accepted/frozen consumption summary; stale Finished dates and independent current-runtime rewatch multiplication removed. Details separates aired/announced counts and validates catalogue dates.137 targeted tests and actual-component browser checks at4widths pass; visual owner acceptance pending. No production mutation/deploy/Git mutation. Report: modules/watching/personal-sidebar-2026-11-01.md. Supersedes prior discussion-only sidebar note.


## Detail rails and broadcast years - 2026-11-01

Owner-requested season/BestEpisodes repeated-navigation fix implemented: measured card-aligned shared hook, stable end/rapid targets, matching header/gutter spacing. Bleach Broadcast years now uses whole-work aired season dates instead of original-run-only OMDb.44targeted tests, TS/lint/DS/diff and4width actual-component browser checks pass; screenshots read. No production/schema/Git/deployment change. Right-sidebar critique prepared for discussion, especially persistent Top10 editor, large list carousel and contribution matching. Report: modules/watching/detail-rail-alignment-2026-11-01.md.


## Watching sidebar polish - 2026-11-01

Approved compact spacing/KPI cells/always-visible parts/plain titles/compactTop10+InLists implemented locally. Rank/favorite decoupled in writes and reads. Contributing membership shares the existing live-goal evaluator; no Goals module edit or receipt-rule change.101targeted tests,20saved-snapshot goal parity comparisons and4width actual-component interaction checks pass; screenshots inspected. Owner visual acceptance pending; no production/schema/Git/deploy mutation. Report: modules/watching/sidebar-polish-2026-11-01.md.


## Watching list artwork / awards surface - 2026-11-01

Owner-approved collage tiles replace compact list rows; membership actions stay in Manage. Accolades uses dark gold with lighter nomination-only emphasis. No business data/schema changes, production writes, deployment or Git mutation. Report: modules/watching/list-collage-award-surface-2026-11-01.md. Owner visual acceptance pending.

Owner correction to list/artwork direction: horizontal65percent list tiles with three portrait cells, shared aligned navigation and complete-poster fitting; Accolades now solid visible gold with dark copy. Prior stacked/dark-tint presentation is superseded. Four-width component checks cover tile width, horizontal arrangement, navigation and Manage actions. No data/schema/Git/deploy mutation.


## Accolades hierarchy - 2026-11-01

Owner-approved graphite/counter/featured-category direction implemented, superseding the solid gold surface. Tracked ceremony metrics and broader external totals are explicitly separated; no competing12/16Emmy headlines or truncated categories. Nomination-only/external-only/empty states retained.38targeted tests plus4width actual-component checks pass; screenshots inspected. No production/schema/Git/deploy mutation. Report: modules/watching/accolade-hierarchy-2026-11-01.md. InLists unchanged; owner visual acceptance pending.


## Detail page final polish - 2026-11-01

Owner-requested local corrections implemented: four aligned rails, unchanged poster tokens, natural trailing-edge navigation without synthetic tail space, concise empty collection states, animated Lucide More details and four-line My Take with conditional Read more.45 targeted tests, TypeScript/affected lint/DS/diff and four-width actual-component browser checks pass. Supersedes terminal left-alignment spacer behavior; single non-overflowing list tile remains65%. No production/schema/Git/deployment mutation. Next: owner reviews localhost UX. Report: modules/watching/detail-final-polish-2026-11-01.md.


## Lists table source quality - 2026-11-02

Owner accepts prior detail UI and requested list-table image quality. Table poster now uses existing native density helper at actual40px token size (w92 at1x/2x,w154 at3x), unchanged layout/lazy/custom-image behavior.9tests, TypeScript/lint/DS/diff and12 browser source/frame checks pass. Focused review: Knip CI passes; one unused Watching helper remains, LegacyConsumption is used despite extended Knip warning; list mutation invalidation consistency and keyboard navigation are next recommendations, not implemented. No global performance/no-dead-code certification or production/Git/deploy mutation. Report: modules/watching/list-table-quality-2026-11-02.md.


## Watching consolidation - 2026-11-02

Owner-approved pastel badge palette/skin, identical caught-up labels, inline Last Watched title badges, image-only hover zoom, For You direct add, native table keyboard controls and list cache consistency implemented locally. Removed the identified unused helper; retained used LegacyConsumption. Shared all-time Stats memo avoids duplicate computation:516-item warmed A/B median29.65->15.89ms, equal outputs; selected-year path stable around29ms.109 targeted tests and four-width mocked browser checks pass; desktop hover movement reproduced13.86->0px. Live automated localhost session cannot load workspace, so global network/hydration performance is NOT certified. No production/schema/Git/deploy mutation. Next: owner visual acceptance and connected-page performance check when automated workspace access works. Report: modules/watching/badge-accessibility-performance-2026-11-02.md.


## Badge experiment REJECTED / hover restored - 2026-11-02

Owner rejects the pastel skin and image-only carousel zoom. Original badges/CSS/Marks restored from saved working-tree baseline; experimental adapter removed. Whole-card MediaCarousel/ForYou hover restored. Independent add/accessibility/cache/cleanup/Stats fixes retained. Detail rails regain6px hover clearance with matching negative margins, preserving aligned content and token sizes.99 targeted tests, TypeScript/lint/Knip/DS/diff pass. Production test session requires login; native Chrome inspection automatically stopped because URL could not be determined reliably; no live production hover comparison claimed. Report: modules/watching/badge-rollback-hover-2026-11-02.md. Supersedes the previous skin/motion acceptance entry.


## Detail alignment + hover - 2026-11-02

Owner requests both horizontal edge alignment and vertical rhythm, preserving the accepted hover. All four headers now reserve36px (Seasons previously36 vs other32); first-image gap18px, section gaps20/24px verified. Shared rail clipping aligns unused gutters at rest; delegated edge-aware hover opens only the needed side, including after paging and at the final card.47 targeted tests, TypeScript/lint/DS/diff and four-format actual-component browser checks pass; captures inspected. Badges/MediaCarousel retain restored design; independent fixes preserved. No production/schema/Git/deploy mutation. Report: modules/watching/detail-alignment-hover-2026-11-02.md.


## Exact image edge / trailing episode anchor - 2026-11-02

Owner clarifies actual image edges vs toolbar axis. Layout-consuming frame borders replaced by inset rings in Seasons/Best Episodes/More Like This. Trailing Best Episode zoom anchors right (and retains anchor during mouse-out); first episode and outward poster zoom preserved. Sizes unchanged. Desktop proof: image951->952px at rest; trailing episode hover955.96->952px against952px toolbar. Four-format browser animation checks and47 targeted tests/TypeScript/lint/DS/diff pass. No production/schema/Git/deploy mutation. Report: modules/watching/exact-image-edge-2026-11-02.md. Owner localhost review pending.


## Best Episodes partial trailing card - 2026-11-02

Owner clarifies partial S3.E4 card, not final list record. Reproduced image AND caption leak at initial/scrolled positions. Scoped episode clipping now holds partial-card right boundary; first/full-final cards retain outward zoom. Removed incorrect persistent right-anchor machinery. Seasons untouched. Four-format physical-pointer browser checks and47 targeted tests/TypeScript/lint/DS/diff pass. No production/schema/Git/deploy mutation. Report: modules/watching/partial-episode-hover-2026-11-02.md. Supersedes prior terminal-anchor interpretation.


## Dark status badges and CI harness - 2026-11-02

Owner requested dark tinted status pills with left icons and black poster veil, replacing the rejected pastel direction. Local implementation preserves accepted hover/alignment. CI AniList harness now registers real provider-refresh dependency and independent heads fixture; all791tests pass. Four-width component screenshots checked. No production/Git/deployment action. Owner visual acceptance and remote CI rerun pending. Report: modules/watching/dark-status-ci-2026-11-02.md.


## Responsive poster actions / bottom status mask - 2026-11-02

Owner-requested local fix:24px visual +/dismiss circles with independent44px touch targets and white add emphasis; status darkening confined to bottom40percent. Season score mask already bottom-only; availability masks unchanged.41targeted tests and8touch/mouse browser configurations pass. Badges remain owner-UNVALIDATED. No production/Git/deploy action. Report: modules/watching/responsive-actions-mask-2026-11-02.md.


## Watching design discussion / personal action controls - 2026-11-02

Discussion only for badge sizing, add/follow hierarchy, People sidebar, trailer, temporal labels and inline Caught Up. No approval inferred. Only explicitly requested mobile heart/menu correction implemented:32px visible,44px touch target,8browser configurations and41tests/TypeScript/lint pass. Badges remain nonvalidated. Report: modules/watching/design-discussion-record-actions-2026-11-02.md.


## Watching design harmony implementation - 2026-11-02

Owner-approved proposals implemented locally: Add/Follow roles, compact/regular status sizes, inline Caught Up, quieter time typography, shared trailer/secondary actions and coherent Person sidebar. Shared AccoladesSummary removes duplicated presentation without merging person/work data.47targeted tests, TypeScript/lint/Knip/DS/diff and16touch/mouse component cases pass; screenshots inspected. No production/Git/deployment action. Owner visual acceptance pending. Report: modules/watching/design-harmony-2026-11-02.md.


## Correction stance / Your highlights - 2026-11-02

Owner-approved local follow-up complete: suspended progress corrections retain stance/reason, do not invent dates or auto-complete/resume; shared app/service guard and pending prospective transaction guard20261102000000. Same-place pencil for all3states. Your highlights uses80px full-width posters and merged duplicates; weekly Caught Up and position share compact geometry.807tests and disposable SQL/16browser configurations pass. No production/Git/deploy change. Migration LOCAL/PENDING; next owner visual review and controlled application. Report: modules/watching/correction-stance-highlights-2026-11-02.md.


## Network / animation studios - 2026-11-03

Owner-approved follow-up prepared: TV legacy studio value relabeled Network; films Production company. Separate primary animation studios from AniList retained per entry and shown from existing cache. Live public API verifies the3reported anime and their studio changes.82targeted tests, local PostgreSQL and28component browser cases pass. New20261103000000 and anime-cours robot changes remain LOCAL/PENDING owner rollout; no production/Git mutation. Prior20261102000000 application reported successful by owner, not freshly queried here. Report: modules/watching/network-animation-studios-2026-11-03.md.


## Viewing dates / tracking intent - 2026-11-04

Owner-approved business+robots block implemented locally. Replaces invalid frontier-date sort: Silo1st, Hill House14th in stored read-only snapshot replay. Corrections do not invent dates/resume; prospective growth waits for explicit Resume, preserving consumed history/hours and People membership. 749Watching tests/43files, disposable PostgreSQL, TypeScript/lint/Knip/schema/DS/diff and4actual-component browser formats PASS. New migration20261104000000 adds1availability column, functions replaced without backfill; watchdog reference868columns. Migration20261104000000 application reported successful by owner on2026-11-05; no independent remote recheck in the following UI task. Series-worker diagnostic rename deployment not reported. Report: modules/watching/tracking-intent-2026-11-04.md.

UI follow-up implemented locally2026-11-05: one stable status shell with measured height, section-scoped visible-image decode/reveal, no per-card stagger, cached/refetched content retained, matching rail/header skeleton geometry and media-specific route loading.171targeted tests plus4format real-component browser cases, TypeScript/lint/Knip/schema/DS/diff PASS. Owner localhost visual check pending; exact live-network flash absence not certified. Report: modules/watching/section-transitions-2026-11-05.md. Updates implementation now recorded below (2026-11-05). Earlier studios migration/worker rollout was reported completed by owner.


## Completed recency correction — 2026-11-05

Owner-reported prior UI/business commit pushed and visually accepted. New targeted fix LOCAL: completed claims now rank/label by their completion projection, excluding legacy activity timestamps. Read-only real-data replay moves The Promised Neverland from2nd to14th (2022-05-23); Silo ongoing recency preserved.152tests/4files PASS; TS/lint/diff verified. No production mutation or migration. Report: modules/watching/completed-recency-2026-11-05.md. Updates now implemented locally; see its current milestone below.


## Watching Updates — 2026-11-05

Latest Updates polish implemented locally: Coming next/TBA share four-desktop-card horizontal rails,112px posters with verified2x/3x source selection, Don't Miss-style poster ambience and matching skeletons. Redundant month heading and paused/dropped control removed; paused announcements stay visible/dropped hidden. TBA orders started stories before Want to watch. Available carousel image behavior unchanged.54targeted tests and four-format component/DPR image checks pass; authenticated read-only localhost captures inspected; TS/lint/Knip/DS/diff verified. No migration/data/edge/Git/deploy action. Report: modules/watching/updates-page-2026-11-05.md (latest carousel/ambience section). Next: owner visual review. debug.log excluded.


## Lists / Awards loading alignment - 2026-11-05

Owner-approved focused implementation complete locally: dedicated route/client skeletons, explicit read errors/retry with cached data retained, list child-read failures propagated, Awards categories independent from full shelf loading.28targeted tests, TS, affected lint, Knip, DS and production build pass; four-format read-only browser checks and failure/retry scenarios exercised. No production/Git/deploy action. Report: modules/watching/loading-architecture-audit-2026-11-05.md (implementation section). Next: owner visual validation; workspace-aware cache context remains separate.

Skeleton harmony follow-up2026-11-06: standalone shared Pulse, Person Bar removed, placeholder colors/radius tokens aligned, nested animations removed.28isolated browser cases verify common color/no nested pulse/reduced motion; TS/lint/Knip/DS pass. Existing page layouts retained. Owner visual review pending.

MediaCarousel navigation2026-11-07: replaced click-only index/viewport calculations with existing useAlignedRail (same as Updates). Native scroll now controls arrows; clicks use actual card geometry and pending target.6rail tests + isolated browser native-scroll/Prev checks pass; TS verified. Live page check unavailable because owner dev server is off. Season-rating eligibility correction remains pending.


## Watching stats / navigation - 2026-11-08

Local implementation ready for owner review: annual ratings and Wrapped exclude rewatch-only titles; Wrapped keeps stable third-place ties with circular poster navigation. Hours rows show applicable ratings and hide film scope jargon. For You/Cast/People share scroll-aware navigation. Activity/distribution baselines and label rows aligned with token-based zero-activity color.74tests, TS/lint/Knip/DS and isolated browser checks pass. No production/data/Git mutation. Report: modules/watching/stats-navigation-2026-11-08.md. Next separate correction: fully aired + fully watched eligibility for season/part ratings (still pending).


## Current - Watching integrity (2026-11-17)

Owner reports Anime Themes block already committed; starting tree clean atd79b0be. Local corrections prepared: deleted/orphan history no longer creates active titles/hours or revives on re-add; complete-airing+viewing rating gates with stable saving controls; Watching-only Back scroll restoration and existing identity-loading skeleton. Read-only replay3,870h26 ->3,807h24; removes63h02 across Dark Matter/Godfather of Harlem/deleted The Night Of. Dragon Ball Z replacement explains52h47 and is explicitly confirmed correct by owner.917tests/61files PASS, four-format rating-control stability and real mobile Back1097->0->1097 verified. New20261117000000 migration LOCAL/PENDING owner application, no backfill. Disposable PostgreSQL complete-scope/RLS/retry/no-backfill regression PASS; sole pending migration confirmed remotely. Lint/TS/Knip/DS/schema and final production build PASS. Four-format real Back checks and live Black Clover panel verified. Report: modules/watching/integrity-rating-scroll-2026-11-17.md. No production/Git/deploy action.
