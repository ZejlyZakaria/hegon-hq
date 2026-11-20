# Detail rail navigation and broadcast years - 2026-11-01

Owner requests fixes for cropped leading cards after repeated season/Best Episodes navigation, section alignment and Bleach Broadcast years, plus a critical review of the right sidebar.

## Root causes and local corrections

Season and episode arrows scrolled fixed320/560px rather than actual card boundaries. Shared useAlignedRail now measures the real cards and viewport, navigates by whole-card pages, keeps rapid-click targets, remeasures on resize and disables boundary arrows. A measured trailing flex pseudo-element makes the final scroll stop align without changing percentage-based mobile card widths. The initial padding-based experiment caused a percentage-width measurement feedback and was replaced before completion. Native touch scrolling remains available. Both section headers reserve32px and rails use matching6px vertical headroom, preserving their established horizontal gutters and still image quality.

Bleach's OMDb yearRange describes the original2004-2012 run. The saved personal snapshot's catalogue includes S1 air/end2004/2012 and S2 air/end2022/2026, aired366/50. catalogueBroadcastYears now prefers stored actually-aired season dates across the work, labels returning works start-present, and falls back to external catalogue text only without sufficient local evidence. No use of personal watched dates or guessed part dates; no stored metadata rewrite. Saved snapshot is historical evidence, not a fresh provider lookup. The actual Details component regression deliberately supplies stale OMDb2004-2012 plus these season dates and verifies2004-2026.

## Validation

44 targeted tests pass across rail-navigation, catalogue-facts and review-ui. TypeScript and affected-file lint pass; DS coverage regenerated and git diff --check passes. Isolated native-browser tests at390/768/1024/1440 render actual SeasonHistoryStrip, Episodes and MediaDetails with deterministic fixtures, exercise repeated next/previous/end/rapid clicks, check each settled leading edge and heading alignment, confirm corrected broadcast years and no page overflow or JS errors. Screenshots inspected. Fixtures use12seasons to reproduce overflow; this is not a claim about Bleach's season count. No authenticated live-page verification, provider query, production mutation, migration, Git mutation or deploy.

## Critical review for owner discussion only

Keep the status card as the sole daily control; reduce technical correction prose. Viewing panel now explains consumption but unknown dates should stay discreet and viewable without creating work for the owner. Provider logos remain subordinate. QuickStats is now a compact ledger summary; keep known/incomplete duration explicit. Top10's permanent10-slot editor is visually heavy and its current rank writer couples removing a rank to removing favorite, with a two-request swap; challenge these semantics before a separate change. InList's large card carousel and hover edits are disproportionate for this narrow column; prefer small membership rows with a touch-accessible manage action. Accolades should remain compact, gold and conditional. Details should prioritize work-wide catalogue evidence over an external sub-work's year span. Music belongs low and keeps its controls. ContributingToGoals currently matches watched posture/global watched_at, so season/rewatch-specific contributions need an evidence review before stronger claims; its always-present checkmark can suggest completion even when the goal is incomplete. No additional business or UI changes applied to these reviewed modules in this block.


## Owner screenshot follow-up: cross-section right edge

Owner screenshot reveals Best Episodes extends closer to the right column than Cast & Crew. Earlier verification compared Seasons/Best Episodes leading cards, not their viewport edges against Cast & Crew. Root cause: both used sm:-mx-1.5 with compensating padding (6px viewport bleed on each side), while Cast & Crew uses sm:mx-0 sm:px-0. Removed that tablet/desktop bleed in both affected rails; mobile edge-to-edge gutters remain identical to Cast & Crew. No card size, image quality or business-rule change. Browser fixture now includes actual CastCrew and checks all three rail left/right bounds together at all four widths, in addition to repeated navigation.
