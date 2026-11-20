# Responsive artwork actions and bottom mask - 2026-11-02

Owner reports oversized For You +/dismiss circles relative to the poster menu and rejects full-poster status darkening. Fixed locally: shared artwork-action CSS paints24px circles with44px independent touch targets,24px pointer targets, aligned8px visual inset, no target overlap. Add is white/dark ink; dismiss remains dark/white. Existing AddMark writer, pending/done states, keyboard labels and stopped propagation retained; no alternate status writer. AddMark callers share the correction. Existing hover/rail sizes untouched.

Status mask is now a bottom40percent gradient to transparent; top60percent of the artwork is unchanged. SeasonHistoryStrip already limits its score mask to the bottom. Its full-poster Coming soon/ahead-of-position overlays are intentional availability/unwatched indicators, not score/status backgrounds; no season mask change.

Recommendation for discussion: at most one useful personal consumption-status pill on library artwork. Completed default needs no badge. Badge palette remains owner-UNVALIDATED; this correction is not product sign-off.

Validation:41targeted tests pass. Actual ForYouCard/AddMark/LibraryCard/StatusMark/MovieCard browser fixtures at390/768/1024/1440 with both touch and mouse pass;24px painted circles,44/24px targets, equal vertical position, nonoverlapping targets, add/dismiss distinct actions without accidental navigation, bottom-only masks, no overflow/runtime errors. Captures inspected. Generated DS coverage updated. No production mutation, migration, Git mutation or deployment.
