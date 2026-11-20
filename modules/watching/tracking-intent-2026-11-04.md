# Viewing dates and explicit resumption - 2026-11-04

## Scope and status

Owner-approved business/robots block only. Local implementation; new migration NOT applied remotely. No production read/write in this implementation; earlier owner-authorized read-only Silo snapshot replayed locally. No Git staging/commit/push/deployment. Studio disclosure from the preceding task remains in the working tree. Details transition/skeleton/image synchronization work and Updates page are separate next blocks.

This supersedes the earlier studios-silo-recency report's invalid max(caught_up_at) sort. caught_up_at remains a historical frontier marker, including completed shows; it is never consumption-date evidence.

## Final rules

- Position corrections can clear invalid frontier evidence but cannot create a new frontier timestamp, move last_watched_at forward, assign today's year or implicitly resume. Completion can be acknowledged without manufacturing its date. Existing 2E scope-aware backward retractions and immutable audit remain.
- Additional corrected episodes inherit only an unambiguous existing scope year, with YEAR/INFERRED precision. No new month/day. Conflicting scope years remain UNKNOWN; unidentified anime cours cannot borrow a global/flat-season date. Existing assertions remain.
- Last Watched sorts and labels by one latestViewingAt helper: newest valid watched_at/last_watched_at, never caught_up_at or updated_at. UTC normalization avoids browser/server ordering disagreements. Real owner snapshot replay: Silo1st, Hill House14th, outside top10.
- Unknown/nonpositive announced totals cannot prove season completion. A delayed episode is not an invented cour/part.
- A new season announcement does not resume. Proven aired growth after a completed title, season or identified AniList part sets resume_required. Weekly growth inside an active unfinished scope keeps tracking active. Paused/Dropped/Want to watch/reference decisions remain.
- Growth can revoke the old whole-title watched flag when its scope grows; previous dates, ratings, position, frozen coverage, first-watch/rewatch history remain. No automatic in_progress flip. Last Watched keeps the consumed title with a new-content notice until explicit Resume or a new forward viewing. The same classification feeds Stats, with an awaiting-resume category instead of false completed/active counts.
- Resume changes intention only, never a watch date. Actual forward consumption resumes and records its own date; correction does not. A correction that now reaches all available content makes an old hold non-actionable without claiming a new viewing.
- Existing consumed-library/People/Quick Stats/goal-summary membership survives the hold, including SQL people_ranking. One hasViewingEngagement predicate serves the client summaries.

## Database and robot boundary

20261104000000_watching_tracking_intent.sql adds only resume_required boolean DEFAULT false NOT NULL. Five existing functions replaced: observe_provider, personal_history_command, receipt_source, personal_revision_guard, people_ranking. No backfill or personal data repairs. Expected objects:82tables/868columns/226policies/51functions/279indexes/20crons; only+1column. ACL/security contexts/RLS/idempotent operation receipts retained.

The provider transaction owns membership and growth. Application updates strip unaccepted count/status refreshes for existing rows; direct authenticated updates and stale personal RPC calls are rejected at DB boundaries. Initial inserts retain their existing catalogue contract. Corrections are defended inside the atomic command, including missing-intent resume forgery. Transactions normalize genuine forward viewing from old clients into explicit resumption.

Both watching-series-sync and watching-anime-cours-sync were inspected: their accepted evidence flows through observe_provider, not local status patches. Anime mapping updates remain descriptive; no invented parts/dates. Series robot response now calls its counter growth_detected rather than repaired; failures/retries/status codes unchanged. Applying the migration changes the common transactional behavior; owner may deploy the updated series robot afterward for accurate diagnostics. No anime worker change/deploy is needed for this block.

## Validation

Rehearsal script: node scripts/test-watching-milestone-receipts.mjs --correction-stance --animation-studios --tracking-intent. Uses disposable PostgreSQL16 on127.0.0.1:55432, never production. Includes earlier backward/cour identity regressions, ACL/count/no-backfill checks, announcements, series/anime revivals, confirmed cour vs weekly delays, suspended states, explicit resume and repeat-sync idempotence, date-forgery rejection, direct/raw catalogue-write rejection, People preservation and unchanged history/rewatch/rating hashes.

