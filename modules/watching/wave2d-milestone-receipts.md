# Watching Wave2D: permanent milestone and Goal receipts

Current status2026-10-28: Wave2D CLOSED for its validated scope; remote schema verified through20261028000000 and deployed worker v11 source verified against commit d2a041a. Next automatic worker execution remains routine monitoring, not observed runtime proof. See [final targeted validation](final-wave2d-review-2026-10-28.md). Waves1/2A/2B/2C remain locked. Wave2E scope is proposed, not implemented; Bleach part-year repair remains deferred. Dated rollout notes below are historical and superseded by this status.

## Exact manifest

Watching:
- src/modules/watching/lib/milestone-receipts.ts
- src/modules/watching/lib/milestone-receipts.test.ts
- src/modules/watching/lib/receipt-ui.test.ts
- src/modules/watching/lib/history-analytics.ts (read-origin type, accepted legacy calendar dates, shared first-view deduplication)
- src/modules/watching/lib/history-consumption.test.ts
- src/modules/watching/components/stats/computeStats.ts
- src/modules/watching/components/stats/computeStats.test.ts
- src/modules/watching/lib/stats-charts.ts
- src/modules/watching/lib/stats-charts.test.ts
- src/modules/watching/lib/stats-org.test.ts
- src/modules/watching/service.test.ts
- src/modules/watching/hooks/query-keys.ts
- src/modules/watching/hooks/useWatchingStats.ts
- src/modules/watching/service.ts (existing selected viewing_date typed)
- src/modules/watching/receipt-actions.ts
- src/modules/watching/hooks/useAchievementReceipts.ts
- src/modules/watching/hooks/useUpdateMedia.ts
- src/modules/watching/hooks/useRewatches.ts
- src/modules/watching/lib/sync-goals.ts
- src/modules/watching/components/stats/achievements.ts
- src/modules/watching/components/stats/StatsPage.tsx
- src/modules/watching/components/stats/AchievementPanel.tsx

Goals (the owner explicitly authorized the minimum Watching receipt integration despite the general Goals freeze):
- src/modules/goals/lib/completion-receipts.ts
- src/modules/goals/lib/completion-receipts.test.ts
- src/modules/goals/receipt-service.ts
- src/modules/goals/service.ts
- src/modules/goals/components/WatchingCompletionStatus.tsx
- src/modules/goals/components/GoalDetailPage.tsx

Schema/checks:
- supabase/migrations/20261021000000_watching_milestone_receipts.sql
- supabase/tests/watching-milestone-receipts.sql
- scripts/test-watching-milestone-receipts.mjs
- supabase/checks/expected-objects.json
- src/modules/styleguide/coverage.generated.json

HQ: this document, now.md and Watching architecture/decisions/backlog. No other product module is unfrozen.

## Achievement classification and live state

Ten consumption/experience milestones get permanent receipts: cinephile,series_devotee,otaku,marathoner,time_traveler,genre_nomad,auteur,big_year,long_hauler,ever_present. Connoisseur and Hall of Fame describe CURRENT opinions and remain live-state achievements; neither produces a permanent receipt. No new achievement definitions or configurable engine.

Existing computeWatchingAchievements remains the live calculator, with current value/target and existing breakdown. withUnlockReceipts independently supplies historical unlocked/date state. It never derives a permanent date from watched_at. A receipt can remain unlocked with live349/350; an unverified legacy400/350 does not invent an unlock/date. The panel explicitly says first observed satisfied and current progress. Rating-only badges say Currently satisfied. A receipt read/observation failure is shown, not replaced with a guessed unlock.

## Authoritative observation and eligibility

The browser supplies only orgId to an authenticated Next server action, never a metric, target, unlock date, userId or eligibility flag. Server getClaims verifies identity; the owner/org RPC returns a fresh coherent single-statement snapshot. SQL jsonb aggregation returns one RPC row containing all history/ranges without a nested PostgREST collection cap. No third-party provider render request. All-time compact observations are required for milestones; Goal definitions are owner-filtered and paginated in1000-row pages. Goal metric reads are batched, with per-year computed-stat memoization.

