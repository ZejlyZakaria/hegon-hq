# Top Picks rated subjects and owner-confirmed Bleach Part4 viewing year

Current status corrected2026-10-29: migration20261028010000 is ALREADY APPLIED remotely, and its exact archived file is restored locally. Earlier not-applied claims below were incorrect or preflight-only; do not execute the historical push instructions. No production modification during restoration. See migration-alignment-2026-10-29.md for identity proof and the unresolved full fresh-replay limitation.

## Product contract and findings

Top Picks presents the subjects supplying its annual score, not every season touched by activity. Hannibal's8.75 comes from S1 and S2; progress at S3E1 does not add a third rated subject. Compact identifiers distinguish one season (`S3`) from a rated range (`S1–S2`) and separate subjects (`S1, S3`). Nested separately rated parts may say `S2 · Part4`. The existing card/panel layout is retained; no extra progress line competes with the score. All-time uses the global opinion without inventing a seasonal scope. An unrated favorite may still qualify under existing policy, but does not acquire a false rated-season label.

Fresh SELECT-only Bleach backup confirms nested Part4 is S2E41–50, with E49 dated2026-10-20 and E50 dated2026-10-28. E41–48 remained in accepted legacy2023 coverage. The owner explicitly confirms all ten Part4 episodes were watched in2026. There is no confirmed month/day for E41–48.

Important correction to the earlier UI hypothesis: the stored current9 is a **SEASON** assertion with all50 S2 episode IDs, not an independent Part4 opinion. Its Top Picks label must therefore be `S2`. Consumption details may say `Season2 · Part4 · EP41–50` after the year repair. No score or frozen rating membership is copied into a part or rewritten.

## Single source of truth

`historicalRatingResult` returns both the existing selected score and the exact selected subjects, after lineage, period qualification, cleared assertions and overlapping season/part suppression. Numeric consumers call the existing `historicalRating` wrapper over this result. Top Picks consumes that same result; no independent vote/eligibility calculator. Legacy seasonal votes retain existing qualification rules.

`ratedScopePresentation` formats those subjects in display coordinates, preserves stable cour identities for overlays/nested parts, refuses missing identities and chooses a season poster only when a single scored season is represented. Activity/hour labels keep their separate consumption meaning. Hannibal's poster stays global because S1/S2 both supplied the note; JJK uses S3 artwork.

## Data correction and safeguards

The new versioned migration calls the existing guarded personal-history command once. It appends only a confirmed YEAR2026 FIRST_WATCH assertion for E41–48, with accepted episode IDs. E49/E50 keep their existing precise records. Exact overlap subtraction removes the same eight episodes from the legacy2023 remainder; no baseline rewrite, manual SQL-editor operation or new writer.

Duration remains the existing frozen legacy estimate24min/episode (192min), preserving the agreed all-time total. Accepted provider runtimes differ for some episodes; this year correction deliberately does not recalculate them or claim recovered exact duration. Provenance is explicit in the command reason. No release date supplies a personal viewing date.

Guarded by owner/org/media identity, revision8, accepted catalogue snapshot, exact accepted cour map, frozen legacy baseline, posture/position, old season stamps, absence of existing overlapping first-view claims and the precise E49/E50 evidence. Provider-before-media locking matches the existing command order. Fixed operation identity makes repeats harmless; stale/malformed input aborts the transaction. Prior assertions, ratings, rewatches, receipts, completion date and other titles stay unchanged.

Projected Bleach first-view totals:

| Period | Before | After |
|---|---:|---:|
| 2020 |146h24|146h24|
| 2023 |19h12|16h|
| 2026 |48min|4h|
| All time |166h24|166h24|

The new YEAR-only contribution does not invent monthly/daily activity. S2 period remains2023–2026, old season stamps remain S1=2020/S2=2023, finish remains2026-10-28. All-time milestone metrics unchanged.

## Validation

