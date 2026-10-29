# Targeted rewatch hardening — 2026-10-21

Owner approved the full-show default, season-level UI (multiple seasons allowed), no episode-picker burden, and a scope frozen at the selected viewing period. The Black Clover historical poster example illustrates the required stability but its poster logic is explicitly outside this task. No general Wave 2E reconciliation or final Wave 2D review is started.

## Product and data contract

- A rewatch is one independent viewing event. It never changes first-watch dates, progress, posture or ratings. Whole-title is the default; multiple displayed anime/TV seasons are one event.
- Available content is evaluated from accepted episode air dates up to the supplied day, or the last possible day of a MONTH/YEAR period capped at today. That bound is only an availability calculation: the persisted watch date retains its actual precision. UNKNOWN date freezes currently available content without inventing a viewing date.
- Historical full-title no longer selects today's episode counts and then rejects later seasons. Known future episodes are excluded, ongoing seasons are clipped to dated membership and noncontiguous eligible episodes are grouped without filling holes. Unknown historical availability fails explicitly instead of reconstructing a past from today's count. Missing dates in an unrelated future season do not block an explicit older-season selection.
- Anime choices use the existing MediaView display seasons/cours. Selection translates through stable cour identity and announced storage offsets with validated membership, including partially aired/open-ended cours. Actual episode IDs, storage ranges and durations are frozen; later seasons, runtimes or cour remapping cannot grow an existing event.
- Movie runtime freezes directly. TV runtime comes from accepted per-episode provider facts, never the representative show runtime or whichever season happens to be cached in the browser. Known subtotals survive missing episode durations. No fabricated hours.
- An explicit partial/unsure event is valid with UNKNOWN scope and duration. It counts once but adds no invented episodes/hours.
- Canonical correction retracts and supersedes through the existing command. Legacy entries expose Complete; owner confirmation atomically creates one canonical event and replaces its compatibility legacy row using the existing remove_rewatch command with records. A failed or repeated operation cannot lose/duplicate an event. No automatic conversion of the nine personal legacy rewatches to full coverage.

## UI

RewatchForm defaults to Entire show, offers multi-select season cards with available episode counts and disables unavailable seasons. The existing WatchDatePicker supplies YEAR/MONTH/DAY precision with an explicit unknown-date choice. A live summary previews membership/hours before the separate save action; pending controls are disabled and errors stay visible. Existing canonical selections can be corrected without exposing an episode grid. Detail rows show stored dates, episode counts and frozen durations; legacy rows offer Complete.

The unknown-duration notice now stays with Hours Watched instead of occupying its own chart-grid column. The message describes incomplete viewing-event durations, including known subtotals; it does not imply every unknown event is necessarily a rewatch. No poster or rating-qualification change.

## Manifest

- src/modules/watching/lib/rewatch.ts and new rewatch.test.ts
- src/modules/watching/components/detail/RewatchForm.tsx, StatusCard.tsx
- src/modules/watching/components/stats/StatsPage.tsx (duration notice grouping/copy only in this task)
- src/modules/watching/hooks/useRewatches.ts (remove transient episode-cache duration dependency)
- src/modules/watching/service.ts, types.ts, service.test.ts, lib/review-ui.test.ts, lib/provider-ratings.test.ts
- supabase/functions/_shared/provider-evidence.ts (optional bounded episode runtime retained by watching-series-sync)
- supabase/migrations/20261021020000_watching_rewatch_episode_runtime.sql
- supabase/tests/watching-rewatch.sql; scripts/test-watching-milestone-receipts.mjs optional --rewatch rehearsal
- supabase/checks/expected-objects.json (watermark only; object counts unchanged)
- src/modules/styleguide/coverage.generated.json

Existing Wave 2D, approved estimated-date migration 20261021010000, Goals and unrelated worktree changes were preserved. Several files remain mixed with those earlier tasks; this is not a staging manifest for the whole working tree.

## Rollout boundary

The runtime field is optional and backward compatible, but the current remote provider-facts validator rejects unrecognized episode keys. Migration 20261021020000 replaces that validator to allow numeric runtimes in (0,1440] or null, keeping all old facts valid. No tables, columns, functions, policies, indexes or crons are added; no existing data is rewritten. Local PostgreSQL rehearsal confirms unchanged media data and measured object counts.

Owner must apply the migration before deploying watching-series-sync (it imports the changed shared evidence helper). The robot's subsequent normal observations populate episode runtimes; existing snapshots with no runtime remain honest UNKNOWN. Existing immutable rewatches do not retroactively gain hours: explicit correction is required. Dates/membership unavailable from current accepted evidence remain blocked or explicitly unsure, never guessed. Application and edge deployment have not been performed here; no production writes or observing receipt/Goals action were invoked.

## Verification

214 targeted tests across eight files pass: new period/cour/partial-scope/frozen-Stats regressions plus history consumption, provider contracts/actual robot handlers, service, UI, annual Stats and receipts. Local PostgreSQL history/consumption/receipt RLS, stale revisions, retries, concurrency and rewatch replacement tests pass; failed replacement rolls back, retry stays single and a later movie-runtime change leaves frozen minutes unchanged. Optional runtime validation accepts old facts and bounded numbers/null, rejects invalid values.

