# Watching Updates — 2026-11-05

Implemented locally at `/perso/watching/updates`. This revision supersedes the initial horizontal-row/unseen-backlog implementation rejected during owner review. No migration, worker deployment, production data mutation, staging or commit.

## Final product rules

- Available now is release news, not a backlog: one evolving card per explicit season/accepted anime part, with unseen dated episodes. Keep it during a confirmed airing schedule (including a dated hiatus), then through 30 days after the latest release. An old fully released season disappears even if unwatched. Unknown episode dates cannot prove freshness; personal edits and robot observation timestamps never substitute for release dates.
- Weekly release: EP1 available, then the actual unseen episode count, then Full season/part available when all expected members are proven released. Fully consumed scopes disappear. No per-episode duplicate cards or inferred parts from airing gaps.
- Coming next remains chronological and grouped by month. TBA uses explicit catalogue scopes only. Paused announcements remain visible by default; paused available content and all dropped works require the existing opt-in. The Gentlemen S3 is consequently visible without resuming anything.
- Existing buildMediaView is the coordinate source, including flattened anime and accepted nested parts. Premiere uses the first member, never a later episode or parent-season date for a later part. No fabricated dates or next season from returning status alone.
- Vertical poster cards use the existing Seasons desktop token `--poster-lg` (144px), 2:3 ratio, shared image density helper (up to w500), and responsive 2/3/4-column grids. Grid width is bounded to avoid spreading small posters across enormous tracks; individual posters never stretch beyond the token. Month groups and text remain outside artwork. Matching skeleton geometry and section-scoped initial image reveal retained.
- Continue watching opens an already active work without mutation. Resume watching uses the existing central writer and opens its detail page only after success. Neither action changes the selected scope/episode cursor or invents consumption dates. One page-level writer, no hook per card.
- Global breadcrumb now includes Watching / Updates.

## Read and cache architecture

Scoped user/org query with Range pagination; batched shared cours and accepted TMDB heads/snapshots for remaining scopes. Candidate selection uses the same lens/scope iterator before date filtering, including ordinary TV and weekly episodes. This replaces the previous anime-only evidence optimization, which was insufficient for release freshness. No provider calls during render, N+1 query or schema change. Previous 6-catalogue payload measurements describe the superseded implementation and are not performance claims for this revision.

Updates query invalidates on all personal series/anime edits (revision-safe Resume targets), foreground refresh every 5 minutes, stale 1 minute/gc 1 hour. Cached content remains visible during refresh.

## Validation

- 82 targeted tests across updates, service and actual writer/cache UI regression files pass. Scenarios: The Bear June bulk drop vs November backlog; exact 30-day boundary; Silo premiere to complete season; weekly unseen count/caught-up removal; dated hiatus; missing/invalid/future dates; paused/TBA/drop visibility; canceled/ended scopes; accepted anime overlay/parts and immutable inputs; TV catalogue batch reads; selected Resume and cache revision invalidation.
- `node scripts/test-watching-updates-ui.mjs`: actual components, deterministic transport, 390/768/1024 touch and 1440 mouse. Asserts 2/3/4 columns, 144px poster token, no overflow/runtime errors, loaded images, paused announcements vs opt-in, type filters, Continue without write, Resume/navigation, loading/error/retry/empty and skeleton/card dimensions within 1px. Captures inspected.
- Authenticated localhost read-only smoke at all four widths succeeds. Desktop/mobile screenshots inspected: The Bear absent, The Gentlemen S3 TBA visible, Silo S4 dated, breadcrumb present. HTTPS writes were blocked; no live action buttons clicked. Sentry POSTs intentionally blocked. Initial sandbox attempt could not reach Supabase/CDNs; authorized network-enabled read-only retry succeeds.
- Saved real-owner 86-work/47-cours/86-accepted-catalogue replay confirms the same cases without modifying inputs. Catalogue release dates are stored provider facts, not an independent external release-date audit.
- TypeScript, affected-file lint, Knip CI, schema reference and whitespace checks pass. DS regenerated (26 primitives, 13 modules, existing 442 off-system hits). Production build PASS with the Updates route present; network-enabled retry was required only for the Inter Google Font. Existing middleware-to-proxy deprecation warning unchanged.

