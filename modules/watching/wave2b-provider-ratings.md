# Watching Wave 2B: provider evidence and scoped ratings

Status: locally validated, ready for owner review. Migration is NOT applied remotely; workers/application are NOT deployed. No production personal writes, staging, commit or push. Wave 1 and committed Wave 2A remain the baseline. Resume review completed 2026-10-13 from the interrupted working tree.

## Scope and changed files

- `supabase/migrations/20261013000000_watching_provider_ratings.sql`
- `supabase/checks/expected-objects.json`
- `supabase/tests/watching-provider-ratings.sql`
- `scripts/test-watching-provider-ratings.mjs`
- `supabase/functions/_shared/provider-evidence.ts`
- `supabase/functions/_shared/provider-progress.ts`
- `supabase/functions/watching-series-sync/index.ts`
- `supabase/functions/watching-anime-cours-sync/index.ts`
- `src/modules/watching/lib/scope-ratings.ts`
- `src/modules/watching/lib/provider-ratings.test.ts`
- `src/modules/watching/lib/scope-rating-ui.test.ts`
- `src/modules/watching/lib/robot-handlers.test.ts`
- `src/modules/watching/lib/wave1-contracts.test.ts`
- `src/modules/watching/lib/history-actions.test.ts`
- `src/modules/watching/service.ts`
- `src/modules/watching/service.test.ts`
- `src/modules/watching/hooks/useScopeRatings.ts`
- `src/modules/watching/components/detail/SeasonPanel.tsx`
- `src/app/(main)/perso/watching/[id]/page.tsx`
- `src/modules/styleguide/coverage.generated.json`

HQ updates: this document, `now.md`, Watching `architecture.md` and `decisions.md`.

Separate dirty work is preserved: availability guards (`watchable.ts`, its tests, discover page, StatusCard, ListDetail, MediaActionMenu, NextEpisodeButton, QuickAddPanel, useAddMedia, useWatchActions and review-ui tests); badge color work (StatusBadge, LibraryCard, MediaCarousel); untracked AGENTS.md. The generated coverage file represents the whole current tree, so it is shared with that work. The history-actions test harness imports the availability helper required by the existing pre-2B handler changes. Do not stage the entire dirty tree as Wave 2B.

## Additive schema

Delta: 3 tables, 36 columns, 3 SELECT policies, 5 functions, 8 indexes; no new cron. Reference totals: 79 tables, 836 columns, 223 policies, 44 functions, 274 indexes, 20 crons; migration watermark `20261013000000`.

`provider_snapshots` (historical, 10 fields): `id` UUID PK; `provider` tmdb/anilist; `work_key` TV namespace; `observed_at` server timestamp; `contract_version` 1; `facts` compact validated contract; `fingerprint` SHA256 checked against facts; `previous_snapshot_id` same provider/work FK; `classification` six allowed states; `reason_code`. Index by provider/work/time; composite identity uniqueness prevents cross-work pointer links. No full payload retention.

`provider_heads` (mutable, 8 fields): provider/work PK; `accepted_snapshot_id`; `latest_observed_snapshot_id`; `checked_at`; `accepted_checked_at`; `refresh_outcome`; nullable bounded uppercase `error_code`. Both pointers have same-provider/work FKs. Accepted pointer and trusted check timestamp must be present together. Trusted checks advance without extra snapshots.

`scope_ratings` (historical assertions, 18 fields): `id`; `org_id`; `user_id`; nullable `media_item_id`; `work_key`; `scope_kind`; stable `scope_key`; `scope_number_at_recording`; nullable `viewing_record_id`, `provider_snapshot_id`, `mapping_snapshot_id`, frozen `episode_ids`; `score`; nullable `previous_rating_id`; `action`; `recorded_at`; `operation_key`; `operation_hash`. Score (0,10], null only CORRECTION; membership positive/nonempty without nulls when known. Unique user/operation and unique successor. Account/org deletion cascades; library removal nulls media link and retains opinion/coverage. Ordinary clients and service role have SELECT only, no direct history writes. Provider reads shared; opinions owner plus org RLS.

Functions:
- `valid_provider_facts`: immutable compact-contract validation, whitelist, identity/count/calendar/partition consistency. Unknown air dates/counts remain null, never known zero.
- `observe_provider`: service-role-only transactional evidence classifier; owns accepted head, cache projection and eligible growth posture effects under work lock.
- `provider_scope`: internal read resolver of accepted SEASON/COUR identity and episode membership; no public EXECUTE.
- `scope_rating_context`: authorized owner read, derives current validity and immutable receipt context.
- `scope_rating_command`: authorized, non-demo owner write; appends opinion and known completion coverage, updates only the scope compatibility map and personal receipt/revision. Global score is untouched.

