# Wave2E - generic backward correction, 2026-10-29

Current rollout2026-10-29: generic migration20261029000000 and separately approved three-anime data repair20261029010000 APPLIED and verified;33/33versions aligned, no pending. WAVE 2E LOCKED / COMPLETED by owner approval2026-10-29. See wave2e-final-reconciliation-2026-10-29.md. Earlier no-application/three-anime-untouched verdicts below are historical.

Current R1 follow-up2026-10-29: stable-identity collision fixed locally, READY FOR RE-REVIEW; see wave2e-r1-fix-2026-10-29.md. No remote application.

Historical pre-application review update2026-10-29: NOT LOCKABLE / NOT SAFE untilR1 (same-score/ordinal, different stable identities) is fixed. See wave2e-backward-review-2026-10-29.md. No remote application. The validation results below remain historical positive coverage and do not coverR1.

Status: generic fix implemented and locally validated; new migration20261029000000 is LOCAL ONLY. No correction of the three personal production rows, no other production mutation, migration application/repair, deployment or Git mutation. Existing20261028010000 restoration remains in the working tree from the previous task, not a new data correction. Empty-database reconstruction stays separately deferred and nonblocking for2E.

## Owner-authorized contract

Current position is an explicit first-view correction boundary. Retire only assertions whose identified coverage is contradicted by a backwards correction; preserve legitimate remaining coverage, independently scoped opinions, rewatches and unknown identities. Dropped/Paused never select the cleanup rule. Known seasons/cours beyond the true boundary lose invalid active opinion/year; partially retained first viewing keeps only actual episodes with original date precision and frozen duration. A partial legacy opinion without evidence of its full membership is not blindly erased. Frozen membership takes priority over ordinals. Unknown or incompatible cour identity remains unknown instead of being deleted.

## One transaction and one authority

service.ts passes existing history_intent metadata to personal_history_command, not into the media patch. The existing SECURITY DEFINER command owns all series/anime backwards invalidation under operation/provider/media locks and expected revision. Removed the previous TypeScript backwards clipping loop rather than keeping a parallel implementation; captureHistory still captures prospective activity, explicit date edits and the independent film-removal behavior.

The private SQL progress_scope_relation resolves KEPT/PARTIAL/REMOVED/UNKNOWN using frozen storage membership first. Legacy seasons use known storage identities; cours use stable AniList IDs/current contiguous mapping and frozen storage sizes. A stable identity absent from the accepted mapping never falls back to its old ordinal. Earlier frozen subjects remain valid even if their recorded ordinal is now higher than the cursor's display ordinal.

Within that same command:
- Retract wholly removed non-rewatch records and contradicted completions; clip/supersede partially retained FIRST_WATCH ranges, preserving original precision and proportional frozen duration.
- Resolve a coarse completion by its known season/cour scope even without ranges. The latest unbounded FINISHED_TITLE assertion can be contradicted by a rollback of the currently claimed full-title completion; earlier genuinely bounded completions remain retained. Unidentified assertions are not blanket-deleted.
- Remove false legacy year/rating slots. Append NULL CORRECTION successors for invalid scoped opinion lineages; do not mutate/delete original ratings. Retire invalid accepted legacy count suffixes so future progress cannot resurrect old false consumption. Runtime estimates and valid prefix membership remain untouched.
- Retire a global/legacy date only with exclusive removed-scope/date evidence and no surviving supporting first-view/rewatch/calendar fact. Do not invent another date or globally suppress legitimate old calendar facts. Reserved-field validation still prevents the client from directly patching legacy_consumption; only this audited command adjusts invalid membership/calendar.
- Append a private immutable progress_corrections record containing exact before/after projections, operation hash and reason. This is an audit journal only, never a source for Stats/status. Authenticated owner/org SELECT only; no browser/service direct writes. Existing operation/revision protections remain, including retries after later actions.

useUpdateMedia installs the authoritative returned media row and invalidates history/scoped opinions, Last Watched and rating rails for cursor corrections. A server-retired date/score absent from the input patch therefore cannot remain cached on those surfaces.

## New migration and schema reference

20261029000000_watching_backward_correction.sql has no personal-row backfill and contains no real owner/organization/media ID. It creates only one internal audit table with10columns, one owner read policy, two indexes and one private helper; the existing command function is replaced. Measured local object delta:+1table,+10columns,+1policy,+1function,+2indexes,+0crons. Prepared watchdog target:82/867/226/50/279/20; watermark20261029000000. These describe the PENDING local migration, not current remote counts. Old applied SQL files remain unchanged (apart from the previously restored exact archived file).

