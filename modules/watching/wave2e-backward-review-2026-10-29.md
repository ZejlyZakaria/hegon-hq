# Wave2E backward migration - targeted pre-application review, 2026-10-29

Current rollout2026-10-29: generic migration20261029000000 and separately approved three-anime data repair20261029010000 APPLIED and verified;33/33versions aligned, no pending. WAVE 2E LOCKED / COMPLETED by owner approval2026-10-29. See wave2e-final-reconciliation-2026-10-29.md. Earlier no-application/three-anime-untouched verdicts below are historical.

Follow-up2026-10-29: R1 is corrected locally and its exact regression plus mutation control pass. Current status READY FOR RE-REVIEW; see wave2e-r1-fix-2026-10-29.md. No remote application. The verdict and candidate identity below describe the original failing review, not the amended candidate.

Original verdict: NOT LOCKABLE. Migration verdict: NOT SAFE FOR REMOTE APPLICATION until findingR1 is corrected and re-reviewed. No application attempted; candidate file unchanged throughout review. This review supersedes the preceding locally-validated verdict for deployment readiness.

## R1 - retained identity loses its active projection when another invalid identity has the same ordinal and score

Priority:P1, blocker. In20261029000000_watching_backward_correction.sql, the score-clear lineage loop correctly distinguishes frozen membership. But its compatibility-map cleanup at lines339-340 checks only scope_number_at_recording and numeric score. It does not prove that this active map slot belongs to the invalidated scope_key.

Concrete disposable PostgreSQL/authenticated-role counterexample, extending the existing remapped-ordinal regression:
- IdentityA anilist-cour:993900 has frozen storage membership S1E1-12, score7, recorded ordinal3. Its canonical opinion and completion remain valid after correcting storage episode36to24.
- Older identityB anilist-cour:994900 also has recorded ordinal3 and score7, but frozen membership S1E25-36. It must be retracted/cleared.
- The later active compatibility slot cour_ratings[3]=7 represents retainedA, which the initial map pass preserves using its latest frozen opinion.
- The subsequent lineage pass clears invalidB, then removes slot3 solely because its numeric value also equals7. It never restores retainedA.
- Expected active map {"3":7}; actual {}; retained canonicalA opinion count remains1. Prior lower-level regressions pass in the same run. This is active opinion loss, not a hypothetical concern or deletion of its immutable original.

Ignored proof: scripts/.shots/review-backward-identities.mjs and backward-review-identities.log. No real owner/media data used; fresh disposable supported Watching schema, real PostgreSQL and authenticated-role command. Database removed and local test server stopped. This is not full empty-Supabase replay (separate deferred debt).

Required next correction: bind an active compatibility slot to the retained/invalidated stable subject before removing it; on ambiguity preserve legitimate/unknown state. Add regressions with two different identities sharing ordinal AND score, alongside different-score controls and frozen-membership preservation. Do not weaken retraction/audit/security guards. This review did NOT modify the candidate or implement a fix.

## Other reviewed properties

The candidate DDL is additive: audit table/policy/indexes/private helper plus replacement of existing transactional RPC. No top-level user DML, no data backfill, no three-anime identifiers. Audit owner/org policy, helper execution restrictions, existing revision/operation guards and independent-rewatch handling are present. Existing PostgreSQL backward/receipt/history/rewatch suite passes beforeR1 is exercised. These positives do not overrideR1 or justify application.

## Pre-migration audit

Linked project is HEGON femvhonlpafdajyamvcu. Fresh read-only Management endpoint registry:31remote versions,32local files; only20261029000000 pending. No remote-only version, no historical replay. Previously restored20261028010000 is recorded remotely. Candidate remote registration count0 and progress_corrections absent. Remote current personal_history_command body matches the prior applied20261028000000 generic command after whitespace normalization.

Official cached CLI2.121.0 dry run succeeds:

npx --offline supabase@2.121.0 db push --linked --dry-run

Plan contains ONLY20261029000000_watching_backward_correction.sql, seeds=[],roles=[]. No actual push or migration repair. Local schema-reference/diff checks pass; planned post-migration counts82/867/226/50/279/20 are not current remote counts (81/857/225/49/277/20).

## Existing data verification

Captured deterministic count/SHA256 fingerprints before and after this review using only SELECT, one SQL snapshot per capture. Scope: owner/org Watching personal fields (provider metadata/update timestamp excluded from media hash), viewing_records, owned viewing_ranges, scope_ratings, rewatches, achievement_unlocks and episode_highlights. All7fingerprints match exactly. Counts:518owned media rows (includes references),21records,56ranges,4scope ratings,10rewatches,3receipts,337highlights. These are fresh current counts, not the earlier515personal-nonreference inventory.

Full JSON values of Tokyo Ghoul, The Seven Deadly Sins and Fire Force match before/after; no correction applied. Candidate remains absent from the remote registry after review. Existing data changes:NONE observed within the compared scope. This is after-REVIEW verification, NOT post-MIGRATION proof because no migration was applied.

Ignored before/after proof: backward-migration-before.json, backward-migration-after-review.json, backward-migration-verification.sql, backward-migration-review-hash.txt. They contain private snapshots and must not be staged. No git add/commit/push, production mutation, observer invocation or deployment performed.

## Next step

FixR1 locally and repeat the targeted review before any remote application. Only after a clean review, owner-run single-migration push and unchanged-data post-verification may the next business step be controlled correction of Tokyo Ghoul, The Seven Deadly Sins and Fire Force. Those corrections remain a separate task and require explicit approval. No push command is issued whileR1 blocks readiness.
