# ARCHIVE — Watching, la constitution d'architecture (version du 2026-07-23)

> **Archivé le 2026-09-13** (audit Watching, phase 3). Ce document était
> `hq/modules/watching/architecture.md`. Ses **invariants** (§1) sont repris, vérifiés dans le code,
> dans le fichier vivant du même nom — au format tableau du template. Ses **sections d'état** (§2.1,
> §2.3, §2.6, §4, Annexe) décrivaient l'adoption surface par surface au 23/07 ; elles sont
> remplacées par `hq/modules/watching/audit-2026-09.md`. Ce qu'elles affirmaient de faux au
> 2026-09-13 : « `getClaims()` partout » (4 `auth.getUser()` vivent dans `service.ts`) · « régression
> `useTmdbDetails` sous `WATCHING_KEYS.all` » (corrigée : la clé est `TMDB_KEYS.bundle`, via
> `useTitleBundle`) · « `discover/Episodes` non vérifié » (couvert par le garde ESLint, vérifié) ·
> « staleTime 10-15 min / gcTime 1 h » (remplacé par les paliers `STALE.*`, doctrine R4).
> Le raisonnement (§0, §2.5 « traduire tard ») reste vrai — §2.5 vit aussi dans `decisions.md`.

---

# WATCHING — Architecture (la constitution)

> **LA source de vérité d'architecture du module Watching.** Une seule, finale, robuste.
> Le raisonnement historique (audits, diagnostics) est archivé dans `hq/archive/` — il explique le
> *pourquoi*, ce document énonce le *quoi* et *où c'est forcé*.
>
> **Comment s'en servir :** avant tout changement sur Watching, vérifie-le contre les **5 invariants**
> du §1. Si ton changement touche un fait qui a déjà un point d'étranglement (§1), tu es couvert. Si
> tu découvres un invariant **sans** point d'étranglement, **c'est là — et seulement là — qu'on code
> une barrière**. Ce protocole (§6) remplace les audits à répétition.

---

## 0. La loi fondamentale — pourquoi ce module a saigné

Un invariant vit à un **point d'étranglement** (un seul endroit qui le *force*), **jamais dans la
mémoire d'un call site**. Toute la saga du module est une seule maladie, rejouée : une règle **écrite
en commentaire**, puis **violée vingt lignes plus bas**.

- `watched` confondait « à jour » et « terminé » → 25 lignes fausses.
- Le statut était assemblé **drapeau par drapeau** à 4 endroits → ils ont divergé.
- La liste était **stockée** au lieu d'être **dérivée** → 3 portes fabriquaient des lignes fausses.
- L'overlay de cours est appliqué **à la main** sur 2 surfaces, **oublié sur 6** → la 7ᵉ (les années)
  a resurgi le 20/07 (Blue Lock).

La leçon n'est pas « il faut mieux se relire ». C'est : **rendre la mauvaise chose impossible à
écrire.** Un invariant qui dépend du prochain développeur qui s'en souvient n'est pas un invariant.

---

## 1. Les 5 invariants — chacun : la règle · le point d'étranglement · la faute évitée

### (a) UN STATUT EST UNE DÉRIVATION PURE — jamais un objet assemblé à la main
**Règle.** Le statut d'un titre (`watched`/`in_progress`/`paused`/`dropped`/`want_to_watch`/`reference`)
se **dérive** de la position et des faits du monde. Aucune décision de statut hors du cœur pur.
**Point d'étranglement.** `lib/watch-status.ts` + `lib/series-state.ts` — **functional core** : zéro
I/O, toutes les décisions. `claimedStatus`, `addStatusPatch`, `markWatchedPatch`, `positionPatch`,
`deriveWatchStatus`, `seriesState`. Les hooks/composants ne font que *transporter* le patch.
**Faute évitée.** La ligne limbo (aucun statut vrai), le titre « watched » sur une série en cours,
« Watched » sur un écran et « Dropped » sur le suivant.
**Pattern.** *Functional Core, Imperative Shell* (Bernhardt) + *make illegal states unrepresentable*.

