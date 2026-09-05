> ⚠️ **ARCHIVE — OBSOLÈTE (vérifié 2026-07-10).** Audit **pré-V3** (23 mai). Le Top 5 « priorités absolues » est **résolu** (Library supprimable, `watch_status` optionnel, rating /10, sweep dead code, accents unifiés). Reliquats possibles hors fiche détail (débounce stepper, N-requêtes par table sur les pages liste, clamp resize MediaCarousel) = mineurs, non fonctionnels. **Trace historique — pas un backlog actif.** Backlog courant = `hq/now.md`.

# Audit Watching — final (sénior, sans concession) — 2026-05-23

> 2e passe après corrections. Beaucoup de points du 1er diagnostic ([watching-diagnostic-2026-05-23.md](watching-diagnostic-2026-05-23.md)) sont réglés : pub/sub mort supprimé, `WatchingHero`/`MediaDetailModal`/`LibraryFilters` supprimés, animation de nav ajoutée (layout `motion.div`), For You implémenté, namespace `TMDB_KEYS` séparé.
> Légende : 🔴 Critique · 🟠 Important · 🟡 Mineur.

---

## 1. TypeScript

🔴 **`watch_status` est un type menteur.** `types.ts:27` déclare `watch_status: WatchStatus` **requis**, mais la colonne n'existe pas en DB — elle est dérivée des booléens. Seul `getListItems` (`service.ts:424`) la calcule via `deriveWatchStatus`. `getMediaItems`/`getMediaItemById`/`getAllWatchedMedia` font `data as WatchingMedia[]` sans setter le champ → **`media.watch_status` vaut `undefined` à l'exécution** hors listes, alors que TS le garantit présent.
- Fix : rendre optionnel (`watch_status?:`) ou dériver dans tous les chemins de lecture.

🟠 **`any` à l'échelle du fichier service.** `service.ts:1` `eslint-disable no-explicit-any` global. `getForYouRecommendations` mappe `item: any` (`service.ts:379-393`), jointures Supabase `d: any`/`item as any`. Les types `TMDBMovie`/`TMDBTVShow` (`types.ts:106-134`) existent et ne sont **jamais** utilisés.
- Fix : interface `TmdbResult`, scoper le disable aux fonctions TMDB.

🟠 **Zod `addMediaSchema` mort.** `media.schema.ts:7` définit un validateur + `AddMediaInput`, mais `useAddMedia.ts:19` redéfinit son propre `AddMediaInput` local (`selectedItem: any`) et n'appelle jamais `.parse()`. Validation à l'insert = zéro. Seul `updateMediaSchema` est branché.
- Fix : `addMediaSchema.parse(insertData)` avant insert, ou supprimer le schéma.

🟡 **Cast en cascade** : `[id]/page.tsx:206` `items={similar as any[]}` — `useSimilarTitles` retourne déjà `any[]`. `SimilarItem` jamais propagé au hook.

---

## 2. TanStack Query

🟠 **Sur-invalidation DB persistante.** `useUpdateMedia`/`useDeleteMedia`/`useMarkAsWatched`/`useUpdatePriorities` font tous `invalidateQueries({ queryKey: WATCHING_KEYS.all })`. Un toggle favori sur la page détail refetch toutes les listes, leurs items, lists-with-thumbnails, search-for-list, episode-highlights et les 4 sections. (TMDB bien isolé désormais, mais côté DB = marteau-pilon.)
- Fix : `setQueryData(WATCHING_KEYS.detail(id), …)` + invalidation ciblée par type.

🟠 **Suppression de highlight silencieuse.** `EpisodeHighlights.tsx:108` `removeHighlight.mutate(h.id)` fire-and-forget, et `useRemoveEpisodeHighlight` n'a pas de `onError`. Échec → carte réapparaît sans toast.
- Fix : `onError: () => toast.error(...)` dans le hook.

