# Watching Wave 2A — historical foundation

2026-10-12. Wave 1 is locked. Wave 2A is local implementation for review; no production migration, data mutation, deployment, commit or push. Waves 2B–2E are excluded.

## Source-of-truth boundaries

`media_items` remains the current library/posture projection. `viewing_records` and its `viewing_ranges` retain personal assertions independently of current posture and provider changes. This is prospective capture, not full event sourcing or reconstructed history. Stats, People, achievements and Goals retain their existing reads.

`history_version = 0` means no prospective history has been captured through this row. Existing rows receive only this deterministic default and `personal_revision = 0`. `history_version = 1` means **partial prospective history exists**, not that all earlier viewing is known. New rows also use 1 when a viewing assertion is captured; even a newly added title may describe an old, incomplete memory. No value in 2A means complete history.

Wave 2C must use active canonical assertions for the scopes/dates they actually cover, with explicitly separate legacy fallback for uncovered history. It must not sum the entire legacy projection and canonical history, or treat version 1 as permission to discard all legacy evidence. FIRST_WATCH ranges describe coverage assertions and may overlap across distinct claims/corrections; analytics must union first-watch coverage by user/work, while explicit REWATCH cycles remain separate. COMPLETION adds zero duration. No such analytics cutover occurs here.

## Schema inventory and constraints

Migration: `supabase/migrations/20261012000000_watching_history.sql`.

| Object / fields | Purpose, constraints and authority |
|---|---|
| `media_items.personal_revision` | Nonnegative bigint, default 0. Server trigger owns it; stale expected revisions fail. Personal command advances once, including rewatch/retraction. Unrelated direct provider metadata refresh does not advance it; a trusted robot's existing posture change does, without creating/retracting history. |
| `media_items.history_version` | Smallint 0 or 1, server controlled, partial-history semantics above. |
| `media_items.personal_operation_key`, `personal_operation_hash` | Latest non-historical/projection command receipt; protects immediate retry without inventing viewing facts for notes/favorites. Older history-bearing commands retain durable receipts on assertions. An older projection-only retry after another command fails its revision check rather than replaying a stale write. |
| `media_items.viewing_date` | Nullable validated JSON `{year, month, day}` used to seed the picker. Null for legacy/unknown. Projection of explicit input, never decoded from `watched_at`. |
| `viewing_records.id` | UUID primary key. |
| `org_id`, `user_id` | Required FK ownership with existing account/org deletion cascades retained. Command derives these from the authorized media item; read policy checks both user and organization membership. |
| `media_item_id` | Nullable FK with `ON DELETE SET NULL`; current library deletion retains evidence. |
| `work_key`, `title_snapshot`, `media_type_snapshot` | Server-frozen identity. `movie:<tmdb_id>` versus `tv:<tmdb_id>`; local UUID identity if no TMDB ID. Type constrained to film/serie/anime. |
| `kind` | FIRST_WATCH, REWATCH or COMPLETION. |
| `scope_kind`, `scope_key` | WHOLE_TITLE / SEASON / COUR / EPISODES / UNKNOWN. SEASON and COUR require a key. Scope labels do not imply unbounded/full coverage; ranges state the bounds. |
| `completion_basis` | AVAILABLE_SCOPE / FINISHED_TITLE, present only and necessarily for COMPLETION. Current finished-title command emits FINISHED_TITLE. |
| `year`, `month`, `day`, `date_precision` | Calendar-valid YEAR / MONTH / DAY / UNKNOWN. Omitted precision components are SQL NULL. Years 1–9999; leap days validated. |
| `date_confidence`, `scope_confidence` | CONFIRMED / INFERRED / UNKNOWN. Direct date input is confirmed; cursor-derived range coverage is inferred; missing facts remain unknown. |
| `recorded_at` | Server transaction capture timestamp, never personal viewing time. |
| `operation_key`, `operation_index`, `operation_hash` | UUID command key, nonnegative assertion ordinal, exact payload fingerprint. Unique `(user_id, operation_key, operation_index)`; one command can capture first-watch coverage plus a milestone. |
| `origin` | USER_ACTION / USER_CORRECTION / LEGACY_IMPORT / EXPLICIT_REPAIR. Public command permits only the first two; no legacy import or repair mechanism is implemented. |
| `supersedes_id` | FK to retained original, unique successor. Replacement requires authorized original retraction in the same command. |
| `retracted_at`, `retracted_by`, `retraction_reason` | Audited retraction; actor and nonblank reason required with timestamp. Original facts/ranges remain intact. |
| `retraction_operation_key`, `retraction_operation_hash` | Durable retry receipt for a retraction-only command, including Undo/new-history rewatch removal. |
| `viewing_ranges.id`, `viewing_record_id` | UUID PK and required parent FK (cascade only if the historical parent itself is removed during account/org cleanup); only the command attaches ranges to its newly generated record ID. No caller-supplied foreign parent accepted. |
| `season_key`, `season_number_at_recording` | Frozen storage coordinates: `tmdb-season:N` (N >= 1); film uses `movie`, number 0. No provider payload copied. |
| `episode_from`, `episode_to` | Inclusive positive bounds, end >= start and <= 1,000,000. Overlaps within a record/season are explicitly rejected. |
| `episode_ids` | Nullable bigint array; null means unknown membership. If supplied, cardinality must equal inclusive coverage, so an empty array cannot masquerade as unknown. |
| `minutes`, `duration_basis` | Nonnegative finite numeric snapshot or NULL. RUNTIME_SNAPSHOT requires minutes, UNKNOWN requires null, MILESTONE requires zero. Command refuses nonzero/non-milestone duration on COMPLETION. |