### (b) UNE ÉCRITURE EST UN PARSE QUI REFUSE L'ILLÉGAL
**Règle.** Toute écriture en base passe par un schéma Zod qui **refuse** l'état illégal — pas « valide »,
**refuse**. Un `parse()` qui strippe, exécuté dans le tuyau.
**Point d'étranglement.** `schemas/media.schema.ts` : `insertMediaSchema` (parsé **dans**
`insertMediaItem`) et `updateMediaSchema` (parsé **dans `updateMediaItem`** — pas seulement dans
`useUpdateMedia`). Invariants forcés : **exactement un** statut vrai (insert) · une **position porte**
son `caught_up_at` (insert + update) · une série qu'on regarde **a** une position (insert).
**Faute évitée.** La ligne limbo insérable depuis n'importe quelle porte (y compris pas encore écrite) ;
une position écrite sans recalcul de `caught_up_at` (le bug de l'Undo du « +1 »).
**⚠️ La barrière est DANS LE TUYAU, pas dans la mémoire des call sites.** `updateMediaItem` était le
maillon faible (barrière seulement dans `useUpdateMedia`, contournée par `addTmdbItemToList`) —
corrigé : le parse vit **dans `updateMediaItem`**, comme l'insert. C'est un parse de **validation
seule** : on ne réécrit **jamais** sa sortie (on écrit `data` brut), donc les colonnes « faits du
monde » non déclarées sont écrites intactes ; le parse ne fait que **jeter** si l'invariant est violé.
**Pattern.** *Parse, don't validate* (Alexis King). Le mauvais objet **ne parse pas**, comme au niveau
types il **ne compile pas**.
**⚠️ Piège vécu.** `updateMediaSchema` **strippe** les clés non déclarées → une branche qui y fait
passer un patch perd **en silence** les colonnes non déclarées (`season_episodes`, `priority`…).
Déclarer = préserver.

### (c) LES COORDONNÉES SONT UNE LENTILLE UNIQUE — *le point d'étranglement manquant, à construire (§2)*
**Règle.** Aucune surface ne lit `current_season`/`current_episode`/`season_*`/`*_years` brut pour
**afficher** ou **stamper**. Elle consomme un `MediaView`, seul objet capable de traduire entre
l'espace de stockage (TMDB à plat) et l'espace d'affichage (cours), **en lecture ET en écriture**.
**Point d'étranglement.** `lib/media-view.ts` + `hooks/useMediaView.ts` (§2). Aujourd'hui : `lib/
anime-overlay.ts` fait la moitié (les **positions**), appliqué à **2 surfaces sur 8**.
**Faute évitée.** L'overlay « oublié » : StatusCard épisode plat, cartes carousel brutes, et **le bug
du 20/07 — les années d'un anime overlay écrites dans `season_years` (coord TMDB) mais affichées
depuis `cour_years` (coord cours), deux colonnes qui ne se parlent pas.**
**Pattern.** *Read model / view-model* (CQRS-lite). Une lecture dérivée, un constructeur unique.

### (d) UN FAIT A UN SEUL PROPRIÉTAIRE — « derive don't store »
**Règle.** Chaque fait a **un** écrivain. **Les faits du monde** (`season_aired`, `status`,
`season_end_dates`…) → le **cron de sync**. **Ta posture** (`current_*`, stance, `*_years`,
`*_ratings`) → **toi**. Deux écrivains, **colonnes disjointes**. Tout ce qui est dérivable
(`seriesState`, « caught-up », « last watched ») est **calculé à la lecture**, jamais stocké.
**Point d'étranglement.** La séparation des colonnes + `series-state.ts` (les dérivations).
**Faute évitée.** Deux copies d'une série qui divergent ; un booléen figé qu'on oublie de rebasculer.

### (e) LA SÉCURITÉ VIT DANS LA BASE — la RLS est le mur, l'app est du confort
**Règle.** L'isolation est garantie par **RLS `org_isolation`** (via `my_orgs()`), pas par du code
applicatif. Les tables de **référence partagée** (`trending_cache`, `for_you_cache`, `anime_cours`,
`series_facts` à venir) = **lecture globale / écriture service-role, sans `org_id`**. Garde démo
read-only sur les tables mutables.
**Point d'étranglement.** Les policies RLS Postgres. Un check applicatif oublié n'ouvre aucune brèche.
**Faute évitée.** Une fuite cross-org par un `select` sans filtre — la RLS la refuse à la source.
**Détail clé** (CLAUDE.md §6 bis) : les crons ne portent **aucun secret** ; la clé privilégiée
(`HEGON_SECRET_KEY`) ne vit que dans 3 endroits privés.

### (f) LA POSSESSION EST DÉCIDÉE PAR `tmdb_id`, PAS PAR `type` — le type est un indice, pas une identité
**Règle.** « Possèdes-tu déjà ce titre ? » se répond par `tmdb_id` **seul**. Le `type`
(`film`/`serie`/`anime`) est un **choix de routage**, jamais une clé de possession.
**Point d'étranglement.** `findOwnedMediaId(userId, tmdbId)` **sans** filtre `.eq("type")`, qui renvoie
`{id, type}[]` (en excluant les stubs `is_reference`). `tmdbResultType` est rétrogradé au rang d'indice.
**Faute évitée.** Un anime rangé en `serie` (ajouté avant le type « anime », ou deviné à tort par
`tmdbResultType`) échappe à `findOwnedMediaId(..., "anime")` → discover le croit neuf → **doublon
créé** (l'index unique le permet). C'était le « candidat n°1 » du bug « want-to-watch n'ajoute pas
l'anime ». **✅ Corrigé et vérifié dans le code** (`service.ts`, `findOwnedMediaId`/`findOwnedMediaIds`) :
aucun `.eq("type")`, la fonction ne filtre que sur `tmdb_id` + `is_reference`, avec un commentaire
in-file qui documente exactement cette règle.

---

## 2. La lentille `MediaView` — le chokepoint des coordonnées (✅ construite, adoptée sur la majorité des surfaces)

Le module a guéri les **statuts** avec un constructeur unique (`claimedStatus`). Les **coordonnées**
ont eu le même remède : `lib/media-view.ts` (+ `hooks/useMediaView.ts`) existe, est testé, et est
**câblé** dans le détail (StatusCard, strip, puce de position), les cartes (`MediaCarousel` via
`useMediaViews`), l'écrivain unique (`useWatchActions`, `NextEpisodeButton`) et la modale d'ajout
(`AddMediaModal` construit une vue synthétique via `buildMediaView`, alimente `HowFarDidYouGet`).
Le garde ESLint `no-restricted-syntax` sur les lectures de coordonnées brutes dans `components/**`
est en place et au niveau **`error`** (pas `warn`). Reste non vérifié à ce jour : l'adoption sur
`discover/Episodes`. Le détail historique (l'overlay `anime-overlay.ts` limité à 2 endroits, la
lentille manquante) est **antérieur au 20-21/07** — gardé ci-dessous pour le raisonnement, pas comme
état courant.

### 2.1 Deux espaces, deux traductions — dont une neuve
| | Stockage (TMDB à plat) | Affichage (cours) | État aujourd'hui |
|---|---|---|---|
| **Position** | `current_season/episode` | `flatToCour`/`courToFlat` | ✅ existe (`anime-overlay.ts`) |
| **Carte d'années** | `season_years` (clé = saison TMDB) | `cour_years` (clé = cour) | ❌ **routé nulle part** → bug 20/07 |
| **Carte de notes** | `season_ratings` | `cour_ratings` | ❌ même trou |

Pour un anime « lumpé » (1 saison TMDB), `season_years` **ne peut exprimer qu'une** valeur pour tout
le show → une année par cour **ne peut vivre que dans `cour_years`**, aujourd'hui écrit **uniquement
par l'édition manuelle**. Donc aucun stamp automatique (`markWatchedPatch`, `positionPatch`,
`addStatusPatch`) n'apparaît jamais sur un anime overlay. **C'est le bug, et sa racine est
l'absence de routage.**

### 2.2 Le contrat
```ts
export interface MediaView {
  media: WatchingMedia;                 // ligne brute (coord stockage) — pour les libs pures
  overlaid: boolean;
  position: { season: number; episode: number; label: string };  // coord AFFICHAGE
  seasons: { episodes: number; aired: number; poster: string | null; endDate: string | null }[];
  yearMap:  Record<string, number> | null;   // cour_years si overlay, sinon season_years
  ratingMap: Record<string, number> | null;
  // LES SEULS convertisseurs sanctionnés — position ET cartes d'années/notes.
  toStorage(season: number, episode: number): { season: number; episode: number };
  fromStorage(season: number, episode: number): { season: number; episode: number };
  // Écriture d'une carte : route vers la BONNE colonne (season_years vs cour_years) et
  // renvoie le patch d'update prêt pour la barrière. C'est ÇA qui ferme le bug du 20/07.
  writeYear(season: number, year: number): Partial<UpdateMediaInput>;
  writeRating(season: number, value: number | null): Partial<UpdateMediaInput>;
}
export function buildMediaView(media: WatchingMedia, cours: AnimeCour[] | null): MediaView;
export function useMediaView(media): MediaView | null;      // branche useAnimeCours seul
export function useAnimeCoursMap(items): Map<number, AnimeCoursRow>;  // 1 lecture batch, pas N
```

### 2.3 Ce qui rend l'application MÉCANIQUE (pas une convention)
1. **Les props changent.** `StatusCard`, `SeasonHistoryStrip`, la puce de position, `HowFarDidYouGet`
   prennent **`view: MediaView`**, plus `media` brut. Une nouvelle surface **ne peut physiquement
   pas** afficher une position sans passer par le constructeur : le composant qu'elle réutilise
   n'accepte plus la ligne brute. *La mauvaise chose ne compile plus* — exactement le truc de
   `claimedStatus`. La rustine `hideSeasonStepper` disparaît.
2. **Un garde ESLint** `no-restricted-syntax` sur `.current_season|.current_episode|.season_episodes|
   .season_years|.cour_years` dans `components/**` attrape le résidu.
3. **Le stamp automatique passe par `view.writeYear`.** ⚠️ **QUATRE** fonctions codent `season_years`
   en dur, pas une — `markWatchedPatch`, `markCaughtUpPatch`, `addStatusPatch`, `positionPatch` — et
   **les quatre** doivent router par `view.writeYear`, sinon « Mark as watched »/« Caught up »/l'ajout
   reproduisent le bug par les portes que le fix n'a pas touchées. Elles travaillent en « saisons de la
   vue », le convertisseur route vers `season_years` **ou** `cour_years`. Pour un anime overlay, finir
   un cour stampe **`cour_years` en coord cour** → visible. **C'est le fix du bug, structurel.**

### 2.5 La direction de conversion — TRADUIRE TARD (décision d'archi, tranchée)
Le point le plus subtil, et le seul qui rende le fix correct. **Les call sites cessent de pré-convertir**
(aujourd'hui `handleCourSetPosition` convertit cour→plat **avant** `positionPatch`, si bien que
`positionPatch` raisonne à plat et ne **voit jamais** une frontière de cour → aucun stamp). À la place :
> **Les fonctions pures reçoivent des unités VUE + le `MediaView`, et convertissent au BORD** (via
> `view.toStorage()` pour la position, `view.writeYear()` pour l'année). Le stockage n'entre **jamais**
> dans une décision pure pour un titre overlay.

Pas de cycle de dépendance : `media-view.ts` est **pur** (se construit depuis `media` + cours, aucune
I/O) ; `watch-status.ts` **importe** `media-view.ts`, jamais l'inverse. Sens unique entre deux modules
purs.
**Garde-fou schéma manquant** : `cour_years`/`cour_ratings` ne sont légitimes que si `shouldOverlay`
est vrai — le schéma doit refuser un `cour_years` écrit sur un film ou une série non-overlay (contrainte
type-conditionnelle, à ajouter).

### 2.6 Adoption — surface par surface (état vérifié dans le code, pas déduit)
Séquence prévue : Detail (StatusCard, strip, puce) → cartes `MediaCarousel` → `NextEpisodeButton` →
`LibraryCard` → Dashboard « IN PROGRESS » → modale (`HowFarDidYouGet`) → discover `Episodes` → garde
ESLint en dernier.
**✅ Faites et vérifiées dans le code :** Detail (StatusCard prend `view?: MediaView | null`, commentaire
in-file « THE LENS » — toutes les saisons affichées viennent de `view.seasons`) · cartes `MediaCarousel`
(`useMediaViews(items)`, `view` passé à chaque item) · `NextEpisodeButton` (`actions.view.fromStorage`)
· modale d'ajout (`AddMediaModal` construit une vue synthétique, `HowFarDidYouGet` reçoit
`addView.seasons.map(s => s.aired)` — coordonnées cour, pas brutes) · **garde ESLint au niveau
`error`** (pas `warn` : la promotion prévue a été faite).
**Non applicables, pas des surfaces en attente :** `LibraryCard` et le widget Dashboard n'affichent
aucune position — rien à adopter là.
**Non vérifié à ce jour :** l'adoption sur `discover/Episodes` (aucune trace trouvée lors du dernier
passage dans le code — à confirmer avant de le considérer clos).
**Historique (pour mémoire, plus l'état courant) — les 2 bugs qui ont motivé cette adoption :**
- **StatusCard** recevait des coords **brutes** (`page.tsx` passait `currentSeason/currentEpisode`, pas
  `overlay.current…`) → sur Blue Lock, le stepper affichait l'épisode **à plat** (« 47 ») à côté d'un
  historique en cours. `hideSeasonStepper` ne masquait que la saison, pas l'épisode. **Fixé** — le
  composant prend maintenant `view`.
- **`HowFarDidYouGet`** lisait `seasonAired` **brut TMDB** (jamais `buildOverlay`) → on ne pouvait dire
  que « tout » ou « rien » d'un anime lumpé, jamais « j'ai vu le cour 1 ». **Fixé** — alimenté par
  `addView.seasons` (coordonnées cour) depuis `AddMediaModal`.

---

## 3. Connectivité inter-modules — la règle unique

**Un module ne touche JAMAIS les tables d'un autre.** Watching **n'écrit pas** dans les tables de
Goals ; il **appelle une fonction que Goals expose**. Le contrat est une **interface étroite et
typée**, jamais un accès direct. C'est ce qui empêche le monolithe modulaire de pourrir en boule de
boue.

- **Aujourd'hui** : `syncWatchingGoals` / `syncWatchingHabits` appelés dans le `onSuccess` d'une
  mutation. À **centraliser** en un seul point de fan-out au bord de l'écriture — pas éparpillé.
- **Pas de bus d'événements.** Choix assumé : à l'échelle mono-utilisateur, un event bus paie de
  l'indirection et du debug asynchrone pour un découplage dont on n'a pas besoin. L'orchestration
  explicite au bord de l'écriture suffit. (Reconsidérer **seulement** si 3+ modules réagissent à un
  même fait de façon non triviale.)