No media schema alteration, UPDATE/backfill of legacy media, manual production SQL or reconciliation. Replay tested against a fresh disposable local fixture with existing Wave 2A rows. The versioned migration is applied once by migration registry, not repeatedly to an already migrated DB.

## Provider classification

BASELINE: first valid observation, unless it demonstrably downgrades existing legacy catalogue projection. It cannot reopen personal posture.
GROWTH: retained stable prior available IDs/coordinates/date evidence, known prior counts, plus individually identified newly aired episodes dated strictly after the trusted prior check UTC calendar date. Only this class may reopen an eligible completed/caught-up title behind the accepted frontier; cursor, personal dates, ratings and history are retained. Paused/dropped/wanted/reference posture is protected.
REGRESSION: loss of accepted membership, season identity, or reduced trusted counts. Evidence retained; accepted head/projection/posture unchanged.
RESTORATION: observed facts return to the accepted fingerprint after a rejected observation. Accepted original remains; not growth, no reopen.
CORRECTION: same available membership metadata changes, valid complete cour mapping changes, or conservative resolution of previously unknown airing without new-content proof. No reopen. Cour identity is AniList ID, never ordinal.
AMBIGUOUS: incomplete/unknown identity or airing, count-only changes, late discovery, unstable prior identity, incomplete/mismatched cour mapping. Evidence retained; accepted head/projection/posture unchanged. Empty/malformed responses and transport failures mark FAILED, retain accepted state.

Identical latest facts create no additional snapshot. Failure creates no invented snapshot. Accepted snapshot/cache/posture writes are one transaction. Noop/restoration checks update trusted time/cache freshness without treating restoration as growth.

## Locked lifecycle proofs

Kuroko synthetic fixture: accepted26 -> observed25 REGRESSION (26 retained, no personal regression) -> observed26 RESTORATION (not GROWTH, no reopen); identical polling deduplicates.

Bleach synthetic fixture: scope13, explicit9 freezes13 IDs. Identified/date-proven +10 to23 preserves old9/13, becomes NEEDS_RECONFIRMATION. Catching up remains pending. Explicit confirm9 creates current23 coverage; explicit revise8.5 preserves old9 and links previous assertion. Global title score stays independent. Analytics remains on the existing legacy projections, with zero-minute/unknown-date rating completion receipts; repeated assertions do not add weighted annual scores. No analytics cutover in 2B.

Validity: CURRENT = current accepted membership exactly covered; NEEDS_RECONFIRMATION = accepted membership contains additional episodes; SCOPE_UNVERIFIED = unknown coverage/current membership, incompatible remap, or retracted completion; UNRATED = absent/cleared score. Progress does not confirm opinions.

Legacy map scores remain opinions with unknown coverage. Only an explicit subsequent user edit may preserve an overwritten scalar as a NULL-coverage legacy assertion. Known historical ordinals never seed an unrelated remapped subject. Unbound legacy opinions overwritten by a known subject are retained separately, without invented dates or episodes. No bulk seed or backfill.

## Interrupted cour-cache safety fix

`scopeMatchesView` requires a resolved lens, matching display number and known aired ceiling. For overlaid cours it also requires stable AniList ID and both current flat bounds. The panel blocks stars/confirm on a mismatch and offers a small refresh action. Page handler repeats the guard before any write. Refresh invalidates detail, anime-cour mapping and scope-context queries; editing resumes only after they match. Canonical score strips use matching contexts rather than raw remapped ordinal scores.

RPC requires expected personal revision, previous assertion, scope key, accepted TMDB snapshot and accepted mapping snapshot. Provider refresh and rating writes share the same work advisory lock; the click cannot silently acquire larger membership after refresh. Lost-response retry sends the identical frozen UUID/payload once for a transport error; coded database conflicts require refresh. Same-key same-payload retry returns its receipt; different payload rejects.

## Workers and validation

Both actual workers call observe_provider; neither directly writes catalogue/posture/cache projections. Series fetches compact season/episode identity instead of trusting reused counts. AniList mapping acceptance/cache projection is atomic. HTTP200 for successful/noop,207 partial,500 complete substantive failure. Rejected evidence counts as unsuccessful refresh, not silently accepted.

