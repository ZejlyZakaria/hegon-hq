# Watching Wave 1 review ? 2026-10-11

Implementation only; no commit, migration application, robot deployment or production mutation. The pre-existing untracked AGENTS.md is outside this block. Goals changes are restricted to Watching-linked completion, under explicit owner authorization.

## A. Files changed

- `src/app/(main)/perso/watching/[id]/page.tsx`
- `src/app/(main)/perso/watching/discover/[type]/[tmdbId]/page.tsx`
- `src/app/(main)/perso/watching/person/[id]/page.tsx`
- `src/modules/goals/lib/watching-target.ts`
- `src/modules/goals/service.ts`
- `src/modules/styleguide/coverage.generated.json`
- `src/modules/watching/components/detail/CastCrew.tsx`
- `src/modules/watching/components/detail/QuickStats.tsx`
- `src/modules/watching/components/person/PersonJourney.tsx`
- `src/modules/watching/components/person/SeenTogether.tsx`
- `src/modules/watching/components/sections/WatchlistPanel.tsx`
- `src/modules/watching/components/stats/StatsPage.tsx`
- `src/modules/watching/components/stats/achievements.ts`
- `src/modules/watching/components/stats/computeStats.test.ts`
- `src/modules/watching/components/stats/computeStats.ts`
- `src/modules/watching/hooks/useAddMedia.ts`
- `src/modules/watching/hooks/useDeleteMedia.ts`
- `src/modules/watching/hooks/useMediaCredits.ts`
- `src/modules/watching/hooks/useMediaItems.ts`
- `src/modules/watching/hooks/useOwnedTmdbIds.ts`
- `src/modules/watching/hooks/useUpdateMedia.ts`
- `src/modules/watching/lib/anime-overlay.ts`
- `src/modules/watching/lib/credit-roles.ts`
- `src/modules/watching/lib/episode-counts.ts`
- `src/modules/watching/lib/media-view.test.ts`
- `src/modules/watching/lib/media-view.ts`
- `src/modules/watching/lib/mutation-dependencies.ts`
- `src/modules/watching/lib/pagination.ts`
- `src/modules/watching/lib/person-experience.ts`
- `src/modules/watching/lib/possession.test.ts`
- `src/modules/watching/lib/possession.ts`
- `src/modules/watching/lib/watchlist-page.ts`
- `src/modules/watching/lib/wave1-contracts.test.ts`
- `src/modules/watching/service.test.ts`
- `src/modules/watching/service.ts`
- `src/modules/watching/types.ts`
- `supabase/checks/expected-objects.json`
- `supabase/functions/_shared/batch-outcome.ts`
- `supabase/functions/_shared/credit-roles.ts`
- `supabase/functions/_shared/pagination.ts`
- `supabase/functions/_shared/provider-progress.ts`
- `supabase/functions/for-you-refresh/index.ts`
- `supabase/functions/watching-anime-cours-sync/index.ts`
- `supabase/functions/watching-people-sync/index.ts`
- `supabase/functions/watching-people-sync/unreleased.ts`
- `supabase/functions/watching-series-sync/index.ts`
- `supabase/migrations/20261011000000_watching_director_roles.sql`

HQ: `now.md`, `modules/watching/architecture.md`, `decisions.md`, `backlog.md`, and this review. HQ is a separate repository.

## B?I. Calculation and mutation contracts

- **B Annual ratings:** previous eligible seasonal maximum (2 and 10 -> 10) becomes arithmetic mean (6). Average rating, histogram, Top Picks and Wrapped share the same calculation. All-time title ratings remain. Annual series/anime ratings have no title-rating fallback: eligible seasonal/cour scores are required, including for single-year titles; absent scores are null. Regression tests cover the consumers, not merely the helper.
- **C Airing:** known zero remains zero; null/undefined/missing metadata is distinguishable via `airedKnown`. Duration fallback is per missing season and explicitly estimated in detail. Stats, achievements, overlays, claimable ceilings and rewatch estimates preserve zero. Known viewing without a season stamp remains in all-time totals; its annual attribution stays unknown. Historical legacy fallback remains deferred.
- **D Identity:** `movie:id` and `tv:id`; anime is TV classification. Ownership maps, discovery resolution, existing/reference additions, TV exclusion in recommendation ownership and filmography dedup preserve namespace. Person page filters numeric candidate reads by canonical credit key before consumption.
- **E Roles:** Director/Series Director => director; TV Creator => creator; Producer/Executive Producer => production, never director. Acting and anime voice buckets remain separate. Creator information remains available in cached credits and combined filmography roles; directing wins classification when a person is also a creator. No three-person storage cap. Untyped legacy film crew retains prior directing semantics; legacy TV crew remains stored but is not claimed as verified directing. Stats, detail, People counts and SQL ranking agree.
- **F Seen With:** experienced titles only; each collaborator/provider work counted once despite repeated cast, crew, dual roles or duplicate TV classifications. Distinct shared works remain distinct; acting takes display precedence over crew. Creators are labelled Creator, never Actor without an acting credit.
- **G Your Person:** engaged viewing participates; future/unexperienced ratings do not. Canonical duplicate works collapse; completion is separate. Journey says Experienced, Released filmography, and gives the completed count with partial-viewing wording. Visual structure retained.
- **H Goals:** exact current count >= positive target; 499/500 may display 100% but stays active, 500/500 can complete. A prior rounded 100% no longer prevents later genuine completion. Completion-write errors propagate. Only active goals recalculate; completed-goal reopening after edits remains a separate policy decision.
- **I Invalidation:** supplied-property checks include null clears and anime cour maps. Targeted dependencies cover Stats/library, person titles, People counts/rankings, rating percentiles, Top Picks and watchlists. Notes/routing metadata do not invalidate analytics; no blanket module invalidation added.

