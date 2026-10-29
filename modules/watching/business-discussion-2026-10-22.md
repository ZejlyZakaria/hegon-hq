# Watching business discussion — 2026-10-22

Scope: owner requests business discussion before Black Clover poster correction, poster-quality/performance inspection and diagnosis of inconsistent achievements/slow Marathoner. Read-only investigation and offline reproductions; recommendations below are NOT approved business-rule changes. No application/schema/production modification or general Wave 2E reconciliation.

## Fresh evidence

Owner/org-scoped SELECT snapshot: 513 personal media rows,12 history assertions,10 compatibility/legacy rewatch rows. Existing achievement receipts read directly. The private receipt_source function is not executable by the dedicated read-only endpoint role; the authorized SELECT body was used against the permitted tables instead. No observer or receipt/Goals writer invoked. Local ignored snapshot/results: scripts/.shots/watching-business-audit-current.json, watching-business-audit-results.json, audit-watching-business.cjs.

Black Clover is In Progress at storageS2E3. season_years={1:2022},season_ratings={1:8.5}; global user_rating8.5. Current aired counts170/3, both season posters available, MediaView not overlaid (old cour_years residue does not drive this view). Actual computeStats reproduces2022=68hours,rating8.5,label Season1,poster null;2026=1.2hours,no annual rating,label Season2,poster null. Thus no old rating leaks into2026. seasonPosterFor incorrectly requires >=2 distinct stamped years before considering a one-season annual poster. Only S1 has an explicit year stamp, while S2 has valid current/history consumption. A single-season display must depend on the actual period scope, not on a second explicit year stamp. Simply removing the guard alone does not resolve history-only periods with no year-map entry.

No blanket transfer of title-level fields to S1 is recommended. Personal season/cour consumption, dates, scoped opinions and rewatch membership should stay attached to their recorded historical scope; original global title opinion stays distinct. Prospective single-season events should already capture membership before future growth. Ambiguous old global opinions/scope cannot silently become confirmed S1 evidence. Existing frozen rewatch ranges/durations survive growth; old legacy rewatch events require explicit completion, not lifetime expansion.

Legacy first-watch duration is not universally immutable: offline change of Black Clover runtime24→48 changes its2022 estimate68→136hours, while stored scope/year remains2022. This is a counterexample, not a database mutation. A product decision on accepting/freezing legacy estimates is needed before promising all older hours are immutable. New canonical rewatch hours already freeze exact available membership and available episode runtimes.

## Achievements: different display and eligibility inputs

| Key | Live displayed total | Permanent eligibility | Existing receipt |
|---|---:|---:|---|
| long_hauler |500episodes (Naruto Shippuden)|0|none|
| ever_present |132months|9months|none|
| marathoner |3729.9hours|3729.9hours|present, first observed2026-10-21T17:48:27Z at3331.0333333333333hours|

Live computeWatchingAchievements reads completed legacy projections. receiptMetrics deliberately requires explicit canonical FINISHED_TITLE completion membership for Long Hauler and confirmed canonical dates for Ever-Present. withUnlockReceipts keeps the live500/132 progress but gates unlock on a separate receipt. Hence100% and locked are a real policy/UI mismatch; merely clamping the bar or asserting a receipt locally would not fix it. Tests currently protect this strict policy; do not silently reverse it.

Ever-Present has a third definition mismatch: live achievement counts watched_at months of finished titles only, whereas actual all-time Activity ledger has133months including other usable consumption/rewatches. Recommended discussion: define Active across120months using the same accepted monthly activity ledger, with no invented month for YEAR-only data.

Recommendation for this personal tracker: use one accepted business metric for progress, explanatory groups and permanent eligibility; include owner-accepted legacy information transparently, retain unknown/missing exclusions, and record first observed satisfaction now rather than inventing historical unlock dates. Clarify Long Hauler's finished-scope contract so a future season cannot erase an earned completion or qualify merely from newly announced counts. Existing immutable receipts stay immutable. These choices need owner discussion because they change the previously locked strict eligibility policy.

## Marathoner loading/performance

Fresh receipt exists already, but useAchievementReceipts only obtains it after observeWatchingReceipts completes fresh auth/source read, metric evaluation, demo/Goals checks, achievement writer, optional Goal writer and final receipt SELECT. Query is async independently of Stats and defaults its missing response to an empty receipt list, so an already acquired badge initially appears locked. Current AchievementPanel resolves the open key against refreshed achievements; stale open-object capture is NOT a current finding.

Read-only snapshot serializes to7,367,302bytes of JSON, not measured HTTP compressed transfer. Warm local Node/V8 medians (3warmups,9runs): computeStats19.36ms,receiptMetrics21.90ms,live achievements39.33ms. Earlier same-method run varied to25.91/29.52/47.66ms. These bound pure computation only; authenticated action round-trip latency was not invoked/measured because it writes production observations. Do not claim an end-to-end speedup from these CPU results.

Recommendation: read small existing immutable receipts first and show them immediately; keep fresh observation separate from initial display, reuse accepted cached receipts, distinguish pending verification from a locked state, reduce unnecessary snapshot fields/repeated derivation after measurement. Do not remove freshness/hash checks, forge receipts or modify frozen Goals merely to make a badge fast.

## Poster quality

Top Picks and panel rows share MediaRow:40CSS-pixel token, tmdbImageFor(url,40) with DPR2 gives TMDBw92; Next image optimization is globally disabled intentionally, so increasing Next quality alone cannot change these source bytes. On DPR3, the40px slot needs120physical pixels and w154 is the next supported source size. Custom/AniList images pass through the TMDB helper untouched. Raising every poster to w500/original is not justified.

Read-only CDN A/B on3public poster paths, bytes downloaded per same image:5609→10092,5612→11897,5325→24346 forw92→w154. Extra4.5–19.0kB per image in this small sample; not free, not a measured page-latency result. Recommend adaptive density/source selection on relevant surfaces, cache/lazy-loading retained, and actual page A/B before performance claims. Source identity/annual scope must be corrected before evaluating sharpness of the wrong poster.

## Discussion outcome pending

No new migration, production mutation, receipt observation, app edit, deploy or Git mutation. Owner should discuss/choose the acceptance policy for legacy achievement evidence and legacy-hour freezing; then implement coherent period-scope presentation, source-size selection and receipt-first display with targeted regressions and measured end-to-end validation.