- Targeted Vitest: 9 files, 162 tests passed (provider/scoped/UI/handlers, Wave1, Wave2A history/actions, review UI, stats and service retry contracts).
- Local PostgreSQL16 rehearsal: full Wave2A contracts plus Kuroko/Bleach/cour retention, permissions/RLS, stale revisions/snapshots, two-connection refresh/rating race, simultaneous exact-operation retries and library-removal retention passed. Final SQL tests also verify new remapped subject never inherits another cour's old opinion, and overwriting another legacy slot preserves it without coverage.
- Fresh fixture object counts: 5/103/3/4/10 -> 8/139/6/9/18 (tables/columns/policies/functions/indexes). All existing media JSON hashes unchanged across migration.
- TypeScript passed. Affected app/test/script ESLint passed with zero warnings. Cached Deno typecheck and lint passed for both workers/helpers (existing inline URL import style excluded from Deno lint).
- Schema-reference check, design-system coverage generation, git diff --check passed.
- Earlier pre-interruption production build passed; required resume rerun after frontend edits failed solely fetching Google Fonts Inter. No repeated font retries, source-map upload or deployment. Final complete production build remains environmentally unverified.
- Prior local fixture receipt renders tested at390/768/1024/1440 without overflow; screenshots read. These are isolated receipt smoke checks, not live private application end-to-end verification. Current full UI cannot use the new remote RPC before owner-approved migration/application rollout.

## Review/rollout and deferred scope

Review the explicit Wave2B manifest and separate dirty hunks. Owner applies versioned migration only after review; coordinate worker/application deployment afterwards and reload old browser tabs. No rollout authorized in this resume. Review readiness is local evidence, not production readiness.

Deferred to explicit2C+ authorization: scoped rewatch UX, history-based analytics/achievements/goals, provider reconciliation/backfill, broader receipts and complete historical reconstruction. Runtime provider/long-series budget is not measured in production; no performance claim. Live end-to-end verification follows an explicitly authorized rollout. No Wave2C work started.

## Finalization and controlled remote migration (2026-10-13)

The owner explicitly authorized the two final wording fixes and controlled schema push. Owned detail comment now states scoped/global independence. Cour receipts say `Originally rated N episode(s)`; season endpoint wording remains unchanged. Relevant receipt tests:5 passed, including singular/plural cour cases. TypeScript, affected-file ESLint, design-system coverage generation, schema reference and git diff --check passed. No broad test or build rerun.

Project confirmed by CLI: linked `digital-second-brain` / HEGON, `femvhonlpafdajyamvcu`, ACTIVE_HEALTHY. Pre-push registry:23 matching historical versions, local24, exactly `20261013000000_watching_provider_ratings.sql` pending. Normal CLI dry run named only that migration; no unknown remote version, additional pending migration or historical replay. `supabase db push --linked --yes` applied it successfully. Post-push registry:24 matching versions, Wave2B recorded once.

Read-only remote verification: all3 tables with36 columns, all5 functions with expected ACLs, RLS on all3 tables,3 SELECT policies,8 indexes, zero unvalidated Watching constraints. Global counts exactly match reference:79 tables,836 columns,223 policies,44 functions,274 indexes,20 crons. Direct INSERT/UPDATE/DELETE revoked for authenticated/service roles on new tables. Media revision/history fields are queryable on all925 rows. A direct private resolver smoke was denied to the dedicated read-only role as intended; successful catalogue/media SELECTs supplied compatibility verification instead. No production write RPC was invoked.

Immediate before/after media counts:925 ->925; film662 ->662, serie125 ->125, anime138 ->138. Identity hash `607a18b67b270fab500645d97d3e02fa` unchanged; aggregate personal-field hash `036fc09b3f32bd185bf9cfbfa3cbdf8d` unchanged. This fresh baseline supersedes the older924-row Wave2A snapshot; the difference existed before this push. New provider_snapshots/provider_heads/scope_ratings all empty; viewing_records/viewing_ranges remained empty. No backfill, Kuroko repair, legacy coverage inference or personal rating rewrite.

Only schema migration/registry effects occurred remotely. No application/Edge Function deployment, production personal mutation, Git staging/commit/push or Wave2C work. Earlier production-NONE statements describe the pre-migration stage. Verdict: WAVE 2B REMOTE READY AND LOCKED. Wave2C is deferred to the next session.
