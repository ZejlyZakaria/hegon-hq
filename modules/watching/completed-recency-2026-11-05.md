# Completed recency / The Promised Neverland — 2026-11-05

Read-only production diagnosis confirmed the owner's row c44b17a3-916a-4ee0-a545-6dda422d370b: watched=true, watched_at=2022-05-23 12:00:00, season_years={1:2020,2:2022}; last_watched_at=caught_up_at=2026-09-19 14:43:25.894+00. No canonical viewing records or legacy rewatches for this row. A separate demo-org row was identified and excluded; these are not two copies in the owner's library.

Root cause proven: latestViewingAt used max(watched_at,last_watched_at) for every posture, so the legacy activity timestamp won over the completed viewing date. Details used the completion/year projection and therefore showed2020–2022. The older markCaughtUpPatch code could write current time for a claim, and old position handling could stamp frontiers during corrections. The exact September user action cannot be proven without an operation log; do not label it as known or a real viewing.

Fix: completed claims (watched=true) use only their valid completion projection; missing/invalid completion dates remain unknown. Ongoing titles retain max(completion,forward-viewing timestamp), preserving Silo's legitimate recency. Shared helper changes both rail order and time badge; no title hardcode, timestamp-equality heuristic, extra per-card history request or production mutation. Current correction writer/transaction guards already forbid manufacturing a new viewing date and are unchanged.

Live minimal SELECT of69owner anime rows, replayed locally:38LastWatched candidates; target before rank2/date2026-09-19, after rank14/date2022-05-23, outside the displayed ten. Existing dates, years, notes, ratings, position and history remain untouched. No migration required.

Validation:152targeted tests across4files PASS, including exact legacy timestamp/unit immutability, unknown completion, ongoing/new completion preservation, real service ten-item cap and existing correction guards. TypeScript, affected-file lint and git diff whitespace checks PASS. No live browser/prod deployment claim. Raw legacy activity timestamp is retained for audit; this fix corrects its interpretation, not its stored value. No promise that all possible date errors in all other fields are impossible.

Files: src/modules/watching/lib/series-state.ts, series-state.test.ts, src/modules/watching/service.test.ts. No Git mutation or remote writes. Updates page remains pending.