receiptMetrics reuses the Wave2C computeStats/historical-contribution path. Marathoner uses the full fresh canonical Stats hours total, including accepted legacy contributions. Other receipt metrics retain explicit USER_ACTION/USER_CORRECTION evidence; imports/repair-origin rows remain live inputs for those metrics until an explicitly approved evidence policy. No guessed historical receipt date. Retracted evidence stays retracted. Repeated IDs and overlapping FIRST_WATCH membership do not inflate first-view counts/time; unknown partial aggregate runtime remains unknown rather than apportioned. REWATCH is additional consumption, completion contributes only uncovered frozen membership and never milestone zero as known runtime.

Receipt-eligible title completion is explicit: movie consumption or FINISHED_TITLE TV completion; AVAILABLE_SCOPE is not a finished series. Long Hauler reads frozen finished membership, not current episode counts. Marathoner uses the same mixed canonical hours total as Stats, including legacy runtime estimates and known frozen durations, recomputed server-side from the fingerprinted snapshot. Missing durations add no guessed hours. Its evidence source is WATCHING_STATS_HOURS_V1; the other milestone sources remain WATCHING_HISTORY_V1. Release decades/genres use currently available reference metadata only for canonically consumed completed works; absent metadata stays absent. Auteur additionally requires explicit director role, not untyped legacy credits. Year/month milestones require confirmed recorded date components; YEAR never manufactures a month. No updated_at date inference. Removed-library history is retained without inventing missing metadata.

Eligibility is raw finite actual>=positive target. No rounded percentage. A currently authoritative satisfied metric creates a FIRST_OBSERVED_SATISFIED receipt using trusted DB observation/write time. This is explicitly NOT the true historical first-crossing date. Source hash,metric key,rule version,target and actual value are retained, not a full analytics dump.

## Trust, stale state and persistence

watching.achievement_unlocks has one primary-key identity per org/user/achievement,immutable under ordinary flows. Trusted server evaluator calls record_achievement_unlocks through the existing server-only service credential after user authentication; no secret is returned or browser-bundled. SQL rechecks the exact snapshot fingerprint, rejects unknown rule/version/target, compares raw values and inserts ON CONFLICT DO NOTHING under an owner/org advisory lock. Changed source data fails closed; normal subsequent observation may retry. Goal definition locks and the receipt PK provide separate Goal concurrency safety. Partial failures cannot delete an already-created legitimate receipt.

RLS permits SELECT only to the receipt owner within their organization. Authenticated/anon/service roles have no direct receipt DML. Only the trusted service role can call the two writers; direct browser-forged metrics cannot create receipts. The source helper is private. Demo accounts may read their snapshot but cannot run the observing writer action. Existing Watching history/provider/rating write boundaries are unchanged.

Observation is prospective on Stats, media analytical/status changes, add/delete bridges, rewatch correction/removal and Watching Goal create/edit/recalculation. Existing sync failure reporting remains: a receipt/Goal bridge failure does not roll back an already-successful watch action, and subsequent observation can retry. This is first-observation persistence, not a promise of reconstructing every past threshold crossing or a new event-sourcing system.

## Goals boundary and permanence

Watching transports {metricKey,currentValue,liveValue,measuredAt,source,evidenceVersion}. currentValue is receipt-qualified canonical evidence; liveValue is the current canonical count and may retain explicit legacy fallback. Goals owns its definitions, raw threshold decision and public.goal_completion_receipts writer/table. Its SQL contains no Watching table query or Watching-owned duplicate Goal state. Definition/target/period/year are checked under row lock; old observations expire after30seconds. The server-only writer checks the raw decision again.

