# Watching — restore saved season ratings and stabilize the season panel

Owner reported that all legacy season/cour scores disappeared from posters and panels, and that expanding content stretched/cropped the poster. Discussion first; subsequent explicit go authorized this correction only. No final Wave2D review or Wave2E started.

## Cause and policy

The preceding point1 change required CURRENT membership and write-ready episode availability for both displaying a saved score and editing it. Legacy scores legitimately have no frozen membership and may have no accepted episode catalogue, so SCOPE_UNVERIFIED hid them even without evidence of growth. That conflated incomplete evidence with an obsolete opinion.

Read-only remote verification confirms JUJUTSU KAISEN still stores cour scores 8.5 / 9 / 9.5; no score was lost or deleted. Display now accepts saved unverified opinions for the same subject, while keeping verified snapshot/membership requirements on writes. The server's existing suppression of remapped legacy ordinal fallbacks remains authoritative; known different cour identities/ranges cannot inherit an old score. A truly grown scope (NEEDS_RECONFIRMATION) still moves its old note into history and requires an explicit new opinion/confirmation. No analytics eligibility, annual rating calculation, provider evidence, DB row or API write policy is weakened by showing the saved score.

## UI and code

- `scope-ratings.ts`: shared identity checks, separate read-score and write-eligibility functions, same policy for season/cour/part presentation. Missing catalogue information cannot erase an existing unverified score; stale CURRENT context and known remappings remain guarded.
- `SeasonPanel.tsx`: catalogue card with normal-flow 2:3 poster,96×144 on mobile /120×180 from sm; image no longer fills a content-driven absolute height. Restored the owner-approved poster-derived blurred background behind catalogue facts, clipped and darkened for readability. Removed the growing artwork frame and redundant personal facts in that frame; the foreground poster keeps its fixed ratio. Rating and watched period use a separate personal-record section. The period appears once, with the existing year editor. Episode/part navigation remains.
- Saved legacy score is visible as **Your rating**, on poster cards and in the panel. Optional **About this rating** uses one plain explanation, with a small refresh action inside when needed. No duplicated “Scope unverified” / “Rating scope unverified”, no blanket “Previous rating” for legacy scores and no large refresh CTA competing with the record.
- Corrected the grown-scope confirmation callback: it submits the retained previous score, not the empty current display value0. The existing transactional rating command remains the only writer.
- Targeted test updates cover saved series scores, unbound anime slots, remapping refusal, unknown write evidence, part ratings, true growth and plain disclosure copy.

## Validation

169 targeted tests /7files pass, including historical hours/annual scoring regressions. Actual React components exercised in an isolated browser fixture for anime legacy9.5, series legacy9 and a grown season with prior9:12cases at390/768/1024/1440. Notes verified on the season poster and in the panel; grown9 stays behind history; explicit keep action delivers9 to the callback. Poster ratio exactly1.5 at all widths and dimensions unchanged when disclosure opens. No horizontal overflow or page errors. Seven representative screenshots read, including the four anime widths and series/growth examples.

Fixture uses fresh read-only JJK media/cour metadata and its public reference poster, served locally; provider states/episode rows and mutation callbacks are controlled. No authenticated production-backed mutation smoke test, deployment or browser observer invocation. Ignored evidence: `jjk-season-ui.json`, `season-polish-browser-results.json`, `season-polish-*.png`. No new migration/edge change, production data mutation, staging, commit or push.

Manifest: `src/modules/watching/components/detail/SeasonPanel.tsx`, `lib/scope-ratings.ts`, `lib/provider-ratings.test.ts`, `lib/scope-rating-ui.test.ts`, `lib/season-history.test.ts`, generated styleguide coverage. Earlier point1 files remain dirty and intact. Worker deployment from the preceding block is still a separate owner step, not performed by this correction. Next: owner discusses the result before final2D validation.


Final checks: TypeScript `--noEmit --incremental false`, affected ESLint (zero warnings), Knip, regenerated design-system coverage and main/HQ `git diff --check` pass. Knip retains its three pre-existing configuration hints. No production build or broad suite required for this targeted presentation correction.
Owner-requested blur restoration: repeated all 12 isolated browser cases across four widths, read the four anime screenshots, confirmed unchanged 2:3 poster geometry and no browser errors. Affected ESLint, regenerated design-system coverage and diff checks pass.
