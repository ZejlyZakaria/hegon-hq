# Detail page final polish - 2026-11-01

## Scope

Owner-requested local UI corrections only. No production data, schema, migrations, deployment, staging or commits.

- Seasons, Best Episodes, Cast & Crew and More Like This share the same rail bounds/header spacing. Seasons aligns cards at the top; More Like This and its skeleton no longer retain a different desktop gutter.
- Poster tokens are unchanged: --poster-lg is 9rem (144px at the standard root size), mobile uses --rail-peek. Best Episodes retains its separate 264px, 16:9 still frame. List tiles retain the owner-approved 65% width and three portrait cells.
- Shared navigation stops at the natural trailing edge. Removed synthetic tail spacers from Seasons, Best Episodes and In Lists. This supersedes the previous terminal left-alignment spacer: intermediate pages align to card starts, the final page aligns the last card to the right edge. A non-overflowing single list tile still occupies 65%; no stretching or fabricated filler.
- Empty Top 10 / In Lists use concise positive guidance and explicit Rank title / Add to list actions, keeping normal empty states compact.
- More details uses an accessible button, Lucide ChevronDown, animated height/opacity and reduced-motion support. Closed content is inert.
- My Take clamps the displayed review to four lines. Read more / Read less appears only for overflowing text, independently of the full-text editor/autosave.

## Verification

45 targeted tests pass (rail navigation, review UI, catalogue facts), TypeScript and affected-file lint pass, DS coverage regenerated (444 existing off-system hits), git diff --check passes. No broad suite required.

Actual-component browser fixtures cover 390/768/1024/1440px: all four rail bounds match; repeated/rapid navigation ends without synthetic blank space; long/short notes and disclosure controls work; empty actions remain accessible; no page overflow or JS errors. Separate list fixture verifies the last 65% tile ends flush right at all four widths. Screenshots inspected. These are deterministic local fixtures, not authenticated production end-to-end checks.

Owner review of the rendered localhost result remains the next step. Earlier unrelated working-tree changes remain intact.
