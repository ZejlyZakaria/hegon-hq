# Watching image quality and Top Picks presentation audit - 2026-10-29

DISCUSSION / READ-ONLY AUDIT. No application/worker code change, migration, deployment, production business mutation or Git mutation. Wave2E remains LOCKED. Prior Kuroko repair SQL/reference changes remain uncommitted from the preceding task.

## Proposed product contract, awaiting owner implementation approval

Hide a Top Picks season label only if the displayed rating's represented viewing scope covers the entire relevant work in the selected year. Kuroko with all three seasons viewed/rated2018 can display only title/score. A shared year stamp alone is insufficient: Hannibal's S1/S2rating whileS3isunfinished must still sayS1–S2, and JJK's S3must remainS3. Partial episode/cour coverage remains explicit. Keep exact score calculation and qualification unchanged. Do not replace the removed label with an equally redundant Completed/Entire series label. Remove film release year from both Top Picks card and See all; this does not remove years globally from library/search/detail contexts.

## Current sources of truth

- globals.css defines visual frame sizes: poster-xs40px,sm80,md112,lg144,xl170; mobile rails use --rail-peek.
- lib/tmdb-image.ts chooses CDN variant from declared CSS width and default DPR2; selected heroes request3. Compact MediaRow alone already wraps its Next Image in a native picture with1x/2x/3x candidates.
- next.config.ts globally sets images.unoptimized=true. Most Image sizes props therefore do not create an optimized responsive srcset. Historical comments in MediaCarousel claiming Next adds AVIF/WebP/srcset no longer describe current configuration. Keep the existing quota-related reason for disabling optimization; this audit does not recommend re-enabling it.
- AniList worker already selects coverImage.extraLarge before large. Non-TMDB/custom images pass through the helper; rewriting them as TMDB variants would break them.

## Per-surface comparison

| Image family | Current frame/source | Assessment / possible improvement |
|---|---|---|
| Top Picks card and See all |40px MediaRow;w92atDPR1/2,w154atDPR3 |Already enough for40pxx3=120physicalpx. No reason to forcew500. Tiny text inside a40pxposter is a physical-size/artwork limit, separate from source resolution. |
| Hours, person timeline/insights and search MediaRow panels |Same40pxnative picture |Same adequate density policy; retain shared implementation. |
| Season panel header poster |96pxmobile/120pxsm+;fixedw342 from tmdbImageFor(120) |At120pxx3needs360px, so native responsivew500candidate can improve if source has real detail. Mobile96x3=288already covered byw342; no blanket increase. |
| Season strip |Mobile fluid/144pxdesktop;rawTMDBw300orpassed AniList/custom URL |At144x3needs432px: density-awarew500can improve;w342already coversDPR2. Native-source cap still matters. |
| Detail hero poster |112px/w342mobile,170px/w780desktop;explicitDPR3 |Already prioritizes quality. Do not blindly enlarge. Somew780variants exceed original resolution and can be heavier than original. |
| Detail backdrop |w1280mobile/originaldesktop |Quality is already high; leave visual treatment intact. Responsive source selection can optimize bytes, no guaranteed need for sharper image. |
| Poster carousels / poster ForYou |Declared170px,w342fixed |Can under-serve true170pxDPR3:510needed. Responsivew500/w780conditional on actual frame/native image; avoid globally fetching larger files for long rails. |
| Landscape carousel / ForYou |Declared360px,w780fixed |True360pxDPR3needs1080:conditionalw1280can improve native high-resolution sources. |
| Library / large list grids |tmdbImageFor(200)=w500 irrespective of actual cell |Often already sufficient on current small columns; mainly adapt to actual responsive cell to avoid excess mobile bytes. Do not automatically raisew780. |
| More Like This |144pxdesktop/fluidmobile,w500 |Already covers144x3=432; normally keep. |
| Season-panel episode still |144px,w500 |Already covers144x3. Native400pxsources will not gain detail byw780. |
| Best Episodes |264px,w500 |NearDPR2needs528;DPR3needs792. Higher variant can help only high-native-detail stills. Real Kuroko400pxoriginal cannot improve through upscaling. |
| PersonFace / person miniatures |Declared120px/w342,small36px/w92,56px/w154 |Evaluate actual rendered diameter; conditional density selection can help larger faces, miniature sources usually adequate. |
| Person hero |112px/w342mobile,160px/w500desktop,DPR3 |Already adequate3xsource policy. |
| Wrapped export |148pxcenter/116pxsides,storedURLthroughsame-originproxy;exportpixelRatio3 |Main148x3=444is covered by normalw500. Explicit export source policy avoids inheriting an undersized stored URL; no blanketoriginal. |
| Blurred ambient poster backgrounds |SeasonPanel/Don'tMissw92 |Intentional low-detail blurred layer; retain small source. Detail hero glow currently uses storedURL instead of small variant, an optimization opportunity rather than sharpness problem. |
| Provider logos/theme covers |Provider assets/localSVG/custom or iTunes source |Separate image families, not fixed by changing TMDB poster variants. SVG remains vector; inspect unusually small raster sources individually. |

## Executed evidence

1. Official TMDB configuration queried directly with existing key only in memory. Published sizes differ by type:postersw92/154/185/342/500/780/original;stillsw92/185/300/original;profilesw45/185/h632/original;backdropsw300/780/1280/original. Extra CDN width variants can currently return200 even when not published for that family; a generic width list is not a guarantee of native detail.
2. Real fetched image buffers measured with sharp, not URL names. Three poster samples:Hannibal,BreakingBad,JJK. Hannibalw34236333bytes vsw50087235;JJK68182vs104900. These are payload comparisons, not page-latency A/B claims. Original Hannibal680x1000/106941bytes versusw780780x1147/169208bytes proves CDN upscale can cost more with no added source detail. Kuroko episode original400x225/19891bytes;w500500x281/32764;w780780x439/58388. Upgrading that still is not a real quality improvement. Hannibal backdropw1280129532bytes/original1920x1080229613bytes.
3. Chromium native picture fixture reproducing current MediaRow candidates confirms40CSSpx ->w92atDPR1/2,w154atDPR3. It is an isolated source-selection check, not a full application visual review.
4. Authenticated localhost Playwright navigation did not yield usable image views; stopped rather than claim four-device validation. Browser business mutations/worker requests were blocked. No full native-frame screenshot/actual-device visual quality sign-off is claimed. Frame sizes outside the native picture fixture come from current source/CSS and remain to measure during implementation.

Official references:https://developer.themoviedb.org/docs/image-basics and https://nextjs.org/docs/app/api-reference/components/image#unoptimized.

Ignored evidence/scripts:watching-image-assets-audit.json,top-picks-native-density-audit.json,audit-watching-image-assets.cjs,check-top-picks-native-density.cjs,audit-watching-image-dom.cjs under scripts/.shots. No keys stored in these outputs.

## Recommendation

Reuse and extend the existing image helper/shared components: visual frame + appropriate native density candidates + known original-resolution ceiling, type-aware where relevant. Keep low-resolution blur and lazy loading. Preserve the good detail imagery and existing AniList extraLarge selection. Improve season headers/strips and genuinely undersized large rails first; compact Top Picks is already density-aware. Validate actual four-device frames and warmed page-level transfer/latency before promising zero performance impact. No implementation authorized by this discussion request alone.