## Scope and next step

Owner visual review of the revised Updates page, then commit/push when accepted. The earlier completed-recency fix remains a separate pending workstream in series-state.ts/tests. Unrelated debug.log excluded. HQ is a separate repository. No user data changes, historical Bleach redistribution or Python work in this task.


## Owner review supersedes visual direction — first-section implementation

Owner rejected the plain poster grid. Following discussion of reference mockups, requested section-by-section implementation, starting ONLY with Available now. Coming next/TBA remain at their previous rendering until separately reviewed; the planned compact poster-left cards for those sections are not implemented.

Available now now uses four backdrop cards per desktop viewport, three at landscape tablet, two at portrait tablet and 85%-width cards with a next-card peek on mobile. Shared aligned rail navigation, touch/keyboard scroll, shared header/arrows and See all gallery panel beyond four results. Title/scope/availability sit on a bottom-only dark gradient; a small secondary action sits below artwork. Native density srcset is retained; missing backdrop uses a contained poster without cropping it into a false landscape asset. Initial reveal remains grouped per section; matching aspect-ratio skeleton cells.

Recent series/anime Want to watch releases now participate in Available now, preserving the same dated freshness rules. No old backlog, status mutation or invented viewing dates. Watchlist cards navigate through View details, active work through Continue, other work through the existing explicit Resume. Service reads include watchlist rows and backdrop_url, scoped/batched as before. Watchlist upcoming/TBA announcements are deliberately deferred with the later sections in this incremental rollout.

An empty Available now is completely hidden; no empty title/card. No data/schema/edge/Git/deploy mutations. Removed old Available branch/actions from the generic announcement poster component and unused skeleton action branch.

Validation: 82 targeted tests across updates/service/writer passed; after final service query assertions the 43 updates/service cases reran successfully. TypeScript and affected-file lint pass. Updated actual-component browser harness passes four formats (390/768/1024/1440): carousel geometry, navigation, 6-result gallery, watchlist navigation without mutation, Continue/Resume, grouped images, skeleton geometry, filters/error/retry and hidden empty section. Authenticated read-only localhost captures checked desktop/mobile with real watchlist results; no application read failures, Sentry POST intentionally blocked. DS regenerated. Prior full build evidence above belongs to the previous revision; no new production build claimed for this isolated section change.


## Coming next / TBA compact cards — owner-authorized follow-up

Both announcement sections now share a compact graphite card: natural80px poster at2:3 on the left, title/scope on the right, bottom-aligned calendar date plus the existing releaseCountdown label for Coming next, quiet Date TBA for undated announcements. Whole-card link; no follow/notification buttons. Chronological month groups and all eligibility/data rules unchanged. Grid1mobile/2tablet/4desktop; Available now untouched. Replaced the rejected isolated144px announcement posters rather than retaining duplicate variants.

Skeleton announcement cells match poster ratio, card dimensions and breakpoints. Actual-component browser harness now covers four announcements in a month, long titles,80px/2:3 geometry, four-format1/2/4columns, no announcement buttons, matching skeleton height/width, plus all prior Available interactions. Four-format checks pass; desktop/mobile fixture screenshots and authenticated localhost screenshot inspected.21pure Updates cases, TypeScript/affected lint/DS/diff pass. No production data/schema/Git/deploy action. Owner reviews all three sections next.


## Shared MediaCarousel / watchlist announcements — latest owner-approved revision

