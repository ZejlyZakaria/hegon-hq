# Audit architecture — Watching, post-chantier « diffusé ≠ annoncé »

> Fable · 2026-07-14 · **Lecture seule respectée : zéro fichier de code modifié, zéro écriture en base, zéro appel réseau en écriture.** Unique livrable : ce fichier.

**Lu en entier** : `lib/series-state.ts`, `supabase/functions/watching-series-sync/index.ts`, `lib/progress.ts`, `lib/status-flags.ts`, `lib/season-years.ts`, `hooks/useUpdateMedia.ts`, `hooks/useSeasonRefresh.ts`, `components/shared/NextEpisodeButton.tsx`, `hq/now.md`. **Lu en larges extraits ciblés** : `page.tsx` (handlers 100-380), `StatusCard.tsx` (180-390), `SeasonHistoryStrip.tsx` (40-170), `Episodes.tsx` (55-120, 400-455), `MediaActionMenu.tsx` (100-245), + sondages grep systématiques (imports de series-state, `season_episodes`, `air_date`, `select("*")`, supabase hors service, schéma update). **Non lus en entier** : `series-state.test.ts` (existence + 337 l. vérifiées, contenu survolé), `useAddMedia.ts` (sondé aux lignes clés), `AddMediaModal`/`HowFarDidYouGet`, `computeStats.ts` (sondé), `service.ts` (sondé). Chaque constat porte sa confiance.

---

## Verdict d'ensemble (demandé : challenger le modèle)

**Le modèle est juste, et c'est le meilleur morceau d'architecture du repo.** Deux nombres (vu / diffusé), des positions plutôt que des drapeaux, l'annoncé qui s'affiche sans jamais décider, un job qui fait bouger la vérité du monde — c'est la bonne décomposition, et les commentaires du fichier valent une spec. Je ne propose **pas** de le remplacer. Mes trois critiques structurelles honnêtes :

