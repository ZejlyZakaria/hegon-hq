# Best Episodes image quality — scoped follow-up

Owner authorizes only improving Best Episodes images. Other recent local UI/business changes remain rejected/unvalidated; this work does not approve, roll back or deploy them.

Changed Episodes.tsx: preserve the264px16:9card, hover scale, alt text, lazy loading and blurred no-still fallback. Native srcset uses TMDB's published w300still at1x (covers272pxhover) and original at2x/higher. This replaces the previous fixed500px delivery on modern high-density screens. The original exposes available native detail without requesting an artificially upscaled width for low-resolution sources. Original HD payloads may be larger; no zero-cost/performance claim. Existing tmdbImage formatter reused; w300added to its valid size type. No parallel formatting helper, metadata fetch, database/worker/schema change or poster/panel change.

Validation:45targeted image/UI tests PASS; TypeScript and affected-file ESLint PASS; DS coverage regenerated; diff check PASS. Browser source-selection fixtures at390/768/1024/1440 ×DPR1/2/3 (12cases) PASS using known400px/1600pxraster originals. Each image triggers one selected request,1xchoosesw300,2x/3xchoosesoriginal; no fallback double-fetch, overflow or changed frame. Mobile capture read. Fixtures prove source selection, not an authenticated page review, measured production latency or universal native image quality. Evidence: ignored scripts/.shots/best-episode-quality-browser.json and best-episode-quality-*.png.

Files: src/modules/watching/components/detail/Episodes.tsx; lib/tmdb-image.ts (size type only); lib/review-ui.test.ts. No commit, push, deployment or production mutation.
