# Watching integrity fixes - 2026-11-17

Status: LOCAL implementation; owner applies migration/deploys. No production write, commit or push.

## Active consumption and deletion

Cause: getWatchingStatsData and receiptMetrics recreated deleted titles from the retained history (synthetic history: identities). History matching by work key also revived orphan consumption after a title was re-added. Frozen runtime was then correctly summed against an incorrectly active identity.

Fix: activeHistoryForItems binds both the current owned media row ID and work key; orphan/null/deleted-row records and ratings never contribute to active Stats, annual calendars, goal metrics or new achievement observations. Legacy orphan rewatches are excluded too. recordsFor uses the same current identity. Remove synthetic rows from service/receipts. History/rating audit rows remain physically unchanged; permanent achievement/Goal completion receipts remain permanent. No retrospective data repair or deletion migration.

Read-only owner/org snapshot: scripts/.shots/watching-snapshot-2026-11-17.json. Old snapshot: stats-ratings-live-2026-11-08.json. Before/after computation artifacts remain ignored local files.

| Scope | Before | After |
|---|---:|---:|
| All-time | 3,870 h 26 | 3,807 h 24 |
| 2026 Shows | 182 h 23 | 164 h 01 |

Removed active contributions: Dark Matter 18 h 22; Godfather of Harlem 36 h 40; The Night Of 8 h (deleted old row; current row is Want to watch). Total 63 h 02. Audit snapshot is byte-for-byte unchanged by projection.

Reconstruction of remembered total: Nov8 3,748 h 47 -> Nov17 uncorrected 3,870 h 26, +121 h 39. The three orphan contributions explain 63 h 02. Dragon Ball Z replaces Dragon Ball Z Kai (+52 h 47 net); owner explicitly confirms Dragon Ball Z is correct on Nov17. Remaining +5 h 50: Oldboy 2 h; MHA Heroes Rising 1 h 44; Mr Robot 1 h 42; Black Clover +24 min. No Dragon Ball personal-data change.

## Rating eligibility and stable controls

Caught-up is not full season completion. A writable rating requires known complete released membership, the full announced season watched and matching stable scope/snapshot. A fully released, watched independent part remains writable inside an unfinished parent season. Unknown evidence blocks a new rating; existing historical opinions are preserved.

SeasonPanel and page command gates enforce the rule. New migration 20261117000000_watching_complete_scope_rating.sql adds one private stable helper and replaces scope_rating_command with the complete-membership guard. Original locks, revision/ownership/RLS/snapshot checks, journal, retry receipts and partial-watched guard remain intact. Clearing an old false opinion is still allowed. No backfill or user-data change on application. Watchdog functions56 ->57; other counts unchanged.

Stars/confirmation controls remain mounted and temporarily disabled during save. Pending selected score is a local UI preview keyed by stable scope identity; canonical assertion IDs/cache are not forged. Other parts keep their control nodes and scores. Failure drops the preview. Four-width real SeasonParts/StarRating browser fixture:40buttons/4groups, zero removed groups, all DOM identities retained during save.

Real Black Clover snapshot and authenticated live local fiche: S2 EP7,7 aired/26 announced; Caught up, Not rated, zero rating radio groups in the Season2 drawer. Screenshot read. No production rating mutation used for verification.

## Back navigation

Cause: Watching layout unconditionally set main.scrollTop=0 on pathname change, overriding browser Back. Next's normal window restoration does not reliably restore this nested main container.

Watching-only hook saves route positions, distinguishes popstate from new navigation, restores before paint/after router focus, and observes real content height while a skeleton is shorter. Deliberate touch/wheel interrupts deferred restoration. WatchingPageWrapper now shows its existing dedicated skeleton while user identity resolves instead of returning an empty surface. No shared auth/layout or other module changes.

Actual owner-session localhost mobile check: before1097px -> fresh detail0px -> Back1097px. After using the proper identity skeleton, no intermediate56px scroll event. Four-format real-page checks PASS:390px1097->0->1097;768px1414->0->1414;1024px820->0->820;1440px747->0->747. No intermediate wrong-position scroll event in the final runs.

## Validation

- Full Vitest:917 tests/61files PASS.
- New regression suites: deletion/re-add/stale-row/orphan rewatch, season/cour completion eligibility, stable pending preview and rollback, nested scroll/skeleton/intent.
- Disposable PostgreSQL: PASS full legacy transaction/provider/rollback/single-scope/stance/tracking/period suites and the new complete-scope migration suite. New migration changes zero personal rows, +1 private function only.
- TypeScript/lint/Knip: PASS.
- DS coverage/schema reference/diff: PASS (generated coverage included).
- Production build: PASS including final shared-gate compile, TypeScript and39static pages. Google Fonts required network access; existing middleware convention notice only.
- Screenshots: mobile Back and saving-star fixtures read; no production mutation used.

## Rollout

Read-only remote migration history verified:41remote versions,42local; sole pending version20261117000000, no remote-only version. Owner applies the new migration, then commits/pushes the code. No Edge Function deployment needed. Old audit rows are intentionally retained. This block does not alter the user-confirmed Dragon Ball Z identity or any other genuine viewing date/progress/rating.
