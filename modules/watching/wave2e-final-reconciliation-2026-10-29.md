# Wave2E controlled rollout and reconciliation - 2026-10-29

Status: WAVE 2E LOCKED / COMPLETED. Owner explicitly approved closure on2026-10-29. Owner explicitly authorized agent-run remote application and the previously approved three-anime corrections in this task, overriding the usual owner-run migration convention. No Git staging/commit/push or application/worker deployment.

## Remote migration and unchanged-data boundary

Linked HEGON project femvhonlpafdajyamvcu confirmed. Fresh registry:31remote/32local, only20261029000000_watching_backward_correction.sql pending, no remote-only versions. Official CLI2.121.0 dry-run confirms that single migration with no seeds/roles. Applied using normal db push --linked --yes, no manual SQL or migration repair.

Immediately after the generic migration:32/32versions aligned, exactly one registry entry, both remote function bodies match the final R1-corrected local SQL. Seven personal-table fingerprints and full three-anime rows are identical before/after this migration. It performs no backfill or personal-data change.

## Separately authorized controlled data correction

Added and applied20261029010000_watching_reconcile_three_anime.sql, a data-only migration, after a second single-pending dry-run. This keeps historical cleanup separate from prospective generic behavior and versions every database change. It creates no schema objects.

The transaction checks exact owner/org/media identity, revision, true cursor/posture, all relevant dates/maps and frozen consumption. It refuses unexpected independent viewing/opinion/rewatch evidence. Provider-before-media locks and exact result checks protect the batch; any stale target rolls back all three. It does not change any status or cursor or fabricate a temporary backwards/forwards transition. Original full rows and resulting full rows are saved in three progress_corrections entries with fixed operation identities/hashes. Repeat application is a no-op. No fake viewing or opinion assertions are created.

| Title | Before -> after | Legitimate data preserved |
|---|---|---|
| Tokyo Ghoul | Remove S3year2019 and S3score7.5; false watched_at2019-12-31T12:00:00 -> NULL; explicitly retire the corresponding legacy calendar with calendar_date=NULL | S2E12/Dropped; S1/S2years2017, scores8.5/8.5; global8.5; frozen counts[12,12,0,0]/24min;9.6h in2017; S1E12highlight |
| The Seven Deadly Sins | Remove S3year2024 and S4year2021 | S2E24/Dropped; S1year2017/S2year2019; global7.5; empty season scores; frozen counts[24,24,0,0]/24min;19.2h (9.6h per valid year) |
| Fire Force | Remove S2year2025 | S1E18/Paused; S1year2024/score8.5; no global score/date; frozen counts[18,0,0]/24min;7.2h in2024 |

Personal revisions advance0to1; history_version stays0 because no canonical history was fabricated. No new dates, ratings or viewing coverage. Bleach and every other media row remain exactly unchanged in the full before/after snapshots.

## Verification

- Real loopback-only PostgreSQL16 rehearsal of the actual data migration on the captured three-row state: exact before/after, unchanged unrelated fields and schema, complete audit, stale last-target rollback of the entire batch, idempotent repeat PASS. Existing history/rewatch/receipt/concurrency, generic backward correction and exact R1 tests also PASS. Disposable databases removed and local server stopped.
- Final fresh remote read:33versions equal33local, each new migration recorded once, no pending/remote-only version. Remote generic function bodies match local R1 candidate; audit table exists and contains exactly the three expected full before/after pairs.
- All518owner media rows retained;516non-reference rows in the final full snapshot. Only the three specified full rows changed. Canonical records/ranges, scoped opinions, rewatches, receipts and highlights fingerprints unchanged. No inferred/unknown data was promoted to confirmed.
- Actual current computeStats replay on fresh remote snapshots for all-time and every available year: hours, counts, episodes, rewatches, unknown-duration/scope counts, average/distribution and Top Picks score/scope unchanged. Total3730.7h ->3730.7h. Per-title annual/all-time hours match the table above. Only intended monthly delta:Activity2019-12 events3to2, removing Tokyo's false calendar contribution. No valid activity is reassigned to an invented month/day.
- Watchdog reference matches remote82tables/867columns/226policies/50functions/279indexes/20crons; watermark20261029010000. Schema-reference and main/HQ whitespace checks PASS. Final CLI dry-run reports up to date.
- No UI changes in this task; verification is remote SELECTs plus actual statistics-code replay, not a new production browser mutation test. No broad suite or performance claim.

## Files and evidence

New tracked-intent SQL: supabase/migrations/20261029010000_watching_reconcile_three_anime.sql. Updated supabase/checks/expected-objects.json; HQ current-status pointers/now/backlog and this report. The prior generic SQL/application/tests and restored applied20261028010000 remain part of the earlier uncommitted Wave2E work.

Ignored personal evidence, do not stage: scripts/.shots/wave2e-rollout-before.json, wave2e-rollout-after-schema.json, wave2e-rollout-final.json, wave2e-pre-repair-snapshot.json, wave2e-post-repair-snapshot.json, wave2e-repair-specs.json, wave2e-final-verification.json, wave2e-repair-rehearsal.log and diagnostic scripts.

## Remaining disposition

No Wave2E blocker remains. UNKNOWN scopes/dates/durations remain unknown, INFERRED historical date estimates retain provenance, and Bleach personal reconciliation stays excluded. Full empty-database reconstruction remains separately deferred technical debt, expressly nonblocking; this new guarded data repair also requires its historical targets and is not a claim of empty-data replayability. Earlier no-application/no-three-anime-correction and READY FOR RE-REVIEW statements in historical reports are superseded by this rollout. Next: owner commits/pushes this locked block when ready; no Python/refactor/new workstream started.