## J. Robots

Shared executable contract: success/legitimate no-op = HTTP 200 + ok true; mixed outcome = HTTP 207 + outcome partial + ok false; all substantive work failed = HTTP 500 + outcome failure + ok false. Existing watchdog >=400 detection needs no change.

| Robot | Prior issue | New behavior |
|---|---|---|
| watching-series-sync | Per-show failures could produce success; a season failure could abort subsequent shows; count restoration could look like new content | Successful writes counted, season failure continues batch, count regression rejected before facts/posture writes, dated growth checked |
| watching-anime-cours-sync | Failed fallback/cache writes could be counted as success | Write errors checked before counters; all failures 500, mixed 207; legitimate cached mismatch/none results count only after successful write |
| for-you-refresh | Seed/read failures could be swallowed into empty recommendations/ok; partial seed failure hidden | DB errors propagate, every failed seed counted, all seeds failing cannot pass; successful empty provider response is legitimate no-op and mixed seed results are partial; previous list preserved on empty candidates |
| watching-people-sync (including unreleased refresh) | Secondary season/unreleased errors could hide behind success, especially no followed people | Secondary failures and unreleased writes enter batch outcome; complete failure 500, mixed 207 |

Trending and awards contracts reviewed; their existing total-failure behavior required no modification. No cron/key/auth/watchdog changes.

## K?M. Complete collection surfaces

- **K Watchlist:** TanStack infinite query, 50 rows/page, exact total; server-side released/search/priority filters and title/year/recency sorting. Priority sort pages across four fixed buckets, not individual titles. See more exposes later pages. Live film list: 50/56 -> 56; search preserves the released filter. Service request regression verifies final priority bucket after item 100, with total 120.
- **L Top Picks:** existing SlidingPanel and MediaRow; favorite OR effective rating >=8, same rating-first/favorite-tie ordering, minimum UUID fallback for exact ties. Six remain visible, all qualifiers in panel. Live four-device check: six visible, 30 panel rows, no horizontal overflow; Escape closes.
- **M Other reads:** Range pagination with stable ordering for library, Stats, owned IDs/maps/index/status, rewatch history/stats, ratings, People titles/counts, episode highlights, list items and robot candidates/follows/seeds/cache reads. Anime overlays additionally batch 200 IDs to bound request URLs; real service request test returns all 1,201 overlays. Intentionally bounded provider result/robot processing batches remain bounded.

## N. Kuroko investigation

Read-only live app response observed on 2026-10-11:

```text
Kuroko's Basketball
status: ended; watched: false; in_progress: true
current_season: 3; current_episode: 25
season_aired: [26,26,25]
season_episodes: [26,26,26]
season_end_dates: [2012-09-30,2014-03-30,null]
```

The relevant robot is `watching-series-sync`; it counts dated season episodes and stores announced/actually-aired counts, end dates and status. Previously a decreased count could replace the coherent snapshot. Later restoration increased that degraded count and the robot interpreted the increase as growth, physically setting `watched=false`, `in_progress=true`, and seeding the cursor from the previous last-aired position. Thus the reopened state can be persisted personal posture, not solely presentation. `seriesState` then derives behind/caught-up/ended from the cursor and current aired facts.

If the current degraded row restores S3 to 26, its S3E25 cursor is behind 26; no automatic watched repair branch exists. The row already has no completed claim and no old S3 end date to prove restoration. Refresh does not clear seasonal years/ratings, watched_at, rewatches or history; the completion claim/posture can change, and current-metadata calculations can reinterpret durations. The present row alone cannot establish the original incident sequence or prove all historical facts were retained; no past robot log replay was performed.

