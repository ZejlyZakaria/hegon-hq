# Watching Wave 2D — final targeted validation, 2026-10-28

Current verdict (supersedes dated rollout notes below): **WAVE 2D CLOSED for the validated scope**. Application commit d2a041a is present; remote migrations/watchdog were verified, and deployed anime worker v11 plus its four shared dependencies match committed source. No post-v11 cron execution was observed: that remains routine monitoring, not a closure blocker. Bleach part-year reconciliation remains explicitly deferred. Wave 2E is at scope proposal only; see wave2e-reconciliation-scope.md.

## Review correction

The detail page now shares a paginated history query between period labels and the audit history. The normal personal mutation invalidated detail/Stats/library, but not that history key or the scoped-rating context. A progress or finish action could therefore leave the period and rating eligibility stale. A date-only `history_date` correction was also missing from the shared analytical dependency list.

Reproduced against the real `useUpdateMedia` callbacks and a real TanStack QueryClient; assertions failed before the fix. The existing mutation now invalidates the affected media's history and scope-rating reads, and the existing analytical field list includes `history_date`. No second writer, calculation helper, cache-wide reset or parallel status path was introduced. Notes-only edits retain their caches.

Files changed by this review:

- `src/modules/watching/hooks/useUpdateMedia.ts`
- `src/modules/watching/lib/mutation-dependencies.ts`
- `src/modules/watching/lib/history-actions.test.ts` — five new regression cases (progress, finish, season year, date-only edit, notes-only control).
- HQ current-state/report updates. Generated coverage regenerated, retaining 444 measured hits and the prior blur change.

Earlier point-1, completion-date and season-panel changes remain intact; their manifests are in `season-periods-and-parts-2026-10-28.md`, `bleach-completion-diagnostic-2026-10-28.md` and `season-panel-polish-2026-10-28.md`. Pre-existing `debug.log` is unrelated and untouched.

## Business and data verification

Fresh owner-authorized SELECT-only snapshot: 515 personal library items, 20 history records, 4 scoped rating records and 10 compatibility rewatches. Same accepted evaluator produces identical live UI and server achievement values: Marathoner 3730.7 hours, Long Hauler 500 episodes, Ever-Present 133 months. All three existing permanent receipts remain unlocked with their original observation dates; Connoisseur remains a live opinion badge. Immutable receipt dates are not recomputed from finish dates.

- Bleach: finish projection 2026-10-28, S1 2020, S2 period 2023–2026, four nested verified parts. First-view hours retain 146.4h in 2020, 19.2h in 2023 and only 0.8h/EP49–50 in 2026. No historical release-year reconstruction or redistribution.
- Hannibal: actual 2026 annual score remains 8.75.
- Black Clover: 2022 S1 has 68h and rating 8.5; 2026 S2 has 1.6h and no historical-rating qualification leak. Season-specific posters selected correctly.
- Activity: fresh 2021/2022/2023/2026 snapshot rendered at all four widths with all 12 month positions. Legacy stored dates remain accepted, without a pre-2024 cutoff. Zero months stay zero. Frozen rewatches, partial replacement, retractions and mixed legacy/history remain covered by tests.
- Rating Distribution: actual numeric positions/labels align; maximum 10, narrowed 6–8 and single 10 cases render without overshooting segments. Fractional 8.75 maximum also covered by unit tests.
- Scope reads preserve saved unverified opinions; edits require verified membership. Proven growth hides the prior aggregate note behind history, while unchanged part scores and independent global opinions are retained. No season/part overlapping annual votes.

Reviewed replacement paths: completion-date resolution remains at the existing personal-command boundary; coarse year-map handling remains a necessary legacy fallback. History/Stats/receipt calculations share the accepted ledger. Display/write scope checks intentionally answer different questions and share identity checks. No newly orphaned helper/file/dependency or competing business calculator found; Knip passes. This is a targeted review, not a claim that all legacy code in the repository is dead-code free.

## Remote schema verification

Linked project `femvhonlpafdajyamvcu`: all 30 local migration versions match the remote registry through `20261028000000`. No pending migration in the current local set. Watchdog reference matches exactly: tables81, columns857, policies225, functions49, indexes277, active crons20. Existing full media counts: 665 films / 126 series / 137 anime (references included).

Remote bodies of `watching.receipt_snapshot`, `watching.receipt_source`, `watching.record_achievement_unlocks`, `public.record_goal_completion_receipts` and `public.preserve_goal_completion_receipt` match the latest local migration bodies (MD5, newline normalization only). SECURITY DEFINER and ACLs inspected: snapshot authenticated-only, writers service-only, private source/trigger not publicly callable. No receipt observer/writer was invoked against production.

The read-only diagnostic role correctly cannot execute private `receipt_source`. After an approval-review rejection of the broader personal-data read, the owner explicitly authorized it; subsequent snapshot uses direct SELECT queries without changing grants or invoking private functions. No access restriction was bypassed.

## Executed validation

