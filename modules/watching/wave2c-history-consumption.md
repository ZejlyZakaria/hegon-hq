# Watching Wave 2C: history-aware viewing, rewatches and analytics

Status: WAVE 2C REMOTE READY AND LOCKED,2026-10-20. Waves1/2A/2B remain locked. The owner-authorized controlled migration is applied; no application/worker deployment, production personal data mutation, reconciliation, Git stage/commit/push or Wave2D work. Earlier local-only statements below describe implementation-stage evidence; the finalization section supersedes rollout status.

## Exact Wave2C manifest

- `src/modules/watching/lib/rewatch.ts`
- `src/modules/watching/lib/history-analytics.ts`
- `src/modules/watching/lib/history-consumption.test.ts`
- `src/modules/watching/components/detail/RewatchForm.tsx`
- `src/modules/watching/components/detail/StatusCard.tsx`
- `src/modules/watching/components/stats/computeStats.ts`
- `src/modules/watching/components/stats/computeStats.test.ts`
- `src/modules/watching/components/stats/StatsPage.tsx`
- `src/modules/watching/components/stats/HoursBreakdownPanel.tsx`
- `src/modules/watching/components/stats/WrappedModal.tsx`
- `src/modules/watching/components/stats/WrappedCard.tsx`
- `src/modules/watching/components/stats/achievements.ts`
- `src/modules/watching/hooks/useRewatches.ts`
- `src/modules/watching/hooks/useWatchingStats.ts`
- `src/modules/watching/hooks/useSeasonEpisodes.ts`
- `src/modules/watching/service.ts`
- `src/modules/watching/service.test.ts`
- `src/modules/watching/types.ts`
- `src/modules/watching/lib/review-ui.test.ts`
- `supabase/migrations/20261018000000_watching_history_consumption.sql`
- `supabase/tests/watching-history-consumption.sql`
- `scripts/test-watching-history-consumption.mjs`
- `supabase/checks/expected-objects.json`
- `src/modules/styleguide/coverage.generated.json`

HQ: this document, Watching architecture/decisions/backlog and now.md. Existing availability, badge and uncommitted Wave2B hunks remain separate; shared files are mixed. Do not stage the whole working tree as Wave2C. `achievements.ts` only accommodates nullable rewatch duration in its existing arithmetic; no persistence/receipt work. Current-watchlist CSV remains a current-state export, not a historical analytics export. Frozen dashboard/Goals modules untouched.

## Rewatch and viewing history

