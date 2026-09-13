# Watching — Décisions

> **Le seul fichier de `hq/` qui ne périme jamais.** Un arbitrage pris le 20 juillet reste vrai le
> 20 juillet, même si tout le code a changé depuis. C'est ce qui le rend plus précieux qu'une
> description : on y revient pour savoir *pourquoi*, pas *comment*.

Une entrée par arbitrage, la plus récente en haut.

---

## 2026-09-13 — Renommer « Don't Miss » en « Trending » (spec V2 de mai, jamais fait) : on le fait ?

**Choisi :** non — le libellé « Don't Miss » reste. Ligne rayée de la spec V2.
**Pourquoi (owner) :** la section n'est pas une liste de tendances : c'est **un** titre qui porte le
badge Trending, suivi de pépites récentes. C'est une sélection. L'appeler « Trending » mentirait sur
son contenu — le nom actuel décrit ce qu'elle fait.
**Dans le même arbitrage :** Books ↔ Watching et Journal ↔ Watching (les deux autres lignes de la
V2 sans décision) sont **voulus**, mais bloqués par la condition 3 de l'amendement 6 (ils touchent
Books et Journal, non audités) → `backlog.md` avec ce déclencheur. La notification « nouvelle
saison / nouvel épisode » est **coupée en deux** : la détection + l'affichage côté Watching (panneau
saison, *Waiting for*) sont Watching-only → **v4** ; la tuile dashboard attend l'audit de dashboard-os.

## 2026-07-21 — SSR sur les 3 pages grille (Movies/Series/Animes) : on bascule ou on reste en CSR ?

**Choisi :** on reste en CSR (client-side, TanStack Query).
**Pourquoi :** gain mesuré = **~440 ms (27 %), et seulement sur la PREMIÈRE visite** — `/library` (SSR)
1165 ms / 4 requêtes vs `/movies` (CSR) 1607 ms / 14 requêtes. Les visites suivantes sont déjà
instantanées grâce au cache TanStack, donc le gain SSR ne joue que sur le tout premier chargement.
**Refusé :** le SSR, parce que le coût réel n'est pas dans le rendu mais dans la couche de lecture :
**aucune fonction de `service.ts` ne peut tourner côté serveur** — toutes utilisent le `createClient()`
de `infrastructure/supabase/client.ts` (module `'use client'`, pas de session cookie). Importée telle
quelle dans un Server Component, la RLS renvoie **zéro ligne sans erreur** → la page affiche « tu n'as
rien » avant que le client ne monte. Il aurait fallu une **2ᵉ implémentation des lectures**, exactement
la maladie que ce module a déjà eue trois fois (statut assemblé à la main, coordonnées lues brutes,
liste stockée au lieu d'être dérivée). 440 ms de première visite ne valent pas une duplication
permanente du layer de lecture.
**Ce qui la rouvrirait :** l'ouverture des inscriptions (multi-utilisateur, où le premier chargement
compte plus) ou une page qui redevient lente. Si on y revient un jour, **la bonne forme n'est pas un
fork** : injecter le client Supabase en paramètre (`getMediaItems(client, userId, …)`) — une seule
implémentation, deux appelants, vérifiable par le compilateur. `client.ts` et `server.ts` restent
INTOUCHABLES (CLAUDE.md §8) — on les *appelle*, on ne les modifie pas.

---

## 2026-07-20 — Re:Zero (et 3 autres animes) montrent 5 saisons, IMDb en dit 4 : on refusionne les cours ?

