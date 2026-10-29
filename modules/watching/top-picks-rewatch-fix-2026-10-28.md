# Annual Top Picks: exclude rewatch-only activity

Owner reported Interstellar and Fight Club in 2026 Top Picks and explicitly approved the rule/fix: an annual pick requires first-view consumption in that year. A rewatch alone remains activity and additional hours, not a new annual pick. All-time qualification and existing favorite-or-score threshold/ranking stay unchanged.

## Reproduction and cause

Replayed the fresh owner-authorized2026-10-28 snapshot used by final2D validation. Interstellar first-view date2019-08-04; Fight Club2024-08-20. Both have only canonical REWATCH consumption in April2026, also represented in the compatibility table without double counting. Both are favorites. Top Picks used the same broad `inPeriod` predicate as Activity/counts, so either favorite status or a high legacy rating admitted a rewatch-only title. Identity exhibited the same favorite-path defect.

## Fix and manifest

`computeStats.ts` builds the Top Picks eligible ID set from the existing deduplicated consumption ledger. Annual sets exclude rewatch contributions; all-time sets retain them. The broad activity predicate stays unchanged. Main card and See all receive the same computed `topFavorites` list. No new business helper, second ledger, persisted flag, score rewrite or database operation.

Changed source/tests:
- `src/modules/watching/components/stats/computeStats.ts`
- `src/modules/watching/components/stats/computeStats.test.ts`
- `src/modules/watching/lib/history-consumption.test.ts`
- `src/modules/watching/lib/accepted-consumption.test.ts` — existing cour-poster fixture now uses first viewing rather than assuming rewatches qualify for annual picks.

Seven added regressions cover legacy favorite/high-score qualification, frozen film/series/anime rewatches, same-year first-view plus rewatch, unknown first-view year, preservation of original-year/all-time picks, hours/calendar/counts. Six counterexamples failed before the source fix; the same-year control already passed. A temporary optional-argument predicate was rejected by the full snapshot equality check because Array.filter supplies an index; it was removed. Final code keeps the single-argument activity predicate and a dedicated ledger-derived eligibility Set.

## Validation

-205 tests/7targeted files pass, including Hannibal8.75, Black Clover historical-rating isolation, frozen consumption, receipts and part/season histories. TypeScript and affected ESLint pass; main/HQ diff checks pass. No broad suite or repeated production build for this pure selection correction.
- Real snapshot:2026 picks33→30, excluding Interstellar/Fight Club/Identity. Exact deep equality for **every other computeStats output** in2026 and all-time. All-time Top Picks list unchanged.2026 hours remain270.31666666666666; rewatch hours8.433333333333334. Existing scores, dates, monthly activity and receipts are untouched.
- Actual TopFavoritesCard and SlidingPanel exercised from current source in an isolated browser at390/768/1024/1440. Both films absent from2026 card/panel, present in All time and their original2019/2024 panels. Hannibal See all score8.75 retained. No page errors/horizontal overflow; mobile/desktop screenshots read. Images may be pending in the early mobile capture; no image rendering code changed. No authenticated production observer/mutation test.

Ignored evidence: `scripts/.shots/top-picks-rewatch-before.json`, `top-picks-rewatch-results.json`, `top-picks-rewatch-browser-results.json`, `top-picks-rewatch-2026-*.png`, harnesses alongside them. No new migration, production mutation, staging, commit, push or2E work. No remaining blocker for this correction. Prior AniList worker activation still requires owner confirmation before full2D operational closure.
