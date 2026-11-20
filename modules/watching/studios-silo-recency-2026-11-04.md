# Studio disclosure / Silo rail recency - 2026-11-04

Owner reports migration20261103000000 and watching-anime-cours-sync deployed successfully; not independently reverified here.

Animation studios now starts as one aligned summary row. Multiple scoped entries expand via accessible animated disclosure, grouped by studio with original AniList titles. Uniform complete credits stay static. Uses existing cached data; no new provider query or migration.

Read-only production diagnosis scoped to owner Silo and same-user series candidates: Silo is S3EP10, aired[10,10,10,0], announced[10,10,10,1], ongoing, engaged and not suspended. Last-view/caught-up stamps2026-11-04 but legacy watched_at2024-12-31. Existing classifier correctly assigns last-watched; null-coalescing recency wrongly prioritised the old date. Real-data replay: rank12 before, rank1 after. Service now takes newest valid viewing timestamp across watched_at,last_watched_at,caught_up_at; updated_at remains only for In Progress. No data repair or production mutation.

Validation:72targeted tests (service,series-state,catalogue-facts), TypeScript, affected ESLint, diff check pass. Regression reproduces exact Silo season arrays and dates with ten competing titles, invalid-date fallback and caught-up-only stamp.28actual Details component scenarios across390/768/1024/1440 pass, disclosure open/close verified, mobile screenshot inspected. DS coverage regenerated with442existing off-system hits and no tracked change. No commit/push/deploy performed.

Next owner topics: intermittent Details page flash; additional business concern about newly aired episodes moving a title back into In Progress. Do not infer or implement that unspecified change yet.
