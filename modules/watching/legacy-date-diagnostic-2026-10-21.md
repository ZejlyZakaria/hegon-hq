# Watching legacy date diagnostic - 2026-10-21

Scope: read-only personal HEGON library, user+organization filtered, consumed/current-progress/paused/dropped non-reference titles with watched_at before2025.291 dated titles. No date or production write. The owner has now applied migration20261021000000 according to the fresh remote registry SELECT; this is not a complete post-migration schema/receipt audit.

## Unknown-duration message and layout

StatsPage.tsx places the conditional unknown-duration paragraph as a direct child of the three-column chart grid. When nonzero, it occupies an entire column, displaces Activity and wraps Rating Distribution onto a second row. This is the screenshot's blank middle area, unrelated to missing personal data. The paragraph should be grouped with Hours Watched; no code/UI change was requested/executed in this diagnostic.

The2022 unknown event is the legacy Game of Thrones rewatch dated2022-03-01, with no matching canonical viewing record. The current title runtime is56minutes but the legacy row does not establish episode/season/whole-title scope or frozen duration. It contributes calendar activity, not invented hours. All291 old dated titles have a positive current runtime, so the screenshot message is not evidence of a missing title runtime. Other years' warning sources should be inspected per event, not guessed from this one example.

## Stored-date concentration

| Year | Dated titles (watched_at year) | December | December31 | December31 at12:00:00 |
|---|---:|---:|---:|---:|
|2013|13|6|1|0|
|2014|12|7|6|6|
|2015|12|5|3|3|
|2016|9|4|4|3|
|2017|36|9|7|7|
|2018|40|13|10|9|
|2019|32|13|12|12|
|2020|28|10|9|4|
|2021|20|8|7|6|
|2022|29|15|12|11|
|2023|30|15|13|11|
|2024|30|22|17|17|
|Total|291|127|101|89|

These raw date counts differ from annual season-aware title counts and include no rewatch rows.89 noon markers comprise49films,14series,26anime; they are strong synthetic-date candidates, not proof that every December31 was fabricated.101 total December31 rows include12 with other times. No currently active canonical viewing assertions with year<=2024 exist. The one nonnull viewing_date among these legacy rows belongs to later2026 partial history, not an old calendar recovery source. No watched_at earlier than nonnull stored release_date was found; that is only a lower-bound consistency check, not proof of real watch dates.

## Provenance found in repository

- scripts/fix-watched-at.mjs explicitly writes latest season-year + December31 noon for completed TV/anime through2025.
- scripts/repair-watched-at.mjs also projects year-only information to December31 noon. It may use an airing boundary to remove impossible dates; airing is not personal watch-date evidence.
- src/modules/watching/lib/watched-date.ts buildWatchedAtProjection maps past YEAR precision to December31 noon, across media types. This also explains why TV-only scripts cannot account for all49film markers.

The exact old writer for each row cannot be reconstructed from these signatures alone. Scripts were READ, not executed. None of these old write scripts should be used as a real-date recovery source.

## Possible next correction work

Restore real dates only from personal exports/backups/watch diary/owner-supplied recollection, with source/confidence and an explicit reviewable mapping. Release/air dates do not establish when this owner watched something. No uniform/random month redistribution. If only the year is known, a product choice is needed: retain the previously accepted placeholder calendar date or represent YEAR precision honestly. This diagnostic does not override the locked policy accepting existing legacy dates; no precision downgrade or2E reconciliation is started.

Local ignored CSV: scripts/.shots/watching-dates-through-2024-audit.csv,291 rows, original date, candidate marker, empty proposed_date/evidence_source columns. No mutation script created, no guessed replacement date, no SQL apply, no observing receipt action, no Goals update, deploy or Git mutation.

Targeted estimate proposal2026-10-21: owner states no personal evidence exists and explicitly requests plausible synthetic dates; authorized offline proposal only, not a DB write or general2E work.89 strong31Dec/noon candidates reviewed with49 public TMDB release reads and stored TV dates.85 estimates prepared (2TV metadata-gap flags),4ambiguous rows left blank;202other dated rows unchanged. Dates are explicitly ESTIMATED_WITHOUT_PERSONAL_EVIDENCE, never recovered/confirmed history. Original291-row CSV preserved. Review files: scripts/.shots/watching-dates-through-2024-proposal.csv and watching-date-proposal-review.md. No mutation/migration/receipt observation, deploy or Git write. Application requires approval of this concrete mapping and preservation of estimated provenance; years unchanged.


## Approved estimated-date repair prepared (2026-10-21)

Owner approved the concrete 85-row proposal and requested implementation. Fresh read-only preflight confirmed all 85 original watched_at values and personal revisions match; full rows backed up locally in scripts/.shots/estimated-dates-before.json. Four ambiguous candidates and 202 non-candidates remain excluded.

Prepared data-only migration 20261021010000_watching_estimated_legacy_dates.sql. It locks the exact owner/org targets, checks old dates/revisions/history versions/viewing_date, and aborts the entire batch if stale. Only watched_at, viewing_date and the normal personal revision increment can change. Every estimate retains original values, proposal SHA256, INFERRED confidence and NONE personal evidence in viewing_date.legacy_date_estimate. Existing canonical viewing_date components (Bleach 2026) are preserved; no history assertions are invented. SQL checks fingerprints of all non-target media, viewing records/ranges/ratings, rewatches, Goals and receipts. Years, statuses, ratings, durations and canonical history stay unchanged. Reapplication fails safely. Schema reference watermark updated; all object counts unchanged.

Local PostgreSQL16 rehearsal passes with 85 seeded mappings: application/provenance, original canonical date preservation, unchanged rows/counts/object counts; a changed revision rejects the full batch without writes; repeat rejects without increment. Existing history/consumption/receipt RLS, concurrency and atomicity tests pass in the same disposable database. Saved 512-title personal-library simulation plus fresh 85-row backup confirms all-time and 2013-2026 annual counts, hours, rewatches, ratings, favorites and activity-event totals unchanged; month/day positions change only through the approved estimates. Results: scripts/.shots/estimated-dates-stats-validation.json. Affected-script ESLint, schema-reference and diff checks pass. No TypeScript/application code changes in this repair.

REMOTE APPLICATION STILL PENDING: repository policy reserves migration execution to owner via npx supabase db push --linked. No production writes, observer RPCs, deployment, Git mutation or Wave2E. After owner push, verify registry, exact 85 mappings, revisions, provenance and unchanged canonical data against the backup. This is estimated personal-date repair, not evidence recovery or a new final Wave2D review.