The existing films/finished-series/finished-anime/titles Goal contract is retained: starting a series does not complete a finish-series Goal. Annual finished TV receipts require a confirmed finish-period context; all-time completion may remain valid with unknown date. Goals progress is independently computed from liveValue with floor, not used to authorize completion. Receipt state forces completed status; current49/50 can remain historically completed. A BEFORE UPDATE guard prevents ordinary recomputation/manual status edits from removing an existing receipt-backed completion/date. Existing explicit legacy completion/date is preserved without manufacturing a receipt. Updating a Watching Goal returns its fresh persisted status after receipt evaluation. Goals' detail shows historical completion alongside current raw progress and its existing stable completion date.

## Corrections and deferred scope

Ordinary drops, provider changes, library deletion or history corrections never revoke receipts. No invalidated_at/admin UI is added speculatively. If original proof is later shown invalid, an owner-reviewed controlled corrective migration must preserve the original evidence/audit and explicitly define invalidation/reopening; current app/service recomputation cannot do this. This future controlled correction flow is deferred, not auto-revocation. Goal deletion does not cascade-delete its receipt; owner/account removal remains governed by existing tenant/account lifetime.

Wave2E+ only: legacy reconciliation/backfill, guessed lifetime crossings (not implemented), and separately authorized invalid-receipt correction tooling/policy. No broad Goals redesign, new achievement engine, provider repair, Kuroko repair or other frozen module work.

## Migration and validation evidence

20261021000000 is LOCAL ONLY. It creates2 tables,20 columns,2 owner SELECT policies,5 functions and3 indexes; no existing-row update/backfill and no writer invocation at apply time. Measured fixture counters before/after:11/164/6/13/20 ->13/184/8/18/23 (tables/columns/policies/functions/indexes). Expected full reference becomes81/856/225/49/277 with20 crons; watermark20261021000000. No extra version or remote operation.

Final targeted Vitest:11 files,195 tests pass. Covers raw99.6 thresholds, canonical-vs-legacy evidence, immutable live drops, prospective dates, duplicates, completion deduplication, unknown duration/date, provider regressions, finish-series semantics, private UI state and previous Wave1/2A/2B/2C contracts (Black Clover,Hannibal8.75,lineage,cour,1105-row reads).

Real PostgreSQL16 fresh loopback cluster repetition passes: no migrated data changes, no automatic receipts, legacy explicit completion/date retained, fresh/stale source hashes and Goal definitions, raw thresholds, stable timestamps, sticky49/50 progress, direct-write denial, same-org foreign-user and foreign-org isolation, correct service/browser ACLs. Two independent connections race for the same achievement and then the same Goal: one receipt each. All test databases are dropped. The older disposable cluster had a missing internal file; it was stopped and replaced by a fresh ignored local cluster. No production host/credential is accepted by the runner.

TypeScript and affected-file ESLint pass. Schema reference, generated coverage, diff whitespace and CRLF checked. Production build succeeds with37 generated pages after fixing an import/directive ordering error and allowing the existing Google Fonts fetch; Sentry upload auth and telemetry disabled, no deployment. Final pure metric/date-confidence refinements pass the final targeted tests/TypeScript/lint; no repeated full build for those pure changes. Existing middleware/Vite notices are unrelated.

Actual AchievementPanel and WatchingCompletionStatus SSR fixtures checked at390/768/1024/1440 with zero horizontal overflow; mobile/desktop PNGs read. All browser network requests blocked. These are isolated component fixtures, NOT live authenticated production E2E writes.

Existing data changes:NONE. Production changes:NONE. No remote migration, deployment, staging,commit,push or Wave2E. Verdict:WAVE 2D READY FOR REVIEW.

## Two targeted user-visible corrections before final review (2026-10-21)

Owner supersedes the earlier legacy calendar policy: every stored legacy full watch date is accepted as DAY, including pre-2024, without a cutoff or DB rewrite. Explicit partial viewing_date/history components retain their recorded precision; sorting projections do not manufacture missing components. A title date supplies month/day only to compatible season/cour year buckets, preserving explicit cross-year attribution. Legacy rewatch dates are usable calendar data without inventing scope/duration. history_version is never a lifetime-completeness switch; frozen history masks only matching legacy coverage. Shared first-view deduplication retains the existing receipt overlap guard while keeping Marathoner and Stats equal; no repeated FIRST duration inflation.

