# Wave2E - three anime awaiting owner clarification, 2026-10-29

Fresh read-only snapshot captured2026-10-28T23:21:27.270401Z (2026-10-29 00:21:27 Paris). SELECT only, limited to the three personal titles and their stored grouping/history/opinions. No provider call, mutation, receipt observation, deploy or Git mutation. Exact raw IDs/evidence in ignored scripts/.shots/wave2e-three-anime-current.json and wave2e-three-anime-results.json.

All three use ordinary TMDB season coordinates in HEGON (overlaid=false), with no cour year map. No canonical viewing/rating assertions exist for these titles in this capture. Their accepted legacy coverage confirms the stored analytical baseline, not independently verified real-life viewing. PROVEN: stored fields disagree. UNKNOWN: which field needs correction and what was personally watched.

| Title | Stored status and position | Recorded season years | Accepted coverage used by Stats | Inconsistent-looking entries |
|---|---|---|---|---|
| Tokyo Ghoul | Dropped, S2E12 (S2 has12episodes) | S1=2017;S2=2017;S3=2019;S4 unknown | [12,12,0,0],24minutes each;9.6h in2017 | S3=2019 lies beyond current cursor; S3 also has a saved7.5 opinion |
| The Seven Deadly Sins | Dropped, S2E24 (S2 has24episodes) | S1=2017;S2=2019;S3=2024;S4=2021 | [24,24,0,0],24minutes each;9.6h in2017 and9.6h in2019 | S3=2024 and S4=2021 lie beyond cursor; reversed years alone do not disprove nonsequential viewing |
| Fire Force | Paused, S1E18 out of24 | S1=2024;S2=2025;S3 unknown | [18,0,0],24minutes each;7.2h in2024 | S2=2025 lies beyond current cursor |

Tokyo Ghoul saved season opinions: S1=8.5,S2=8.5,S3=7.5. Fire Force S1=8.5; noS2/S3opinion. The Seven Deadly Sins has no saved season/cour opinions. These are existing legacy opinions, not proof of completed seasons; do not silently delete or promote their scope.

## Proposed dispositions - none applied

Tokyo Ghoul: keep current data until owner clarifies whether S3 was finished, partly viewed or never viewed, and whether2019 is correct. The saved S3opinion is an additional reason not to assume an accidental year. If owner confirms onlyS1/S2were seen, propose retiring the erroneous S3year through an audited correction; ask separately what the saved S3opinion represents, never erase it automatically. If S3was viewed, reconcile the cursor and missing first-view coverage instead of deleting2019. S4remains unknown absent owner facts.

The Seven Deadly Sins: ask which seasons/episodes were actually seen and whetherS3=2024/S4=2021 are first-watch years or a mistaken/rewatch attribution. If onlyS1/S2were seen, propose retiring the two erroneous later stamps with retained audit. If S3/S4were seen, reconcile personal history and cursor to those facts; do not reorder the years or infer missing episodes merely to make the sequence chronological. Non-contiguous consumption must not be represented by a falsely complete linear cursor.

Fire Force: ask whether progress really stops atS1E18 or whether anyS2episodes were seen in2025. If S2was never viewed, propose retiring only its erroneous2025stamp; retain S1year/position/status/opinion unless separately corrected. If S2was viewed, record exact supported coverage and reconcile the cursor accordingly; do not infer fullS1/S2completion from an existing year stamp.

## Verified current analytical impact

Actual current buildMediaView/computeStats replay: Tokyo Ghoul2017=9.6h/Top Picks8.5 S1-S2,2019=0h/no pick; The Seven Deadly Sins2017=9.6h and2019=9.6h,2021/2024=0h/no pick; Fire Force2024=7.2h/Top Picks8.5 S1,2025=0h/no pick. Missing months remain missing. The initial whole-library offline probe already showed removing the four out-of-position stamps from copies changes no hours/counts/activity/ratings/Top Picks. Removing them would therefore be a data consistency correction only, not a known current-Stats repair. Conversely confirmed additional viewing may legitimately add hours; do not promise unchanged totals before the owner supplies those facts.

Stop here before correction. Owner supplies real viewing facts; next prepare a concrete before/reason/after mapping with expected Stats deltas and approval before any write. Bleach remains outside personal reconciliation. Full empty-database reconstruction is separately tracked technical debt and does not block2E.
