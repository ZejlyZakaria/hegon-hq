# Watching migration alignment - 2026-10-29

Current rollout2026-10-29: generic migration20261029000000 and separately approved three-anime data repair20261029010000 APPLIED and verified;33/33versions aligned, no pending. WAVE 2E LOCKED / COMPLETED by owner approval2026-10-29. See wave2e-final-reconciliation-2026-10-29.md. Earlier no-application/three-anime-untouched verdicts below are historical.

## Result

Repository migration history is aligned with the linked HEGON remote registry:31local versions equal31remote versions. Restored20261028010000_watching_bleach_part4_viewing_year.sql is the original archived migration, not a new/empty replacement. No remote mutation, migration repair, reapplication, rollback, user-data correction, robot invocation, deploy or Git mutation performed. Three anime cursor/year cases remain untouched.

## Exact identity proof

A fresh SELECT through the dedicated read-only endpoint returned the registry version, name and three stored SQL statements. Version20261028010000 and name watching_bleach_part4_viewing_year match the archive. BEGIN, DO repair body and COMMIT all match after whitespace/trailing-separator normalization. The restored file is byte-identical to scripts/.shots/deferred-bleach-part4-viewing-year.sql, SHA256 b9500912d4add71740a3b2cb912675692fc7d80b392ab4b372beddfde8816564. The Management API registry stores parsed statements rather than original file bytes, so byte identity is asserted against the archive, statement identity against remote.

The original SQL guards, approved operation ID, provider snapshot, frozen IDs,192minutes and expected revision8 are unchanged. This migration was already applied remotely before restoration; earlier documentation claiming it was never applied was incorrect. Fresh evidence from the prior audit confirms the resulting stored revision9. No inference is made about who applied it or exactly when.

Watchdog watermark advances to20261028010000; counts stay81tables/857columns/225policies/49functions/277indexes/20crons. Local schema-reference checker and diff whitespace checks pass.

## Rehearsal results and honest replay limit

1. The exact restored SQL PASS on a disposable PostgreSQL16 database with synthetic prerequisites matching its historical guards. Checks cover unchanged unrelated fields/history/ratings/rewatches/receipts/schema; exact new E41-48 membership and192minutes; revision8to9; stale/malformed rollback; and idempotent repeat. Existing local history, rewatch, receipt, accepted-consumption and concurrency checks also pass. This is a targeted fixture rehearsal, NOT a complete empty-database replay.
2. Raw ordered replay of the31unmodified migration files was attempted in a newly created empty PostgreSQL16 database on127.0.0.1:55433. It fails immediately at20260906000000_baseline.sql because native Windows PostgreSQL lacks pg_cron. Docker daemon is not available in this session; no full Supabase local environment was started and no extension was silently stubbed or removed.
3. Independent local SQL probes on the supported Watching test schema, without the original personal repair targets, prove further empty-data blockers:20261021010000 aborts with Stale approved mapping:0/85;20261028000000 aborts because the approved Bleach row is absent;20261028010000 aborts because the accepted provider catalogue is absent. All probes roll back and preserve existing fixture rows. Therefore even a Supabase-capable local engine would not make a literal empty-data replay pass without addressing the historical data-repair prerequisite policy.

No historical migration was weakened or edited to hide these failures. Exact restoration is complete; full fresh-database replay is NOT validated and currently NOT self-contained. Owner decision2026-10-29: this remains an unvalidated technical capability, but is explicitly a SEPARATE TECHNICAL-DEBT WORKSTREAM and does NOT block Wave2E completion. A separately reviewed bootstrap/baseline or historical data-repair execution strategy is needed; do not invent placeholder personal rows or silently skip migrations and call that a successful full replay.

All disposable databases were removed; the local test server was stopped. No production credentials are used by local test harnesses.

## Evidence and next step

Ignored local evidence: scripts/.shots/wave2e-migration-alignment-remote.json; wave2e-migration-restoration-results.json; wave2e-full-replay-results.json; verify-restored-part4.mjs; deferred-test-watching-bleach-part4.mjs. Diagnostics are not commit inputs. No new product source or schema objects introduced.

Next: resume Wave2E read-only clarification of the three anime. Full empty-database reconstruction is deferred; reopen only when the owner explicitly schedules a dedicated bootstrap/reconstruction task. Applied migrations must not be rewritten for this purpose. Remaining2E personal-data findings stay at audit-only; no authorization to correct Bleach or the three anime follows from this repository restoration. Do not run db push to reapply this already-recorded version or run migration repair.
