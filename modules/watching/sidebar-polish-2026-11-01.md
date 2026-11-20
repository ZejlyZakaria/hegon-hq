# Sidebar polish and contribution consistency - 2026-11-01

Owner approved the compact spacing, always-visible nested parts, first-view KPI cells, plain consistent section headings, compact Top10/InLists, reduced Details footer padding and truthful contributing goals.

## Local implementation

StatusCard/WhereToWatch use tighter12px group spacing and8px footer separation while preserving44px touch controls. ViewingHistory renders date/duration/episodes as one grouped KPI surface (date spans the mobile first row; films omit episodes), and nested seasons/parts are permanently visible with no accordion arrow. No viewing facts, dates or runtime sources changed.

Top10 shows rank plus Change, with an on-demand5-column editor and explicit Remove from Top10. Rank writes no longer change favorite; ranked reads no longer filter favorite=true, so removing a heart does not silently hide an independently ranked title. Existing revision-guarded two-write swap remains; it is not claimed atomic. No new RPC/migration. InLists now uses Panel and compact thumbnail/name rows with always-visible accessible removal, Manage membership/create and See all; old poster carousel/scroll implementation removed. Accolades heading has the same text-title style and no trophy in the title (award details keep their separate semantics). Details has an explicit small footer padding, eliminating cumulative bottom spacing. List skeleton replaced with a compact card silhouette.

ContributingToGoals uses the existing cached media history and lens, not only watched_at. createLiveGoalEvaluator is extracted from receiptMetrics.liveGoalValue and shared by the live goal counter and detail membership. It retains existing finished-title eligibility, canonical rewatches and type/year semantics. No Goals module files changed; strict goal receipts unchanged. Completion checkmark appears only when current>=positive target; progress is clamped. Unknown or failed history does not assert contributions.

## Validation

101 targeted tests pass in live-goal-metrics, goal-contribution, viewing-timeline, review-ui, milestone-receipts and service. New regressions: canonical year versus stale global timestamp; type mismatch; once-per-title canonical rewatches; unfinished title rejection; completion/retraction; ranked query without a favorite filter. Saved516-item2026-10-29 snapshot replay matches the former live evaluator for20key/period combinations. This is historical replay, not a fresh production read.

Browser fixture at390/768/1024/1440 renders actual StatusCard, ViewingHistory, TopTenRank, InList, Accolades, QuickStats, MediaDetails and ContributingToGoals with deterministic data and mocked writes. Verifies no overflow/errors; KPI cells; permanently visible parts; four concise GOT rewatches; rank removal payload excludes favorite; list manage/removal remains accessible on touch; menu-only rewatch and distinct direct episode versus inline correction callbacks. Screenshots inspected. Local authenticated full-page E2E is not claimed. TypeScript, affected lint, KnipCI and regenerated DS coverage checked; whitespace gate checked. Owner visual acceptance remains pending.

No production data/schema mutation, migration application, deployment or Git mutation. Existing unrelated mixed worktree and pending scope-rating rollout remain separate.
