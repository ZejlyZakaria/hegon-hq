# Watching stats and rail alignment - 2026-11-08

## Owner-approved behavior

Annual rating distribution and its average/rated count use titles with first-view consumption in the selected year. Rewatch-only titles remain in activity/counts/hours and available years, but cannot contribute an old rating to these annual rating metrics. All-time rating eligibility unchanged. New seasons first-viewed in the selected year remain eligible with their period rating.

Wrapped posters and title counts/genre use that same first-view cohort. Known hours still include rewatches. Ranked by period rating, favorite only for equal-score ordering, then stable ID. Keep first three entries and all ties at the third entry's score. Exclude unscored titles from the poster selection. Manual circular fan shows up to three of these entries, with focused title/score/index, clickable sides and arrow-key navigation; no automatic rotation. Existing PNG export surface retained; no production screenshot/export certification claimed.

Hours panel rows use the rating computed alongside their period-aware contribution. Rewatch ratings use the film/whole-title score or a resolved season/cour score; unknown/episode-only scopes do not borrow a season opinion. Film rewatch scope hidden. Series/anime scopes translated into readable labels; resolved display season numbers included.

For You, Cast & Crew and Library People rails now use existing useAlignedRail, like MediaCarousel/Updates/season/episode/list rails. Removed independent click-only indices, navigation page state and duplicated target calculations; card sizing and layout preserved. Availability follows actual scroll position, including native scrolling.

Activity/Rating charts share header spacing, flexible plot height, baseline and12px label rows. Baseline color is the Watching vivid token at12%, matching zero activity bars. Rating zero-count points lie on the baseline. Existing numeric x scale and responsive card order remain intact.

## Verification

-74tests PASS: computeStats, stats-charts and rail-navigation. New regressions cover rewatch-only annual film/series exclusion, first+rewatch in one year counting once for ratings, newer seasonal rating retained, stable ties including five entries and Wrapped first-view counts. TS and affected ESLint PASS; Knip PASS; DS coverage regenerated; diff checked.
- Actual chart functions extracted into isolated browser fixture at390/768/1024/1440. Plot baseline and label top positions match for side-by-side cases; equivalent stacked plot dimensions; matching token-based axis colors. Four-format screenshots captured; desktop chart capture inspected.
- Actual WrappedCard in fixture: all five entries become center by side click, wrap to first, ArrowLeft works; screenshot inspected. Actual HoursBreakdownPanel/MediaRow with mocked navigation/images: movie rewatch WHOLE_TITLE absent, film and rewatch scores visible. No backend calls.
- Actual ForYouSectionClient/CastCrew/PagedRail function bodies exercised with mocked data/card dependencies, real useAlignedRail and CarouselNav: native scroll to end updates both arrow states; Previous uses actual scrolled position. For You includes loading-to-content transition.
- Evidence remains ignored under scripts/.shots/: verify-stats-fixes.mjs, create-stats-fixes-fixture.mjs, verify-rail-consumers.mjs and screenshots. Fixtures do not certify live production personal data, provider networking or full route layout.

## Scope

No schema, migration, provider robot or production/user-data write. No staging/commit/push. Pending separate task: season rating requires a fully aired AND fully watched season, or a separately identified finished part; caught-up alone is insufficient. This block does not change that write eligibility.


## Owner follow-up: cohort explanation and row grammar

40Wrapped films versus43KPI films follows the intentional cohort distinction (first-view titles versus all titles with activity, including rewatch-only titles). No live personal-data query or count correction in this follow-up. Hours first-view season labels and Top Picks panel labels now use MediaRow suffix on the title line; title truncates before the reserved suffix. Personal scores use existing ScoreMark source=mine (teal), including Top Picks preview; world score colors unchanged. Wrapped own score uses matching RGB teal for its existing export-safe inline appearance. Unknown-duration rows follow the same suffix grammar.
