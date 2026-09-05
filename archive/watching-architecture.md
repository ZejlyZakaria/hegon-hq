# WATCHING — Architecture de connectivité entre les couches

> **Audit Fable (2026-07-18), croisé et vérifié par Opus.** Commandé par l'owner : *« la meilleure architecture pour toute la connectivité entre les couches, qui implique qui, tous les flows, tous les cas possibles — une architecture qui ne changera pas, et si une nouvelle connectivité apparaît elle doit être simple à intégrer. Insiste sur la performance et la rapidité. »*

## ✅ VÉRIFICATIONS OPUS (les affirmations que Fable avait marquées « à vérifier »)

| Affirmation | Verdict | Preuve |
|---|---|---|
| **Lignes « limbo » sans statut à l'insertion** | ✅ **CONFIRMÉ** | `useAddMedia.ts:123` `claimedStatus(facts, position ?? null, stance)` + `:132` `isInProgress = isSeries ? !!claim?.in_progress : …`. `claimedStatus` **retourne `null` si position est nulle** (`watch-status.ts:230`, « a claim of nothing must change nothing » — juste pour un UPDATE, fatal pour un INSERT) ⇒ tous les drapeaux faux. |
| **`library/page.tsx` a une copie périmée des colonnes** | ✅ **CONFIRMÉ** | Le `select` omet `status, season_episodes, season_aired, season_years, caught_up_at` — exactement ce que `LIBRARY_COLUMNS` dit être nécessaire pour que le menu « … » agisse honnêtement. |
| **Durées de cache trop courtes** | ✅ **CONFIRMÉ** | `useMediaItems.ts` : `staleTime: 2min`, `gcTime: 5min`. |
| **`key={pathname}` remonte tout l'arbre** | ✅ **CONFIRMÉ** | `watching/layout.tsx` : `<motion.div key={pathname}>` ⇒ démontage/remontage complet à chaque navigation interne. **C'est la cause du « rechargement complet de page ».** |

---

## 0. Le verdict en un paragraphe