Other objects: immutable `valid_viewing_date` validator; `personal_revision_guard` function and BEFORE INSERT/UPDATE trigger; `personal_history_command(jsonb)` SECURITY DEFINER RPC with empty search path. Two SELECT-only history policies. Seven new indexes: two PKs, operation uniqueness, media lookup, owner/work lookup, unique supersession and range parent lookup. Measured deltas: tables +2, columns +42, policies +2, functions +3, indexes +7; no cron changes. Reference counts updated locally for this migration.

## Date write-path audit

| Current path | Canonical input/capture | Compatibility projection |
|---|---|---|
| Film detail date picker | Original YEAR/MONTH/DAY parts passed as `history_date`; correction retains original assertions. Picker seeds only from `viewing_date`. | `buildWatchedAtProjection` preserves the old sorting convention. |
| Single-season year / multi-season Watch History | Changed explicit map entries become YEAR assertions; clearing a prospective date produces UNKNOWN replacement, retaining bounded coverage. Cour date edits retain the supplied cour key; where no frozen mapping evidence exists, no ranges are invented. | Existing season/cour maps and derived `watched_at` continue serving Wave 1 reads. |
| `useWatchActions` mark watched / caught up / claimed position | No user date supplied means UNKNOWN. Automatic legacy year stamping is not promoted to canonical date input. New known coverage is frozen; completion is zero-duration context. | Existing `watch-status.ts` patches remain the status writer and produce existing timestamps/maps. |
| Explicit +1 / forward viewing | The viewing gesture supplies today's local DAY; only additional bounded known coverage is captured. | Existing `last_watched_at` / status projections remain. |
| Backwards correction / Undo | Original precision is retained for remaining coverage; affected prospective assertions are superseded/retracted. Compare against the user's cursor, never a regressed provider count. | Position/caught-up/last-watched patch is in the same transaction. |
| Add / merge / top-ten / progress claim | `useAddMedia` accepts `historyDate` separately from compatibility `watchedAt`. Existing UI without a date input produces UNKNOWN canonical date. Existing title classification is retained on namespace-equivalent merge. | The existing claim/status functions still derive legacy fields. |
| Rewatch picker / today shortcut | YEAR or MONTH input has no invented day; explicit today supplies DAY. Current UI asks no scope, so new rewatch scope remains UNKNOWN with no invented coverage. UI labels read canonical components for new records. | Legacy `rewatches.watched_on` uses Jan 1 for year-only or month day 1; this is projection-only for unchanged Stats. |
| Remove legacy/new rewatch | New historical rewatch is retracted atomically with projection deletion; a legacy rewatch has no invented historical assertion. | Legacy-only deletion remains supported inside the authorized command. |