🟠 **Invalidation redondante dans useAddMedia.** `useAddMedia.ts:201-209` invalide `typeKey()` (`['watching','movies']`) déjà couvert par `WATCHING_KEYS.all` (ligne 190). (L'optimistic For You au-dessus est bien fait.)

🟡 **For You ne se rafraîchit pas quand les favoris changent.** `useForYouRecommendations.ts:10` `staleTime 24h` ; les mutations DB ne touchent pas `TMDB_KEYS`. Acceptable par design.

🟡 **`initialData` mort sur les sections.** Les 4 sections passent `initialData: initialItems` mais `WatchingPageWrapper` ne fournit jamais `initialItems` → toujours `undefined`.

---

## 3. Performance

🟠 **Logique de carousel dupliquée.** `ForYouSectionClient.tsx:97-154` ré-implémente `cardsPerView`/resize/`scroll`/`itemWidthStyle` de `MediaCarousel.tsx:350-414`. Deux listeners resize.
- Fix : extraire `useCarousel()`.

🟠 **`sortedItems` inutile sur 3 carousels/4.** `MediaCarousel.tsx:385` trie par `priority || 999` à chaque render — no-op pour Recently/Want/InProgress, écrase l'ordre `updated_at DESC`, alloue un tableau par render.
- Fix : ne trier que si `showRankBadge`.

🟠 **Stepper = tempête d'écritures.** `CurrentlyWatching.tsx:58-66` chaque clic → `[id]/page.tsx:133` `mutateAsync` immédiat → `WATCHING_KEYS.all` invalidé. 5 clics = 5 écritures + 5 refetch complets.
- Fix : debounce 500ms ou save sur blur.

🟠 **N requêtes pour une table (non corrigé).** /tv-shows : 4 round-trips Supabase sur `media_items` (inProgress/recentlyWatched/wantToWatch/topTen) qu'un seul SELECT partitionné client-side remplacerait.

🟡 **Double scroll-to-top.** `layout.tsx:23` + `[id]/page.tsx:61`.

---

## 4. Architecture & dead code

🟠 **Library : aucun moyen de supprimer un média.** `LibraryGrid.tsx:7-13` déclare `onUpdate`/`onDelete` mais ne déstructure que `items` → props ignorées. `LibraryCard` n'a aucun delete (juste `onClick`→détail), et la page détail non plus. Conséquence : un film en Library mais hors carousels (au-delà des 10 récents / Top 10) **ne peut plus jamais être supprimé**. `handleDelete`/`handleUpdate`/`deleteMediaMutation` de `LibraryClient.tsx:95-107` = code mort.
- Fix : delete sur `LibraryCard` (menu ou hover), brancher `onDelete`.

🟠 **`handleUpdate` + `useUpdateMedia` morts dans 3 sections.** InProgress (`55-67`), RecentlyWatched (`44-54`), WantToWatch (`55-66`) : `handleUpdate` jamais passé (MediaCarousel a retiré `onUpdate`) → `updateMediaMutation` instancié pour rien.

🟠 **4 fonctions service mortes.** `getTrendingMovie` (`539`), `getTrendingTV` (`545`), `searchTMDB` (`560`), `getGenres` (`628`) — jamais importées.

🟡 **Dead code divers** : `AddCard` (`MediaCarousel.tsx:323`) ; `StatRow` (`ListDetail.tsx:87`) + `watchingCount`/`planCount` (`571-572`) ; prop `initialItems` ×4 sections ; en-tête commentaire `WATCHING SERVICE (TMDB API)` dupliqué (`service.ts:508 & 513`).

🟡 **Shape mapping dans le composant.** `ForYouSectionClient.tsx:134-150` reconstruit un faux objet TMDB inline → à extraire en helper.

---

## 5. Cohérence

🟠 **Rating /5 ↔ /10 : triple conversion + impossible de remettre à 0.** `[id]/page.tsx:87` stocke `starRating = user_rating/2`, sauve `*2` (`105`). `MyTakeRecord.tsx:180` reçoit (0-5), `*2` pour le slider, `/2` au retour. `RatingSlider` clampe `Math.max(1, …)` (`MyTakeRecord.tsx:21,53`) → impossible de redescendre à 0/null. AddMediaModal utilise un slider 0-10 brut. Nom `starRating` mensonger.
- Fix : stocker en /10, virer la conversion, autoriser 0.

🟠 **3 systèmes d'accent.** Layout `accent="#2dd4bf"` (teal, `layout.tsx:33`), AddMediaModal tout en bleu (`bg-blue-600` 1170, `ring-blue-500/50` 676), reste en `accent-watching` (violet) + amber. CLAUDE.md dit violet `#8b5cf6`.

🟡 **FR/EN mélangés.** `LibraryClient.tsx:190-196` "médias/films/séries/animes" ; `useAddMedia.ts:135` erreur en français.

🟡 **Helpers dupliqués.** `formatUpdated` identique dans `ListsClient.tsx:19` et `ListDetail.tsx:62` ; `mediaTypeLabel` aussi.

---

## 6. Edge cases & bugs

🟠 **`addMediaToList` : race sur `position`.** `service.ts:254-268` lit `max(position)` puis insère — deux ajouts concurrents → même position, pas de contrainte unique. (Mono-user → faible.)

🟠 **For You / existingIds plafonné.** `service.ts:335-337` `select("tmdb_id")` sans `.limit` → cap 1000. OK pour 127 films.

🟡 **`MoreLikeThis` toujours non cliquable.** `cursor-pointer` sans `onClick`. Cul-de-sac (For You a réglé ce pattern via `openModalWithItem`).

🟡 **EpisodeHighlights incohérences.** `MAX=20` (`14`) vs spec 12 ; skeleton 3 cartes (`94`) pour `PAGE_SIZE=4`.

---

## 7. UX

🟠 **`MediaCarousel` : `currentIndex` non clampé au resize.** `MediaCarousel.tsx:372-383` change `cardsPerView` sans reclamper `currentIndex`/scroll → espace blanc/bouton next incohérent après resize.

🟡 **For You : clic = modal sans CTA au hover** ("+ Watchlist" manquant).

🟡 **Pas d'optimistic sur mark-as-watched** (léger lag jusqu'au refetch).

