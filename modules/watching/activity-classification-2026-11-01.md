# Watching activity classification ? 2026-11-01

Owner requests the generic fix for A Knight of the Seven Kingdoms appearing in In Progress despite completed viewing. The preceding read-only remote check found personal S1E6, aired/announced [6], stored status ended, watched=false/in_progress=true, watched_at 2026-04-06 and rating 8.5. No provider status claim is made beyond that stored catalogue evidence.

## Root cause and replacement

The series state engine computes completed, while In Progress previously excluded only caught-up awaiting-season titles. Last Watched queried watched=true and merged only awaiting-season candidates. Both capped candidates before classification, potentially losing matching titles.

activityBucket in series-state.ts now owns rail classification from actual aired progress. Completed or awaiting-next-season titles belong in Last Watched; behind/new and weekly followers belong in In Progress. Unknown catalogue data retains stored posture, films retain existing semantics, paused/dropped and unstarted titles are not promoted. The existing service reads paginated lean engaged candidates, classifies before sorting/capping, and the two components no longer implement parallel filters or merge logic. Stats status counts consume the same rule. No personal flags are rewritten to hide an inconsistent projection.

## Verification

122 tests in activity-bucket.test.ts, series-state.test.ts, service.test.ts and computeStats.test.ts pass. Regression includes the exact personal Knight state, season wait versus weekly airing, actual growth with stale watched posture, unknown catalogue, paused/dropped, films, >1000-row pagination and classification before a ten-card cap. Knight remains 204 minutes with its original date and unchanged input; appears in the completed status count. No production mutation, schema change, migration, deployment or Git mutation. Last Watched remains limited to the ten most recent matching titles: an April completion need not be visible in November's top ten.

Localhost route did not respond within the initial 3-second connectivity probe; authenticated live rendering is not claimed. Existing unrelated/rejected UI changes and pending scoped-rating rollout remain separate.
