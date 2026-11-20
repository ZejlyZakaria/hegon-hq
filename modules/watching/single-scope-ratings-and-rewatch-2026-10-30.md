# Single-scope opinions, nested parts and rewatch polish

Status: locally implemented and validated; **NOT remotely applied or deployed**.
Owner approved this block after the discussion of global/S1 equivalence, Bleach part ratings and duplicated rewatch entry points. Watching Waves1/2A/2B/2C/2D/2E remain locked. This is a subsequent scoped fix, not a reopened reconciliation or Python workstream.

## Final business contract

- A fully seen work with exactly one proven season/cour can expose the same opinion globally and on that scope. The accepted TMDB catalogue must have complete aired/announced membership; anime also requires accepted AniList mapping proving there are no independent additional parts. Unknown evidence never proves equivalence.
- Explicit edits and clears update both compatibility projections in the existing transaction. The append-only scoped opinion ledger retains old assertions; it supplies the scope rather than creating two analytical votes. Existing divergent personal scores are not backfilled or rewritten by the migration.
- Once a new season/part makes those scopes different, scoped edits leave the global opinion independent. Previously frozen scoped membership is retained. No automatic average or copied score for newly released content.
- An explicit scoped edit retains its existing legacy annual attribution when that season/cour has a usable stored viewing year. This does not manufacture a dated canonical assertion, a month/day, or any consumption hours. Independently dated first-view history still wins; retracted completions remain excluded.
- Parts use stable AniList identities and frozen episode membership, not cour ordinals alone. Existing season/part overlap exclusion in annual ratings stays in force. Part ratings cannot recover unknown viewing years; Bleach part-year reconciliation is still excluded.

## Bleach diagnosis

Dedicated remote read-only query found an accepted TMDB head for `tv:30984`, but **no AniList head**. Its cached display mapping alone does not satisfy the existing scope-rating command's membership/snapshot requirements. This explains the unavailable controls without weakening the write guard.

The anime robot now prioritizes missing accepted evidence independently of a recent legacy cache, with a retry cooldown for unsuccessful bootstrap attempts. Accepted mappings keep their normal refresh schedule. No browser-to-provider fetch and no automatic acceptance of an arbitrary cached mapping.

Local Bleach-shaped PostgreSQL fixture (366 + 50 episodes) reproduces missing mapping, then accepts five stable cours and independently rates the four nested S2 parts, including Part4 flat407–416 / S2EP41–50. No personal dates or hours are created. Actual remote repair remains dependent on owner deployment and a successful ordinary robot sync; this task did not invoke or deploy it.

## Rewatch UX

One visible `Log rewatch` action for finished/caught-up titles; the partial-progress action uses the same name. Removed duplicate past-rewatch menu actions and unused CalendarPlus import. All open the existing shared form, defaulting to today and allowing a past/unknown date. Individual history corrections remain available.

The form uses shared Button controls, compact selection choices and a right-aligned primary save / secondary cancel. Mobile and coarse-pointer tablet targets retain44px minimum; desktop actions are32px. Short episode-count labels avoid clipping in a narrow desktop card. Full-show frozen membership, optional seasons and date precision rules are unchanged.

## Files in this block

- `supabase/migrations/20261029040000_watching_single_scope_rating.sql`: prospective replacement of existing context/scoped/personal commands and one private `whole_rating_scope` helper. No backfill. Applied migrations left intact.
- `supabase/checks/expected-objects.json`: **planned post-migration** watermark20261029040000 and functions51 (+1); other counts unchanged82/867/226/279/20. Remote is not claimed to have this watermark or count yet.
- `supabase/functions/watching-anime-cours-sync/index.ts`, `_shared/provider-refresh.ts`: missing-head bootstrap.
- `src/app/(main)/perso/watching/[id]/page.tsx`, `SeasonPanel.tsx`: stable part lookup, explicit awaiting-sync copy.
- `MyTake.tsx`: adopt the server's revised rating projection without overwriting a queued unsaved draft.
- `StatusCard.tsx`, `RewatchForm.tsx`: unified entry and design-system controls.
- `lib/history-analytics.ts`: preserve usable existing annual attribution on explicit scoped edits.
- Tests: `provider-ratings.test.ts`, `provider-refresh.test.ts`, `history-consumption.test.ts`, `review-ui.test.ts`, `scripts/test-watching-scope-equivalence.mjs`; runner `scripts/test-watching-milestone-receipts.mjs --scope-equivalence`.
- Design-system coverage regenerated. Earlier image/Kuroko files and unrelated `debug.log` are pre-existing work, not introduced by this block.

## Validation and limits

-212 targeted Vitest tests /8 files pass, covering shared analytics, presentation, scope guards and rewatch contracts.
- Disposable loopback PostgreSQL rehearsal passes: migration changes no existing personal rows; measured delta is one function only; bidirectional season/cour edits, clearing, delayed retries/payload mismatch, growth separation, four nested parts, immutable old opinions, private helper and existing RLS/provider/backwards-correction contracts.
- TypeScript, affected-file ESLint, Deno cached typecheck, schema-reference gate, Knip CI gate and diff checks pass. Knip retains its three existing configuration hints.
- Actual form JSX/shared controls/CSS rendered in an isolated browser fixture at390/768/1024/1440; no page overflow, compact save/cancel, screenshots read. This is not an authenticated end-to-end production test or a provider-sync success claim.
- No production mutation, application/function deployment, staging, commit or push. No personal Bleach redistribution.

## Owner rollout still required

Before rollout, confirm linked HEGON project and exact pending set (expected new pending version20261029040000; do not replay already applied Kuroko versions). Owner applies the normal linked migration push and deploys `watching-anime-cours-sync`. Verify registry, counts, unchanged personal rows and successful accepted mapping before claiming the Bleach controls work remotely. No manual SQL or migration repair. The application rollout is separate and remains owner-controlled.
