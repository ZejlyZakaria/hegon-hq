# Watching accepted consumption — 2026-10-22

Owner approved implementation after the business audit (`business-discussion-2026-10-22.md`). Targeted corrections only; no final Wave 2D review or general Wave 2E work. No Git staging/commit/push, remote mutation, application deployment or edge deployment.

## Business contract

One accepted consumption ledger drives Stats, annual artwork, achievement explanations/progress and server achievement eligibility. Usable owner-accepted legacy consumption and stored calendar dates count; YEAR-only/unknown dates never invent monthly activity. Canonical first-watch coverage replaces only matching legacy coverage, including tombstones. Rewatches remain independent frozen events. An unresolved legacy rewatch never borrows today's runtime or catalogue.

An annual TV/anime row wears a season poster only when its actual contributions resolve to one display season. Multiple or ambiguous scopes use the global poster. The same resolution supplies the label, Top Picks, Hours breakdown and Wrapped. Recorded AniList cour identities take precedence over later episode offsets. No blanket promotion of a global opinion into a confirmed S1 opinion. Annual rating and Top Picks qualification rules are unchanged.

Legacy estimates must not expand when another season appears or a runtime changes. Migration `20261022000000_watching_accepted_consumption.sql` captures existing consumed, non-reference rows' storage coverage, storage offsets, runtime, fallback year and prior completion in one immutable `media_items.legacy_consumption` JSON column. This is an estimate, not new confirmed viewing history. Current explicit year maps and canonical assertions retain precedence. Explicit personal progress reset can reduce a legacy contribution. New canonical viewing supplies future consumption. Old unknowable scope/duration is not fabricated.

The server and UI now use the same achievement evaluator. Long Hauler uses frozen FINISHED_TITLE membership or accepted legacy finished coverage; partial activity alone cannot finish a show. Ever-Present uses exactly the accepted Activity months, including rewatches. Live opinion badges remain live. Permanent receipts are still server-only, immutable and first-observed-now; no inferred historical unlock date. Existing receipts are not changed. Goals' explicit-history completion policy is retained separately; no frozen Goals files changed in this block.

## Loading, assets and cleanup

Existing receipts are read with one small owner/org-filtered service query before a separate Watching-only observation. An existing Marathoner receipt remains visible during an observation and after an observation failure; old cached empty observations cannot erase it. Receipt data is memoized so a React render does not restart achievement calculation unnecessarily. Goals synchronization remains on its bridge/Goals paths, outside the Stats badge dependency. Strict Goal calculations are lazy and are not executed for a Watching-only observation.

The receipt SQL projection now selects only fields used by the evaluator instead of descriptions, casts and unrelated personal payload. On the same 513-row audit copy, serialized source size changes from 7,367,302 to 712,227 bytes (about 90.3% less). This is a payload measurement, not a production end-to-end latency claim.

Compact MediaRow posters use native 1x/2x/3x source selection: w92 at 40 CSS px on 1x/2x, w154 on 3x. Lazy loading and URL caching stay intact; custom and AniList URLs pass through. Earlier three-CDN-sample measurements put the incremental w154 cost at roughly 4.5–19 kB per image. There is no blanket w500/original upgrade or Vercel optimizer activation.

Removed the duplicate strict achievement counters/overrides, the raw-storage historical label helper, the no-history Stats aggregation branch, the mutable legacy rewatch runtime path, and the second calendar grouping pass. There is one contribution ledger, one calendar grouping and one achievement rule evaluator.

## Manifest for this correction block

- `src/modules/watching/lib/legacy-consumption.ts` (new), `history-analytics.ts`, `milestone-receipts.ts`, `tmdb-image.ts`
- `src/modules/watching/components/stats/{computeStats.ts,achievements.ts,StatsPage.tsx,AchievementPanel.tsx}`
- `src/modules/watching/components/shared/MediaRow.tsx`
- `src/modules/watching/hooks/useAchievementReceipts.ts`, `src/modules/watching/{service.ts,receipt-actions.ts}`
- `src/modules/watching/lib/{accepted-consumption.test.ts,milestone-receipts.test.ts,stats-org.test.ts}`, `src/modules/watching/components/stats/computeStats.test.ts`
- `scripts/test-watching-accepted-consumption.mjs` (new), optional flag in `scripts/test-watching-milestone-receipts.mjs`
- `supabase/migrations/20261022000000_watching_accepted_consumption.sql` (new), `supabase/checks/expected-objects.json`
- `src/modules/styleguide/coverage.generated.json`, this HQ report/decision/current-state update

The worktree already contains earlier Waves/rewatch/Goals/edge changes. Those are preserved and are not claimed as changes made by this block.

## Executed validation

- 218 targeted tests /11 files pass: accepted consumption, receipts/loading/org isolation, Stats, historical consumption, actual panel rendering/UI contracts, image sizes, rewatch freezing, provider ratings and chart domains.
- TypeScript `--noEmit --incremental false` passes; affected-file ESLint with zero warnings, Knip files/dependencies/unlisted/unresolved, design-system coverage, schema-reference and diff checks pass.
- Disposable PostgreSQL16 rehearsal passes prior history/rewatch/receipt RLS, stale/retry/atomicity and independent-connection concurrency contracts, then the new migration. Exactly +1 column, no new table/policy/function/index/cron. Full-row fingerprints excluding the added column and history/range/rewatch/receipt fingerprints remain unchanged. Direct authenticated writes, RPC patch forgery and robot updates of the snapshot are rejected. The private source/writer ACLs and source-hash freshness guard remain enforced. No-catalogue legacy progress is retained.
- Replayed the same migration against a local import of the 513 personal audit rows and canonical history. SQL snapshots match offline expected coverage. Every available year plus all-time retains identical counts, hours, activity and rating distribution. Total stays3729.9hours;133 accepted months. Black Clover2022 is170episodes/68hours/S1 artwork/rating8.5;2026 is3episodes/1.2hours/S2 artwork with no historical rating leak. Hannibal annual8.75 remains exact. UI/server values: Long Hauler500/500, Ever-Present133/133, Marathoner3729.9/3729.9.
- Real React components rendered in a local isolated browser fixture at390/768/1024/1440px, using actual Stats/evaluator/MediaRow/AchievementGrid/AchievementPanel and light Next image/link adapters. Posters really downloaded from CDN; Chromium selectsw154 atDPR3 andw92 atDPR2. No horizontal overflow, panels settle inside the viewport, Escape/selection work, zero page errors. Captures read after animations. No authenticated app/observer/production route loaded.
- Ignored artifacts: `scripts/.shots/watching-accepted-verification.json`, `watching-business-browser-results.json`, `watching-business-*.png`, local rehearsal/fixture scripts. The audit copy is not a production data repair script.

## Activation and remaining boundary

Fresh dedicated read-only registry comparison on linked HEGON `femvhonlpafdajyamvcu`:28 remote versions match the first28 local versions; local version29 (`20261022000000`) is the sole pending migration. No push/repair/manual SQL performed. Watchdog reference is prepared for the new migration:81tables,857columns,225policies,49functions,277indexes,20crons.

Owner applies the migration before running/deploying the updated Stats selector:

```powershell
npx supabase db push --linked --dry-run
npx supabase db push --linked
```

The dry run must list only `20261022000000_watching_accepted_consumption.sql`. No edge deployment is needed for this correction. After application, verify registry/schema/unchanged personal fields read-only, then refresh localhost. The next normal authenticated observation can create newly eligible permanent receipts; the migration itself does not backfill them. Remote activation and post-application verification remain pending. Local implementation validated; do not call the new remote schema applied or the Wave2D final review complete.
