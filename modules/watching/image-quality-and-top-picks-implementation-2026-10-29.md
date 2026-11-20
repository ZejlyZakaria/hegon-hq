# Watching image quality and Top Picks presentation - implemented 2026-10-29

Owner approved implementation of the preceding audit. Local application changes only; no new migration, production data mutation, robot invocation, deployment or Git mutation. Wave2E remains LOCKED; preceding Kuroko data-repair migration/reference files remain separate working-tree changes.

## Presentation contract

Top Picks card and See all no longer show a release-year fallback. Redundant season identifiers disappear only when completed posture, the accepted first-view ledger and the rating subjects jointly cover the entire known aired/announced work in the selected year. Unknown coverage, other first-view years, an unfinished/unrated season, or an upcoming season retain the explicit rated scope. Rewatches do not select or alter this scope. All-time behavior and rating/qualification logic remain unchanged.

The existing ratedScopePresentation owns this decision with an optional full-work context; no second label formatter was created. Full-work presentation falls back to global artwork. Kuroko2018/8.5 and BreakingBad2020/9.4 drop S1–S3/S1–S5; Hannibal2026 stays8.75/S1–S2. Multi-year and partial cour/season identifiers remain explicit. Hiding a full-work label never exposes the release year in its place.

## Image changes

Extended/renamed the existing compact-poster helper to tmdbImageSrcSet; no duplicate compatibility alias or old export remains. It supports density candidates for fixed frames and width candidates with sizes for fluid frames, plus per-consumer width limits. Custom uploads/AniList art remain untouched. No runtime third-party API discovery or speculative original-image fetches.

- SeasonPanel header: responsive96/120CSSpx frame; native width candidates through500. Actual ×2 selection remains342; ×3 at120px chooses500, mobile96px remains342. Frame ratio and existing blurred background unchanged.
- SeasonHistoryStrip: adaptive width candidates through780; desktop144CSSpx and mobile --rail-peek. Native ×2 chooses342, typical ×3 chooses500 instead of previous fixed300. Existing lazy loading retained. Corrected the obsolete comment claiming all retired personal values automatically return after advancing.
- Shared MediaCarousel and ForYou: native picture candidates; poster rails170px choose185/342/500 for1/2/3density, with500cap. Landscape cards360px can select500/780/1280; landscape poster fallbacks cap780 to retain previous high-density resolution without raising them to1280.
- Low-count Don'tMiss expanded poster:220px candidates through780. Its ambientw92 blur stays unchanged.
- MediaRow Top Picks/Hours/search/timeline keeps the existing40px/w92/w154 policy through the same generalized helper.
- Changed affected eager native pictures from Next priority preload to loading=eager + fetchPriority=high, so an unoptimized fallback URL is not preloaded in addition to the chosen dense source. Lazy/eager boundaries remain unchanged. Corrected the stale comment claiming the disabled Next optimizer supplies responsive AVIF/WebP sources.

No detail hero/backdrop, episode/best-episode still, image frame tokens, provider robot or database image URL rewrite. The known400pxKuroko still is not upgraded to a larger fake-detail variant. Source metadata is not available for a universal native-resolution cap; the bounded consumer policy avoids claiming such a capability. Higher-density files can add bytes; no zero-latency-cost or page-performance improvement is claimed.

## Validation

-192tests across7targeted files PASS: image candidates/non-TMDB pass-through, full/mixed/unrated/upcoming rated scope, actual Top Picks JSX card/panel without release year, rating/history/accepted-consumption regressions. Updated the older Hannibal rating-lineage fixture to represent its actual three-season partial state; confirmations still add no vote and8.75is unchanged.
- TypeScript noEmit and affected-file ESLint --max-warnings0 PASS; Knip passes with3existing configuration hints. Old tmdbPosterSrcSet export/imports removed. Design-system coverage regenerated:26primitives/13modules/444off-systemhits, no generated content change. CRLF preserved in touched source files.
- Replay of actual current computeStats before/after (Git HEAD baseline) against the retained post-Kuroko snapshot across15all-time/annual periods: all scores, qualification, order, hours, counts, distributions, activity and non-TopPicks presentation equal; only12redundant full-work labels removed. No new production read/mutation required.
- Real JSX fragments/components rendered with current Tailwind CSS and real Next Image configured unoptimized, in isolated Chromium fixtures at390/768/1024/1440andDPR2/3. Eight cases PASS: no page overflow, fixed2:3season poster, native source selection, retained Hannibal scope and absent release-year fallback. Native card usesw500atDPR3; compact posters remainw154. Separate eager-card request test downloads onlyw500, not the oldw342fallback. Screenshots inspected for mobile/desktop. Fixture navigation does not authenticate or mutate production; this is not a full authenticated application/E2E sign-off.
- Schema-reference and main/HQ whitespace checks PASS. No broad suite/build or performance claim.

## Exact application manifest

lib/tmdb-image.ts; lib/season-history.ts; components/stats/computeStats.ts; components/stats/StatsPage.tsx; components/shared/MediaRow.tsx; components/shared/MediaCarousel.tsx; components/sections/ForYouSectionClient.tsx; components/sections/DontMissSectionClient.tsx; components/detail/SeasonHistoryStrip.tsx; components/detail/SeasonPanel.tsx.

Tests: lib/tmdb-image.test.ts; lib/season-history.test.ts; components/stats/computeStats.test.ts; lib/review-ui.test.ts; lib/accepted-consumption.test.ts; lib/history-consumption.test.ts. Paths relative to src/modules/watching/.

HQ: this report, current status/decision and architecture pointer. Ignored diagnostic scripts, sample assets, saved private snapshot, screenshots and browser/replay JSON under scripts/.shots are not staging candidates. debug.log is unrelated.

Next: owner reviews the local result; Git commit/push and deployment remain owner actions. No blocker identified in the targeted checks.