Actual React form, real design-system date picker and real pure logic were bundled into an isolated browser fixture. Service/query data and save callbacks are local mocks; no authenticated production flow is exercised. Chromium interactions pass at 390/768/1024/1440, including future-season exclusion, selection and YEAR precision. Mobile/desktop screenshots read; no horizontal overflow or browser errors. The initial HTTP fixture harness timed out; a bundled offline fixture removed that harness dependency. QA artifacts live under ignored scripts/.shots/rewatch-*.png and verify-rewatch-bundled.mjs.

No commit, push, deployment, production mutation or poster correction.

Final checks: TypeScript --noEmit --incremental false, affected ESLint --max-warnings0, schema-reference, design-system coverage generation, both Git diff --check and source CRLF checks pass. Additional actual-browser checks pass for YEAR to MONTH to DAY date-picker changes, multi-season live preview and a single save. Local PostgreSQL stopped after rehearsal; disposable database removed. Production remains unchanged.

Correction after owner screenshot2026-10-21: the previously reported Stats duration-notice grouping had NOT actually applied; the replacement script silently matched no block. Fixed the actual source with an asserted match. Three new regressions parse/render the real Stats chart grid for0/1/4 incomplete events and require exactly three grid children with the notice inside the Hours column.27 UI tests, affected ESLint, coverage generation and four-width actual-card/grid browser layout checks pass; desktop screenshot read. This corrects the earlier inaccurate UI verification claim. Duration data and the four unknown legacy events are unchanged. No cache/restart explanation, production write or migration needed.


## Missing initial episode evidence after owner migration (2026-10-21)

Owner reports Entire show blocked by Historical episode availability is unknown. Fresh dedicated read-only remote query confirms latest registry version20261021020000 (owner applied), but no TMDB/AniList provider head exists for personal Game of Thrones, Breaking Bad, both Hunter x Hunter identities, or Black Clover. Stored media season totals/boundaries exist; they are not the per-episode identity/date/runtime evidence required by the historical rewatch contract. No remote write or provider/observer invocation performed.

Corrected watching-series-sync bootstrap scheduling: bounded paginated batch reads of media, accepted TMDB heads and legacy rewatch IDs; missing evidence independent of last_synced_at/weekly ended cadence. Prioritize legacy-rewatch works; cap the special bootstrap queue at13 of25 to preserve airing-show capacity; de-duplicate works across queues, keep normal accepted-ended weekly cadence and avoid immediately hammering failed initial checks. All facts still pass through the existing observe_provider transaction; no direct personal writes or guessed historical episode membership. RewatchForm explains missing sync data and exposes Check sync status rather than implying an ordinary UI refresh initializes provider facts.

82 targeted tests across actual robot handlers, rewatch domain and UI pass (four new scheduler cases plus missing-catalogue feedback regression). Offline actual React/browser form checks still pass at390/768/1024/1440, including real date picker and multi-season save. Existing UI tests use the host Error constructor in their VM to match real-browser error identity. No new migration needed: optional runtime validator is already applied. Owner must deploy watching-series-sync then invoke its normal batch (or wait for its cron); verify accepted catalogues/runtime availability read-only afterwards. Until this activation, blocked historical scope is deliberate and cannot be honestly unblocked by substituting today's season counts.


## Hunter x Hunter absolute episode numbering / edge typing (2026-10-22)

Owner deployed/invoked the robot and supplied partial207:12 successful BASELINE catalogues, Hunter x Hunter rejected Invalid episode evidence. Fresh read-only TMDB observations reproduce the2011 title46298: counts62/74/12, but episode_number uses1..62,63..136,137..148.1999 title45952 has normal1..62. This is absolute broadcast numbering, not a missing episode or invalid personal watch date.

compactSeason now optionally accepts the proven cumulative preceding-season count. Only a full block exactly inside that offset interval can normalize to local1..N; IDs/dates/runtimes unchanged, input untouched. Wrong/missing offsets, partial membership, gaps, duplicate IDs/numbers and first-season absolute offsets still reject. Worker supplies cumulative announced counts from verified consecutive sorted season identities, rejects mismatched returned season identity, and keeps the existing observed-provider boundary. No schema/personal data rewrite, fabricated dates or exception for a hardcoded title.

Explicit ProviderHead interface, Map<string,ProviderHead> and typed tuple entries replace imprecise Map inference reported by the owner's IDE. Initial-failure retry cooldown is10minutes rather than reusing the6-hour airing freshness threshold; recent failures remain throttled, normal accepted show cadence unchanged.

69 targeted tests/three files pass, including the real worker producing all148 normalized episodes and existing initialization/staleness regressions. Fresh raw TMDB replay through compactSeason and buildRewatch confirms2021 whole-title coverage148episodes/3503known minutes and preserved identities. Deno check passes on the actual edge entrypoint (the app tsc excludes edge code). Default modern Deno lint flags the pre-existing inline esm.sh import; all affected edge checks pass with only no-import-prefix excluded, preserving the established deployment import. Node affected tests lint and diff/CRLF checks apply. No function deployment, production write or new migration in this correction. Next: owner redeploys watching-series-sync and runs its normal Test invocation; confirm actual acceptance before claiming Hunter remotely ready.
