# Watching loading architecture — read-only audit, 2026-11-05

Baseline main commit:370d8eb. No application source, auth/provider, database/schema, staging, commit or deployment change. Only ignored local verification scripts/artifacts and this HQ report. Owner restarted localhost after the initial connection refusal.

## Verdict

Keep the existing client-side TanStack architecture. It already supplies one QueryClient, keyed reference reads, concurrent independent reads, deduplication, stale/gc tiers, cached renders and background refetch. Lists/Awards need a focused loading/error/skeleton alignment pass, not a rewrite or a second fetch layer.

## Proven findings, ordered by impact

1. **Read errors masquerade as empty data.** ListsClient reads data default[]/isLoading without isError. Simulated media_lists503 (browser-only) retries once then shows only Create a list, with no failure state. AwardsClient also defaults winners/owned data to[] without error handling: simulated award_entries503 shows an empty trophy shelf and category0/0, despite existing personal data. Separately, getListsWithThumbnails ignores the child media_list_items error and returns count0/thumbnails[]; executing the actual AST-extracted function with a failed child read reproduces this. Counts must remain unknown/stale on failure, not fabricated as0.
2. **Two legitimate phases are not visually coordinated in Awards.** AwardsClient's loading branch calls CarouselSkeleton with default Last Watched/add=true props; the actual rail is Your trophy shelf with ceremony/See all controls. On data readiness MediaCarousel mounts its own grouped-image reveal skeleton. Browser traces show data placeholder -> image reveal placeholder -> content. The default title text is invisible skeleton sizing, not a visible mislabeled heading; DOM overlaps during reveal do not mean two visible panels. Preserve section-scoped image gating, but use the same surface/chrome configuration across both phases and avoid needless remount/restart.
3. **Independent categories wait for shelf hydration.** AwardsClient uses one loading condition covering categories,winners,owned AND useShelfRows. The full-card shelf query genuinely depends on the derived owned winners, so its later read is justified; category cards do not require those full rows and should not be gated on their delay. AwardCategoryClient also legitimately waits for category identity before reading its rows; do not replace this with an unscoped broad read.
4. **Route skeleton structure remains inconsistent.** Lists/loading.tsx returnsnull; a shell-only interval before client skeleton mounting was observed. ListsClient contains its full grid skeleton inline. Awards overview/category/ceremony routes lack dedicated loading boundaries and contain local/full inline skeleton compositions. Extract named view/section skeletons, reuse at route/client boundaries and preserve tiny local micro-loaders. Existing core Watching/Updates routes already use named skeletons and should stay unchanged.
5. **Additional cache identity concern, source inspection only.** useOwnedTitles accepts userId but AWARD_KEYS.owned() has no user/org arguments; shelfRows key is based on ids only. QueryProvider clears cache on signout, so no cross-user disclosure is claimed. Workspace/context switching was not exercised. Review Watching-owned cache context before adding workspace-aware behavior; do not modify auth/provider in this scope.

## Execution evidence

- `scripts/.shots/audit-watching-loading.mjs`: actual authenticated localhost, only HTTPS GET/HEAD allowed; all writes aborted. Cold Lists/Awards/category/ceremony,400ms artificial read delay,4viewport screenshots390/768/1024/1440, cached Lists return, Oscars->Emmys->Oscars and local browser503 failures. Screenshots inspected, no page runtime error or responsive overflow on the exercised surfaces. These delayed dev observations are not production latency benchmarks.
- Cached Lists return makes zero additional media_lists/media_list_items reads. Awards cold requests include winners offset0+1000 and a distinct ceremony-year2027 read: legitimate Range pagination/different datasets, not duplicate fetching. Ceremony switch reads new Emmy data; cached Oscar return restores it.
- `scripts/.shots/audit-watching-background.mjs`: browser Date advanced3minutes (local clock only), confirmed navigation back to Awards, two background media_items reads (owned index + full shelf), existing personal data retained. A brief image-reveal placeholder on remount is observable; no claim of zero flashing on every cached return. Clock-adjusted performance timestamps are invalid as duration metrics and are not used. Category Watched filter triggers zero additional backend reads.
- `scripts/.shots/audit-watching-cache.mjs`: real installed QueryClient/QueryObserver, two simultaneous observers execute one query; fresh cached remount makes no read; forced background refetch retains prior data and isLoading=false. Actual extracted getListsWithThumbnails function reproduces swallowed child-read error. No production connection for this unit check.
- Error screenshots confirm false empty Lists/Awards states without mutating any real data. Sentry writes are intentionally aborted, not app data read failures. Network fault scenarios are synthetic; they do not imply an existing production outage.
- Raw timelines/request-shape summaries/cache evidence remain ignored in scripts/.shots/. No tokens or auth headers logged. Audit does not certify all Watching routes, real production performance, exact route/client pulse continuity or workspace-switch cache correctness.