---

## 4. Performance — distinguer l'architecture des tactiques

**Le seul pattern PERF architectural** (à nommer, à réutiliser partout) :
> **Déplacer la latence dans un cron, lire une ligne de cache.** `trending_cache`, `for_you_cache`,
> `anime_cours` : un job précalcule le coûteux/partagé, l'app lit instantanément. La meilleure idée
> perf du module.

**Tactiques** (pas de l'architecture, mais obligatoires) :
- **Cache client discipliné** : `staleTime` 10-15 min / `gcTime` 1 h **sans risque**, parce que chaque
  écriture **invalide explicitement**. Un `gcTime` court fait revenir les skeletons pour rien.
- **`placeholderData`** depuis les caches de sections → la fiche peint instantanément (la carte cliquée
  contient déjà titre/poster/position).
- **Pas de `key={pathname}`** sur le layout : il démonte/remonte tout l'arbre → faux « rechargement ».
- **`append_to_response`** : collapser le fan-out TMDB (6 aller-retours → 1).
- **`getClaims()` (pas `getUser()`)** partout — vérif JWT in-process (CLAUDE.md §8).
- **⚠️ Régression connue** : `useTmdbDetails` a sa query-key sous `WATCHING_KEYS.all` → chaque ajout
  (`refetchType:"all"`) **refetch tous les détails TMDB en cache**. La déplacer sous `TMDB_KEYS`.

---

## 5. Délibérément exclu (filtre anti-sur-ingénierie)
Pas de types « branded » pour les coordonnées (le changement de props + le garde ESLint obtiennent la
même contrainte à un dixième du bruit) · pas de migration RSC des pages de type (combat le cache client
choisi) · pas de bus d'événements · pas de `series_facts` immédiat (différé, déclencheur = AniList v2
**ou** ouverture des inscriptions) · pas de « registre d'invariants » abstrait (trois barrières
concrètes valent mieux qu'une abstraction).

