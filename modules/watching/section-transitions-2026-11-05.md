# Section images and status transitions — 2026-11-05

Owner-approved UI block implemented locally, after the owner reported successful application of migration 20261104000000. No fresh remote registry/schema read in this UI task; no production write, new migration, deploy, staging, commit or push. Updates page remains unimplemented.

## Changes

- StatusCard retains one mounted content shell: removed the three independent keyed presence crossfades. One chip is rendered at a time, controls are not duplicated. ResizeObserver measures natural content and animates only container height for 200ms, with clipping and reduced-motion support; text is never scaled. Existing optimistic/server/rollback data path stays intact.
- SectionImageReveal coordinates initial native image loads and decoding per section, not per page. It selects the visible responsive tree and horizontally visible images, warms one viewport ahead only near the vertical viewport, and reveals once (150ms; cached immediate). Failure/decode rejection is tolerated; a 2500ms deadline prevents an indefinitely blocked section. After the deadline, still-missing artwork can arrive later; this is the explicit exception to grouped reveal.
- Once revealed, cached content stays visible during rerenders/refetch; scroll/resize preload never resets the section. Hidden responsive image trees do not receive eager scheduling. No duplicate detached image fetch/source selection or provider requests added.
- MediaCarousel, For You and Don't Miss share that policy. Removed the carousel's copied props state and per-card entrance stagger, plus obsolete eager props and individual fades on these surfaces. Standalone image reveal remains used elsewhere.
- CSS now owns the initial 3/5 desktop widths, removing the effect-driven initial 5-to-3 geometry change. Existing poster tokens and hover behavior preserved.
- Skeleton headers use actual SectionHeader typography, wrapping, spacing and toolbar geometry. Desktop carousel padding matches the real rail; For You mobile padding matches its skeleton. All three loading routes now identify their media type and mirror the section order. Data-dependent absent sections/counts remain unknowable until queries resolve; no claim of a pixel-exact whole page for unknown data.
- For You waits for initial ownership evidence as well as recommendations/dismissals, avoiding transient already-owned cards.

## Validation

- 171 targeted tests / 5 files PASS (review UI, series state, watch status, activity bucket, correction stance).
- New executable browser regression: `node scripts/test-watching-section-reveal.mjs`. Actual Carousel/ForYou/DontMiss/StatusCard components with deterministic read-only hook fixtures and controlled image transport. Four widths: 390/768/1024 touch, 1440 mouse. PASS: slow images, independent sections, cache remount, rerender retained content, broken image, deadline, multiple state transitions and rollback-shaped return, reduced motion, no runtime errors or horizontal page overflow.
- Actual carousel card width, first-card offset and total section height match skeleton within 1px at all four widths; measured height difference at 1024 is 0.016px, others zero. Existing diagnostic had two concurrent chips; current frame traces have exactly one through ongoing/caught/paused/completed transitions and many intermediate height values rather than a single height jump. Screenshots at mobile/desktop inspected.
- TypeScript, affected-file lint, Knip CI, schema reference, regenerated DS coverage, git diff whitespace PASS. Coverage adds the new Watching component (221 files), existing 442 off-system hits unchanged. Knip retains only three existing config hints.
- No production build or authenticated live-network end-to-end action run. The exact intermittent owner flash under production timing is not certified absent; owner localhost visual check remains appropriate. No broad performance improvement claim.

## Files in this block

- src/modules/watching/components/shared/SectionImageReveal.tsx (new)
- src/modules/watching/components/shared/{MediaCarousel,WatchingSkeletons}.tsx
- src/modules/watching/components/sections/{DontMiss,ForYou,InProgress,RecentlyWatched,TopTen,WaitingFor,WantToWatch}SectionClient.tsx
- src/modules/watching/components/detail/StatusCard.tsx (mixed with the previous tracking-intent block)
- src/modules/watching/hooks/useImageReveal.ts (comment clarifies ownership)
- src/app/(main)/perso/watching/{movies,tv-shows,animes}/loading.tsx
- scripts/test-watching-section-reveal.mjs (new; creates ignored .shots fixtures/results)
- src/modules/styleguide/coverage.generated.json
- supabase/checks/expected-objects.json (owner-reported migration status comment only)

Other prior business/studios changes remain in the working tree. debug.log is unrelated and excluded. HQ is a separate repository.