---

## Scores par axe

| Axe | Note | Justification |
|---|---|---|
| TypeScript | 5/10 | `watch_status` menteur, `any` global, Zod schema mort |
| TanStack Query | 7/10 | namespace TMDB isolé + For You optimistic propres ; invalidation DB en marteau + delete highlight silencieux |
| Performance | 6/10 | carousel dupliqué, N requêtes, stepper non débouncé, tri inutile |
| Architecture | 6/10 | service layer respecté ; dead code (sections, library, service) |
| Cohérence | 5/10 | rating /5↔/10, 3 accents, FR/EN, helpers dupliqués |
| Edge/bugs | 6.5/10 | pas de crash dur ; échecs silencieux, race position |
| UX | 6.5/10 | MoreLikeThis mort, resize non clampé, library non supprimable |

**Global : ~6/10** — propre en surface, mais traîne du dead code post-refactor et deux vrais trous fonctionnels.

---

## Top 5 priorités absolues

1. 🔴 **Library non supprimable** — delete sur `LibraryCard`/page détail + brancher `onDelete`. (vrai trou fonctionnel)
2. 🔴 **`watch_status` menteur** — optionnel ou dérivé partout. (bug latent garanti)
3. 🟠 **Rating /5↔/10** — stocker en /10, virer la triple conversion, autoriser le reset à 0.
4. 🟠 **Sweep dead code** — `handleUpdate`+`useUpdateMedia` ×3, props `onUpdate/onDelete` LibraryGrid, `AddCard`, 4 fonctions service TMDB, `addMediaSchema`, `StatRow`/`watchingCount`/`planCount`, `initialItems`.
5. 🟠 **Stepper CurrentlyWatching** — debounce les écritures + resserrer l'invalidation DB.