Watching contient **déjà** une architecture excellente — mais pour **un seul** de ses quatre invariants inter-couches. Le **statut** a un écrivain unique (`lib/watch-status.ts`), un lecteur unique (`deriveWatchStatus`), une façade que toutes les surfaces appellent (`useWatchActions`) et une barrière zod qui refuse les écritures de position à la main (`schemas/media.schema.ts:59`). C'est du travail de maître, et **c'est le modèle**. Les trois symptômes rencontrés sont **la même maladie sur les invariants qui n'ont jamais reçu ce traitement** : l'overlay de cours est une lentille optionnelle que chaque surface doit penser à lever (cinq l'oublient) ; le statut à l'ajout dérive d'une position réclamée, mais deux portes ne passent aucune position et produisent silencieusement des lignes **sans aucun statut** ; et la lenteur perçue est un empilement de **quatre petites causes réparables**, pas une grosse. **Rien ici n'exige de réécriture.** Il faut appliquer deux fois de plus la médecine existante, plus une passe de réglage du cache.

---

## 1. La carte des couches (telle que le code est aujourd'hui)

```
ROUTES (coquilles fines)   app/(main)/perso/watching/
                            layout.tsx ── tabs + WatchingSearch + motion(key=path)
                            movies|tv-shows|animes → WatchingPageWrapper
                            [id]/page.tsx  (orchestrateur CLIENT de 486 lignes)
                            discover/[type]/[tmdbId]  (non commité)
                            library/ lists/ stats/ person/ + loading.tsx
        ↓ rend
COMPOSANTS                  sections/* · shared/MediaCarousel, NextEpisodeButton,
                            MediaActionMenu, AddMediaModal · detail/StatusCard,
                            SeasonHistoryStrip, Episodes…
        ↓ lit                                    ↓ écrit
HOOKS (TanStack)   useMediaItems/useMediaItem     useWatchActions (FAÇADE)
                   useAnimeCours, useLibrary       → useUpdateMedia (barrière zod)
                   query-keys.ts : WATCHING_KEYS   useAddMedia, useDeleteMedia
                   vs TMDB_KEYS (namespaces !)     (les invalidations vivent ici)
        ↓ appellent des fonctions PURES dans
LIBS DOMAINE       lib/watch-status.ts   ← LE writer de statut
(pures, testées)   lib/series-state.ts   ← état dérivé (diffusé ≠ annoncé)
                   lib/anime-overlay.ts  ← lentille cour ↔ TMDB
                   lib/resolve-transition.ts · season-years · watched-date
                   schemas/media.schema.ts ← barrière zod sur les UPDATE
        ↓ toute I/O passe par
SERVICE            service.ts (1514 lignes) — Supabase + TMDB via /api/tmdb
INFRA              /api/tmdb (getClaims + ratelimit Upstash + cache 1h)
                   Supabase · edge functions/crons (series-sync 4h, cours 6h…)
```

**Le nombre de couches est juste. Ne pas en ajouter.** Les trous sont **trois points d'application manquants**, pas des étages manquants.

---

## 2. Les invariants qui traversent les couches

### (a) STATUT — point unique : **oui pour les updates, non pour les inserts**

**Trois portes oubliées, toutes dans le chemin d'AJOUT :**

1. **🔴 `useAddMedia` insère des lignes SANS STATUT.** La porte « In Progress » ne collecte **jamais** `position` (le `HowFarDidYouGet` ne s'affiche que pour library/recentlyWatched/topTen — `AddMediaModal.tsx:812`), et ses champs saison/épisode partent en `currentSeason/currentEpisode`, pas en `position`. ⇒ **une série fraîche ajoutée via « In Progress » s'insère avec `in_progress: false`** — invisible partout, StatusCard affiche « Want to Watch ». Même limbo sur les portes de réclamation si l'utilisateur ne touche pas la bande de saisons (rien ne l'y force : `isSubmitDisabled`, `AddMediaModal.tsx:502`).
2. **Les branches update de `useAddMedia` contournent la barrière zod** (`:259`, `:304`, `:323` appellent `updateMediaItem` directement). `update:inProgress` écrit une position **sans recalculer `caught_up_at`** — exactement l'écriture que la barrière a été construite pour refuser.
3. **Il n'existe aucun schéma d'insertion.** `insertMediaItem(data: Record<string, any>)` accepte n'importe quoi.

### (b) POSITION + overlay de cours — **aucun point d'application**

La lentille (`lib/anime-overlay.ts`) est parfaite : pure, bidirectionnelle, fallback sûr. Le choix « le stockage ne bouge jamais » est **le bon, à garder**. Mais `shouldOverlay/buildOverlay` sont importés dans **exactement 2 endroits**.

| Surface affichant une position | Overlay ? |
|---|---|
| Detail : SeasonHistoryStrip | ✅ |
| Detail : Episodes | ✅ |
| **Detail : StatusCard** | ❌ épisode **plat** (« Episode 47 ») à côté d'un historique disant « S2 E23 » ; rustine `hideSeasonStepper` |
| **Cartes MediaCarousel** | ❌ `formatPosition(current_season…)` brut |
| **NextEpisodeButton (toast)** | ❌ coords TMDB brutes |
| **LibraryCard** | ❌ |
| **Dashboard « IN PROGRESS »** | ❌ |
| AddMediaModal : champs inProgress | ✅ (patch non commité) |
| **AddMediaModal : HowFarDidYouGet** | ❌ — **contradiction dans le MÊME modal** : « max 2 saisons » à 3 cm d'une bande à 1 saison |
| **Discover : Episodes** | ❌ — 1×38 avant l'ajout, 2 cours après |

**Huit surfaces, trois couvertes.** C'est la situation d'avant `watch-status.ts`, rejouée sur les coordonnées.

### (c) DIFFUSÉ vs ANNONCÉ — point unique : **oui** (`series-state.ts`), une dérive vérifiée
`library/page.tsx` : copie **périmée** de `LIBRARY_COLUMNS`. Colonnes manquantes ⇒ `canComplete` lit `status: undefined` ⇒ une série terminée se voit offrir « Seen everything that's out » au lieu de « Mark as watched ». **Fix : exporter `LIBRARY_COLUMNS` et l'importer.**

### (d) POSSESSION — le point faible est le **`type`**
Partout ailleurs le type est **un choix de l'utilisateur** ; `tmdbResultType` (non commité) le fait **deviner par la machine**. Deux autorités pour un même fait. Si elles divergent, `findOwnedMediaId` (filtré `.eq("type")`) rate ta ligne ⇒ discover croit le titre neuf ⇒ **une seconde ligne du même show est créée** (l'index unique le permet). **Candidat n°1 pour « want to watch n'ajoute pas l'anime » : la ligne a atterri en `serie` et l'owner regardait la page Animes.**

---

## 3. LA PROPOSITION CENTRALE — rendre l'oubli impossible

Trois mouvements, tous des copies de médecine déjà éprouvée. **~3 fichiers neufs, plusieurs suppressions.**