Read-only remote audit: local .env.local points to femvhonlpafdajyamvcu, localhost /auth responds200. Remote registry remains20261018000000,925 media; achievement/Goal receipt tables, snapshot RPC and achievement writer absent. No mutating RPC or authenticated observer was invoked. Browser session identity was not inspected. The pre-migration local UI consequently cannot verify/persist receipts, independently of the old eligibility mismatch.

Fresh read-only real-library reproduction (512 items,2 historical assertions):199862 minutes=3331.0333333333333 hours.185750 minutes belong to legacy-only breakdown entries;14112 to mixed entries, including48 frozen minutes. Strict pre-fix eligibility was0.8 hours. After the fix the trusted server Marathoner metric equals the full Stats total exactly; neither cached client values nor rounded display authorize a receipt. Canonical accepted legacy time is legitimate for this hours milestone, not proof of a reconstructed historical first-crossing date. Other milestone/Goal eligibility policies unchanged.

On the same immutable input, calendar aggregation rises from1 populated month to119.2022 now has10 populated months, including March;2026 has8 (March through October). Hours remain exactly3331.0333333333333. All changes are local; no migration application, receipt backfill, date repair, Goals writes, deployment, commit/push, Wave2E or final2D review.

Targeted correction validation:7 Vitest files /166 tests pass (history/calendar, Stats, milestones, Wave1 contracts, history precision and scoped-rating/receipt UI contracts). TypeScript --noEmit --incremental false and affected-file ESLint --max-warnings0 pass. Local PostgreSQL rehearsal passes raw2999.999 rejection, raw3331.0333333333333 prospective receipt, idempotency, evidence-source label, ownership/RLS and independent-connection achievement/Goal races; disposable databases removed and loopback server stopped. Schema reference, app/HQ diff checks and changed-source CRLF/whitespace pass. No component UI changed in this correction, so no coverage regeneration or production build. No authenticated live UI observer was executed against production.

## Remaining monthly Activity + Rating Distribution corrections (2026-10-21)

Owner identified2021/2022/2023/2026. Fresh SELECT-only Watching snapshot reproduced512 items,2 canonical assertions,9 legacy rewatches. No observing/writer RPC or production mutation. Three calendar issues: missing zero-month slots compressed the UI axis; the latest media viewing_date suppressed older unrelated legacy watched_at (Bleach2023 vs prospective2026); calendar facts vanished when the stored full title date disagreed with annual season/cour stamps (Dexter/Presumed Innocent2026). An unscoped UNKNOWN assertion also cannot prove a matching legacy scope, so a same-year coincidence no longer suppresses it.

Accepted legacy calendarDate is retained per remaining legacy contribution, independent of annual scope-year attribution. Monthly/daily Activity uses the stored calendar fact as-is. A partial newer history date does not replace an older unrelated legacy date. Explicit same-year partial date components still prevent decoding their sorting projection. Frozen history/tombstones mask only represented legacy coverage; fully replaced coverage cannot resurrect its old calendar date. Monthly/daily duration/episode details remain unknown where a conflicting season year prevents attribution; annual hours/episode counts are not relocated. This refines the earlier compatible-year-only calendar policy. No inferred cross-year date, no2024 cutoff, no DB rewrite, no fabricated scope/duration.

Reproduction on the same input:2021 and2022 consumption counts unchanged, with12 UI month positions (March/August2021 and May/June2022 now visible as zero). December2023 rises14->15 (Bleach); March2026 rises5->6 (Presumed Innocent); April2026 rises4->5 (Dexter). Existing dated rewatches remain2/1/2/4 events for2021/2022/2023/2026; canonical replacement IDs never duplicate compatibility rows. Annual hours remain436.01666666666665 /262.06666666666666 /239.96666666666667 /261.0833333333333 respectively; all-time3331.0333333333333 unchanged.

