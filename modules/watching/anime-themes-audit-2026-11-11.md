# Anime Themes audit and correction - 2026-11-11

Status: implemented locally; owner review pending. No migration, deployment, commit, or personal-data mutation.

## Starting state and scope

Working tree was clean before edits. Inspected current local HEAD `5b1005f` and the preceding absolute-episode commits; owner reports the earlier stats work completed manually. Those implementations were not reverted. This block audits recent season/Highlight semantics and the complete Themes read/player/favorite flow.

## Diagnosis

- Detail used `searchAnimeThemes` directly from the browser against AnimeThemes. HTTP failure returned `[]`, so the card vanished; CORS/network errors instead entered default query retry behavior. Browser transport, absence and provider failure were conflated.
- Main Anime / My Themes read persisted `theme_favorites`, including song names and audio/video URLs. They do not rediscover the AnimeThemes catalogue. Existing playback can therefore work while detail discovery fails. Catalogue and personal favorites are different domains, not competing copies of one complete catalogue.
- A public read during verification independently returned upstream HTTP503. The new real authenticated endpoint returned HTTP200/status=unavailable. Live provider recovery cannot be certified during that outage.
- Positive design retained: module-level player, durable favorites, natural favorite key, covers requested after songs, cached reference data and shared artwork resolver.
- Weaknesses found: duplicated cover hooks/query keys, unbounded artwork fan-out, missing timeouts, HTTP failures poisoning persistent artwork cache, order-dependent detail track IDs, permissive short-name substring matching, and organization-unbounded favorites read/delete.

## Implemented boundaries

Browser -> authenticated `/api/watching/anime-themes` -> fixed AnimeThemes search endpoint. No URL forwarding, provider key or privileged data in the response. Existing local JWT claims and rate limiter patterns retained. Server logic lives in Watching; route remains thin.

Only validated successful catalogue results, including legitimate empties, enter Next's successful-result cache (24h). Provider fetch waits at most7s and does not retry. Concurrent cold requests share a promise. A bounded process-local60s failure cooldown supplements the cache; it is not a distributed outage store. HTTP/network/timeout/malformed failures produce a controlled unavailable envelope, never an empty success. Auth/validation/throttle errors retain401/400/429. Responses are private/no-store; no workspace data enters the shared public-catalogue cache.

Client uses a namespaced query key, DAY stale/gc policy,10s network deadline, no automatic retry/focus/reconnect retry and no retry-on-mount loop. Existing successful query data survives failed refresh. A cold outage shows a quiet Panel message with manual Try again; a genuine empty catalogue stays hidden. Optional enrichment does not gate the rest of the fiche.

Both surfaces now use one cover hook/query key and one deduplicated service path. Four workers bound each batch; simultaneous identical song lookups share a promise.5s per-request deadline. Existing positive localStorage artwork stays usable; null/failed entries are no longer persistent negative cache. Tracks still render before covers. iTunes still exposes a per-song lookup, so cold requests remain N (bounded, not claimed to have disappeared).

Detail player IDs now use the already-established favorite natural key rather than group index. No favorite key migration or overwrite. Playback error stops with a discreet message rather than automatically cascading through broken URLs; transport and queue remain available. Video failure closes the failed video and leaves the queue intact.

Favorites queries and cache keys are scoped to the active organization, deletion includes that organization, and mutation writes capture it explicitly. Existing org/track uniqueness and RLS unchanged. Reads use shared pagination; optimistic updates only modify an already-existing cache entry.

## Recent manual Watching audit

- SeasonPanel passes viewing for Watched through, correction for I stopped after. Page handler propagates intent through the single writer. Existing positionPatch dates/stamps actual forward viewing and preserves correction dates; no rewrite of this correct behavior. Added regression checks for this and Discover caught-up recency.
- Normalized highlight/progress coordinates remain separate from source_episode. Best Episodes retains the owner's provider-number display. Found/fixed the panel's bestHere counter comparing provider numbers against normalized stored keys.
- Found/fixed the metadata fallback calling TMDB with normalized episode instead of sourceEpisode. SeasonPanel already supplies metadata, so the ordinary pin path still performs no extra TMDB lookup.
- MAX_HIGHLIGHTS remains30, as manually chosen. This is a UI convenience cap, not a database authorization constraint. No blanket coordinate guard suppression added. Removed obsolete comments claiming Watched through is always a correction and normalized highlight coordinates are provider-native.

## Validation

- npm test:908tests /59files PASS (including catalogue matching, provider errors, endpoint auth/validation/throttle, cooldown/deduplication, cache retention, artwork concurrency/failure recovery, stable favorites/player transport, workspace scoping and source_episode metadata lookup).
- TypeScript, lint, Knip, DS coverage and schema-reference checks: PASS; production build PASS (initial restricted-network attempt could not download Inter; authorized network retry succeeded).
- Read-only real localhost fiche with controlled provider response:390/768/1024/1440 widths, no panel overflow, no JS page errors, zero browser AnimeThemes-search calls. One cold attempt; manual retry restores tracks. Actual private endpoint safely reports unavailable during real upstream503. Captures inspected: scripts/.shots/themes-outage-*.png, themes-recovered.png. Isolated tests additionally cover retention of cached tracks during a failed refresh.
- No live favorite/progress mutation or real-media playback was performed. A synthetic broken audio URL was exercised on the real fiche: player stopped, displayed recovery text and retained the selected track/queue UI without a JS page error. Actual CDN recovery remains dependent on external media availability.

## Remaining limits

Title/synonym/year matching remains evidence-based title matching, not an external-ID binding; future provider renames can change favorite natural identities. iTunes first-result artwork matching may be imperfect. A cold instance cannot recover a catalogue that has never been cached while the upstream is down. The failure cooldown is process-local; durable successful cache is managed by Next. None of these justify fabricating a complete catalogue from a partial favorites list.

## File manifest

- src/app/(main)/perso/watching/[id]/page.tsx
- src/modules/styleguide/coverage.generated.json
- src/modules/watching/components/ThemePlayer.tsx
- src/modules/watching/components/detail/AnimeThemes.tsx
- src/modules/watching/components/detail/SeasonPanel.tsx
- src/modules/watching/hooks/query-keys.ts
- src/modules/watching/hooks/useAnimeThemes.ts
- src/modules/watching/hooks/useEpisodeHighlights.ts
- src/modules/watching/hooks/useThemeCovers.ts
- src/modules/watching/hooks/useThemeFavorites.ts
- src/modules/watching/hooks/useWatchActions.tsx
- src/modules/watching/lib/watch-status.test.ts
- src/modules/watching/service.ts
- src/app/api/watching/anime-themes/route.test.ts
- src/app/api/watching/anime-themes/route.ts
- src/modules/watching/anime-themes-provider.ts
- src/modules/watching/anime-themes-server.test.ts
- src/modules/watching/anime-themes-server.ts
- src/modules/watching/components/detail/AnimeThemes.test.ts
- src/modules/watching/hooks/useEpisodeHighlights.test.ts
- src/modules/watching/lib/anime-themes.test.ts
- src/modules/watching/lib/anime-themes.ts
- src/modules/watching/lib/theme-favorites.test.ts
- src/modules/watching/lib/themes-client.test.ts

HQ files updated (separate ignored HQ repository):
- hq/now.md
- hq/modules/watching/anime-themes-audit-2026-11-11.md
- hq/modules/watching/decisions.md
- hq/modules/watching/backlog.md