### 3.1 La lentille devient un view-model avec UN constructeur

`lib/media-view.ts` + `hooks/useMediaView.ts`. Les composants **cessent de lire** `current_season/current_episode/season_*` pour l'affichage ; ils consomment un `MediaView`, et **seul `buildMediaView` peut en construire un**.

```ts
export interface MediaView {
  media: WatchingMedia;                      // ligne brute (coords stockage) — pour les libs
  overlaid: boolean;
  position: { season: number; episode: number; label: string };   // coords AFFICHAGE
  seasons: { episodes: number; aired: number; poster: string | null; endDate: string | null }[];
  yearMap: Record<string, number> | null;    // cour_years si overlay, sinon season_years
  ratingMap: Record<string, number> | null;
  toStorage(season, episode): { season, episode };   // le SEUL convertisseur sanctionné
  fromStorage(season, episode): { season, episode };
}
export function buildMediaView(media, cours): MediaView;
export function useMediaView(media): MediaView | null;   // branche useAnimeCours tout seul
export function useAnimeCoursMap(items): Map<number, AnimeCoursRow>;  // 1 lecture batch, pas N
```

**Ce qui rend l'application automatique pour une 6ᵉ surface — deux contraintes mécaniques, pas une convention :**

1. **Changer les props des composants d'affichage.** `StatusCard`, `SeasonHistoryStrip`, la puce de position, `HowFarDidYouGet` prennent **`view: MediaView`**, plus `media: WatchingMedia`. Une nouvelle surface **ne peut physiquement pas** afficher une position sans passer par le constructeur : les composants qu'elle réutilise n'acceptent plus la ligne brute. *C'est exactement le truc de `claimedStatus` : la mauvaise chose ne compile plus.* La forêt de ternaires de `[id]/page.tsx:386-404` s'effondre en `view.seasons`, et `hideSeasonStepper` (rustine qui n'existe QUE parce que StatusCard ne voit pas la lentille) **est supprimée**.
2. **Un garde ESLint pour le résidu** : `no-restricted-syntax` sur `.current_season|.current_episode|.season_episodes` dans `components/**`.

Les **écritures ne changent pas** : on convertit une fois au bord (`view.toStorage(...)`) puis on passe à `positionPatch`.

### 3.2 Le chemin d'ajout reçoit les deux verrous du chemin de mise à jour

1. **Chaque porte énonce une réclamation** : la porte inProgress passe ses champs **comme position**. Dans `useAddMedia`, supprimer l'assemblage drapeau par drapeau et tout dériver de `addStatusPatch(door, claim, type)` — le frère côté ajout de `claimedStatus`.
2. **Une barrière d'insertion**, miroir de celle d'update : `insertMediaSchema` avec `superRefine` — **exactement un** statut vrai, et toute écriture de position porte `caught_up_at`. ⇒ **une ligne limbo devient impossible à insérer, depuis n'importe quelle porte, y compris celles qui n'existent pas encore.**
3. **Fermer le contournement** : les 3 branches `update:*` passent par la barrière.
4. **Honnêteté UI** : si série + porte de réclamation + `position === null` ⇒ submit désactivé, « Dis-nous jusqu'où tu es allé ».

### 3.3 La possession reçoit une autorité unique
`findOwnedMediaId(userId, tmdbId)` **sans filtre `.eq("type")`** → renvoie `{id, type}[]` ; exclure les stubs `is_reference`. `tmdbResultType` est **rétrogradé au rang d'indice de routage**, jamais d'identité.

---

## 4. Tous les flux — points de désaccord entre portes

**7 portes d'ajout.** Désaccords **réels et actuels** :
- Insert série via porte inProgress (`in_progress:false`) **vs** même geste sur une ligne existante (`update:inProgress` force `in_progress:true`). **Même geste, résultat opposé selon que tu possédais déjà la ligne.**
- `update:inProgress` écrit une position sans `caught_up_at` ; `setPosition` ne le peut pas (schéma). Même champ, une porte gardée, une porte ouverte.
- Le mapping TMDB→ligne existe en **4 copies** (`useAddMedia.insertData`, `addTmdbItemToList.meta`, `getTmdbDetails`, `selectResult`). Elles s'accordent aujourd'hui **par discipline**. Un `tmdbToRow(details)` supprimerait cette taxe — priorité moyenne.

**Flux modèle à copier** : le « +1 » (`NextEpisodeButton` → `nextStep` dérivé du diffusé → `setPosition(kind:"viewing")` → `positionPatch` → barrière zod → invalidations ciblées, Undo par le même builder en `"correction"`).

