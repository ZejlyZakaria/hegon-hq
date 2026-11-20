# Exact image edge and Best Episode zoom anchor - 2026-11-02

Owner clarifies the axis is the actual image edge relative to toolbar buttons, not merely the rail rectangle. Posters must reach that axis at rest and may zoom past it. The trailing Best Episode must reach it at rest but keep its right edge fixed while zooming. Preserve the first episode hover and all size tokens.

## Implementation

- Seasons, Best Episodes and More Like This image frames use an inset ring instead of a layout-consuming1px border. This removes the internal image retreat without changing the external144px poster/264px episode frame widths or responsive mobile dimensions.
- Delegated hover geometry stores the card's current trailing-edge identity. Only Best Episode artwork uses right-center transform origin for a trailing card. Retain this anchor on mouse-out, so reverse zoom does not jump back to a centred origin; recompute if hovered again after scrolling. Posters retain centred outward zoom and the first episode retains its existing hover.
- Existing header36px/vertical rhythm, rail clipping/clearance, natural terminal navigation and prior independent fixes remain intact. No badge redesign restored.

## Verification

Actual-component before/after fixture, standard root sizing, desktop1440px: toolbar axis952px. Before: More Like This actual image ended951px inside a952px frame; trailing Best Episode expanded to955.96px. After: actual poster image ends952px at rest and expands to954.88px; Best Episode stays952px during zoom and mouse-out while width grows264->271.92px.

390/768/1024/1440px browser checks pass: image/frame/toolbar alignment at natural rail end (within0.6CSSpx browser scroll rounding), outward poster zoom, fixed trailing Best Episode edge sampled across24 animation frames in each direction, preserved first-card expansion, no horizontal page overflow/JS errors. Actual JSX and CSS with deterministic local image fixtures, not authenticated live-production comparison. Captures inspected. Evidence: scripts/.shots/exact-edge-browser.json and exact-edge-{width}-{section}.png (ignored).

47 targeted tests, TypeScript and affected lint pass. DS coverage regenerated (444 existing off-system hits); diff check passes. No schema/data/production changes, deployment, staging, commit or push. Next: owner checks localhost rendering.