-202 tests/8targeted files pass: scored scope selection, Hannibal partial S3, compact contiguous/separate identities, nested part labels/remapping refusal, unknown scopes, year-only overlap correction, preserved precise dates, original annual scores, no rewatch-only picks and receipt/organization contracts.
- TypeScript, zero-warning affected ESLint, Knip (three existing config hints), schema reference and main/HQ diff checks pass. No repeated production build/full suite for pure calculation/label changes. React component source/styles unchanged; existing design coverage remains valid.
- Fresh real-data replay verifies JJK9.5=`S3`, Hannibal8.75=`S1–S2`, Bleach9=`S2`. Exact equality of all non-Top-Picks Stats outputs against the pre-refactor snapshot, and identical selected numeric scores. Proposed data repair preserves all-time hours, first-year2020, scores, counts, monthly/daily activity, and milestone metrics; only the intended192 annual minutes move.
- Real PostgreSQL16 disposable rehearsal: migration application/repeat, revision/catalogue stale rejection, malformed-date rollback, frozen IDs/192min, YEAR precision, identical prior history/ranges/ratings/rewatches/receipts/other media, identical schema counters. Loopback-only runner drops test databases; server stopped.
- Actual TopFavoritesCard/SlidingPanel tested at390/768/1024/1440, screenshots read: compact JJK/Hannibal/Bleach labels, Hannibal8.75, no EP49–50 or ambiguous3-seasons label in picks, no rewatch-only Interstellar/Fight Club, original-year/all-time inclusion retained, zero overflow/page errors. Isolated current-data fixture; early mobile artwork is still loading in its capture. No authenticated production writer/observer smoke.

## Exact manifest

- `src/modules/watching/lib/history-analytics.ts`
- `src/modules/watching/lib/season-history.ts`
- `src/modules/watching/components/stats/computeStats.ts`
- `src/modules/watching/lib/season-history.test.ts`
- `src/modules/watching/components/stats/computeStats.test.ts`
- `src/modules/watching/lib/history-consumption.test.ts`
- `src/modules/watching/lib/accepted-consumption.test.ts`
- `supabase/migrations/20261028010000_watching_bleach_part4_viewing_year.sql`
- `supabase/checks/expected-objects.json` — watermark advanced, all six counters unchanged.
- `scripts/test-watching-bleach-part4.mjs`, `scripts/test-watching-milestone-receipts.mjs` — reusable local rehearsal option `--bleach-part4`.
- HQ report/current state/decisions/architecture.

Ignored proof/backup: `scripts/.shots/watching-point1-current.json`, `bleach-part4-provider-current.json`, `bleach-part4-command.json`, `bleach-part4-proposed-snapshot.json`, `bleach-part4-replay-results.json`, `rated-scope-top-picks-*.png`, `rated-scope-top-picks-browser-results.json`, local replay/parity/browser harnesses. These are diagnostics, not commit inputs. Earlier dirty implementation and unrelated debug.log remain intact.

## Owner application

Fresh read-only preflight: linked HEGON registry has30 matching historical versions through20261028000000; only local20261028010000 remains pending. Bleach still revision8 and accepted TMDB snapshot80ae912f-ba80-4597-82e8-3dff90ad2afd, matching the migration guards. No dry-run/push/repair/manual production SQL was executed.

Per AGENTS.md §8, owner runs the normal linked push:

```sh
npx supabase@2.121.0 db push --linked
```

Then read-only verify registry, one new YEAR-only assertion/range, revision9, old precise dates/ratings/receipts preserved, all-time166.4h,2023=16h/2026=4h. If any stale guard fires, re-audit; never bypass it. Application/deployment remain separate owner steps.


## Superseding owner decision — Part4 data correction deferred

Owner asks to keep Bleach data unchanged and discuss each S2 part's actual viewing period later.The earlier assertion that20261028010000 was never applied was disproven by the read-only registry/body/row audit. At the time of deferral it and the dedicated test were archived under ignored scripts/.shots/deferred-bleach-part4-viewing-year.sql and deferred-test-watching-bleach-part4.mjs. The runner option is removed and schema-reference watermark restored to applied20261028000000. The preceding Owner application section is historical and MUST NOT be executed. Generic rated-scope application changes stay in scope; no production rollback/write. Reopen only after owner-approved part-by-part mapping, not automatically with2E.
