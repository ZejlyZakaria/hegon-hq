# List table image quality and focused code review - 2026-11-02

## Applied

ListDetail table poster retained --poster-xs (40x60 CSS px), but previously declared 28px and used only a fixed w92 source. It now reuses tmdbImageSrcSet at40px in picture/source, matching MediaRow, with a40px fallback request. Native density selection: w92 at1x/2x, w154 at3x. Non-TMDB custom artwork remains untouched; lazy loading retained. No changes to list collages or detail sidebar, data, schemas, production, Git staging/commits/deploys.

## Verification

9 existing image-helper tests, TypeScript and affected-file lint pass. DS coverage regenerated (444 existing off-system hits); diff check passes. Source-extracted actual table poster JSX rendered in Chromium at390/768/1024/1440 and DPR1/2/3: all12 source-selection/frame-size checks pass. Local routed image fixture verifies markup/source selection, not photographic quality of the user's actual uploaded/TMDB file; screenshot read.

Knip CI passes (files/dependencies/unlisted/unresolved). Additional exports/types scan reports one genuinely unused Watching helper, hasEstimatedEpisodeCounts in episode-counts.ts. LegacyConsumption is a false positive from inline import types: types.ts and service.ts use it, so do not remove. No dead-code-free claim for the entire module.

## Recommendations, not applied

- Cache consistency: useAddToList/useRemoveFromList invalidate only listsForMedia, while equivalent list-page add operations invalidate listItems and list thumbnail queries as well. This can leave other mounted/cached list surfaces stale. Unify mutation invalidation with regression coverage when the owner approves the next list reliability block.
- Accessibility: table poster and title navigation currently use div/onClick. Replace with accessible links/buttons and verify keyboard activation/focus when the owner approves table UX improvements.
- Remove unused hasEstimatedEpisodeCounts when the owner opens a focused cleanup; retain the used episodeCountsForTime implementation.
- No fresh warmed A/B request/render/bandwidth measurements were taken. Cannot certify all Watching performance as optimal. Batch anime views, shared history query keys and the central status writer are present in the inspected paths.

Earlier pending migration/function rollouts and empty-database reconstruction debt remain separate. No authenticated live-page or production-data verification in this task.