Final Watching suite:749tests/43files PASS. TypeScript and affected-file ESLint zero warnings PASS. Knip CI, schema reference, DS coverage and git diff --check PASS; DS stays442pre-existing off-system hits and generated coverage has no tracked diff. Final PostgreSQL rehearsal PASS (including missing-intent rejection, old-client actual-forward-resume normalization and People ranking preservation). Full production build/live authenticated end-to-end mutation not run; remote rollout remains pending. Actual StatusCard isolated browser fixtures pass390/768/1024/1440: new availability does not display In Progress/Caught Up, explicit Resume is wired, no overflow/runtime error; screenshot inspected. Not a live production mutation or a guarantee about the still-pending transition-flash block. Reports/fixtures under ignored scripts/.shots.

## Changed areas

Service/schema/types; watch-status/series-state/date/history/legacy and engagement helpers; invalidation/action hooks; shared carousel/menu/status labels; detail/People/Stats consumers; Watching detail page engagement gate. New migration and watchdog reference; disposable rehearsal runner/new tracking checks; regression tests. No frozen module modified.

## Remaining rollout / next block

The migration must be owner-applied before using the new column-selecting code against linked Supabase. Existing positions/dates have not been rewritten or historical auto-resumes guessed/backfilled. Review this local block, then controlled migration/series-worker rollout as separately authorized. Next: Details flash, exact hub skeleton geometry and grouped visible-image reveal. Updates remains unimplemented.


## Exact working-tree manifest at completion

- scripts/test-watching-milestone-receipts.mjs
- src/app/(main)/perso/watching/[id]/page.tsx
- src/modules/watching/components/detail/ContributingToGoals.tsx
- src/modules/watching/components/detail/QuickStats.tsx
- src/modules/watching/components/detail/StatusCard.tsx
- src/modules/watching/components/person/PersonTimelinePanel.tsx
- src/modules/watching/components/shared/MediaActionMenu.tsx
- src/modules/watching/components/shared/MediaCarousel.tsx
- src/modules/watching/components/shared/NextEpisodeButton.tsx
- src/modules/watching/components/shared/StatusBadge.tsx
- src/modules/watching/components/stats/StatsPage.tsx
- src/modules/watching/components/stats/computeStats.test.ts
- src/modules/watching/components/stats/computeStats.ts
- src/modules/watching/hooks/useUpdateMedia.ts
- src/modules/watching/hooks/useWatchActions.tsx
- src/modules/watching/lib/activity-bucket.test.ts
- src/modules/watching/lib/completion-date.ts
- src/modules/watching/lib/history-analytics.ts
- src/modules/watching/lib/legacy-consumption.ts
- src/modules/watching/lib/media-view.ts
- src/modules/watching/lib/person-experience.ts
- src/modules/watching/lib/progress-correction-stance.test.ts
- src/modules/watching/lib/series-state.test.ts
- src/modules/watching/lib/series-state.ts
- src/modules/watching/lib/viewing-history.test.ts
- src/modules/watching/lib/viewing-history.ts
- src/modules/watching/lib/watch-status.test.ts
- src/modules/watching/lib/watch-status.ts
- src/modules/watching/schemas/media.schema.ts
- src/modules/watching/service.test.ts
- src/modules/watching/service.ts
- src/modules/watching/types.ts
- supabase/checks/expected-objects.json
- supabase/functions/watching-series-sync/index.ts
- scripts/test-watching-tracking-intent.mjs (new)
- supabase/migrations/20261104000000_watching_tracking_intent.sql (new)

Pre-existing approved studio-disclosure change: src/modules/watching/components/detail/MediaDetails.tsx. Untracked debug.log is unrelated and excluded. HQ itself is a separate repository.

## Rollout update — 2026-11-05

Owner reports migration20261104000000 applied successfully. This is an owner report, not a fresh remote registry verification. The UI follow-up is recorded in section-transitions-2026-11-05.md.
