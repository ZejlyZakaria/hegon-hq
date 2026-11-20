# Read-only stats diagnosis - 2026-11-08

Owner requested actual data verification and proposed replacement logic, not a fix yet. Fresh owner/org-scoped read via Supabase Management read-only endpoint at2026-11-08T18:02:59Z; current application computeStats and includeRetainedHistory replayed locally with diagnostic-only in-memory ledger exposure. Application code and production data unchanged.

## Film counters

Reproduces43film activity titles,40first-view films and4rewatch events. Obsession has stored first watch2026-05-30 and rewatch2026-10-02. Identity first2018, rewatch2026-07-23; Interstellar first2019, rewatch2026-04-07; Fight Club first2024, rewatch2026-04-01. Hence39other first films + Obsession +3older rewatched films =43unique activity titles. All4rewatches remain events; Obsession must not be subtracted from the40first films.

Recommended semantic alignment pending owner go: annual Films/Total watched goal-style KPIs use first-view cohort, with rewatches separate; preserve activity hours/event counts including rewatches. Distinguish title count from event count. Film50goal should use40first completed films, not43activity films or44events.

## Missing annual scores

Fresh snapshot: Gachiakuta global8, Rafa8, Friends8.5, A Knight of the Seven Kingdoms8.5. Their season/cour rating maps are empty and scope_ratings contain no assertions. Gachiakuta has cour/season1year2026; Rafa season1year2026; Friends all10seasons2026; Knight has no season-year map but first-view contribution in2026 from stored watched_at2026-04-06. Accepted catalogues: Gachiakuta1TMDB season24/24 with1AniList cour; Rafa1season4/4; Knight1season6/6; Friends10fully aired seasons. The single-scope equivalence function exists in20261029040000 and scope_rating_context can alias global score without map writes; annual stats read only ratingMap/yearMap plus historical assertions and miss this equivalence. No claim that the owner never rated a season; current persisted values are global-only.

Current replay annual scores null for those4;48rated average7.588541666666667. All4still enter Top Picks through favorite=true. Fifth unscored favorite is Black Clover: global8.5 and S1score8.5 belong to S1first2022;2026consumption is S2EP1-6 of26, noS2score. Its missing annual score is correct; borrowing S1global for2026would be wrong.

## Proposed resolver (not implemented in this diagnostic)

One shared period rating resolver for Top Picks/distribution/average/Wrapped/hour rows: explicit dated valid season/cour opinion first; absent that, read-only global equivalence for a proven single consumed scope, or global fallback when the entire finished rated work was first-viewed in the same selected year (Friends). Require matching coverage and preserve historical/retracted/remapped rules; never infer a partial-season score from global. Derive dates from accepted first-view ledger, not only legacy year maps. Reuse authoritative single-scope proof through a batch read; no per-card RPCs or copied fake season notes. Recommend annual Top Picks require an applicable score even for favorites; keep Black Clover's unscored2026progress out of that rating ranking.

The private whole_rating_scope could not be directly executed under the read-only endpoint role (permission denied). No privilege workaround or role change attempted. Accepted catalogue facts were then SELECTed directly; actual alias response was not claimed verified. Snapshot/diagnostic/catalogue proof remain ignored under scripts/.shots/. No migration, application patch, data write or Git action.
