> ⚠️ **ARCHIVE — OBSOLÈTE (vérifié 2026-07-10).** Diagnostic **pré-V3** (23 mai). Ses points majeurs sont **tous réglés** depuis (Library supprimable ✅, `watch_status` optionnel ✅, rating /10 direct ✅, 4 fns TMDB mortes supprimées ✅, `addMediaSchema` parti ✅, accents unifiés ✅, dead code détail supprimé ✅). Conservé pour trace historique uniquement — **ne pas traiter comme un backlog actif**. Backlog courant = `hq/now.md`.

# Diagnostic Watching — 2026-05-23

> Diagnostic Opus complet du module Watching (perf, animations, dead code, UI/UX, plan reco).
> Légende : 🟢 bon · 🟡 améliorable · 🔴 à corriger.

---

## ✅ Déjà appliqué dans cette session

- **Priorité #1 — fix over-invalidation (FAIT)** : les queries TMDB (`hero`, `similar`, `credits`) sont sorties du préfixe `['watching']` vers un namespace `['tmdb']` dédié (`TMDB_KEYS` dans `hooks/query-keys.ts`). Conséquence : `invalidateQueries({ queryKey: WATCHING_KEYS.all })` (appelé par toutes les mutations DB) ne refetch plus les appels TMDB coûteux. Hooks modifiés : `useWatchingHero`, `useSimilarTitles`, `useMediaCredits`. `WATCHING_KEYS.hero` supprimé (devenu inutile). `tsc --noEmit` propre.

Reste à faire : #2 → #5 ci-dessous (Top 5 priorités).

---

## 1. Performance & vitesse

🔴 **Invalidation catastrophique — problème #1 (CORRIGÉ).** Toutes les mutations faisaient `queryClient.invalidateQueries({ queryKey: WATCHING_KEYS.all })` où `all = ['watching']`. Tout étant préfixé `'watching'`, un simple toggle favori refetchait : hero TMDB (2 appels, `staleTime 24h`), `similar`, `credits`, `whatToWatchNext` (4 requêtes Supabase), 3 requêtes sections, lists… `invalidateQueries` refetch les queries actives **même si `staleTime` n'est pas écoulé**.

Fix appliqué :
```ts
// query-keys.ts — namespace séparé, non invalidé par les mutations DB
export const TMDB_KEYS = {
  all: ['tmdb'] as const,
  hero: (t: MediaType)       => ['tmdb', 'hero', t] as const,
  similar: (t, id: number)   => ['tmdb', 'similar', t, id] as const,
  credits: (t, id: number)   => ['tmdb', 'credits', t, id] as const,
};
```

🔴 **N requêtes pour une seule table.** Sur `/movies`, `RecentlyWatched` + `WantToWatch` + `TopTen` font 3 SELECT séparés sur `media_items` (filtres `recently_watched` / `want_to_watch` / `favorite+priority`). Parallèles, mais 3 round-trips pour la même table d'un user qui a ~127 films. La page Library prouve qu'on peut tout fetcher en 1 fois et partitionner client-side. Reco : hook agrégé `useWatchingPageData(type)` → 1 SELECT `where type=`, puis `useMemo` pour découper en sections.

🟡 **Images.** `MovieCard` : `sizes="(max-width: 768px) 100vw, 50vw"` alors qu'une carte de carousel fait ~1/5 de la largeur desktop → image ~2× trop grosse. Mettre `sizes="(max-width:768px) 100vw, (max-width:1280px) 33vw, 20vw"`. `priority`/`eager` sur les `cardsPerView` premières : OK. Hero `priority` : OK.

🟡 **`MediaCarousel` re-render.** `localItems` dupliqué dans `useState` + `useEffect(setLocalItems)` — copie d'état inutile (les items viennent déjà de TanStack). `useEffect resize` recalcule sans debounce. Mineur.

🟢 **Bon** : `staleTime` hero 24h, `refetchOnWindowFocus:false` global, `Promise.all` dans `getWhatToWatchNext` et `getWatchingHeroData`, `useCurrentUserId` sans network.

**Note perf : 5/10** (avant fix) — l'archi parallèle est là mais l'over-invalidation annulait les bénéfices et faisait du TMDB-spam à chaque clic. Le #1 corrige le plus gros.

---

## 2. Animations de navigation

🔴 **Inexistantes.** Aucune transition d'entrée/sortie sur les pages Watching. Framer Motion installé mais non utilisé pour la nav. Pas de `prefers-reduced-motion` dans `globals.css`.