Safe minimal Wave 1 guard: reject count regression/season removal before any refresh write, preserving previous coherent facts and posture; do not stamp last_synced_at on rejected refresh. If a settled season has unchanged announced count, existing cache reuse also preserves its prior coherent count. On a retained coherent baseline, stable -> degraded -> restored does not reopen viewing. Genuine increased aired count with a later known episode date can still reopen; when the old end date is absent the existing model offers only count evidence, which remains weaker. Tests separately exercise regression/restoration and dated genuine growth.

Already-degraded rows are not automatically repaired. Wave 2 requires provenance/history and verified viewing evidence, including correction-vs-restoration decisions; announced count alone is not proof that episode 26 aired or was watched. The fixture uses the observed [26,26,25] degradation and a coherent [26,26,26] baseline; dates in lifecycle fixtures are test data, not an assertion of current TMDB content.

## O. Validation

- Targeted domain/service suites: 8 files; 220 tests passed in the final run.
- TypeScript, affected-file ESLint, production build, schema-reference gate and DS coverage executed. Coverage regenerated: 26 primitives / 13 modules / 444 off-system hits.
- Edge syntax/local-import check: 14 files from seven robot roots; batch/growth contracts exercised by Vitest. No Deno executable was available: this is not a Deno semantic or deployed runtime test.
- Production app read-only verification: Top Picks four widths (390, 820, 1180, 1440), screenshots read; Watchlist pagination/search observed; current Kuroko series row read.
- Diff reviewed for unrelated changes; protected auth files untouched, frozen modules untouched outside the explicit Watching Goals exception. No broad unrelated tests/performance claims.
- Migration SQL and robot writes are not executed locally/deployed; owner application/deployment and real write verification remain required. Browser used existing session; no viewing/goal mutations made.

## P. Remaining HIGH issues

Intentionally deferred: mutable historical attribution/runtime reconstruction; YEAR/MONTH/DAY precision; rewatch scope/snapshots; permanent achievement evidence; completed-goal reversal policy; broad provider provenance and repair of already-degraded Kuroko posture. The final targeted review identified annual rating fallback and AniList semantic failures as blockers. Both are corrected and covered by regressions; the four listed minor fixes are also corrected. Deployment/runtime-write verification is outstanding evidence, not claimed complete.

## Q. Architecture / schema

One versioned migration replaces the existing `watching.people_ranking` body with role-aware/future-excluding predicates. Expected-objects migration watermark updated; object counts unchanged. Explicit optional role metadata is added inside existing JSON credits, not new columns. Small pure helpers share pagination/role/outcome/growth contracts between app and robots. No new table, event schema, global runtime validation, auth change or historical architecture.

## R. Verdict

WAVE 1 LOCKABLE

## Owner commands ? not executed

```powershell
npx.cmd supabase db push --linked
npx.cmd supabase functions deploy watching-series-sync
npx.cmd supabase functions deploy watching-anime-cours-sync
npx.cmd supabase functions deploy for-you-refresh
npx.cmd supabase functions deploy watching-people-sync
```

Review/commit the validated block first. After deployment verify schema reference and people rankings, then invoke robots with a real write and inspect the outcome and saved facts. Never use a destructive provider payload against personal data to recreate the Kuroko case; lifecycle degradation is covered by fixtures.

Application commit command (Git shell):

```sh
git add -- src/modules/watching src/modules/goals/service.ts src/modules/goals/lib/watching-target.ts src/modules/styleguide/coverage.generated.json 'src/app/(main)/perso/watching' supabase/functions/_shared/batch-outcome.ts supabase/functions/_shared/credit-roles.ts supabase/functions/_shared/pagination.ts supabase/functions/_shared/provider-progress.ts supabase/functions/watching-series-sync supabase/functions/watching-anime-cours-sync supabase/functions/for-you-refresh supabase/functions/watching-people-sync supabase/checks/expected-objects.json supabase/migrations/20261011000000_watching_director_roles.sql && git commit -m "Correct Watching analytics and complete collection reads"
```

HQ commit command (separate repository):

```sh
git -C hq add -- now.md modules/watching/architecture.md modules/watching/decisions.md modules/watching/backlog.md modules/watching/wave1-review-2026-10-11.md && git -C hq commit -m "Record Watching Wave 1 contracts and historical work triggers"
```


## Final targeted review corrections (2026-10-11)

Scope: only annual rating attribution, AniList semantic failure handling, movie/TV filmography React keys, follow role metadata, People partial outcomes and secondary-refresh cache retention. No Wave 2 work, deployment or production data mutation.