Old behavior collapsed year-only input into December 31/now and month-only input into day 1, then decoded those timestamps into false picker precision. New canonical dates never use those conventions. `buildWatchedAtProjection` is explicitly named and cannot be used to decode history; `partsFromISO` was removed. Date-only claims never derive from `recorded_at`.

Intentionally legacy: Stats/Goals/achievement dates and rewatch runtime calculations still consume projections; pre-2A dates/rewatches retain their old display/fallback behavior. Provider/artwork-only refresh uses the existing direct projection update (no personal assertion/revision). Ordinary current library removal is still a direct delete, with retention guaranteed by the FK. There is no remaining application viewing/date write that separately sends a best-effort history write after a projection write.

## Commands, concurrency and security

`service.ts` routes personal insert/update, first-watch/progress capture, date/year correction, rewatch add/remove and audited retraction through one RPC. It prepares typed assertions using `lib/viewing-history.ts`, freezes the command/key, and retries an ambiguous transport failure once with exactly that payload. Validation/security/stale-revision errors are not retried. The RPC serializes a user/key with a transaction advisory lock, then locks the media row, checks expected revision and owner/org, inserts assertions/ranges, applies retractions and rewatch projection changes, and writes the current projection/receipt in one transaction. Any exception rolls back all of it.

Identical historical retries return the current retained projection without recreating assertions or resurrecting superseded rewatch rows. Reusing a key with different payload fails. Different actions use separate UUIDs. A replay after current media deletion returns no current row and preserves the historical receipt. A late retry of an old projection-only command or legacy-only rewatch removal may fail as stale/unavailable after a later command; it never overwrites later state or invents a history receipt.

Interactive rows select their revision. Detail edits/rank swaps preserve the source token. Autosave carries the draft's revision through its timer/unmount flush. Successful writes return and cache the new row; rewatch follow-ups use returned/current cache revision. Watch-action follow-ups carry their own successful revision; **Undo additionally binds to the specific action's returned revision**, so an old toast cannot undo a newer action.

History has no direct write grants/policies for authenticated, anon or service_role. Rewatch projection mutations also lose direct grants. Authenticated direct personal media changes/inserts are refused by the trigger; metadata-only refresh remains allowed under existing RLS. The RPC denies missing users/demo users, foreign users even within the same org, foreign orgs, foreign range parent IDs and unauthorized corrections. Robots cannot execute the RPC or mutate/retract historical rows. Existing robot provider/posture behavior is otherwise unchanged. Authorized corrections preserve originals; ordinary posture/provider changes do not retract them.

## Explicit non-work and rollout boundary

No legacy timestamp-shape inference, old rewatch scope assignment, episode reconstruction, scoped rating migration, Kuroko conversion, achievement reconstruction, completed-Goal changes or historical Stats cutover. No provider snapshots/provenance, Bleach rating reconfirmation, scoped rewatch UI, achievement unlocks, Goal receipts, reconciliation or Python/Data work. 

The owner must review/apply the migration before using the updated application against that database. Historical direct-write restrictions require the matching client; older clients should reload after rollout. No production application is part of this task. Existing Wave 1 uncommitted changes are preserved and remain a separate reviewed block.

## Validation and limits

- `node scripts/test-watching-history.mjs`: creates a uniquely named database only on `127.0.0.1`, replays baseline Watching table definitions plus the existing note trigger/cour constraint and Supabase-style default grants, applies the new migration, runs real SQL contracts, then drops only that test database. Authentication/org helper fixtures simulate request claims; this is not a full local Supabase/Auth/PostgREST deployment.
- Clean rehearsals cover YEAR/MONTH/DAY/UNKNOWN and leap dates, unique retries/distinct operations, insert/history and history/projection rollback, revision ownership/increments, two simultaneous writers, RLS cross-user/org rejection, foreign range rejection, invalid/reversed/overlapping bounds, unknown IDs, zero-duration completion, original/retraction retention, rewatch correction/projection consistency, old retry non-resurrection, demo/provider restrictions and actual deletion/FK survival.
- Targeted Vitest covers canonical capture, UI date submission, Undo and stale callbacks, and locked Wave 1 annual ratings/Black Clover/Hannibal 8.75, zero/unknown, exact Goals threshold, null invalidation, pagination, provider regression and robot outcomes. Final engineering results are recorded in the accompanying completion report/status.
- No production mutation or authenticated browser write against the unapplied production schema was attempted. Production deployment verification remains an owner rollout step.