Implémentation légère, entry-only (l'App Router ne démonte pas proprement pour AnimatePresence), keyée par pathname, dans le layout :
```tsx
"use client";
import { motion } from "framer-motion";
import { usePathname } from "next/navigation";

export function PageTransition({ children }: { children: React.ReactNode }) {
  const pathname = usePathname();
  return (
    <motion.div
      key={pathname}
      initial={{ opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.22, ease: [0.4, 0, 0.2, 1] }}
    >
      {children}
    </motion.div>
  );
}
```
N'utilise qu'`opacity`/`transform` (composités GPU) → ne bloque pas le paint initial. Pour les sections internes, un `staggerChildren` léger quand la data arrive donnerait le côté "Linear-grade" (spec §15).

**Note animations : 2/10.**

---

## 3. Dead code & code mal fait

🔴 **Top 10 drag-reorder cassé.** `TopTenSectionClient` passe `draggable` + `onReorder={handleReorder}` à `MediaCarousel`, mais `MediaCarousel` **n'implémente plus aucun drag** (`onReorder`/`draggable` même pas déstructurés). Donc `handleReorder` n'est jamais appelé → `isDirty` reste `false` → le bouton "Save ranking" n'apparaît **jamais**. Feature morte et silencieuse.

🔴 **Système pub/sub de `WatchingClient` entièrement mort.** `registerOnAdded/Updated/Moved/Deleted` ne sont appelés par **aucun** composant. Donc `notifyDeleted`/`notifyMoved`/`notifyUpdated` itèrent sur des maps vides → no-op. Les sections se mettent à jour uniquement via `invalidateQueries`. ~80 lignes de contexte + refs à supprimer.

🔴 **3 composants morts** :
- `WatchingHero.tsx` — remplacé par `DontMissSectionClient`, plus importé.
- `MediaDetailModal.tsx` — remplacé par la detail page, plus importé.
- `LibraryFilters.tsx` — `LibraryClient` a ses filtres inline, jamais importé.

🟡 **Wrappers inutiles.** `useMovies`/`useSeries`/`useAnimes` ne font que `useMediaItems({...options, type})`. Les 4 sections répètent `const hookMap = { film: useMovies, serie: useSeries, anime: useAnimes }; hookMap[config.type]({...})`. Appeler `useMediaItems({ userId, type: config.type, ... })` directement → supprime 3 fichiers + 4 copies de boilerplate.

🟡 **Indirection rating trompeuse.** La detail page stocke `starRating` en **/5** (`user_rating / 2`), le passe à `MyTakeRecord` qui le `*2` pour `RatingSlider` (/10), qui rappelle `/2`. Double conversion ÷2/×2 + nom `starRating` qui ment. Ça round-trip mais c'est fragile, et le slider clampe à `min 1` → **impossible de remettre une note à 0 / null** une fois posée. Stocker directement en /10.

🟡 **`onUpdate` mort.** Les 4 sections passent `onUpdate={handleUpdate}` à `MediaCarousel` qui ne le déstructure pas → jamais utilisé.

🟡 **`any` partout** dans `service.ts` (TMDB), `useAddMedia`, `DontMissCard`. Les types `TMDBMovie`/`TMDBTVShow` existent dans `types.ts` mais quasi pas utilisés.

🟡 **`MoreLikeThis` non cliquable.** Cartes `cursor-pointer` sans `onClick` (lié au §5).

**Note qualité/dead code : 4/10** — beaucoup d'échafaudage abandonné, dont 2 features mortes silencieuses (Top 10 reorder, pub/sub).

---

## 4. UI/UX vs Letterboxd / Trakt

🟢 La detail page (Hero 21/9, My Take + My Record, Best Episodes, In List, Cast) et le List Detail (table/grid, stats panel, collage hero, notes par item) sont **au-dessus** de Letterboxd en densité/curation. Point fort.

🟡 **"Recommendations" veut dire 4 choses** : le hero "Trending" (discover TMDB), la colonne "Don't Miss" du vieux hero, "What to Watch Next" (heuristique sur la library), "More Like This" (recommendations TMDB d'un titre). Vocabulaire à clarifier : **Trending** (externe/découverte) vs **For You** (perso) vs **More like this** (contextuel détail).

🟡 **`MoreLikeThis` est un cul-de-sac** : titres TMDB pas dans la library + clic sans effet. Letterboxd/Trakt ouvrent toujours une fiche. Ici → ouvrir `AddMediaModal` pré-rempli (cf. §5).

🟡 **Library `lg:grid-cols-10`** : très dense, cohérent avec ListDetail grid (10 cols). OK.

🟡 **Incohérence d'accent** : `SectionHeader accent="#2dd4bf"` (teal), tabs ListDetail en `accent-watching` (violet `#8b5cf6`), `AddMediaModal` tout en **bleu** (`bg-blue-600`), configs `serie` `#0ea5e9` / `anime` `#f43f5e`. CLAUDE.md dit Watching = violet `#8b5cf6`. Mélange teal/violet/bleu/sky/rose sans règle. À unifier.

**Note UI/UX : 6.5/10** — fond excellent, mais vocabulaire "reco" confus + incohérences d'accent.

---

## 5. Challenge du plan "Recommendations for you"

Plan correct dans les grandes lignes. Ce qui cloche / ce que je changerais :

**Ce qui peut mal tourner :**

1. 🔴 **Rate-limit TMDB.** N seeds = N appels `/api/tmdb` (rate-limit Upstash par IP). Au mount + hero (2) + recherches → 8-10 favoris en seed peuvent jeter. → **Cap à 3-5 seeds**, 2 pages max par seed, `Promise.all`.

2. 🔴 **L'over-invalidation va marteler TMDB.** Si la reco vit sous `['watching', ...]`, chaque ajout déclenche `invalidateQueries(WATCHING_KEYS.all)` → re-fetch des N appels TMDB. **Le #1 (fait) débloque ça** : mettre la reco sous `['tmdb','for-you',...]` et **ne jamais l'invalider** (juste splice optimiste).

3. 🔴 **Résultats vides pour un power-user.** 127 films watched → la plupart des recos déjà dans la collection → après filtrage, souvent < 10. Élargir : seeds = favoris **+ top user_rating récents**, 2 pages par seed.

4. 🟡 **Scope par type obligatoire.** "filtrer les tmdb_id déjà présents" doit être `(tmdb_id, type)` — film et série peuvent partager un id (namespaces TMDB séparés).

5. 🟡 **Défaut d'AddMediaModal.** Une reco est un titre **non vu** → ouvrir en `listContext="wantToWatch"`, pas `recentlyWatched`.

6. 🟡 **Conflit conceptuel avec "What to Watch Next".** La spec §10 a déjà "More like [favorite]" mais sur la **propre** library. "For You" = découverte **externe** → proche de Trending. Positionner clairement (titre "Because you liked X", sous What to Watch Next).

**Cache : TanStack, pas Edge Function.** Pour une app mono-user, l'Edge Function est de l'over-engineering. Reco :
- TanStack Query, `staleTime` 12-24h.
- `queryKey: ['tmdb','for-you', type, seedHash]` (hash trié des tmdb_id seeds) → recompute seulement quand favoris/top-rated changent.
- Toute la logique (sélection seeds → N fetch `Promise.all` → merge → dedup vs collection) dans **une** fonction service.
- Ajout → splice optimiste local (10→9), **sans** invalider la query.

Passer à l'Edge Function **seulement si** : (a) cohérence cross-device voulue, ou (b) rate-limit TMDB devient un vrai problème. Pas maintenant → YAGNI.

---

## Top 5 priorités (par impact)

| # | Action | Impact | Effort | Statut |
|---|--------|--------|--------|--------|
| 1 | **Fixer l'over-invalidation** : sortir hero/similar/credits du préfixe `['watching']` vers `['tmdb',...]`. | 🔥🔥🔥 | Petit | ✅ FAIT |
| 2 | **Réparer ou retirer le Top 10 reorder** : réimplémenter le dnd dans `MediaCarousel`, ou retirer `draggable`/`onReorder`/`isDirty`/Save (feature morte). | 🔥🔥 | Petit (retrait) / Moyen (réimpl.) | 🔲 |
| 3 | **Supprimer le dead code** : WatchingHero, MediaDetailModal, LibraryFilters, pub/sub WatchingClient (~80 lignes), wrappers useMovies/Series/Animes, props `onUpdate` mortes. | 🔥🔥 | Moyen | 🔲 |
| 4 | **Animation d'entrée nav** (`PageTransition`) + `prefers-reduced-motion`. | 🔥🔥 | Petit | 🔲 |
| 5 | **Consolider les requêtes sections** en 1 SELECT/type partitionné client-side (`useWatchingPageData`). | 🔥 | Moyen | 🔲 |

Bonus avant de coder "For You" : rendre `MoreLikeThis` cliquable (ouvre `AddMediaModal` en `wantToWatch`) — 80% de la même mécanique, valide le flow avant la version perso.