---

## 6. Le protocole qui remplace les audits
1. **Tout changement se vérifie contre les 5 invariants du §1.** S'il touche un fait déjà couvert par
   un point d'étranglement, c'est mécaniquement sûr.
2. **Un invariant sans point d'étranglement pour ton cas = le signal de coder une barrière** — là,
   pas ailleurs. (Ex : les années overlay n'avaient pas de routage → §2.)
3. **Vérifier en EXÉCUTANT, pas en compilant.** `tsc`/`lint`/tests ne prouvent pas l'app. Preuves
   réelles : le domaine pur en **tests unitaires**, le câblage via **harness screenshot**
   (`scripts/shoot.mjs`) et **lecture DB authentifiée** (session réutilisée, RLS respectée).
   *(Validé le 20/07 : le fix « In Progress » prouvé de bout en bout — ajout Andor → `season_years:
   {"1":2026}`, `in_progress:true`, pas de limbo — puis ligne supprimée.)*

---

## Annexe — état d'avancement (mis à jour, vérifié dans le code au 23/07/2026)
**Revue adversariale Fable faite** (6 trous, tous réels, absorbés ci-dessus).
- (a) statut pur : **en place**.
- (b) parse barrier : **en place + renforcée** — la barrière update descendue **dans `updateMediaItem`**
  (4 tests). ✅ Fable #1 fermé.
- (c) lentille `MediaView` : **✅ construite et adoptée** sur Detail, cartes `MediaCarousel`,
  `NextEpisodeButton`, la modale d'ajout. Ferme les années overlay (les fonctions de stamp routent via
  `view.writeYear`) + les 2 anciens bugs vivants (StatusCard, HowFarDidYouGet — voir §2.6). Direction de
  conversion tranchée : **traduire tard, unités vue** (§2.5). Reste non vérifié : `discover/Episodes`.
- (d) un fait / un propriétaire : **en place**.
- (e) sécurité RLS : **en place** (Fable a confirmé zéro reach inter-module).
- (f) possession par `tmdb_id` : **✅ corrigé** — `findOwnedMediaId`/`findOwnedMediaIds` ne filtrent
  plus par `type`, vérifié dans `service.ts`.
- Perf : `useTmdbDetails` key à déplacer (régression connue — non revérifiée dans cette passe).

Plan d'exécution complet : voir le raisonnement archivé `hq/archive/`. Le détail des arbitrages
(SSR, cour≠saison, etc.) vit maintenant dans `decisions.md` de ce même dossier — pas ici.
