# Detail alignment with preserved hover - 2026-11-02

Owner confirms BOTH requirements: aligned left/right visual edges and consistent vertical spacing, while preserving the accepted card-scale hover. No badge redesign or carousel-hover rollback is reopened.

## Changes

- All four detail rail headers reserve36px. Reproduction found Seasons header measured36px because of its SelectTrigger, while Best Episodes/Cast/More Like This measured32px. Their title-to-image rhythm is now consistent, including skeleton headers.
- Existing overscan/padding remains for transformed edge cards. Shared watching-detail-rail CSS clips its unused gutters at rest, so partially visible trailing cards stop on the same visual column boundary.
- A small delegated detailRailHover handler opens only the relevant edge during hover. It uses current viewport/card geometry, so it works after paging as well as for the actual first/last item. No per-item subscriptions, requests or React state; one shared implementation for all four sections.
- Clip transition follows the existing300ms image scale and respects reduced motion. Cast & Crew shares the same visual bounds. Poster tokens/sizes, image quality and natural-end navigation remain unchanged.

## Verification

47 targeted tests pass (rail navigation, review UI, retained interaction regressions), TypeScript and affected lint pass. DS coverage regenerated (444 existing off-system hits); diff check passes.

Actual-component browser fixture passes390/768/1024/1440px with no overflow or JS errors. Measured all four headers36px, header-bottom to first-image gap18px, section gap20px below lg and24px from lg. First posters remain aligned with titles (x16 mobile/tablet portrait, x32 landscape/desktop in fixture).

Paint/hit-testing verifies that expanded first cards remain visible beyond their original frame, unused opposite gutter stays clipped, middle-page leading cards still expand after horizontal navigation, and final cards can expand at the trailing edge. Screenshots inspected. These are local deterministic component checks; no new authenticated production comparison is claimed.

No production/data/schema changes, deployment, staging, commit or push. Independent For You add/accessibility/cache/Stats fixes remain preserved. Owner visual review is next.