Rating Distribution had two separate issues: labels were equal-width cell centres while curve points used SVG endpoints, making the curve visually pass the10 label; Math.round also moved real decimal maxima such as9.5 to10. Exact score frequencies now feed one numeric scale shared by points, labels and curve. Cubic control points stay inside each endpoint rectangle, preventing x/y overshoot; no beyond-domain zero endpoints are added. All scores remain unchanged; only distribution grouping/display is corrected. Numeric tick spacing stays readable, exact values remain in point titles/accessibility text. Current Watching surfaces, teal fill/curve and card layout retained.

Validation:150 tests in5 targeted files pass, covering monthly legacy/history overlap and full replacement, conflicting dates/partial adoption, UNKNOWN scope, rewatch compatibility,12-month positions, Wave1 Black Clover/Hannibal contracts, fresh Marathoner parity, actual SSR cards, numeric curve bounds, maxima10/9.5 and narrow6.5..8.75 domain. TypeScript --noEmit --incremental false, affected-file ESLint --max-warnings0, ds:coverage regeneration and diff/CRLF checks pass. Actual isolated cards rendered with fresh real calendar data and current Tailwind CSS at390/768/1024/1440; browser network blocked. Each year has12 positions, no horizontal overflow, max-point/label centre error under0.015px. Mobile/desktop PNGs read; this is isolated actual-component validation, not an authenticated production observer. Artifacts: scripts/.shots/wave2d-stats-charts-{390,768,1024,1440}.png (ignored).

No new migration needed or created. Existing local2D migration untouched in this correction. No production/schema/data writes, deploy, Git staging/commit/push, Wave2E or final2D review. No remaining blocker for these two targeted fixes.

## Final targeted review (2026-10-21): WAVE 2D NOT LOCKABLE

Review only: no application/SQL source edits, remote database access/migration, production mutation, deployment, staging/commit/push or2E. Existing225 targeted tests in12 files pass; TypeScript and schema-reference/diff checks pass. Fresh local PostgreSQL rehearsal passes2D no-backfill/data-preservation checks, owner/org RLS, service-only ACLs, raw thresholds, prospective dates, permanence and independent-connection achievement/Goal races. Object deltas remain2tables/20columns/2policies/5functions/3indexes. Disposable database dropped; local server stopped. Three counterexamples are preserved in ignored scripts/.shots/wave2d-final-review-repro.cjs; they are not covered by the current green suite.

1. P1 same-year partial history still overwrites unrelated legacy calendar data. history-analytics.ts addLegacy chooses viewing_date whenever its year equals watched_at's year. Fixture: S1/S2 attributed2026, legacy watched_at2026-03-14, prospective viewing_date2026-10-20, explicit FIRST covers onlyS2 episodes1-2. Expected March legacy plus October canonical; actual Activity contains only October (23episodes/1150minutes conserved but calendar wrong). Date selection must follow represented event/scope, not a same-year test; retain explicit partial sorting-projection protections. Add same-year MONTH/DAY and partial-adoption regression cases.

2. P2 retained local titles lose consumption after library deletion. receiptMetrics creates orphan ID history:local:deleted-film, whereas history-analytics workOf reconstructs local:history:local:deleted-film, failing to match original work_key local:deleted-film. A retained movie FIRST_WATCH with frozen120minutes yields0hours/0film Goal count/0Marathoner instead of2hours/1film. Share stable work-key handling between retained history and ordinary items; cover local movie/TV removal and re-add without guessing metadata.

3. P2 Stats/server parity is conditional on a single organization. getWatchingStatsData/getHistoryAnalytics/getRewatchStats filter user_id only, and WATCHING_KEYS.stats contains no org. receipt_source filters both org and user. A user belonging to orgA and orgB can see3100hours (100A+3000B), while the active-orgA receipt metric is100. This is a static-query/pure-domain counterexample, not evidence that current production personal data spans multiple orgs or that foreign-user data leaks. Align the intended organization scope for Stats/history/rewatches/cache and receipts before locking; add a two-membership regression.