- Black Clover: historical 2022 seasonal score 9 plus unrated current 2026 activity yields 2026 activity/hours but null annual score, zero rated count/histogram contribution and no rating-qualified Top Pick, whether or not the current season has a year stamp. All-time score remains 9. Annual eligible season/cour ratings still use arithmetic mean; Wrapped uses the same attribution when ordering by score.
- AniList: HTTP success is insufficient. JSON/GraphQL errors, absent Page.media or malformed media records fail before any cache write. Existing data and resolved_at remain untouched. Valid empty media is distinguishable from no mapping (mismatch vs none for known nonzero episode totals). Valid writes yield 200; mixed success/failure yields 207; complete substantive failure yields 500. Mapping transport/shape failures also preserve the cache.
- Filmography renders namespace-aware movie:id / tv:id React keys. Actual JSX regression uses identical numeric movie/TV IDs and changes order.
- Cast & Crew forwards explicit Creating, Directing, Acting, Voice Acting or null metadata; unknown legacy TV crew is unverified. Following displays creator/voice labels. Weekly name/portrait refresh preserves credit-derived follow roles.
- People counts successful upcoming/profile writes separately. A failed profile after successful upcoming replacement is partial 207. Any secondary season lookup failure skips the entire upcoming-slice replacement, retains synced_at and reports retained IDs plus failure details; unrelated successful profile writes still count. A retained-only on-demand refresh is failure 500.
- Regression files: computeStats.test.ts, wave1-contracts.test.ts, robot-handlers.test.ts, review-ui.test.ts. Real Deno handlers run in a TypeScript-transpiled VM with faked transport/database boundaries; the separate unreleased-title job is stubbed. This does not claim Deno or deployed-runtime verification.

Final validation: 91 tests passed in the four targeted files; TypeScript and affected application-file lint passed; production build passed (justified by changed page/rail props); DS coverage regenerated; affected Edge TypeScript syntax passed for three files; application/HQ git diff --check passed. All affected source/generated files retain CRLF. Edge files are excluded by repository ESLint configuration; no Deno executable/runtime verification was available. Prior Wave 1 verification above remains historical evidence, not a rerun in this follow-up.

Remaining targeted blocker: NONE. Owner application/deployment and real-write verification remain outstanding release steps.

Follow-up files changed (generated coverage regenerated):

- `src/modules/watching/components/stats/computeStats.ts`
- `src/modules/watching/components/stats/computeStats.test.ts`
- `src/modules/watching/components/detail/CastCrew.tsx`
- `src/modules/watching/components/library/PeopleView.tsx`
- `src/app/(main)/perso/watching/[id]/page.tsx`
- `src/app/(main)/perso/watching/discover/[type]/[tmdbId]/page.tsx`
- `src/app/(main)/perso/watching/person/[id]/page.tsx`
- `src/modules/watching/lib/credit-roles.ts`
- `src/modules/watching/lib/wave1-contracts.test.ts`
- `src/modules/watching/lib/robot-handlers.test.ts`
- `src/modules/watching/lib/review-ui.test.ts`
- `supabase/functions/_shared/credit-roles.ts`
- `supabase/functions/watching-anime-cours-sync/index.ts`
- `supabase/functions/watching-people-sync/index.ts`
- `src/modules/styleguide/coverage.generated.json`
- HQ: `hq/now.md`, `hq/modules/watching/decisions.md`, `hq/modules/watching/wave1-review-2026-10-11.md`.

## Final display micro-fix (2026-10-12)

Top Picks SlidingPanel previously used one fixed decimal (Hannibal 8.75 displayed as 8.8). Its display expression now rounds to at most two meaningful decimal places and strips trailing zeros: 8.75 -> 8.75, 8.5 -> 8.5, 9 -> 9, 8.333333 -> 8.33. No suitable existing formatter was found; one local expression avoids introducing a duplicate helper. Numeric aggregation, qualification, global media ratings, schema/migrations and Black Clover behavior remain unchanged.

Changed application/test files: `src/modules/watching/components/stats/StatsPage.tsx`, `src/modules/watching/lib/review-ui.test.ts`, `src/modules/watching/components/stats/computeStats.test.ts`. DS coverage regenerated with unchanged counts (26 primitives / 13 modules / 444 off-system hits); generated file retains CRLF. HQ now and this review record updated.

Validation: 7 targeted tests passed across two files (41 unrelated tests skipped), covering actual panel JSX for all four requested examples, unchanged Hannibal Top Picks annual score 8.75, and both existing Black Clover stamp/no-stamp regressions. TypeScript and affected-file lint passed; application/HQ diff checks passed. No broad suite/build, live browser or production write was performed for this display-only micro-fix. Existing owner commit commands above include these files; no commit was run.

Remaining blocker: NONE. Verdict: WAVE 1 LOCKABLE.
