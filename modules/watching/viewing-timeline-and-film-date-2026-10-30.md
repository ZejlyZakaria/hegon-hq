# Viewing timeline and film-date default

Status: locally corrected and validated. No production mutation, migration application, deploy or Git mutation.

## Owner report and read-only diagnosis

The old `ViewingHistory` exported from `RewatchForm.tsx` exposed the persistence journal: raw enum values, Season0 movie coordinates, retracted rows, repeated zero-time completion assertions from rating edits, and all rating-audit assertions. Ranges were printed in database return order. A second rewatch popover duplicated this information and its actions.

Remote read-only inspection identifies Boogie Nights (156minutes): the row has a watched_at projection, while its active FIRST_WATCH and COMPLETION both have UNKNOWN personal dates. The previous Mark-as-watched claim path did not pass an explicit date to history capture. A recording timestamp must not silently become the historical watch date.

Owner explicitly chooses a new rule: marking a film watched means today by default; the date remains editable for past viewings. An explicitly supplied year/month/day or unknown date must be preserved. No automatic historical backfill was approved or applied; Boogie Nights' pre-existing canonical date remains unchanged and can be confirmed through the existing Watched picker.

## Changes

- New `lib/viewing-timeline.ts`: pure user-facing read model over the existing records. Uses the existing first-view deduplication; excludes tombstones and available-scope rating milestones; attaches a full-title finish to its terminal viewing where evidenced. Never sums a completion as another consumption. No changes to analytical source data.
- Films show one first viewing, Entire film and a human duration, with no episode/season coordinates. Missing dates stay visibly unset. Legacy-only films may still use their saved date without pretending to have a canonical dated assertion.
- Bleach-shaped supplied journal becomes three bounded actual viewings:2026/S2EP41–48,20Oct2026/S2EP49,28Oct2026/S2EP50 with Series completed. Neither a416episode watch on the finish day nor the three50episode rating milestones become extra viewings. No personal part-year reconciliation.
- GOT retains four separate rewatches, ordered by viewing date. Whole-title entries are concise, e.g. Entire series ·70h32m. Optional episode details use sorted display coordinates; anime overlay summaries share the existing MediaView lens.
- New `components/detail/ViewingHistory.tsx`: shared controls, dates formatted en-GB, compact readable rows, mobile/coarse-pointer action targets, loading/error/empty states, per-rewatch edit/complete/remove menu. Keeps existing correction/removal handlers.
- Removed the old raw component and duplicate rewatch popover/unused imports from RewatchForm/StatusCard. No public rating-audit dump. Scoped opinion audit remains in storage and the existing contextual rating receipt; Stats' canonical opinion read is unchanged.
- `getMediaHistory` no longer fetches unused rating assertions. The shared timeline/season-period query reads records and ranges; a service contract verifies exactly those two requests and zero scope_ratings requests.
- `completionDatePatch` centralizes the prospective film default for updates and inserts. The existing atomic personal command receives matching viewing_date/FIRST_WATCH/COMPLETION dates. Editing an already watched film, historical corrections, and explicit past/unknown dates do not acquire today. Series completion evidence rules are unchanged. Film first viewing carries WHOLE_TITLE scope.

## Files in this follow-up

- New: `src/modules/watching/lib/viewing-timeline.ts`, `viewing-timeline.test.ts`, `components/detail/ViewingHistory.tsx`.
- Changed: `lib/completion-date.ts`, `completion-date.test.ts`, `viewing-history.ts`, `review-ui.test.ts`, `service.ts`, `service.test.ts`, `hooks/useMediaHistory.ts`, `components/detail/StatusCard.tsx`, `RewatchForm.tsx`.
- Design-system coverage regenerated. Prior image/scoped-rating/Kuroko changes remain separate existing work. This follow-up adds no migration and does not apply pending20261029040000 or deploy its robot.

## Validation

-216 targeted tests /7files PASS (timeline, completion date, capture, UI, service, consumption and Stats).
- Boogie-shaped156minute film: one displayed viewing, duration2h36m, no Season0; UNKNOWN not fabricated from recorded_at.
- Prospective film insert and update both send one atomic dated command, including local date across midnight, explicit historical/unknown preservation and no redating on rating edits.
- Exact supplied Bleach shape: three bounded viewings,240known minutes, no repeated milestones/retracted rows, immutable input. GOT: four events, dates/ranges sorted,4232minutes formatted70h32m. Overlay coordinates, tombstone compatibility suppression and independent finish dates covered.
- TypeScript and affected-file ESLint max-warnings0 PASS. Knip CI/schema-reference/diff checks PASS; existing three Knip hints remain. Coverage regenerated.
- Isolated actual component/DS/CSS browser fixtures at390/768/1024/1440: row counts1/3/4, no raw journal codes or horizontal overflow; mobile/desktop screenshots read. Evidence: ignored `scripts/.shots/viewing-timeline-preview.html`, `viewing-timeline-browser.json`, `viewing-timeline-*.png`.
- This is not an authenticated deployed-page or production-write test. No provider invocation, invented date, discarded audit row or user-data mutation.