Owner requests actual shared MediaCarousel reuse, not an Updates-specific copy. MediaCarousel now delegates both its existing personal-media renderer and a typed custom item renderer to ONE layout/navigation/reveal implementation. Updates passes real WatchingUpdate items keyed by scope identity; no fake WatchingMedia rows and no per-card lens queries. Existing personal rails retain their batched lens/prefetch/menu/status renderer. Available now shares native5desktop/3landscape cards and native portrait rails below lg, existing hover clearance,300ms/1.04 whole-card scale and shared CarouselSkeleton. Redundant View details removed; work/scope link remains, explicit Continue/Resume only for started works. See all beyond5; no result cap.

Removed separate Updates rail widths/navigation/reveal skeleton. Carousel skeleton accepts optional inert header actions to keep filters aligned during reveal. Original movie/series/anime layouts remain shared and unchanged; actual TV Shows read-only smoke confirms the existing rails render with no viewport overflow.

Removed redundant Updates heading/subtitle. Controlled filters live in the first visible section header (Available, else Coming next, else TBA); empty filtered page keeps the controls. Responsive toolbar wraps without overflow. Announcement artwork moves to existing --poster-md112px token, ratio2:3, with3desktop/2tablet/1mobile columns; image zoom removed, quiet card-surface hover retained. Announcement skeleton mirrors112px cells. No private image-size token.

Want to watch now participates in Available AND explicit future/TBA announcements. Existing aired/date rules still apply; no fabricated season/date, old-backlog promotion or status/progress mutations. Supersedes the temporary first-section-only eligibility above.

Validation:83targeted tests across updates/service/writer pass; TypeScript, affected lint, Knip, DS/diff pass. Four-format actual-component harness checks native shared dimensions, navigation, gallery, whole-card1.04hover, stationary announcement images,112px artwork/skeleton parity, watchlist release/future/TBA cases, nonmutating navigation, selected Resume, first-visible filters and empty state. Authenticated localhost Updates captures inspected desktop/mobile; TV Shows existing renderer capture inspected. Read-only HTTPS smoke blocks writes/SentryPOST. Production build PASS after network-enabled Google Fonts retry (sandbox font fetch alone failed); existing middleware deprecation warning unchanged. DS returns to442existing off-system hits after semantic radius token cleanup. No production data/schema/edge/Git/deploy action. Owner visual review next.


## Announcement carousels / poster ambience — latest owner-approved polish

Owner approves four desktop cards and horizontal scrolling for BOTH Coming next and TBA. Shared AnnouncementUpdates renders both with useAlignedRail (natural end targets, touch/keyboard/native scroll, arrows only on overflow),112px/2:3 poster cells and4desktop/3landscape/2portrait/mobile90%-peek layout. Matching reveal and route skeleton. Removed redundant month headings, retained chronological Coming next sort and exact card dates/countdowns.

TBA now orders personally started stories before Want to watch, without a new filter or hidden watchlist results. Include paused/dropped UI and its opt-in state/API removed: paused announcements remain visible, dropped excluded. Existing paused Available default unchanged. No provider/user-data mutation.

Card ambience follows existing Don't Miss implementation: w92 poster copy blurred at3xl, scaled1.7, black40% and info-side dark gradient. Sharp poster is separate, unscaled, still112px token with native w154/1x,w342/2x,w342/3x selection; custom/AniList sources remain untouched. No palette API/extraction or high-resolution ambient duplicate. Available now image implementation untouched.

Validation:54targeted cases across Updates, TMDB image helper and service pass; pure regression proves started-before-watchlist TBA without dropping either. Four-width actual-component harness passes carousel geometry/navigation, TBA order, missing month/paused control, image ratio/skeleton geometry, stable hover and all prior shared-Available checks. Browser DPR3 mobile and DPR2 tablet/desktop actually select w342 sharp posters and expose separate ambient image. Authenticated read-only localhost captures pass and desktop/mobile inspected; TBA shows JJK, The Gentlemen and Widow's Bay before watchlist. TypeScript/affected lint/Knip/DS/diff pass; DS443off-system hits (one new announcement card surface), no new broad suite/build claimed. No migration/data/edge/Git/deploy action. Owner visual review next.