## Wave 2A file manifest

Existing uncommitted Wave 1 changes in shared files are preserved. This manifest names only files edited/added for 2A, not the entire dirty worktree.

- `src/app/(main)/perso/watching/[id]/page.tsx`
- `src/modules/watching/components/detail/StatusCard.tsx`
- `src/modules/watching/components/detail/MyTake.tsx`
- `src/modules/watching/components/detail/TopTenRank.tsx`
- `src/modules/watching/components/shared/NextEpisodeButton.tsx`
- `src/modules/watching/hooks/useAddMedia.ts`
- `src/modules/watching/hooks/useRewatches.ts`
- `src/modules/watching/hooks/useUpdateMedia.ts`
- `src/modules/watching/hooks/useWatchActions.tsx`
- `src/modules/watching/lib/viewing-history.ts` (new)
- `src/modules/watching/lib/viewing-history.test.ts` (new)
- `src/modules/watching/lib/history-actions.test.ts` (new)
- `src/modules/watching/lib/review-ui.test.ts` (new in locked Wave 1; 2A adds date-input tests)
- `src/modules/watching/lib/watched-date.ts`
- `src/modules/watching/lib/watched-date.test.ts`
- `src/modules/watching/lib/watch-status.ts`
- `src/modules/watching/schemas/media.schema.ts`
- `src/modules/watching/service.ts`
- `src/modules/watching/types.ts`
- `src/modules/styleguide/coverage.generated.json` (regenerated; content may already match Wave 1 output)
- `supabase/migrations/20261012000000_watching_history.sql` (new)
- `supabase/checks/expected-objects.json`
- `supabase/tests/watching-history.sql` (new)
- `scripts/test-watching-history.mjs` (new)
- HQ repository: `now.md`, `modules/watching/architecture.md`, `modules/watching/decisions.md`, `modules/watching/backlog.md`, `modules/watching/wave2a-history.md` (new).

Ignored `.shots` editing scripts and the isolated PostgreSQL cluster are local verification tools, not product files or migration artifacts.

## Final validation ? 2026-10-12

- PASS: 200 targeted Vitest tests across 9 files (viewing-history, history-actions, review-ui, watched-date, watch-status, wave1-contracts, robot-handlers, Watching service and computeStats).
- PASS: repeated clean PostgreSQL 16 migration rehearsals, including real simultaneous connections, all SQL/RLS contracts above and both library retention and explicit org cleanup. Every run used a newly created database. Raw SQL was not blindly reapplied to an already migrated database; the normal versioned-migration contract remains unchanged.
- PASS: TypeScript, affected-file ESLint with zero warnings, schema-reference gate, design-system coverage generation, main/HQ diff checks and final local production build (37 static pages).
- The initial sandboxed build could not download the existing Google Inter font; the final build passed with approved network access. Existing Next middleware deprecation and Vite tsconfig-paths notices remain unrelated.
- Final source/diff review: no Wave 2B+ objects, provider/rating business logic changes, frozen-module changes or production mutations introduced by 2A. Existing Wave 1 changes remain untouched outside the explicitly shared writer/test files.
- Not verified: a full authenticated browser/PostgREST flow against a deployed 2A schema. Real PostgreSQL tests use auth/org helper fixtures; the production migration was deliberately not applied.

Production changes: **NONE**. Remaining implementation blockers: **NONE**. Verdict: **WAVE 2A READY FOR REVIEW**.

## Owner-only commit commands (not executed)

First separate/commit the already locked Wave 1 block using its existing review manifest. Shared files still contain both blocks in this worktree; do not mistake a whole-file stage now for a Wave-2A-only commit. After that separation, the Wave 2A command is:

```sh
git add -- ":(literal)src/app/(main)/perso/watching/[id]/page.tsx" "src/modules/watching/components/detail/StatusCard.tsx" "src/modules/watching/components/detail/MyTake.tsx" "src/modules/watching/components/detail/TopTenRank.tsx" "src/modules/watching/components/shared/NextEpisodeButton.tsx" "src/modules/watching/hooks/useAddMedia.ts" "src/modules/watching/hooks/useRewatches.ts" "src/modules/watching/hooks/useUpdateMedia.ts" "src/modules/watching/hooks/useWatchActions.tsx" "src/modules/watching/lib/viewing-history.ts" "src/modules/watching/lib/viewing-history.test.ts" "src/modules/watching/lib/history-actions.test.ts" "src/modules/watching/lib/review-ui.test.ts" "src/modules/watching/lib/watched-date.ts" "src/modules/watching/lib/watched-date.test.ts" "src/modules/watching/lib/watch-status.ts" "src/modules/watching/schemas/media.schema.ts" "src/modules/watching/service.ts" "src/modules/watching/types.ts" "src/modules/styleguide/coverage.generated.json" "supabase/migrations/20261012000000_watching_history.sql" "supabase/checks/expected-objects.json" "supabase/tests/watching-history.sql" "scripts/test-watching-history.mjs" && git commit -m "Capture precise Watching history atomically while retaining legacy reads"
```

HQ is a separate repository; after the same Wave 1 separation:

```sh
git -C hq add now.md modules/watching/architecture.md modules/watching/decisions.md modules/watching/backlog.md modules/watching/wave2a-history.md && git -C hq commit -m "Document Watching history contracts and review evidence"
```

## Controlled remote migration ? 2026-10-12

After a read-only registry/catalog audit established 21 matching historical versions and exactly two unapplied migrations, the owner explicitly authorized the normal CLI migration push. `supabase db push --linked --yes` (CLI 2.120.0) successfully applied `20261011000000_watching_director_roles.sql` then `20261012000000_watching_history.sql` to project `femvhonlpafdajyamvcu` (digital-second-brain). The remote registry now contains all 23 local versions. No manual SQL application or history repair was performed.

Read-only post-push verification:

| Check | Before | After |
|---|---:|---:|
| All media rows | 924 | 924 |
| Non-demo Movies | 375 | 375 |
| Non-demo Series | 74 | 74 |
| Non-demo Anime | 69 | 69 |
| Demo Movies / Series / Anime | 287 / 51 / 68 | 287 / 51 / 68 |
| Media identity hash | a9a082233fbbd997fcd0d3bb9e7db564 | identical |
| Personal Watching field hash | 6b6c48628f87e003c25c6f1725bdaabc | identical |

All 924 rows can be queried with `personal_revision = 0`, `history_version = 0`. `viewing_records` and `viewing_ranges` contain zero rows; no legacy reconstruction occurred. Both tables, all five media fields, three history functions, the revision trigger, two read policies and seven indexes exist. No unvalidated Watching constraints remain. The retention FK is `ON DELETE SET NULL`; history RLS is enabled and authenticated direct rewatch insert is revoked. Pure read-only date validation returns true for YEAR/MONTH/leap DAY/UNKNOWN and false for 2022-02-29. The ranking function contains the new Wave 1 logic.

Remote reference counts match exactly: tables 76, columns 800, policies 220, functions 39, indexes 266, active crons 20.

The exact `SECTION_COLUMNS` and `LIBRARY_COLUMNS` selectors return HTTP 200. Using the saved authenticated session, localhost Watching (redirecting to Movies) and Library return 200 with no missing-column/application errors; five observed media reads included `personal_revision` and returned 200. The browser smoke blocked potential non-read Supabase requests; none occurred.

No application or edge-function code was deployed. No personal production write tests, cleanup, registry repair, Git commit or push were run. The earlier production-NONE statements above describe the pre-push implementation/review stage; this section records the subsequently authorized schema application.

Verdict: **WAVE 2A REMOTE SCHEMA READY**.
