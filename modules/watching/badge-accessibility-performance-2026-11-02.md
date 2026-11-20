# Watching badge, accessibility and performance consolidation - 2026-11-02

> Update: owner subsequently REJECTED the badge skin/image-only hover. Those parts were reverted; independent fixes remain. See badge-rollback-hover-2026-11-02.md.

## Initially implemented scope

- New WatchingBadge adapter uses the shared Badge pastel variant; all ten Watching consumers of the old primitive now use the adapter. Removed their old variant selections. Global CSS owns the semantic palette, material and pill radius; reference-image pastel labels supersede the old Watching badge skin. Other modules keep their existing variants. Scores/hearts/bookmarks/rank ribbons remain distinct marks.
- CaughtUpBadge is the single caught-up presentation: StatusMark delegates to it, NextEpisodeButton already uses it. Last Watched puts its timestamp or caught-up badge immediately beside the title in the same flex row.
- Root MovieCard and ForYouCard no longer scale. Only the artwork zooms, respecting reduced motion; labels no longer translate or rasterize under the parent transform. This differs from the prior whole-card zoom, not from business state.
- For You reuses AddMark -> useQuickAdd -> useAddMedia for direct Want to Watch, beside dismiss. Keyboard focus reveals controls, touch controls stay visible with44px hit targets. No new writer or duplicate add workflow. Existing add success invalidates owned data/removes the recommendation from the For You cache.
- Table poster/title navigation uses native buttons with focus indicators; keyboard activation calls the existing navigation callback. Carousel titles also provide a keyboard-accessible entry point.
- Removed unused hasEstimatedEpisodeCounts. Retained LegacyConsumption: types.ts/service.ts use inline imports despite extended Knip reporting it unused.
- Membership add/remove now invalidates membership, affected list items and list thumbnail queries, matching the existing list-page operations.
- Stats All time shares the all-time useMemo result with achievements instead of computing the same ledger twice. Selected-year calculation remains independent; no rating/year/hours aggregation changes.

## Verification

109 targeted tests pass in5files, including6 new regression cases (shared caught-up markup, both membership operations, For You add identity/dismiss isolation, native table controls, single all-time calculation). Earlier targeted series-state coverage also passed. TypeScript, affected lint, Knip CI and regenerated DS coverage verified;444 existing off-system hits. Extended Knip exports scan has no unused Watching runtime export after helper removal; retained type warning explained above. Other modules' warnings are outside scope/frozen.

Actual-component browser fixture at390/768/1024/1440px verifies no overflow/JS errors, equal caught-up visual styles, same-line Last Watched title/status, stable badge coordinates before/during/after hover, mocked direct-add identity and event isolation, and Enter/Space table activation. Screenshots read. All mutation hooks are mocked in the fixture; no production write tests.

## Comparative measurements

Saved read-only snapshot:516 media,27 history records. Same process/machine/data, alternating A/B order,30 warm-up pairs and40 measured runs per arm. Pipelines are extracted from the actual before/after StatsPage useMemo declarations; computeStats implementation unchanged. Full serialized results match for all-time and2026.

| CPU pipeline | Before median | After median | Before P95 | After P95 |
|---|---:|---:|---:|---:|
| All time Stats pair |29.65ms|15.89ms|43.56ms|26.52ms|
| Selected2026 + all-time |29.07ms|29.48ms|55.63ms|41.57ms|

All-time median reduced46.4%; selected-year path has no established gain (same two calculations;1.4% median difference). CPU timings exclude browser hydration/network and do not certify whole-page speed. Earlier isolated profile measured individual year computations around17-20ms and achievements around15ms median; absolute timings vary with host load.

Browser A/B reproduces root-transform movement at desktop: test badge displaced13.86px before,0px after. Touch scenarios do not apply hover scaling. Fixture dimensions are representative test geometry, not the owner's exact real card size.

Live localhost attempt with saved automated session loads the route but reports Couldn't load your workspace, with no media requests. All attempted live timings are discarded as unrepresentative. A complete authenticated end-to-end latency/request/bandwidth comparison remains unverified; do not call global performance perfect or high-level certified from these local tests.

Evidence (ignored local artifacts): watching-stats-ab.json, watching-cpu-profile.json, watching-badge-browser.json and watching-badges-{width}.png in scripts/.shots.

No new migration, user data mutation, deployment, staging, commit or push. Owner visual review is next.
