# Watching — close point 1: viewing periods, nested parts and scoped opinions

Owner approved implementing the discussed rules together before final Wave 2D validation. This is a targeted amendment; no final 2D review, Wave 2E, Python analytics workstream, commit, push or deployment starts here.

## Product contract

- First-viewing hours follow the actual recorded viewing period. A December 2026 / January 2027 season contributes each episode's frozen hours to its own viewing year. Finish is a separate milestone, not another watch or a transfer of all earlier hours. Stats' existing consumed-title counts remain activity counts, not a new finished-title metric.
- Existing legacy year/calendar assignments remain accepted estimates where no finer personal evidence exists. Release dates cannot reconstruct personal viewing dates. No automatic redistribution of Bleach's old 2023 hours to its parts' release years.
- TMDB seasons/episode identities remain the storage coordinates. Existing single-season anime overlays remain unchanged. Multi-season anime can carry nested AniList parts without being recut into new display seasons or moving the old year/rating maps.
- Parts come from accepted AniList identities/boundaries, not airing gaps. Complete, contiguous, uniquely identified ranges must match the TMDB total and fit inside its season boundaries. Unknown/mismatched/crossing mappings fall back to ordinary season/episode display. This is conservative use of accepted provider metadata, not a claim that matching totals alone prove all external catalogue semantics.
- A rating covers its recorded membership. On growth the old season score moves behind **Previous rating**; it is not the displayed current season score, even after catching up. Confirmation/revision remains explicit. A completed part's own unchanged scope retains its independent rating. Unseen/unaired parts cannot be rated. Unknown current availability disables rating rather than manufacturing coverage.
- Annual scoring does not count a season opinion and an overlapping part opinion as two votes; qualified part opinions take precedence over the overlapping aggregate. Existing global title opinion remains independent and is never copied into parts or cleared by growth. Hannibal's annual 8.75 and Black Clover's historical-rating isolation remain covered.

## Implementation

`lib/season-history.ts` owns verified nested boundaries, first-view season period labels, and precise episode-range presentation. `buildMediaView` exposes optional `ViewSeason.parts` while keeping multi-season anime in their existing coordinate space. Part rating context checks live alongside season checks in `scope-ratings.ts`; both UI doors submit the same existing snapshot/revision-bound service/RPC command.

The detail page and expanded audit history share `useMediaHistory` and one query key, with service-layer paginated/batched reads. No hook per season/part and no provider request during render. Season strip/panel can display 2023–2026 without rewriting the original season year. Canonical period-only Stats entries can say `Season 2 · EP49–50`, or name a verified exact part; sparse evidence shows an episode count. Legacy aggregate counts never pretend to establish an exact interval.

`history-analytics.ts` retains storage-season presentation for frozen cour ranges on non-overlaid titles and excludes overlapping aggregate rating votes. The existing first-view deduplication, overlap subtraction, immutable duration and rewatch event rules remain the consumption authority.

The existing `watching-anime-cours-sync` worker pages requests in50-ID batches and now rejects incomplete/duplicate/unexpected AniList response membership, unknown ordering dates, tied ordering dates and unknown interior boundaries. Failures preserve the accepted cache and are reported per work. No new worker, cron, schema object or migration is required for this amendment. The previously prepared Bleach migration remains part of the prior correction block.

## Remote state and real-data replay

Read-only management endpoint confirms migration `20261028000000` is applied (then `20261022000000`, `20261021020000`). Bleach is revision 8, S2E50, watched_at 2026-10-28 noon projection; original season years 2020/2023 and legacy calendar 2023-12-31 are retained. New local backup: ignored `scripts/.shots/watching-point1-current.json`, 8 viewing assertions and 4 ratings. No remote write or observer invocation.

Actual accepted cache gives S2 parts EP1–13, EP14–26, EP27–40, EP41–50. Fresh replay produces S1=2020 / S2=2023–2026, storage coordinates unchanged. Against the previous repaired real-data fixture, annual/all-time hours, consumed-title counts, calendar activity, monthly totals, rating distributions/average and unknown counters match exactly:

| Period | Hours | Current display |
|---|---:|---|
| All-time | 166.4 | Whole title |
| 2020 | 146.4 | Season 1 |
| 2023 | 19.2 | Season 2 (legacy aggregate) |
| 2026 | 0.8 | Season 2 · EP49–50 |

Those numbers reflect available personal evidence; they do not claim the whole TYBW season was watched in 2023.

## Validation and limits

Targeted Vitest coverage includes season parts, periods across New Year, mixed legacy/history, cour rewatches on storage seasons, frozen opinions/growth, overlapping votes, incomplete worker responses, existing completion/date safety, service commands, annual scoring and achievements. Local PostgreSQL rehearsal executes the real provider/rating/history functions, including nested S2 part membership, unaired/unseen rejection, scope growth, frozen prior part scores, independent global opinion and rollback, plus the existing RLS/revision/concurrency/retry checks.

Actual SeasonHistoryStrip/SeasonPanel fixture verified at 390/768/1024/1440 pixels: correct period, four actual mapped parts, old score hidden until history expansion, part rating callback and no page error/horizontal overflow. All external traffic blocked; images/transport are fixture substitutes and writes are local callbacks. No authenticated production-backed rating/observer smoke test was performed. Ignored proof: `point1-replay-results.json`, `watching-point1-browser-results.json`, `watching-point1-*.png`.

Owner deployment is still required to activate the new worker guards remotely:

Final local checks: **378 tests / 15 files pass**, TypeScript `--noEmit --incremental false` passes, affected-file ESLint has zero warnings, Deno check passes, Knip passes (three pre-existing configuration hints), schema reference and main/HQ diff checks pass. Design-system coverage regenerated and CRLF preserved. No production build/full suite run in this targeted block; final 2D validation remains separate. Disposable PostgreSQL server stopped after the successful rehearsal.

```sh
npx supabase@2.121.0 functions deploy watching-anime-cours-sync
```

The existing cron will then use the updated worker. No new migration push is needed for this amendment. Local code validation is separate from owner deployment and final 2D review.

## Manifest

- `src/app/(main)/perso/watching/[id]/page.tsx`
- `src/modules/watching/lib/season-history.ts` and `.test.ts` (new)
- `src/modules/watching/lib/media-view.ts`, `scope-ratings.ts`, `history-analytics.ts`
- `src/modules/watching/hooks/useMediaHistory.ts` (new), `hooks/query-keys.ts`
- `src/modules/watching/components/detail/SeasonHistoryStrip.tsx`, `SeasonPanel.tsx`, `RewatchForm.tsx`
- `src/modules/watching/components/stats/computeStats.ts`
- `src/modules/watching/lib/history-consumption.test.ts`, `provider-ratings.test.ts`, `scope-rating-ui.test.ts`, `review-ui.test.ts`, `robot-handlers.test.ts`
- `supabase/functions/watching-anime-cours-sync/index.ts`
- `supabase/tests/watching-nested-parts.sql` (new), `scripts/test-watching-provider-ratings.mjs`
- Generated `src/modules/styleguide/coverage.generated.json`
- This report, `hq/now.md`, Watching `decisions.md` and `architecture.md`.

Previous Bleach completion/date repair dirty files remain intact; their manifest is in `bleach-completion-diagnostic-2026-10-28.md`. Unrelated `debug.log` is excluded.
