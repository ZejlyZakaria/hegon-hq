# Best Episodes partial trailing card - 2026-11-02

Owner clarifies the screenshot's S3.E4 is partially visible at the right viewport edge, not the final record in the list. Only this partial-card case must stay clipped; fully visible cards, including the real final card, retain outward zoom. Seasons is accepted and untouched.

## Fix

The delegated rail geometry distinguishes a card extending past the content boundary from a fully visible trailing card. A scoped Best Episodes CSS rule holds the right clip boundary for the partial card, covering image AND caption. It applies immediately, avoiding leakage while transitioning from another hovered card. Other rail styles and handlers' visual behavior are unchanged.

Removed the prior incorrect right-anchored artwork rule, its data-rail-artwork marker and persistent per-card anchor metadata. Fully visible final episode cards now use the same centred zoom as other full cards. Dimensions and prior alignment/vertical rhythm remain unchanged. Files: globals.css, lib/detail-rail-hover.ts, detail/Episodes.tsx.

## Reproduction and validation

Actual JSX/CSS fixture at1440px reproduced both image and caption leaking beyond the boundary on the fourth partially visible card at scroll0 and fifth at scroll276. The old final-record-only check missed this. Crucially, use mouse coordinates inside the visible card fragment: locator.hover() can auto-scroll a partial card fully into view and invalidate the scenario.

After:390/768/1024/1440px, both initial and scrolled partial trailing cards show no image/caption beyond the boundary across24 animation frames. Real final cards still expand outward; first-card left expansion is preserved. Browser mutations are mocked, no production writes. Screenshots inspected. Evidence: scripts/.shots/partial-episode-browser.json and partial-episode-{width}-{position}.png (ignored).

47 targeted tests, TypeScript, affected lint, regenerated DS coverage and diff checks pass. No production/schema/data changes, deploy, staging, commit or push. This supersedes the prior right-centre final-card anchor from exact-image-edge-2026-11-02.md.
