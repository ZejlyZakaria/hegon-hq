# Watching personal card and contextual viewing panel — 2026-10-30

Status: implemented locally following owner approval of the revised mockup. Visual acceptance by the owner remains pending. This replaces the rejected per-event history presentation; it does not approve the separate pending scoped-rating migration or robot rollout.

## Product contract

- The petrol personal card owns current state, favorite heart and the next useful action. Progress correction is an explicit edit, routed through the existing `useWatchActions.setPosition(..., "correction")`. Forward episode consumption uses the shared `nextStep` and the existing viewing writer.
- One SlidingPanel owns viewing details, viewing history, rewatch logging/editing and position editing. Forms no longer expand the card. Caught-up waiting text is information, not a button. Completed and caught-up remain distinct states.
- Viewing details appears only for actually consumed nested parts; ordinary seasonal information remains on the existing season posters/panels. Viewing history appears when a rewatch exists, including an independent rewatch retained after progress correction. No redundant film/ordinary-series history entry, and no history for an unconsumed title.
- The panel presents one first-view summary, optional season/part details and separate frozen rewatches. Completion milestones, retractions, recording dates, raw storage coordinates/enums and rating audit assertions stay internal.
- Parts come only from the existing MediaView grouping, never release-gap guesses. An aggregate legacy season year is not a proven viewing date for each nested part. Unknown stays unknown; partially dated coverage explicitly says dates are incomplete. Unconsumed parts are omitted.
- New rewatch form previews reject explicit membership beyond accepted first-view coverage. Existing independent frozen rewatches remain editable. Historical episode availability and frozen runtime checks remain in the existing rewatch builder. This UI check is not a new server-side constraint.
- Where to watch is a separate graphite block, showing actual returned provider names/logos and the returned availability link. Other consumers keep the existing inline provider presentation.

## Shared calculation, not a second hours engine

`seasonContributions` moved unchanged from computeStats to the existing legacy-consumption library. The shared historical contribution ledger now exposes its exact accepted storage ranges, including masked legacy remainders and uncovered completion fallback. Completion fallback ranges have no invented runtime. The display groups this ledger; it neither changes attribution rules nor writes new facts. Legacy coordinate offsets use the same frozen coordinate helper as Stats. Frozen canonical aggregate durations are not arbitrarily divided at an unrecorded part boundary.

The old public event timeline was replaced, rather than retained alongside the grouped projection. Existing first-watch date defaults, image improvements, ratings and all unrelated working-tree changes are preserved.

## Evidence

- Targeted regression coverage: Bleach Part4 chunk grouping, mixed legacy/history masks, completion fallback, tombstones, film first-view precision, four distinct GOT rewatches, unknown/partial dates and durations, overlay coordinates, independent rewatches and consumed-only nested parts. Existing Stats/date/accepted-consumption/UI regressions included.
- Browser harness uses actual React components, shared SlidingPanel/Radix controls and generated application CSS, with deterministic read-only data/mutation stubs. At 390 / 768 / 1024 / 1440: panel navigation, rewatch mode, no redundant/unreleased history, forward vs correction callbacks, caught-up/paused/dropped states and overflow/error checks pass. Screenshots are read, not merely saved. This is not an authenticated end-to-end production-page test.
- Saved post-Kuroko snapshot replay: 516 items, identical first-view minutes/episode totals to the accepted Stats ledger, immutable input. Actual saved Bleach: total 9,984 min; S1 retained; Part4 EP41–50 = 240 min / 2026; Part1–3 dates remain unknown. GOT retains four rewatches; Hannibal has no redundant history entry. The saved snapshot predates some recent interactions; it is not a fresh remote verification.
- Artifacts: ignored `scripts/.shots/right-card-browser.json`, `right-card-*-{390,768,1024,1440}.png`, `right-card-real-snapshot-replay.json`.
- TypeScript, affected-file lint, Knip CI and regenerated design-system coverage checked. Final targeted test and whitespace results are recorded in the completion response.

No production read/write, data correction, migration application, deployment, Git staging, commit or push in this implementation. No new migration is needed. Existing local `20261029040000` remains a separate unrolled-out change.

Next: owner reviews the actual localhost card/panels, especially density and date wording. Do not mark this UX accepted based only on technical checks.