**Choisi :** non — on n'y touche pas.
**Pourquoi :** AniList segmente par **cour** (un bloc de diffusion) ; une saison diffusée en deux
blocs (« Season 2 » puis « Season 2 Part 2 ») donne 2 entrées AniList pour 1 seule saison IMDb. Notre
resolver les numérote séquentiellement sans les fusionner. **Rien n'est cassé** — position, années,
posters, stats restent justes — c'est un **libellé**, pas une donnée fausse. Ampleur mesurée :
**4 titres sur 56** résolus AniList (AoT 6→4, Re:Zero 5→4, Haikyu 5→4, Fire Force 4→3).
**Refusé :** re-segmenter les cours pour matcher IMDb à 100 %. Coût/risque disproportionné : les
numéros de cours sont les **clés** de `cour_years`/`cour_ratings` — re-segmenter impose une
remigration de ces clés, qui peut corrompre des années déjà posées, pour un problème d'étiquette sur
4 titres.
**Remède alternatif retenu (zéro migration) :** afficher le **vrai titre du cour** (déjà stocké côté
AniList) dans le popover du strip de saisons — le « 5 » cesse d'être trompeur, on voit que 2 blocs
forment 1 seule saison. **Pas encore implémenté** — à faire si ça regratte, sans rouvrir le débat de la
refusion.
**Ce qui la rouvrirait :** vouloir un alignement à 100 % avec IMDb un jour (pas une contrainte
aujourd'hui, à un seul utilisateur).

---

## 2026-07-20 — La lentille `MediaView` : les fonctions de stamp convertissent à quel moment ?

**Choisi :** **traduire tard** — les fonctions pures (`markWatchedPatch`, `positionPatch`,
`markCaughtUpPatch`, `addStatusPatch`) reçoivent des **unités VUE** (coordonnées cour) + le
`MediaView`, et convertissent **au bord**, via `view.toStorage()` / `view.writeYear()`. Le stockage
(coordonnées TMDB à plat) n'entre jamais dans une décision pure pour un titre overlay.
**Pourquoi :** l'ancien pattern (`handleCourSetPosition`) convertissait cour→plat **avant** d'appeler
`positionPatch` — la fonction raisonnait donc déjà à plat et ne voyait **jamais** une frontière de
cour, donc ne posait **jamais** de stamp. C'est la cause exacte du bug des années overlay du 20/07
(un « +1 » sur Blue Lock ne stampait rien).
**Refusé :** convertir tôt, au call site — c'est littéralement le pattern qui cassait le stamp.
**Ce qui la rouvrirait :** rien d'identifié — ce n'est pas un compromis coût/bénéfice, c'est la
correction du bug. Le seul risque resterait un futur call site qui réintroduit une pré-conversion ;
c'est ce que le garde ESLint `no-restricted-syntax` (passé en `error`) et le typage de `MediaView` en
paramètre rendent mécaniquement plus difficile à refaire.

---

## 2026-07-XX — Stats : `computeStats` doit-il passer par la lentille `MediaView` comme le reste de la fiche ?

**Choisi :** non — `components/stats/**` reste **exempté** du garde ESLint et continue de lire
`season_years`/`season_episodes` bruts (coordonnées TMDB), jamais `cour_years`/coordonnées cour.
**Pourquoi :** c'est du **calcul agrégé**, pas de l'affichage d'une position. Les totaux (épisodes vus,
heures) sont **invariants par espace** — compter en cours ou en saisons TMDB donne le même total — et
l'indexation par saison TMDB est voulue pour la cohérence historique des Stats (comparer une saison
d'une année sur l'autre a besoin d'un découpage stable).
**Refusé :** faire passer Stats par la lentille — ça changerait la **forme** de ses chiffres (leur
indexation par cour au lieu de saison TMDB), pas ne les corrigerait pas : ce n'est pas un bug à
fixer, c'est un choix d'indexation à ne pas casser.
**Ce qui la rouvrirait :** si Stats devait un jour ventiler par cour plutôt que par saison TMDB
(non demandé aujourd'hui).

---

## 2026-07 (page détail) — Ranking Top 10 : swap ou cascade quand on prend une place déjà occupée ?

**Choisi :** **swap** — prendre une place déjà tenue échange les deux titres, rien d'autre ne bouge.
Source : commentaire en tête de `src/modules/watching/components/detail/TopTenRank.tsx`.
**Pourquoi :** une cascade (« tout ce qui est en dessous descend d'un cran ») est le geste qui semble
le plus naturel — et c'est justement ce qui la rend dangereuse : elle **fait tomber silencieusement**
le dixième titre hors de la liste. Un swap ne déplace que les **deux titres visibles à l'écran**, et
refaire le même geste **annule exactement** ce qu'il vient de faire (idempotence visible).
**Refusé :** la cascade. Elle a été jugée plus intuitive au premier regard mais perd de l'information
sans le dire.
**Ce qui la rouvrirait :** rien — c'est une propriété structurelle (pas de perte silencieuse), pas un
compromis dépendant du contexte actuel.

---

## 2026-07-13 — La grammaire des marques : quelle couleur pour le rang / la note ?

**Choisi :** **la couleur dit la SOURCE**, jamais le rôle. Étoile **or** = la note **du monde**
(TMDB/IMDb) ; étoile **teal** (accent Watching) = **TA** note ; rouge = l'affection (favori), seul
écart volontaire au système car c'est un axe émotionnel, pas une source. Source :
`src/modules/watching/components/shared/Marks.tsx` + `reference_watching_marks_grammar` (mémoire,
2026-07-13).
**Pourquoi :** avant cette règle, chaque badge/étoile était décidé localement (6 implémentations du
même objet) et certaines **marques mentaient** — le carousel affichait TA note avec une étoile OR, la
couleur du monde, sur TA opinion.
**Refusé :** laisser la couleur suivre le contexte visuel (une étoile "qui va bien" avec la carte) au
lieu de la source de la donnée qu'elle représente.
**Ce qui la rouvrirait :** l'ajout d'une 3ᵉ source de note (ex. une plateforme externe supplémentaire)
demanderait de choisir sa couleur dans le même système, pas d'en inventer un nouveau.

---

## 2026-07-18 — Thèmes séries/films façon OP/ED anime (musique de générique jouable) : on le construit ?

**Choisi :** non — **recherché et abandonné**. ⚠️ **Ne pas reposer la question** sans fait nouveau.
**Pourquoi :** règle owner posée avant la recherche — *« soit la MÊME expérience que les OP/ED anime,
soit rien »*. Vérifié par exécution réelle, pas supposé : iTunes/Deezer/Spotify ne donnent qu'un
**extrait 30 s** (Deezer : 469 Ko de MP3 sur un titre de 78 s ; Spotify retire même le preview aux
nouvelles apps) ; Apple Music/Spotify complet exige l'**abonnement de l'utilisateur** + un SDK
propriétaire (MusicKit/Web Playback), donc pas universel ; YouTube donne le son complet mais en
**iframe vidéo** (pas un lecteur audio comme le ThemePlayer existant), avec clé/quota et un matching
peu fiable. AnimeThemes marche pour l'anime uniquement parce que c'est une archive communautaire qui
**héberge les fichiers** — rien d'équivalent n'existe légalement pour les B.O. commerciales.
**Refusé :** toute demi-mesure (extrait 30 s, lecteur différent pour films vs anime) — explicitement
écartée par la règle owner elle-même, donc il ne restait que « la même chose » (impossible) ou
« rien ».
**Ce qui la rouvrirait :** l'apparition d'une source légale qui héberge des B.O. commerciales
complètes en accès libre (aucune ne s'est présentée à date). Le sous-produit réutilisable si ça
arrive : la recette de matching compositeur TMDB (`job: "Original Music Composer"`) + recherche
iTunes ancrée sur son nom + filtre `artistName` — élimine les covers qui polluent le top des résultats,
testée OK sur Interstellar/Inception/GoT/Succession/Stranger Things.

---

## 2026-07 (constitution d'architecture) — Connectivité inter-modules : bus d'événements ou orchestration explicite ?

**Choisi :** orchestration explicite au bord de l'écriture (le module appelle une fonction exposée par
l'autre module, ex. `syncWatchingGoals`/`syncWatchingHabits` dans l'`onSuccess` d'une mutation) —
**pas de bus d'événements**.
**Pourquoi :** à l'échelle mono-utilisateur, un event bus paie de l'indirection et du debug
asynchrone pour un découplage dont le projet n'a pas besoin aujourd'hui.
**Refusé :** un bus d'événements générique pour toute réaction cross-module.
**Ce qui la rouvrirait :** si 3+ modules réagissaient un jour à un même fait de façon non triviale
(aujourd'hui seuls Goals et Habits réagissent à Watching, et de façon simple).