1. **C'est un modèle dérivé posé SUR un modèle stocké qui n'a pas désarmé.** `seriesState()` ignore les booléens ; mais les booléens restent écrits partout, restent lus partout (Library, sections, Stats, cartes), et restent même des ENTRÉES de la dérivation (`hasReachedSeason(!!m.watched…)`). Deux machines à états coexistent — c'est de là que viennent les findings 1 et 2 ci-dessous. Le chantier a fermé les portes une à une ; l'état final devrait être *nommé* : **stocké = les faits indérivables (position, stances, intention, timestamps) ; tout mot de statut = dérivé à la lecture.** Tant que ce n'est pas écrit, chaque nouvelle surface devra re-choisir à quel modèle elle croit.
2. **L'invariant `caught_up_at` repose sur la discipline des call-sites** (« every write of a POSITION must recompute it ») au lieu d'être imposé au point d'écriture. Cinq écrivains le font, un l'oublie déjà (finding 3). Un invariant qui dépend de la mémoire du prochain contributeur n'est pas un invariant.
3. **Le clamp cache les contradictions au lieu de les signaler.** `watchedCount` borne une position > diffusé (bien, pour l'affichage) mais rien ne *remonte* qu'une ligne affirme l'impossible — les 3 lignes que tu dois corriger à la main (now.md) ont été trouvées par accident, pas par le système. Le job de sync voit passer chaque ligne : c'est l'endroit gratuit pour loguer `position > aired`.

---

## Constats, du plus grave au plus léger

### 1 · 🔴 BLOQUANT — La 7ᵉ porte : le menu « … » des CARTES écrit encore `watched:true` sans position ni garde
- **Fichier** : `components/shared/MediaActionMenu.tsx:126-150` (écriture) et `:231-239` (l'offre : `{!item.watched && … item.in_progress ? "Mark as finished" : "Mark as watched"}`).
- **Confiance : certain** (code lu).
- **Quoi** : sur chaque carte de carrousel et chaque carte Library, une série EN COURS non-watched propose « Mark as finished ». Le handler écrit `watched:true, in_progress:false, watched_at:now` — **sans poser la position, sans `isFinished`, sans stampage, sans `caught_up_at`** : mot pour mot la fabrique des 23 lignes fantômes que `page.tsx:119-123` documente et que la fiche a fermée le 14/07. La porte principale et la porte de service de la FICHE sont verrouillées ; **toutes les cartes de l'app restent ouvertes**.
- **Symptôme concret** : un tap sur « Mark as finished » sur la carte House of the Dragon → `watched:true` avec position S1E2. La Library la classe « Completed », `seriesState` (qui ignore le booléen) dit « watching », QuickStats (qui special-case le booléen) compte tout comme vu. Puis la sync « répare » en `caught-up` dans l'heure **et pose `caught_up_at`** (`index.ts:187`) — pour quelqu'un qui n'a jamais été à la frontière → au prochain épisode diffusé, la carte s'allume « New episodes » : une fausse nouvelle fabriquée en deux étapes de bonne foi.
- **Correctif** : même chirurgie que la fiche — l'item devient « Mark as caught up » quand `isSeries && !isFinished(status)`, et le handler pose `lastAiredPosition` + `caughtUpAt` + ne stampe que les saisons `isSeasonComplete` (ou, plus simple : extraire `markWatched`/`markCaughtUp` de `page.tsx` vers un hook partagé et l'appeler des deux endroits — la logique existe déjà, elle est juste emprisonnée dans la page).

### 2 · 🔴 IMPORTANT — Deux ordres de priorité pour « quel est le statut de cette ligne »
- **Fichiers** : `components/detail/StatusCard.tsx:38-44` (`statusOf` : **watched > dropped > paused**) vs `service.ts:486-494` (`deriveWatchStatus` : **dropped > paused > watched**) — et `lib/status-flags.ts:3-4` documente noir sur blanc que la priorité officielle est celle du service.
- **Confiance : certain** (les deux fonctions lues côte à côte).
- **Quoi** : la question « cette ligne est-elle watched ou dropped ? » a deux réponses selon la surface. C'est EXACTEMENT ta classe de bug n°1 (deux règles, deux vérités, un écran) — appliquée au mot le plus central du module.
- **Symptôme concret** : une ligne `watched:true + dropped:true` (héritage pré-RESET_STATUS, ou n'importe quel write partiel — le schéma zod accepte chaque booléen isolément, `schemas/media.schema.ts:22-26`) s'affiche « Watched » sur la fiche et « Dropped » dans une liste (`ListDetail` lit `watch_status`). Le commentaire de status-flags dit que RESET_STATUS existe *parce que* dropped doit gagner — la StatusCard dit l'inverse.
- **Correctif** : une seule fonction exportée (dans `status-flags.ts` ou `series-state.ts`), consommée par les deux. Le duplicata local de StatusCard meurt.

### 3 · 🔴 IMPORTANT — L'Undo du « +1 » viole l'invariant `caught_up_at` (écriture de position sans recalcul)
- **Fichier** : `components/shared/NextEpisodeButton.tsx:111-117` ; l'invariant est énoncé dans `lib/series-state.ts:330` (« Every write of a POSITION must recompute it ») et respecté partout ailleurs (`page.tsx:308, 354, 202` ; le forward du bouton lui-même `:95`).
- **Confiance : certain** pour le code ; le scénario symptôme est déduit, pas reproduit.
- **Quoi** : tap « +1 » jusqu'à la frontière → `caught_up_at` posé. Tap **Undo** → la position recule mais l'écriture ne porte que `current_season/current_episode` : le stamp reste. (Secondairement : le `last_watched_at` posé par l'aller n'est pas repris par l'Undo — tu as « regardé » un épisode que tu viens de désavouer.)
- **Symptôme concret** : tu annules ton +1 (mauvais titre, doigt qui glisse), tu restes objectivement UN épisode derrière ; la sync fait diffuser l'épisode suivant → `seriesState` voit `seen < aired` + stamp → **« New episodes »** s'allume pour quelqu'un qui n'a jamais été à jour.
- **Correctif minimal** : l'Undo envoie aussi `caught_up_at: caughtUpAt({...item, position rétablie}, null)`. **Correctif juste** (recommandé, cf. verdict §2) : recalculer `caught_up_at` DANS `useUpdateMedia` dès que `current_season/current_episode` est présent — l'invariant cesse d'être une consigne et devient une propriété. Un seul endroit, six call-sites libérés.

### 4 · 🟠 IMPORTANT — « Combien d'épisodes ai-je vus » : TROIS implémentations, dont une sur l'ANNONCÉ
- **Fichiers** : `lib/series-state.ts:83-90` (`watchedCount`, clampé sur diffusé — la canonique) ; `components/detail/QuickStats.tsx:50-58` (ré-implémentation locale, clampée elle aussi mais avec son propre special-case `watched`) ; `components/stats/computeStats.ts:61-73` (ré-implémentation **sur `season_episodes`/`episodes`, l'annoncé, sans clamp**).
- **Confiance : certain** sur la triplication ; **suspicion** sur l'ampleur du delta chiffré (computeStats sondé, pas lu en entier).
- **Quoi** : la règle « le diffusé décide » s'arrête à la porte de la page Stats. Les heures et les compteurs y sont calculés depuis l'annoncé et la position brute non clampée.
- **Symptôme concret** : les 3 lignes corrompues que tu connais (True Detective S4E6…) — QuickStats sur la fiche les borne honnêtement, la page Stats compte les épisodes fantômes → **deux totaux d'heures différents pour le même titre selon l'écran**. Et une ligne `watched` héritée sans position compte `episodes` (annoncé total) même si une saison annoncée n'a pas fini de diffuser.
- **Correctif** : `computeStats` (et QuickStats) consomment `watchedCount()`/`airedCount()` — c'est un remplacement de 10 lignes par 2 appels ; le special-case `watched` a sa place DANS la canonique si besoin (`watched → airedCount`).

### 5 · 🟠 IMPORTANT — La sync répare « finished grew » sans nettoyer les stances, et fige `TODAY` au chargement de l'isolate
- **Fichier** : `supabase/functions/watching-series-sync/index.ts:198-212` (repair #2 : pas de `paused:false/dropped:false`, contrairement au repair #1 `:178-191`) ; `:32` (`const TODAY = …` au niveau module).
- **Confiance : certain** sur le code ; **suspicion** sur la durée de vie réelle des isolates Deno en prod.
- **Quoi** : (a) un show fini-qui-repart marqué `watched + dropped` (ligne héritée) ressort du repair en `in_progress:true + dropped:true` → finding 2 s'applique : « Watching » ici, « Dropped » là. (b) `TODAY` est calculé une fois par isolate : un isolate chaud à cheval sur minuit compte avec la date d'hier — un épisode diffusé « aujourd'hui » n'est vu par la sync qu'au recyclage de l'isolate. Pour un cron horaire c'est typiquement quelques heures de retard silencieux, exactement là où la fraîcheur compte (le soir de diffusion).
- **Correctif** : (a) ajouter `...{paused:false, dropped:false, drop_reason:null}` au repair #2 — ou mieux, décider si un revival doit réveiller un show que TU as droppé (probablement non ! auquel cas c'est le repair qui doit préserver la stance et NE PAS mettre in_progress — à trancher, c'est un choix produit) ; (b) `const today = () => new Date().toISOString().slice(0,10)` appelé dans le handler.

### 6 · 🟡 MOYEN — `useSeasonRefresh` : un deuxième écrivain de `season_episodes`, à moitié rendu obsolète par la sync
- **Fichier** : `hooks/useSeasonRefresh.ts:48-54` ; la sync écrit les mêmes colonnes (`index.ts:162-170`) plus `season_aired`/`season_end_dates` que le hook ignore.
- **Confiance : certain** sur le chevauchement ; « redondant » est un jugement.
- **Quoi** : à chaque ouverture de fiche d'un show ongoing : un appel TMDB + potentiellement un write qui met à jour l'ANNONCÉ (et posters/air_dates) **sans toucher le DIFFUSÉ**. Ce n'est pas incohérent (annoncé qui grandit → la saison neuve affiche « Coming soon »), mais c'est un deuxième chemin d'écriture des faits-du-monde avec sa propre périodicité, son propre filtre (`status !== "ongoing"` alors que la sync scanne aussi les finis pour les revivals) et pas de `last_synced_at` — le lot de sync ne sait pas que la fiche vient de rafraîchir.
- **Symptôme concret** : aucun bug utilisateur aujourd'hui ; le coût est un appel TMDB par ouverture de fiche et une classe de divergence future (deux définitions de « seasons réelles » si l'une évolue).
- **Correctif proposé** : la sync apprend `season_posters`/`season_air_dates` (elle télécharge déjà la saison !) et le hook meurt ; ou le hook devient « invoquer la sync pour CE tmdb_id » (edge function avec param). À défaut, le garder mais lui faire poser `last_synced_at`.

### 7 · 🟡 MOYEN — L'affichage « reste de saison » et la barre de progression raisonnent sur l'ANNONCÉ
- **Fichier** : `lib/progress.ts:13-33` (`episodesLeftInSeason`, `overallProgress` — consommés par `MediaCarousel.tsx`).
- **Confiance : certain** (code lu) ; gravité volontairement basse : c'est de l'AFFICHAGE, la règle « l'annoncé peut s'afficher » l'autorise.
- **Quoi/symptôme** : la carte In Progress de House of the Dragon peut dire « 5 left in season » quand un seul de ces 5 existe, et la barre d'un show à jour plafonne à ~50 % — deux messages qui contredisent le « Caught up » affiché juste à côté par `NextEpisodeButton` (lui, honnête). Ce n'est pas une décision, mais c'est le même écran qui dit deux choses.
- **Correctif** : afficher le reste **diffusé** (avec l'annoncé en second souffle si tu veux les deux vérités, comme le « 4 / 8 » du stepper) : `airedInSeason - current_episode`, fallback annoncé si pas de données.

### 8 · 🟡 MOYEN — Perf : la tempête d'invalidations sur chaque changement de statut, et `select("*")` sur la recherche
- **Fichiers** : `hooks/useUpdateMedia.ts:106-129` (statut → ~16 `invalidateQueries` en `refetchType:"all"` : movies/series/animes + library + 4 sections × 3 types) ; `service.ts:521` (`searchMediaForList` : `select("*")` — donc `description`, `notes`, `cast_members` jsonb backfillé, `season_*` arrays — par frappe de recherche).
- **Confiance : certain** sur le code ; **suspicion** sur le coût ressenti (non profilé, et le commentaire documente le POURQUOI Router-Cache du refetchType).
- **Symptôme concret** : un « Pause » sur une carte déclenche jusqu'à ~16 refetchs réseau dont des sections de types non concernés (pauser un anime refetch les carrousels films). Sur la recherche de liste, chaque frappe rapatrie des lignes complètes avec les blobs cast.
- **Correctif** : scoper les invalidations au `item.type` touché (les clés le permettent déjà) ; `searchMediaForList` → `select` des 6 colonnes que la rangée affiche.

### 9 · ⚪ COSMÉTIQUE — Duplication assumée mais non testée entre la sync (Deno) et `series-state`
- **Fichier** : `watching-series-sync/index.ts:33-34` (`isFinished`), `:50-53` (`lastAired` ≡ `lastAiredPosition`).
- **Confiance : certain** sur la duplication ; le risque est spéculatif.
- **Quoi** : deux copies de deux règles, séparées par la frontière Deno/Node. Aujourd'hui identiques ; rien ne le garantit demain (le jour où « canceled » gagne une nuance, une seule copie l'apprendra).
- **Correctif léger** : un `_shared/series-truth.ts` importé des deux côtés (les Edge Functions Supabase acceptent les imports relatifs hors du dossier), ou a minima un test de parité côté client qui fige les deux comportements.

---

## Les 5 booléens — faits, conséquences, falsifiabilité

| Drapeau | Nature aujourd'hui | Peut-on écrire un faux ? |
|---|---|---|
| `want_to_watch` | **FAIT** (une intention — indérivable) | Inoffensif |
| `paused` / `dropped` (+`drop_reason`) | **FAITS** (des stances — indérivables des nombres) | Oui, combinables avec `watched` par write partiel (le schéma accepte chaque booléen seul) → et le finding 2 fait diverger l'affichage |
| `in_progress` | **CONSÉQUENCE à ~90 %** : « engagé et ni paused/dropped/fini ». Encore écrit par 8+ chemins, encore lu par les sections | Oui — trivialement |
| `watched` | **Hybride, le plus dangereux** : FAIT pour un film ; pour une série c'est une conséquence (`seriesState()==="completed"`) qu'on stocke encore **et qui reste une ENTRÉE de la dérivation** (`hasReachedSeason`, `hasFullyWatchedSeason`, `hasReached` prennent `watched` en override) | **Oui — la porte du finding 1 est ouverte en prod** |
| `recently_watched` | **CONSÉQUENCE pure** de `watched_at` (déjà recalculée dans `withRecency`) — stockée par habitude | Oui, sans gravité |

**Ce que j'en ferais** (jugement) : l'état final à écrire noir sur blanc — la ligne stocke **position, stances, intention, timestamps** ; `watched`/`in_progress`/`recently_watched` deviennent des **vues dérivées** (pour les requêtes SQL des sections : colonnes générées ou vue, puisque Library/sections filtrent en base). Chemin incrémental sans migration risquée : d'abord le finding 3 (centraliser `caught_up_at`), puis funnel unique d'écriture de statut (un `writeStatus()` dans le service que TOUTES les surfaces appellent — fiche, cartes, sync, modale), et seulement ensuite discuter colonnes générées.

## Code mort / scories du chantier

- `lib/progress.ts` : le fichier annonce lui-même la mort de `nextEpisode` — ce qui reste est vivant mais annoncé-based (finding 7). RAS de mort réel ici.
- `hasReached`, `hasReachedSeason`, `airedCount` : exportés sans importeur externe — **pas morts** (internes à series-state + tests), mais l'API publique du module gagnerait à ne pas les exposer.
- `useSeasonRefresh` : pas mort, semi-redondant (finding 6).
- Rien d'autre de significatif trouvé côté module (les primitifs morts `tabs`/`command` relèvent de l'audit design-system, hors périmètre).

## Discipline des couches & divers

- **Aucune requête Supabase hors `service.ts`** dans components/hooks (grep) — la règle CLAUDE.md tient. Les hooks parlent à `/api/tmdb`, ce qui est le pattern prévu. ✅
- `select("*")` : 6 occurrences ; défendables (`getMediaItemById` = la fiche veut tout ; tables listes/highlights petites) sauf `searchMediaForList` (finding 8).
- Le schéma zod d'update accepte n'importe quelle combinaison de drapeaux isolés — c'est le terreau des findings 1-2. Un refinement zod (« watched:true ⇒ position présente pour une série ») serait le garde-fou le moins cher en attendant le funnel unique.

## Ce que je n'ai pas pu vérifier (et n'ai pas fait, lecture seule oblige)

- **Reproduire les symptômes** (cartes « New » fantômes, deltas d'heures Stats vs QuickStats) : déduits du code, non exécutés — aucune écriture n'a été faite.
- `series-state.test.ts` ligne par ligne : je n'ai donc pas vérifié si les 59 tests couvrent l'Undo (finding 3) ou la parité sync/client (finding 9). À vérifier : un test « position write ⇒ caught_up_at recomputé » n'existe probablement pas, puisque le bug existe.
- `AddMediaModal`/`HowFarDidYouGet`/`useAddMedia` en entier : sondés (airedFromTmdb, RESET_STATUS, stampage aux lignes clés) — un chemin de faux y est possible mais rien vu de suspect aux points d'écriture ; confiance moyenne.
- `computeStats.ts` en entier (492 l.) : le finding 4 est certain sur les lignes lues, l'ampleur exacte du delta ne l'est pas.
- Les durées de vie d'isolate Deno en prod (finding 5b) et le coût réseau réel de la tempête d'invalidations (finding 8) : non mesurés.
- `getListsWithThumbnails` : `select("*")` vu, l'éventuel N+1 sur les vignettes non inspecté.