Owner must apply this migration before running/deploying the updated correction path. Do not deploy the client-side removal of the old clipping loop against an unmigrated database. No remote push/deploy was performed. Application of this generic DDL/function migration does not repair the three old residual rows: their cursors already equal the correct positions, so no new backwards transition exists. A separately approved controlled data correction will be required.

## Exact proposed personal before/after - NOT APPLIED

| Title | Kept true position/status | Before active residuals | Proposed retirement | Retained data |
|---|---|---|---|---|
| Tokyo Ghoul | S2E12, Dropped | season_years[3]=2019; season_ratings[3]=7.5; watched_at2019-12-31T12:00:00 supplies calendar2019-12 | Remove S3year/rating; clear the proven-false global date and its active legacy calendar source, retaining original values in audit; no invented2017month/day | S1/S2years2017 andscores8.5/8.5; global8.5; counts[12,12,0,0];9.6h; S1E12highlight |
| The Seven Deadly Sins | S2E24, Dropped | season_years[3]=2024 and[4]=2021; no season scores | Remove only S3/S4years with audit | S1=2017,S2=2019; global7.5; counts[24,24,0,0];19.2h; null global date |
| Fire Force | S1E18, Paused | season_years[2]=2025; no S2score | Remove only S2year with audit | S1=2024, S1score8.5; counts[18,0,0];7.2h; null global date |

These three titles have no canonical viewing/completion/rating assertions or rewatches to retract in the read-only evidence. All already have watched=false. No fake history/retraction needs to be fabricated. Tokyo's invalid2019-12calendar event disappears; annual/all-time hours stay unchanged. None of these proposed changes was executed, including locally against the real production rows. Bleach personal reconciliation remains excluded.

## Exact implementation manifest

- supabase/migrations/20261029000000_watching_backward_correction.sql (new)
- supabase/checks/expected-objects.json
- src/modules/watching/service.ts
- src/modules/watching/lib/viewing-history.ts
- src/modules/watching/lib/season-years.ts (outdated cleanup comment corrected)
- src/modules/watching/hooks/useUpdateMedia.ts
- src/modules/watching/service.test.ts
- src/modules/watching/lib/viewing-history.test.ts
- src/modules/watching/lib/history-actions.test.ts
- src/modules/watching/lib/scope-rating-ui.test.ts
- scripts/test-watching-backward-correction.mjs (new real PostgreSQL regressions)
- scripts/test-watching-milestone-receipts.mjs (explicit --backward-correction option)

HQ: this report, Watching architecture/decisions/backlog and now.md. No component/style change or generated coverage update is necessary. debug.log is unrelated; ignored personal diagnostic files must not be staged.

## Validation

-245tests/8targetedfiles PASS (watch-status, viewing-history, history-actions, service, computeStats, history-consumption, provider-ratings, accepted-consumption);6scope-rating UI tests PASS, including actual React receipt rendering and cleared-score refusal after advancing.251tests total across9files.
-Real disposable PostgreSQL16 rehearsal PASS via WATCHING_TEST_PG_PORT=55433 node scripts/test-watching-milestone-receipts.mjs --rewatch --backward-correction. It executes the restored command through authenticated role, not a mocked database. Cases: false season/year/score rollback; flat anime cours; no-range scope completion and unbounded full-title completion; absent identity; remapped ordinal with retained frozen membership; clipped real episodes/precision; retained earlier completion/independent rewatch; same behavior with Paused/Dropped; exclusive calendar invalidation; unchanged migration-time rows; retry/idempotency and stale revision; owner/org RLS and rejected direct journal writes. Prior receipt/Goal concurrency, immutable receipts, history and rewatch checks run in the same harness. This is a supported Watching schema rehearsal, not full empty-Supabase replay.
-TypeScript noEmit, zero-warning affected ESLint, Knip, schema-reference, CRLF and Git diff checks PASS. Existing3Knip config hints unchanged. No broad test suite or production-backed browser mutation. No UI layout changed; actual React rendering is checked locally in tests.
-Only synthetic fixture data was mutated on127.0.0.1; disposable databases removed. Local test server stopped at completion. No performance improvement claimed.

Next: owner reviews the generic fix and proposed three-row mapping. New migration remains pending owner application; do not repair any personal production row until separately authorized.
