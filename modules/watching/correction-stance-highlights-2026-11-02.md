# Correction stance and Person highlights - 2026-11-02

Owner-approved business rule: correcting a position does not resume or complete Paused/Dropped. Keep the same edit access beside the displayed position for In Progress, Paused and Dropped. Resume/Finish remain explicit actions. No dates are invented by a progress correction.

Implementation:
- watch-status.ts removes the implicit paused-frontier resume, prevents suspended completion, and dates landed seasons only for actual viewings. preserveCorrectionStance is shared by the pure patch builder and the freshly-read service boundary; retains stance/reason and omits fabricated finish/last-view timestamps.
- Service applies this before captureHistory, preventing false completion assertions from stale UI patches. Backwards scope cleanup remains owned by the existing atomic command, including cour identities, frozen membership, false opinions/calendar cleanup and independent rewatches.
- New20261102000000 replaces the existing personal_history_command only, adding a post-lock/pre-write guard. Direct/stale RPC clients attempting stance/date changes during a suspended position correction are rejected atomically. No backfill/new object/grant/RLS/idempotence change. Expected objects watermark updated; counts unchanged. LOCAL/PENDING; not applied remotely.
- StatusCard now shows one position/pencil row in the same location for all three states; removes duplicate menu/underlined edit entries and redundant stopped-position rows. Editor explains the retained stance.
- PersonInsights is replaced by Your highlights: existing80px poster-sm full-width rows, typographic rank, teal personal score and distinct TMDB comparison. Deduplicates the same best/gem title without discarding either designation or metadata. Removes old half-width cells and unnecessary firstName prop.
- CaughtUpBadge remains useful for weekly shows and now owns the same compact skin wherever rendered. Position badge uses matching20px height,8px horizontal padding and pill radius, with neutral metadata colour. No duplicated caught-up glyph/style implementation remains in StatusBadge.

Validation:
- Full Vitest:50files/807tests PASS;253targeted business/history/UI tests also pass. New regression coverage includes both types/stances, forward/backward/end corrections, service normalization, explicit resume and deduplicated larger highlights.
- Disposable PostgreSQL16 on127.0.0.1 only: original receipts/provider/backwards/cour/R1/single-scope regressions pass, followed by new4type/stance combinations. New migration leaves existing data hash/object counts/ACL/security context unchanged; rejects stale/null/implicit completion/resume/date commands with unchanged state, preserves audit/idempotence and real season data, retracts invalid legacy maps, allows explicit resume.
-8browser harmony cases and8correction UI cases (3states each),390/768/1024/1440, touch and mouse. Assert80px posters, identical chip geometry, unique same-place edit access, corrections without resume callbacks, functional actions and no overflow/runtime errors. Screenshots inspected.
- TypeScript, affected lint, Knip CI, schema reference and whitespace checks pass. DS coverage regenerated442existing off-system hits.

No remote read/write, production data cleanup, migration application, deployment or Git mutation. Local test server started solely for rehearsal and stopped afterward. Owner visual acceptance and controlled owner migration application remain next steps; all earlier locked waves remain locked.
