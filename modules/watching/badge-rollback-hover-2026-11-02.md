# Badge rollback and rail hover clearance - 2026-11-02

Owner rejects the pastel badge experiment and image-only MediaCarousel zoom. Restored globals.css, shared Badge, StatusBadge and Marks exactly from the saved pre-experiment working-tree snapshot. Removed WatchingBadge and restored previous imports/variants/personal StateChip. Carousel badges return to their previous positions/styles and the whole-card hover returns; keyboard title navigation remains. For You retains the direct AddMark action and touch/keyboard access, with its previous whole-card hover.

Preserved independent work: list table image quality, native table keyboard buttons, membership invalidation, unused helper removal and shared all-time Stats memo. Removed only the regression assertion that required the now-rejected unified caught-up appearance.

Detail rail root cause: desktop/tablet horizontal clearance had been removed during alignment (sm:px-0). A centered144px poster at1.04 scale extends2.88px beyond its original edge; a264px still at1.03 extends3.96px. Overflow-x containers clip that transformed area at the first/last edge. Restored6px padding with matching negative margins and scroll padding for Seasons, Best Episodes, More Like This and matching Cast & Crew. Content starts remain aligned with headers; poster size tokens and natural terminal navigation remain unchanged. Versioned reference has the same negative6px/padding6px treatment. This is not a claim that HEAD is the deployed revision.

99 targeted tests, TypeScript, affected lint, Knip CI, regenerated DS coverage and diff checks pass. Local browser fixture verifies four widths and the first-card hover paint beyond the original frame (including hit-testing the expanded area).

Production comparison limit: isolated browser session redirects to /auth. Computer-use inventory found the owner's existing Chrome window, but the next observation was automatically stopped because it could not determine the current Windows browser URL reliably enough to enforce policy. No further Computer Use inputs issued. Therefore no live production hover comparison is claimed.

No production/data/schema changes, deployment, staging, commits or pushes. Owner visual review of the restored localhost hover is next.