## Recommended bounded implementation

First correct read/error presentation and propagate child list-read failure, then align reusable route/client skeletons, correct trophy shelf placeholder props and split independent readiness gates. Preserve existing cache, grouped image reveal, data semantics and native layouts. Regression checks: success/empty/error, valid counts on partial failures, fresh/stale cached navigation, reference switches, local filters,4device geometry and repeated image reveal. No schema or production mutation needed.


## Implemented bounded alignment

Owner-approved implementation: Lists/Awards now distinguish loading, successful empty, blocking error and cached refresh error. Child media_list_items failures propagate instead of producing invented zero counts. Explicit retry retains the existing query/cache layer. List detail also protects existing metrics from failed reads.

Full Lists/Awards/category/ceremony skeleton compositions live in dedicated files and route boundaries reuse them. Tiny local placeholders remain local. Awards data/image phases share the trophy shelf chrome; categories no longer wait for full shelf rows. Grouped image reveal is unchanged. Route and client boundaries are sequential phases, not a second fetch layer; pulse animation continuity across unmounts is not guaranteed.

Validation: 28 targeted service/read-state tests PASS, TypeScript, affected ESLint, Knip, design coverage generation and production build PASS. Authenticated read-only localhost scenarios cover four surfaces at390/768/1024/1440; no horizontal overflow at checked widths, no page runtime errors. Categories show20links during deliberately delayed shelf hydration. Simulated503 primary/child Lists reads, Awards winners, shelf rows, category and ceremony reads expose errors instead of empty data; explicit retry recovery exercised. Browser checks abort external writes. Screenshots reviewed for mobile Awards/ceremony; data-dependent category/ceremony row counts remain approximations in skeletons. No claim of production performance or exhaustive pixel identity.

No database, migration, robot, auth/provider, staging, commit or deployment change. Workspace-aware cache identity remains outside this bounded task. Local evidence: scripts/.shots/check-loading-alignment.mjs and check-ceremony-retry.mjs, corresponding screenshots and request/state reports.


### Stable trophy shelf header - 2026-11-06

Known trophy shelf title now stays visible through route/data/image loading. Opt-in shared carousel skeleton header uses a fixed256px (container-clamped) subtitle placeholder instead of measuring generic then counted text. Categories unchanged; grouped image reveal unchanged. Authenticated read-only localhost sampling:25loading samples, one256px width, title always visible. TypeScript, affected ESLint, DS generation and diff checks pass.


### Skeleton visual harmony - 2026-11-06

Extracted standalone SkeletonPulse primitive; WatchingSkeletons re-exports it for existing consumers, Person's duplicate Bar removed. Lists white10 placeholders now use surface-2; structural card backgrounds stay surface-1. Placeholder radius tokens harmonized in named skeletons. Removed nested animation from desktop Details/Person heroes, Ceremony rows and Library cells; Stats animates individual placeholders rather than the entire page including static surfaces. Hero backdrop has its own pulse layer. Layout dimensions/breakpoints unchanged.

Verification: isolated actual component fixture,7views x4widths (390/768/1024/1440), all rendered pulse backgrounds rgb(30,30,34), zero nested pulse nodes, reduced-motion duration0.01ms. CastCrew/MoreLikeThis were stubbed in this isolated harness; their source uses unchanged surface-2 pulse leaves. Screenshots Person desktop and Awards mobile inspected. TypeScript, affected ESLint, Knip, DS and diff verified. Evidence scripts/.shots/verify-skeleton-harmony.mjs and skeleton-harmony.json. No production/data mutation. This validates visual primitives, not network loading timing.