- Vitest: **430 tests / 21 files pass**, including Goals completion contracts, history/rewatch/scoped ratings, monthly charts, annual scoring, deleted-local identity, organization isolation, receipts and mutation cache regressions.
- TypeScript `--noEmit --incremental false`, zero-warning affected ESLint, Deno worker check and Knip pass. Three pre-existing Knip configuration hints remain.
- Production build passes, 37 generated pages. No deployment; sourcemap-upload token and telemetry disabled. Existing middleware convention and Vite tsconfig-plugin notices are outside this correction.
- Real PostgreSQL 16 loopback rehearsal passes: permissions/RLS, stale observation and revision rejection, raw thresholds, sticky receipts/Goals, retries, two-connection concurrency, no automatic backfill, frozen rewatches, accepted legacy snapshots, Bleach migration rollback/idempotency and unchanged unrelated data. Provider SQL rehearsal passes nested S2 membership, growth, unaired/unseen refusal, independent opinions and rollback. Temporary databases removed by runners; local test server stopped.
- Isolated actual-component browser checks pass at390/768/1024/1440: season scores and confirmation callbacks, stable 2:3 poster dimensions with disclosure expansion, owner-approved blur, Activity/months, rating geometry, compact poster DPR variants and achievement receipt states. Representative screenshots read, including current-data mobile/desktop charts and achievement views. Controlled no-receipt fixture separately verifies the awaiting-verification state.
- Schema-reference script, regenerated design-system coverage, CRLF and main/HQ diff checks pass. No broad unrelated test suite.

Ignored evidence: `scripts/.shots/final2d-current.json`, `final2d-replay-results.json`, `final2d-function-hashes.json`, `final2d-stats-charts-*.png`, `final2d-business-*.png`, `season-polish-browser-results.json`; harnesses alongside them. These contain local diagnostics and are not product source or commit inputs.

## Limits and remaining owner step

No authenticated production-backed E2E mutation test was run: opening Stats can invoke the receipt observer, so browser validation deliberately uses isolated actual components with production writes blocked/absent. No production data change, migration, function deployment, staging, commit, push or Wave 2E work performed by this review.

The new AniList worker guards are typechecked, covered by provider/robot tests and locally reviewed. Their remote deployment is **unconfirmed**, not asserted absent. The owner can activate the exact current worker with:

```sh
npx supabase@2.121.0 functions deploy watching-anime-cours-sync
```

Then verify the deployed revision and normal cron result before marking operational closure. Do not run destructive write tests or deliberately corrupt production catalogue data. This is the remaining activation verification, not another migration or a newly discovered business-rule blocker. Application deployment remains a later explicit owner decision.


## Owner-found post-review correction: annual rewatch-only Top Picks

Interstellar/Fight Club revealed an annual eligibility leak after this review. Corrected at the existing Stats ledger selection, with seven new regressions,205 targeted tests, TypeScript/lint, exact real-snapshot non-Top-Picks equality and actual card/panel four-width checks. Identity also removed by the same rule; first-year/all-time picks retained. No production data change or2E. See top-picks-rewatch-fix-2026-10-28.md. Worker activation confirmation remains the separate outstanding operational step.


## Rated-scope and owner-confirmed Part4 amendment

Annual labels now represent exactly the selected scored subjects (Hannibal S1–S2, JJK S3, actual Bleach S2-wide9). Shared vote-result refactor preserves numeric scores and all other Stats fields. Owner confirms all Bleach Part4 E41–50 in2026; new data-only migration20261028010000 adds only missing E41–48 at YEAR precision, retaining existing precise E49/E50 and all-time166.4h.2023→2026 transfer192min rehearsed;202 tests/8files, TypeScript/lint/Knip/schema/diff, local SQL rollback/idempotency and four-width UI pass. Correction2026-10-29: this migration is already recorded remotely and its exact archive has been restored locally; no push or remote reapplication is needed. Full fresh-database replay remains a separate documented limitation (migration-alignment-2026-10-29.md). Evidence/manifest: rated-scope-and-bleach-part4-2026-10-28.md. No2E started.


## Read-only owner deployment verification2026-10-28

Owner asked to verify an already-performed deployment. Linked HEGON Management API/CLI metadata confirms watching-anime-cours-sync version11 ACTIVE, updated2026-10-28T21:16:13.931Z (22:16:13 Europe/Paris). Downloaded that deployed source via CLI functions download --use-api into ignored scripts/.shots/anime-worker-download-20261028 only. Main index and all four local shared dependencies (batch-outcome, pagination, provider-progress, retry) match the committed source exactly after CRLF/LF normalization; SHA256 evidence in anime-worker-deployed-verification.json. Application repo remains clean except pre-existing debug.log.

Cron watching-anime-cours-sync-6h remains active at10 */6 * * *. Last retained automatic response2026-10-28T18:10Z was HTTP200, ok=true, failed=0 (10shows,5resolved,1guarded mismatch,4none). This response PRECEDES version11 deployment and is not post-deployment runtime proof. Cached watchdog health has zero consecutive failures. No function invocation, sync, deployment, production write or Git mutation performed. Deployment identity blocker is resolved; execution of version11 remains unobserved until a subsequent normal cron/explicitly authorized invocation. Bleach part-year reconciliation remains owner-deferred, with no pending deployable migration.