Unapplied remote receipt_snapshot remains an expected deployment condition, not one of these business findings. Rating curve/labels and exact-score bounds have no additional finding. Other supported receipts still require explicit canonical proof; Marathoner's accepted mixed-hours policy is retained. Next: targeted corrections for these three findings and regression checks, then re-review before any controlled remote migration.

## Final-review blocker corrections and targeted re-review (2026-10-21)

Owner authorized all three corrections. P1: scoped history's date components no longer override the calendar date of the remaining legacy coverage, even in the same year. YEAR/MONTH/DAY cases are tested; standalone partial-date projections still stay partial, and fully replaced coverage does not revive its old calendar month. Annual scope-year attribution and frozen/legacy overlap rules unchanged.

P2 local identity: shared pure workKeyOf unwraps retained history IDs and is used by history matching/rating/labels, service retained-history deduplication and the receipt evaluator. Frozen local movie/series/anime consumption survives deletion; reintroducing the same original local identity does not double count. A new unrelated local UUID is not guessed to be the same work. No metadata reconstruction or stored-row rewrite.

P2 organization scope: Stats media/history/ranges/ratings/rewatches require explicit orgId plus userId, including nested parent filters on ranges and every paginated page. Stats cache key includes orgId. Stats and receipt hooks wait for an organization and capture the same orgId in their query closure and key, so a request started for A cannot populate A's key with B's data after a switch. No dynamic getCurrentOrgId lookup inside the receipt query. SQL receipt-source scope is unchanged and now matches the browser input scope.

New regressions: same-year DAY/MONTH/YEAR partial adoption and full replacement; deleted/returning local movie/TV/anime; actual PostgREST HTTP request filters through MSW, private owner exclusion, nested history/range scope, real QueryClient caches for organizations A/B, in-flight switch and no-org guards, actual service retained local history. Test harness mocks hook rendering/receipt action only; service requests, query-client cache and pure domain evaluation run normally against local fixtures. A totals100hours and B totals3000hours in BOTH Stats and the server metric, rather than the old3100-vs100 mismatch. The old third branch in scripts/.shots/wave2d-final-review-repro.cjs directly supplies a deliberately unscoped array to computeStats; it is the old control counterexample, not the fixed service path. stats-org.test.ts exercises the corrected actual path.

Final235 tests/13files pass (previous225 plus10 regressions). TypeScript --noEmit --incremental false, affected ESLint --max-warnings0, schema-reference/diff checks and changed-source CRLF checks pass. The isolated Stats org tests were rerun after a helper naming/lint fix:3pass. Saved read-only real-library2021/2022/2023/2026 results remain unchanged from the prior validated fixes. No new production access/mutation, UI component change, schema/migration edit, coverage/build/PG rerun needed for these pure/query/cache corrections. The prior final-review fresh local PostgreSQL RLS/concurrency/no-backfill rehearsal remains applicable; no SQL changed.

Targeted re-review: all three recorded findings resolved; no remaining known local blocker. WAVE 2D LOCKABLE for the reviewed local code. This does NOT mean remote ready:20261021000000 is still local only, and missing receipt_snapshot/PGRST202 is expected until the owner-authorized controlled migration. No migration apply, data backfill, deploy, staging/commit/push or2E.

## Controlled pre-migration check (2026-10-21)

Owner requested verification only. Supabase CLI2.120.0 confirms linked project digital-second-brain (HEGON), femvhonlpafdajyamvcu, eu-west-1, ACTIVE_HEALTHY. All25 prior local/remote migration versions match exactly; only20261021000000_watching_milestone_receipts.sql is pending. Normal db push --linked --dry-run exits0 and plans exactly that file, with no seeds or roles. No historical replay/repair indicated. Local schema-reference watermark/counts and git diff --check pass. The CLI update notice is informational. Dry-run confirms the deployment plan, not remote execution of the SQL. No actual migration, observing/writer RPC, production personal mutation, deploy or Git mutation. Ready for the owner-authorized normal linked push followed by verification.