New actions require WHOLE_TITLE (movies; TV's available episodes), original SEASON, supported COUR or explicit EPISODES ranges. TV starts with no selected scope; clicking Log rewatch opens a form, never immediately saves. Partial TV viewing can explicitly record scoped rewatches without changing posture. Date input accepts YYYY, YYYY-MM, YYYY-MM-DD or blank UNKNOWN; action-today prefill remains visible/editable before the separate Record button.

`buildRewatch` freezes scope and original storage ranges; episode counts derive from frozen inclusive bounds, with accepted episode IDs when available. Stable AniList cour identity and frozen membership survive ordinal changes. Cour recording requires an accepted TMDB season snapshot with validated contiguous season offsets and complete aired episode membership for the selected cour. Cached cour identity/range may be used with that season evidence; aired-count offsets alone are rejected with "Refresh season details before recording this rewatch". Known explicit season/episode coverage still suffices without a snapshot; no third-party request is necessary to record. Missing IDs stay null. Unknown whole-title coverage requires choosing known episodes instead. Overlap/out-of-bounds/changed cour contexts and known dates before selected episodes aired are rejected. Future dates are rejected.

Movie duration freezes the known movie runtime. TV median runtime is NOT used as exact episode evidence for new rewatches. Actual per-episode runtimes already present in the episode query cache may be frozen; otherwise duration is UNKNOWN. Partial runtime knowledge splits ranges so known subtotals are preserved and unknown episodes stay null. Historical totals never recompute these records from today's provider/runtime data.

Detail Viewing history reads original/retracted/replacement assertions and frozen ranges; completions are explicitly milestones, not extra time. Its separate rating-assertion list retains old9 and revised8.5 with recording-date labels, never pretending those timestamps are watch dates. Rewatch rows offer explicit correction/removal. Correction retracts and appends a superseding record transactionally; removal retracts. Original history remains. No new history store.

## Source selection and mixed coverage

The Stats page supplies canonical records and scope ratings to computeStats. Rewatch/history records and range rows are independently paginated; no nested child cap. Every read uses existing RLS, owner filters for global analytics, and per-media filtering for detail.

FIRST_WATCH active assertions provide first-view time/counts. COMPLETION is deduplicated against first-watch coverage, usable legacy membership and other completions. Only uncovered frozen completion membership contributes fallback consumption; MILESTONE duration remains unknown, never known zero or current-runtime recomputation. Without frozen membership, a surviving completion may retain a title with explicit unknown scope/duration, never guessed episodes. Confirmations cannot inflate totals. Active REWATCH assertions contribute one rewatch event regardless of range count. Compatibility rewatches with a canonical ID, including retracted originals, are shadowed. Only unrepresented legacy rewatch rows remain, with UNKNOWN scope/date/duration rather than inferred DAY or whole-title runtime.

For mixed TV coverage, convert legacy display episodes into storage coordinates and subtract only the exact FIRST_WATCH intervals. Uncovered portions on both sides and between disjoint intervals remain explicitly labelled legacy estimates. Retracted first-watch intervals remain tombstones only for their own overlap, preventing resurrection without erasing unrelated legacy coverage. Active replacements contribute their own coverage. Coarse scoped assertions retain the existing conservative unknown-scope behavior; history_version is never a completeness switch.

Movie first-watch history shadows its first-watch projection. Legacy-only movies keep compatible YEAR fallback, without inferring month/day from timestamp shape. `history_version` is not a completeness switch:0 legacy and1 partial/prospective retain their existing meaning.

Canonical records persist through library removal; lightweight display items use stored identity/title/type when no current item remains. No fabricated media rows are persisted. Missing current metadata/global opinions remain unknown.

## Date and analytics contract

YEAR contributes only to annual/all-time; MONTH to annual/monthly/all-time; DAY additionally to exact daily activity; UNKNOWN only to all-time. No fabricated calendar day or month, or updated_at fallback, is used in the history path. Legacy compatibility date projections may still exist as required by2A but are not selected as historical dates when canonical events exist. Legacy dates without trustworthy precision are deliberately conservative.

`monthlyStats` and `dailyStats` also expose full title/rewatch counts, episodes, known minutes, unknown-duration counts and context-qualified ratings; YEAR-only facts cannot enter either calendar bucket.

History now feeds unique title counts, first-watch and rewatch time, episode counts, annual/monthly activity, daily activity data, available years, period genres/rating distribution/Top Picks, hour breakdown and Wrapped totals. Status chips still describe current posture; historical analytics never writes posture. A title may legitimately have current In Progress state while its older record remains historically completed.

Hours are sums of KNOWN frozen duration plus explicitly marked legacy estimates. Unknown/partial durations have separate counts/labels, including Wrapped's known-hours label. Null is never represented as a known zero-duration event. Known subtotals are retained even if the total duration remains unknown. Counts and scope remain visible when duration is absent.

## Rating attribution

Append-only ratings are grouped by stable analytical scope and evidenced viewing-year context. Lineage depth, rather than response/timestamp ordering alone, ensures CONFIRM/REVISE descendants replace predecessors as the single analytical vote for that context. Clears retain history and remove the applicable vote. A rating completion's known watch date is used; when it is UNKNOWN, an active dated first-view record containing the frozen terminal episode can provide context. Operation recorded_at is never treated as a watch date. Without an evidenced context, no annual rating is invented. Untracked legacy season/cour opinions may contribute only through their legitimate existing scope/year; tracked scopes do not fall back to mutable compatibility scores.

Black Clover: old2022 rating remains in2022; genuine2026 activity can count without any2026 rating. Global title score cannot leak into annual TV ratings.

Hannibal: legitimate same-year S1=8.5 and S2=9 yields8.75. Confirmations add no extra vote; no internal decimal rounding. Stats/Top Picks render up to two meaningful decimals. All-time global opinion stays independent.

Growth: historical13-episode9 remains valid in its original period. Watching the added10 does not expand that opinion. Explicit confirmation/revision for23 creates the expanded context; its8.5 does not rewrite the earlier13/9 period. Cour analytic counts/ratings use frozen scope evidence, not current ordinal remapping.

## Small migration, explained before implementation

`20261018000000_watching_history_consumption.sql` replaces only the existing personal_history_command body. No new tables, fields, functions, indexes, policies or object-count deltas; reference watermark moves to20261018000000, totals stay79/836/223/44/274/20.

Why required:2A rejected UNKNOWN-date rewatches because its compatibility table requires a date.2C records them solely in canonical viewing_records, with no invented compatibility date. Removal/correction uses the existing update/retraction mode. A caller-supplied viewing_context additionally takes the shared provider work lock before the media lock and checks accepted TMDB/mapping snapshot tokens, preventing a refresh from expanding scope between click and save. Existing receipt/revision/idempotency/ownership checks remain. No RLS or grants are loosened.

This migration contains function definitions only and no legacy UPDATE/backfill. Fresh local PostgreSQL16 rehearsal proves unchanged media JSON hashes and unchanged function/object counts. It also executes existing2A contracts and new UNKNOWN date, frozen120min movie, correction/removal, retry, stale revision/catalogue, foreign-user read/write and direct-write-denial cases. It was NOT applied remotely. Owner approval is required before a controlled push; deploy the application only after the compatible schema exists.

## Validation

-129 targeted Vitest tests in7 files pass: new historical consumption matrix, viewing-history, history-actions, review-ui, computeStats, scope-rating-ui and Watching service.
-Service tests page1105 records/ranges/ratings with exactly2 calls per collection and explicit owner filters; no per-record queries. Rewatch summary fetches only REWATCH rows/needed fields, not every progress event. Analytics' existing all-time/year selector requires all-time compact history; no arbitrary row limit or date truncation is introduced.
-Real local PostgreSQL16 rehearsal passes; previous media rows/object counts unchanged across2C migration. Temporary DB is disposable and local only.
-TypeScript, affected-file ESLint (zero warnings), schema-reference and generated coverage checks run for the final source. Git diff/CRLF checks recorded at completion.
-Local production build passes with37 generated pages. Existing Next middleware and Vite tsconfig-paths notices are unrelated. Sentry upload auth and telemetry were disabled for build verification; no deployment.
-Actual rewatch form SSR fixtures tested at390/768/1024/1440 with no horizontal overflow; mobile/desktop PNGs inspected. All browser requests blocked. Actual form handler tests cover input precision and stale cour blocking/resumption. This is isolated UI verification, not a live production end-to-end write test.

## Boundaries and next step

Existing data changes: NONE. Production changes: NONE. No historical reconstruction or Kuroko repair. No provider-worker changes in2C. Existing unrelated dirty files are preserved.

Deferred only to explicit2D+: persistent achievement/Goal receipts and policies (2D), legacy date/scope/rating reconciliation/backfill (2E), complete lifetime reconstruction. Provider identity/runtime gaps remain explicit unknowns, not promises of later automatic mutation.

## Targeted blocker fixes (2026-10-20)

The final review rejected frontier masking, completion-only loss and guessed cour offsets. These are now corrected at the domain layer; tests pin partial/disjoint overlap, exact retraction tombstones, movie/season milestone deduplication, retained completion-only and partial/broader completion evidence, ambiguous completion uncertainty, unavailable/missing cour evidence, safe retry and immutable trusted membership. The provider-metadata stability fixture now has no active legacy projection: under the owner's exact-overlap contract, current legacy coverage outside explicit ranges remains a labelled estimate, not an implicitly discarded suffix.

No SQL/RPC/schema changes were needed. The existing local migration is unchanged and remains unapplied remotely. Existing and production data changes: NONE. Final targeted validation is recorded below; Wave2D/2E remain excluded.

Targeted validation:145 tests across the existing7-file Wave2C set pass; the added service rejection regression also passes (9 service tests,146 distinct tests in total). Final TypeScript and affected-file ESLint pass, schema reference and git diff --check pass, and affected source retains CRLF. No PostgreSQL rerun: no command/function/SQL behavior changed. No production build rerun: changes are pure domain calculations/validation with targeted and type coverage, with no framework/UI integration change.

Remaining implementation blockers: NONE. Verdict: WAVE 2C BLOCKERS FIXED, READY FOR RE-REVIEW. Remote migration/application rollout remains a separately authorized follow-up.


## Finalization and controlled remote migration (2026-10-20)

The short final read-only re-review declared WAVE 2C LOCKABLE, no blockers/minor fixes, and SAFE FOR CONTROLLED REMOTE APPLICATION. The owner then explicitly authorized this finalization only. Application code remained unchanged.

Project identity confirmed by CLI: linked digital-second-brain / HEGON, femvhonlpafdajyamvcu, ACTIVE_HEALTHY. Pre-push:25 local migrations,24 remote, all24 historical versions matching through20261013000000. No remote-only version, replay or ordering ambiguity. Normal CLI dry-run listed exactly20261018000000_watching_history_consumption.sql. Normal `supabase db push --linked --yes` applied only that migration, with no seeds/roles. Post-push:25 matching local/remote versions, Wave2C recorded exactly once, no pending migration.

The normalized remote pg_proc.prosrc hash exactly equals the local reviewed migration:79b46794ec41eed3d3385f91bbddbc3e. Thus UNKNOWN-date compatibility handling, provider/mapping context locks/checks, ownership, revision and idempotency match the reviewed body. Signature watching.personal_history_command(jsonb) returns jsonb; SECURITY DEFINER and empty search_path preserved. ACL unchanged:{postgres=X/postgres,authenticated=X/postgres}; authenticated EXECUTE true, anon/service_role false. No write RPC was invoked for verification.

Fresh immediate before/after data invariants:

| Table | Rows before/after | Full-row aggregate hash before/after |
|---|---|---|
| media_items |925 /925|40c22735938a2b8f7c7604537e6fba90|
| viewing_records |1 /1|d64e75c5538910b0dbac22e20f9318b7|
| viewing_ranges |1 /1|0dd2ad004cfe69e5afb93cd597c1c742|
| provider_snapshots |0 /0|d41d8cd98f00b204e9800998ecf8427e|
| provider_heads |0 /0|d41d8cd98f00b204e9800998ecf8427e|
| scope_ratings |0 /0|d41d8cd98f00b204e9800998ecf8427e|
| rewatches |9 /9|2d393e0c3ac6c5134997e0d9f8b0e5f0|

Media type counts unchanged:film662,serie125,anime138. Full media JSON hashing includes personal fields, dates, scope maps and provider fields; no existing media field changed. The existing1 event/1 range and9 rewatches predate this push; no automatic history was created. No reconciliation, Kuroko repair, legacy precision promotion, backfill or rating rewrite.

Read-only compatibility checks succeed: all925 media rows expose non-null personal_revision/history_version, canonical history/range join and scope_ratings SELECTs work, unique function signature/result/ACL resolve correctly. No production mutation smoke test or fake history was run. Only migration-defined function/schema/registry effects occurred remotely. CLI update notice is informational; no CLI upgrade was performed. Initial sandbox/PowerShell restrictions were resolved using approved elevated CLI/read-only calls; credentials stayed in memory and were not printed.

Existing data changes:NONE. Production personal data mutations:NONE. No application/Edge deployment, Git staging/commit/push or Wave2D. Verdict:WAVE 2C REMOTE READY AND LOCKED. Wave2D is deferred to the next session.
