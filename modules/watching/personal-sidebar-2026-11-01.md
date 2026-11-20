# Personal sidebar and viewing details ? 2026-11-01

Owner authorized implementation after rejecting the earlier generated mockup. Scope: personal viewing panel, rewatch entry, provider placement, right-column order and correctness of Quick Stats/Details. No production data change or rollout.

## Implemented

ViewingHistory uses a title/artwork summary, first-view period and known duration, and concise independent rewatch rows. Ordinary season breakdown is absent; only meaningful nested-part summaries display it. Existing edit/remove actions and audit semantics are retained. The shared SlidingPanel is 480px on larger screens and full width on mobile. Log rewatch exists only in the StatusCard menu, not as the dominant action or a viewing-panel toolbar action. Progress correction is an inline draft with explicit Save/Cancel, preserving the correction callback; next-episode viewing stays direct.

Provider logos are a compact strip inside the card footer, with accessible names and region hints. Where a link is available it opens the supplied availability page (not an invented provider deep link). Removed the separate provider-card variant. Rail order: personal card, Quick Stats, Top10, InList, Accolades, Details, AnimeThemes, contributing goals. Skeleton reduces the former six-row QuickStats placeholder and reflects the collection-before-catalogue order.

Quick Stats now consumes buildViewingSummary and its existing accepted consumption ledger. Removed independent runtime multiplication, approximate rounded hours, legacy Finished/Started dates, duplicate season-score mean and rank. First-view episode counts exclude repeats; time includes frozen recorded rewatches. Unknown duration is explicitly incomplete, never multiplied by today's runtime. History loading/error does not present fallback totals as final. Details separates aired and announced episodes, prefers per-season catalogue evidence to a stale scalar, labels broadcast years explicitly, formats valid calendar dates en-GB and omits N/A. Secondary metadata is behind More details. No dates inferred or rewritten.

## Validation

137 targeted tests in viewing-timeline, catalogue-facts, history-consumption, review-ui and completion-date pass. Includes frozen rewatch runtime despite runtime change, unknown-duration partial totals, announced versus aired counts and invalid release dates. Browser fixture renders actual StatusCard, ViewingHistory, QuickStats, MediaDetails, shared controls and CSS with deterministic mocked data/mutation callbacks at390/768/1024/1440. All four widths pass: no overflow or page errors, four GOT rewatches without season breakdown or Log rewatch in the panel, grouped Bleach Part4 with4h, no empty history for unstarted/film cases, one rewatch form via menu, direct advance and inline correction with distinct callbacks. Screenshots read. These are isolated component checks, not authenticated production-page verification. Prior saved/fixture dates do not establish new personal facts.

Knip CI and affected-file lint pass; generated design-system coverage refreshed. No new migration, remote query/write, deployment or Git mutation in this UI block. Earlier pending scoped-rating migration and mixed worktree changes remain separate. Owner visual acceptance is still pending.