**Sync** : les crons écrivent les faits **du monde** ; toi tu écris ta **posture**. Deux écrivains, colonnes disjointes — sain. Le pattern « déplacer la latence TMDB dans un cron, lire une ligne de cache » (trending, for-you) **est la meilleure idée perf du module** et mérite d'être nommée dans CLAUDE.md.

---

## 5. PERFORMANCE — les vraies causes, classées

### P0 — fort impact, quelques heures
- **5.1 La fiche démarre toujours à froid.** `useMediaItem` : `staleTime: 0`, aucune graine depuis les caches de sections — alors que **la carte cliquée contient déjà** titre, poster, backdrop, année, position. ⇒ ajouter `placeholderData` piochant dans `WATCHING_KEYS.all`. **Ce seul changement tue l'essentiel de la sensation de « rechargement complet ».**
- **5.2 Les durées de cache font revenir les skeletons.** `gcTime: 5min` ⇒ toute pause > 5 min vide le cache ⇒ skeleton complet au retour. Ce sont **tes propres lignes**, et chaque écriture invalide explicitement ⇒ `staleTime` 10-15 min et `gcTime` 1 h **sans aucun risque**. Gain de vitesse ressentie le moins cher du module.
- **5.3 Les clés non commitées mettent TMDB dans le rayon d'invalidation.** `useTmdbDetails` et `useOwnedMediaId` sont sous `WATCHING_KEYS.all` — que `useAddMedia/useDeleteMedia` détruisent en `refetchType:"all"` ⇒ chaque ajout **refetch tous les détails TMDB en cache**. → déplacer `tmdb-details` sous `TMDB_KEYS`. Garder `owned-row` sous `WATCHING_KEYS` **délibérément** (c'est ce qui fait marcher la redirection) + un commentaire le disant.
- **5.4 L'animation de route fait passer le cache pour de la lenteur.** `layout.tsx` : `motion.div key={pathname}` **démonte et remonte tout le sous-arbre** à chaque navigation, puis fond 220 ms — et chaque carousel rejoue son stagger (`delay: i*0.04`) **même quand les données viennent du cache**. ⇒ ~400 ms de théâtre sur une navigation instantanée. Retirer le `key`, et conditionner le stagger à la première arrivée des données.

### P1 — fort impact, une journée
- **5.5 Le proxy TMDB taxe deux fois.** Chaque appel = invocation serverless + `getClaims` + **aller-retour Redis Upstash** (20-80 ms) + TMDB. La fiche en tire 4-6. ⇒ (a) sauter le ratelimit pour une session authentifiée ; (b) **collapser le fan-out avec `append_to_response`** (`videos,watch/providers,external_ids,recommendations,credits,content_ratings` en **UN** appel, technique déjà utilisée par `getMediaDetails`) puis redistribuer via `setQueryData`. **6 aller-retours → 1.**
- **5.6 `library/page.tsx` fait encore `getUser()`** — l'aller-retour réseau de ~200-300 ms remplacé partout ailleurs par `getClaims()` (CLAUDE.md §8).

### P2 — réel mais moindre
- **5.7** Une requête par page de type au lieu de 5-7 (à faire **après** 5.2, et seulement si le besoin persiste).
- **⛔ NON recommandé** : passer les pages de type en RSC. Ça combat l'architecture de cache client choisie, réintroduit la péremption du Router Cache que `refetchType:"all"` a été bâti pour vaincre, et les gains ci-dessus le rendent inutile.

### 5.8 Verdict SKELETONS (la suspicion de l'owner)
**Le système de skeletons est BON** : `loading.tsx` reflète l'ordre des sections, `[id]/loading.tsx` réutilise le vrai `DetailSkeleton`, les skeletons épousent la mise en page (pas de saut), et `staleTimes: {dynamic: 180}` fait sauter `loading.tsx` aux revisites. **Les skeletons ne sont pas la cause — la fréquence à laquelle l'app décide de les montrer l'est**, c'est-à-dire 5.1 et 5.2. Corrige-les et les skeletons existants sont exactement justes.

---

## 6. Verdict sur le travail NON COMMITÉ

**Garder, avec 4 retouches avant commit. Ne rien jeter.** L'instinct est juste, et la ligne virtuelle (`getTmdbDetails`) est **élégante** : la fiche EST « un tmdb_id + ta ligne ». La route séparée est le **bon arbitrage**.

1. **RETOUCHER — la carte à 3 boutons.** Bâtie sur l'idée que `listContext` décide du statut ; pour une série ce n'est pas le cas et **ne doit pas l'être**. Mais les **promesses** des boutons sont bonnes → le correctif est §3.2 (position depuis les champs inProgress + `addStatusPatch` + barrière d'insertion). Alors « Start watching » démarre vraiment, et « Mark as watched » refuse de valider tant que la réclamation n'est pas énoncée. **Le texte peut rester.**
2. **RETOUCHER — namespaces de clés** (§5.3). Une ligne.
3. **RETOUCHER — possession sans filtre de type** (§2d), sinon un anime possédé en `serie` obtient une ligne dupliquée.
4. **RETOUCHER — `Episodes` de discover reçoit la lentille**, sinon un anime aplati se contredit de part et d'autre de l'ajout.
5. **GARDER tel quel** — `WatchingSearch` (la séparation ⌘K/catalogue est correcte), le slot `SectionHeader.actions`, `tmdbPathFromUrl`, le patch cours du modal.

---

## 7. Plan de migration — ordonné, chaque étape livrable seule

**Part de Watching : étapes 1-5 ≈ 3-4 jours.** 6-8 optionnelles.

| # | Étape | Vérifier par | Taille |
|---|---|---|---|
| **1** | **Verrou du statut à l'ajout** : porte inProgress passe `position` · `addStatusPatch` · `insertMediaSchema` · brancher les `update:*` sur la barrière · submit désactivé sans position | **Tests unitaires** (modules purs — étendre `watch-status.test.ts`), puis insérer une série fraîche par **chaque** porte et lire la ligne. **Corrige les symptômes AVANT que discover ne sorte.** | ~½ j |
| **2** | **Commiter le travail non commité** avec les retouches 2-4 du §6 | Ajouter depuis discover via les 3 boutons → bon rail ; la redirection vise la bonne fiche ; un ajout **ne refetch pas** tmdb-details (onglet Réseau) | ~½ j |
| **3** | **Perf quick wins** : gcTime/staleTime (5.2) · `placeholderData` (5.1) · retirer `key={pathname}` + stagger première-arrivée (5.4) · `getUser→getClaims` + importer `LIBRARY_COLUMNS` (5.6, §2c) | carte→fiche : le hero peint **instantanément**, sans skeleton · retour après 10 min d'inactivité : **pas** de skeleton · menu Library d'une série finie propose « Mark as watched » dès la 1ʳᵉ peinture | ~½-1 j |
| **4** | **La lentille** : `buildMediaView` + `useMediaView` + `useAnimeCoursMap` ; adoption **surface par surface** (un commit chacune) ; garde ESLint en dernier | Blue Lock/JJK : **le même « S2 E23 »** sur la puce, le stepper, la bande, le modal, le dashboard. Titres non-overlay : pixel-identiques | ~1-1½ j |
| **5** | **`append_to_response`** + ratelimit conditionnel (5.5) | Ouverture à froid : **1** appel proxy au lieu de 5-6 | ~½ j |
| 6 | (Opt.) Une requête par page de type (5.7) | 1 appel `media_items` | ~½ j |
| 7 | (Opt.) `tmdbToRow(details)` unifiant les 4 copies | Ligne insérée via modal vs liste vs discover : méta identiques | ~½ j |
| 8 | (Différé) `series_facts` — conditions inchangées. **La lentille (4) n'en dépend pas.** | — | — |

**Pourquoi cet ordre** : 1 supprime la classe de corruption (lignes limbo) **avant** qu'une nouvelle porte n'ouvre ; 2 livre la feature visible sur terrain sain ; 3 est le gain de vitesse ressentie à risque quasi nul ; 4 est le fix structurel qui rend la **prochaine** surface gratuite ; 5-7 composent. **Chaque étape laisse `main` vert.**

## ⛔ Ce qui a été DÉLIBÉRÉMENT écarté (filtre anti-sur-ingénierie)
Pas de système de types « branded » pour les coordonnées (le changement de props + la règle ESLint obtiennent la même contrainte pour un dixième du remue-ménage) · pas de migration RSC des pages de type · pas de bus d'événements ni de sync push (la dérivation à la lecture suffit) · pas de « registre d'invariants » abstrait (trois barrières concrètes valent mieux qu'une abstraction) · pas de `series_facts` immédiat.

**Surface totale ajoutée : ~3 fichiers + 1 règle de lint, contre ~6 suppressions de rustines par-surface.**
