# Watching — Décisions

> **Le seul fichier de `hq/` qui ne périme jamais.** Un arbitrage pris le 20 juillet reste vrai le
> 20 juillet, même si tout le code a changé depuis. C'est ce qui le rend plus précieux qu'une
> description : on y revient pour savoir *pourquoi*, pas *comment*.

Une entrée par arbitrage, la plus récente en haut.

---

## 2026-09-15 (soir) — Le Musée, deuxième passe sur rendu : cinq arbitrages owner + la source Emmy

**1. Portrait pour les catégories à lauréat unique** (acting, direction — 16 catégories, colonne
`award_categories.portrait`). Le prix est celui de la personne ; la fiche person existe. Visage,
nom, le film en ligne méta (lien séparé vers la fiche), **l'état reste celui du film** (c'est ce
que la bibliothèque connaît). Pas pour scénario/son/musique (2-4 noms → un portrait mentirait).
**Refusé (moi, dans le brief) :** « pas de visages » — mon argument valait pour les notes sous un
visage, pas pour le visage.

**2. Le tag d'année est horizontal, accroché au bord gauche, et porte l'état.** Le ruban vertical
« ne se lisait pas » à la taille d'une affiche. Tokens `--color-award` (or métallique, **vu**) et
`--color-award-dim` (gris foncé, **pas vu**) dans `globals.css` — pas l'ambre (`--color-gold`
= la priorité *medium* du watchlist). Centré sur la rangée du cluster (top-2, h-6) → aligné avec
le cœur et le « … » par construction. **Refusé :** masque noir & blanc sur les affiches (construit,
vu, retiré : « tous les posters identiques, l'info sur le badge ») ; masque sur les cartes de la page
Awards (la carte reste en couleur, le masque est l'affaire de la page catégorie).

**3. Sous l'affiche : titre, puis le mot de la bibliothèque** dans le vocabulaire de la vue liste
de `ListDetail` (point + Watched / In Progress / Want to Watch / Paused / Dropped / Unwatched,
`canonStatus`), ta note à droite, rangée à hauteur fixe (16 px). Filtre All · Watched · Want ·
Unwatched à côté de Winners · Nominees. **Caught up = vu** (`isSeen` : watched OU
`seriesState === "caught-up"`) — Widow's Bay a une saison complète, la performance primée est vue.

**4. L'étagère ouvre la page** : plus d'en-tête Museum/Awards ; Oscars | Emmys dans l'en-tête du
rail (`MediaCarousel.actions`), See all → `TrophyShelfPanel`. L'année de sortie disparaît de la
ligne (un « 2025 » sous un tag « 2026 » — cérémonie ≠ sortie — se lisait comme une contradiction).
**La cérémonie vit dans l'URL** (`?c=emmys`) : le Back du navigateur revient aux Emmys.

**5. Les Emmys viennent d'emmys.com, pas de Wikidata.** Mesuré : Wikidata a 98/98 Best Picture
mais 19 Comedy Series sur 70 ans et zéro award pour Ted Lasso — jamais saisi, trois chemins de
requête testés. emmys.com publie sur chaque page de catégorie un JSON-LD schema.org `ItemList`
(séries, personnes, lauréat/nommé, depuis 1949 ; robots.txt permissif, pas de clause anti-
extraction trouvée) — mais **aucun id**. D'où une RÉSOLUTION qui dit sa confiance (`match`) :
TMDB search + fenêtre d'année → `name` ; Wikipedia → Wikidata → id TMDB, qui prime → `id` ;
rien → `none`, ligne gardée sans lien (`work_tmdb_id` nullable). Exception argumentée à « jamais
par titre » : il n'y a pas d'id, la source est officielle et complète. Deux pièges attrapés à sec :
les homonymes Wikipedia (Scoop 2006 vs 2024, Mr. & Mrs. Smith 1996 vs 2024 → l'année entre
parenthèses la plus récente sous l'année de cérémonie) et les suffixes d'anthologie (« Anna
Karenina Masterpiece Theatre »). Sondé : 2024 = 32/32 résolus, 1978 = 20/20.
**Refusé :** IMDb (pas d'awards dans les datasets, scraping interdit), Kaggle (2019, par titre),
Wikipedia-tables (complet et jointable par id, plan B honnête, mais emmys.com est officiel et déjà
structuré).

## 2026-09-16 — Le Musée : la saison primée, « This year », les crons en saison, la vue par année

**1. La saison primée (Emmys).** Ce que le canon dit, c'est « Succession **S4** a gagné en 2024 ».
emmys.com ne la donne pas ; la règle d'éligibilité (1ᵉʳ juin → 31 mai) + les saisons TMDB la
déduisent (`season_number`, `season_poster_path`). À l'écran : l'affiche de la saison et la puce
« S4 » de la bande Seasons, jamais sur un portrait. Limite : deux saisons dans la fenêtre → la
plus récente.

**2. « This year » = la page cérémonie en mode « avant ».** Une seule page
(`/awards/ceremony/{c}/{year}`), deux moments : avant la cérémonie, les nommés par catégorie, le
compte à rebours, ce que tu as vu ; après, le lauréat en tête avec le tag. La bannière en tête de la
page Awards **est pilotée par la donnée** — elle existe dès que des nommés existent pour une
cérémonie à date future, disparaît le lendemain — pas par une date de nominations à saisir. Les
dates viennent de Wikidata (`award_ceremonies`, P179 « part of the series », **pas** le P31 qui est
une classe générique de 400 prix). Un Emmy tenu en janvier (grève 2024) = l'année précédente.
**Le tag d'année est la porte de cette page.** Validé sur simulation (date poussée en base, remise).
**Refusé :** regrouper les victoires consécutives d'une série en une tuile — construit, vu, refusé :
« cinq affiches identiques montrent la domination au premier regard, c'est une publicité pour la
série ».

**3. Crons en saison, 2×/jour** (03:25/15:25 Oscars janvier→mars, 03:55/15:55 Emmys
juillet→septembre), mensuels hors saison — exigence « dans HEGON sous 12 h ». Créneaux vérifiés
contre les 17 crons : les premiers (04:55/05:55) tombaient dans la traîne de for-you (04:50).

**4. Résolution par nom, les règles apprises sur la donnée réelle :** titre exact, puis **le plus
proche de l'année de cérémonie**, puis popularité (la popularité prenait l'homonyme célèbre : la
série *Sherlock Holmes* de 2013 pour *Sherlock*, la série de 1967 pour le film de 1979) ; l'autre
type TMDB quand le type attendu n'offre qu'une autre époque ; parapluies (35 noms), « Vol. N »,
studio devant, possessif d'auteur en dernier recours seulement (« Schitt's Creek » est un titre) ;
un épisode primé seul → sa série, les deux côtés du « : », Wikidata tranche ; page Wikipedia = titre
exact sans qualificatif. Une table d'alias pour les retitrages US (« Before the Dinosaurs » =
*Walking with Monsters*). Une source `manual` pour les trous de la source (Ngor 1985).

**📌 Dashboard (owner, 16/09) :** ce genre d'information — « les Oscars sont dans 23 jours, il te
reste 38 nommés à voir » — doit aussi remonter **sur le dashboard comme une notification**, au même
titre que « nouvelle saison disponible » (backlog §11 / tableau Watching → Dashboard). Même
condition : la moitié Watching existe (la bannière), la moitié dashboard attend l'audit dashboard-os.

## 2026-09-15 — Le Musée / Awards : le design figé (owner), après deux maquettes et un brief

**Chemin :** ma première vision (frise d'années scrubbable, héros, anneaux teal) → image Gemini →
jugée *almanach* ; brief neutre (`awards-brief.md`) donné à une IA tierce → sa maquette (compteurs →
étagère → catégories) → challengée ; fusion validée par l'owner avec **un** changement de son cru.

**Choisi — la page `/perso/watching/awards`, de haut en bas :**
1. **En-tête** : sur-titre « Museum », « Awards », « Your collection meets the canon », segmented
   **Oscars | Emmys**. **Pas de héros**, pas de statuette (stock + marque déposée de l'Academy).
2. **Your trophy shelf** — rail (`MediaCarousel`) de TES titres lauréats, ruban **or** portant
   **l'année** (le ruban Top 10 dans sa lecture « un chiffre », en or = le monde), ta note, sous
   l'affiche la catégorie la plus prestigieuse (+N si plusieurs). « See all ».
3. **Categories** — **une grille de cartes sur le modèle des Lists** (owner) : mosaïque des 3
   derniers lauréats (les tiens en couleur, les autres atténués), flag or, « 12 / 96 watched » +
   barre teal, clic → **page catégorie = la grille de `ListDetail`** (ruban année, tes notes, « + »
   sur ce que tu n'as pas). *Pourquoi la carte des Lists :* même forme = même comportement — une
   catégorie EST une liste, écrite par le monde (or) et non par toi (teal). 23 cartes = un écran ;
   23 rails = une page interminable.
4. **L'année est une porte, pas une colonne** : cliquer un ruban « 2025 » → la cérémonie (lauréat
   large + nommés, accordéon Don't Miss). Les nommés n'ont de sens QUE là.
**Refusé :** la rangée de compteurs sous l'en-tête (les cartes disent déjà « 12 / 96 » ; les
compteurs restent des achievements dans Stats) · onglets Overview/Oscars/Emmys (un filtre déguisé)
· tuiles de catégorie en photos de stock (dans Watching la seule image est l'artwork) · visages pour
les catégories « personne » (**l'objet est toujours le film**, la personne est une ligne méta) · ruban
« WINNER » sur une étagère qui ne contient que des lauréats.
**Données :** Wikidata → `watching.awards` (référence globale, sans `org_id`), join TMDB par
`P4947`/`P4983`/`P4985`, ingestion par edge function + cron annuel, jamais de SPARQL depuis la page.
**Ordre de construction :** donnée d'abord (l'UI ne se juge que sur 96 vraies affiches), puis page,
puis carte/grille catégorie, puis cérémonie, puis Accolades (fiche) et timeline (person).

## 2026-09-14 — Le panneau saison : trois arbitrages pris sur le rendu (owner), qui dévient de la vision du 12/09

**1. La note de saison = dix étoiles à demi-pas (`StarRating`), PAS le `RatingPicker`.**
**Choisi :** un second contrôle /10 dans le module, réservé au rang « saison ».
**Pourquoi (owner, sur rendu) :** la barre avec ses mots (*Skip it → Masterpiece*) est une cérémonie
— juste pour LE verdict d'un titre, donnée une fois dans My Take. Répétée par saison dans un panneau,
elle pèse. Les étoiles disent le chiffre d'un coup d'œil et le prennent en un tap. **Une seule
échelle** (/10 par 0,5), **deux présentations selon le rang** ; un seul écrivain (`view.writeRating`),
inchangé. Teal, jamais or (grammaire des marques : l'or est celui du monde et du best-ep).
**Refusé :** le `RatingPicker` dans le panneau (vision du 12/09 « un seul contrôle /10 ») — vu à l'écran,
jugé trop lourd ; le Select de l'ancien popover.

**2. Pas d'auto-scroll vers la position à l'ouverture.**
**Choisi :** le panneau s'ouvre en haut (carte de saison visible) ; « Up next » + l'atténuation des
épisodes devant toi disent où tu es. **Refusé :** la vision du 12/09 « saison en cours = l'épisode de la
position visible à l'ouverture » — construit, vu, retiré par l'owner.

**3. Une saison = une note.** Un titre à UNE saison d'affichage n'a pas de note de saison : la note de
My Take EST celle de la saison (`ScoreMark` lecture seule dans le panneau, `user_rating` sur la carte
de la bande ; `season_ratings["1"]` n'est plus écrit). Vérifié : 0 titre à une saison ne porte de
`season_ratings`/`cour_ratings` en base. **Conséquence assumée :** quand une telle série gagne une
2ᵉ saison, la carte S1 perd sa note jusqu'à ce que l'owner la note explicitement — c'est un nouveau
jugement (le titre n'est plus sa saison), pas une perte.

**Dans le même arbitrage (forme, owner) :** la carte de saison est construite **comme la carte
Trending** (poster à fleur du bord gauche, 112 px fixes, son artwork flouté `w92` saturé en halo) ;
pas de titre de cour (une seule forme de carte pour tous les panneaux) ; les corrections (« I stopped
after this season ») vivent derrière un « … » en haut à droite ; *Next episode* = bandeau deux lignes
sous la carte, jamais une carte dans la liste ; « …more » sur la 2ᵉ ligne du synopsis, mesuré, pas
deviné ; vignettes `w500` (panneau ET rail Best Episodes) ; le rail n'existe pas sans best ep.

**Ce qui la rouvrirait :** un 2ᵉ usage d'un contrôle de note hors My Take (→ le `StarRating`
deviendrait le contrôle « secondaire » partout, ou on retombe à un seul).

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

## 2026-10-11 ? Watching Wave 1 locked contracts

Chosen by the owner: annual series/anime scores use the arithmetic mean of seasonal/cour ratings attributed to the selected year, even for a single-year title; without eligible ratings the annual score is null. All-time title-rating behavior remains. Activity does not date a rating. Known aired zero stays zero; missing airing data may support explicitly estimated duration, never proof of airing. Identity is `movie:id` or `tv:id`; anime and series share TV identity. Directing excludes creators/production; TV creators retain their own classification. Legacy untyped TV crew is unverified and excluded from directing, rather than guessed.

Seen With counts each collaborator once per experienced provider work, including engaged viewing. Your Person includes engaged works, separates completion, excludes future/unexperienced ratings, and uses released filmography wording. Top Picks retains rating-first/favorite-tie ordering, six visible items, and the existing SlidingPanel for all favorite-or-rated-at-least-8 qualifiers. Goals complete on actual count >= target, never rounded percentage. Provider count regression must preserve the previous coherent snapshot and personal posture; restoring previously dated content is not genuine growth.

Rejected for this wave: event history/schema redesign, partial date precision storage, scoped rewatches, permanent achievements, broad provider versions, recommendation lab, Python/Data work, UI redesign and an invented completed-goal reopening policy. Reopening trigger: explicit Wave 2 authorization with historical facts and migration policy designed together. Provider corrections cannot be resolved safely from counts alone; already-degraded rows require verified evidence.

## 2026-10-12 ? Wave 2A historical foundation

Owner approved only the historical foundation/date precision/atomic capture slice. Chosen: two historical tables, a transactional personal command, server revisions, retained corrections and library-independent historical identity. Version 1 explicitly means partial prospective history. Unknown rewatch scope and cour coverage without frozen evidence stay unknown; input dates never inherit precision from legacy timestamp shapes. Current Stats/projections remain compatible. Details: [wave2a-history.md](wave2a-history.md).

Rejected for 2A: full event sourcing, provider provenance, scoped rating lifecycle, scoped rewatch UI, achievement/Goal receipts, analytics cutover and historical reconstruction. Reopening trigger: explicit approval for the corresponding Wave 2B?2E slice. No production rollout was authorized or performed.

---

## 2026-10-12 — Poster consumption status colours

**Chosen (owner request):** shared `StatusBadge.tsx` poster tone map uses muted violet
`#a78bda` for Watching / In Progress and calmer indigo `#9295cc` for Caught up.
Top 10 stays teal; Favorite red, Paused blue, Dropped yellow. Icons, badge structure,
status derivation and all business logic remain unchanged. LibraryCard and MediaCarousel
share the change; no global accent change, database change or Wave 2B work.

**Rejected:** keeping consumption statuses in the Top 10 teal family, neon purple,
or introducing Library-only colour overrides. Reopen only on an owner palette request.

**Validation:** TypeScript, affected-file lint and diff whitespace passed; design-system
coverage regenerated. Live Library and hover captures reviewed, including Favorite + status.
Glyph contrast is 6.28:1 / 6.32:1; fills against surface-2 are 5.81:1 / 5.85:1.
Poster artwork failed to load and a workspace error toast was present; artwork contrast
could not be verified. No live Top 10 + status item in the loaded page; composition checked
with a browser-only fixture using the existing ribbon dimensions and colour.

---

## 2026-10-12 — Top 10/status composition and dedicated orange family

**Chosen (owner request, supersedes violet consumption palette above):** Watching uses
`#dc956c` warm orange; Caught up uses `#c48768` calmer burnt orange in the shared poster
tone map. Waiting For keeps `#c4b5fd`, Dropped `#fcd34d`, Paused `#7dd3fc`,
Favorite `#f43f5e`, Top 10 the unchanged Watching vivid teal token (`#2dd4bf`).

Library status sits 6 px right of the 26 px Top 10 ribbon; shared carousel status clears
its 22 px ribbon by the same gap. Paired discs are 20 px, otherwise unchanged at 24 px.
Favorite suppression under Top 10, icons, actions and mobile carousel placement are preserved.
**Rejected:** stacked/overlapping marks and violet consumption statuses (reserved for future
watchlist content). Reopen only on an owner palette/composition request.

**Validation:** real LibraryCard render fixtures cover all requested combinations plus
Top 10 + Favorite + Paused; real carousel card fixture checks Top 10 + Caught up.
Playwright checks 6 px spacing, 20 px discs, Favorite suppression, violet Waiting For and
top-right hover action. Fixture screenshot read; artwork replaced with a neutral test surface.
TypeScript, affected-file lint, ds:coverage and diff --check passed. No business logic,
status semantics, schema or Wave 2B changes. Existing live artwork/workspace loading errors
remain outside this UI-only polish.

---

## 2026-10-13 — Full-text consumption badges on posters

**Chosen (owner request, supersedes the circular status presentation):** shared StatusMark
uses the design-system solid Badge, rounded-chip radius, text-micro typography, a bare
icon and full English label: Caught Up, In Progress, Paused, Dropped. Library badges sit
bottom-left; carousel Caught Up sits in the bottom-left content stack above the title.
Text may wrap on narrow posters rather than truncate. Orange is softened to `#d7a17d`
(In Progress) / `#c49577` (Caught Up); Paused blue and Dropped yellow are unchanged.
Watchlist bookmarks, Top 10 styling/ranking, Favorite suppression, actions and state
derivation remain unchanged. No schema or Wave 2B work.

**Rejected:** circular icon discs, abbreviated labels and violet consumption badges.
Reopen only on an owner poster-presentation request.

**Validation:** TypeScript and affected-file lint passed; design-system coverage regenerated.
Playwright rendered-component fixtures check all four labels, bare icons, bottom-left
placement, text containment and Top 10/Favorite suppression across four viewport widths;
carousel renders one Caught Up badge. Screenshots reviewed on a neutral fixture surface,
not live artwork. git diff --check passed.

---

## 2026-10-13 — Smaller status badges, clearer orange

**Chosen (owner visual feedback):** replace beige orange with warm orange `#f0a166`
(In Progress) and burnt orange `#da8d55` (Caught Up). Shared solid badge uses size sm,
20 px minimum height, text-caption (10 px) with normal tracking and 2 px vertical padding.
Full labels, icons, other colours, placement and all business logic remain unchanged.
**Rejected:** previous beige palette and 24 px minimum height. Reopen on owner feedback.
**Validation:** four-status rendered fixtures pass on four widths; desktop screenshot read.
TypeScript, targeted lint, coverage regeneration and diff whitespace checks run.


## 2026-10-13 ? Consumption actions require available content

Chosen (targeted owner request before Wave 2B): `lib/watchable.ts:isWatchableNow` is the shared availability gate for discover, detail, poster/list menus, quick-add and action handlers. Movies require a past/present release date. TV/anime use actual aired counts: known zero (including an empty known array) blocks consumption; positive counts permit it. Unknown counts fall back to premiere/season air dates. Without dates, an earlier release year or existing personal consumption supports incomplete legacy rows; lifecycle labels and announcements alone do not. Explicit future/invalid dates and known zero override those fallbacks.

Want to Watch remains available; Waiting For and Library status derivation are unchanged. Correction/Undo retains historical correction semantics. Add actions reject consumption claims before writes. No schema, migration, provider provenance or Wave 2B work.

Validation: 118 targeted domain/action/rendered-JSX tests pass, including upcoming media outside the library and released-media compatibility. TypeScript, affected-file lint and diff whitespace checks recorded in the current workstream status. Production personal write tests are excluded.

**Owner refinement (2026-10-13):** orange slightly brighter: In Progress `#ffb477`,
Caught Up `#eea064`. Further compact badge: 16 px minimum height, 4 px horizontal
padding, 2 px gap, icons 10/11 px. Full text and semantics unchanged.
Rejected: keeping the previous size/brightness; reopen on owner visual feedback.

## 2026-10-13: Wave 2B evidence and rating lifecycle, ready for local review

Chosen by the owner-approved2B brief: compact meaningful provider snapshots, separate accepted/latest heads, conservative identity/date growth classification, append-only scoped assertions with frozen membership and explicit reconfirmation. Legacy ratings retain unknown coverage. Cour identity is stable AniList ID; cache mismatch blocks edits and offers query refresh. An ordinal remap cannot seed a known old opinion into another subject, or discard an overwritten unbound legacy opinion.

Rejected for2B: count/date-max-only reopening, progress-based auto-confirmation, global/single-season aliasing, payload archival, reconciliation/backfill, event sourcing and analytics cutover. Reopen broader scope only on explicit2C+ authorization. All local contracts pass; final build rerun is blocked by Google Fonts network. No remote migration, deployment, production personal mutation or Git write. Evidence and separate dirty-file manifest: [wave2b-provider-ratings.md](wave2b-provider-ratings.md).

## 2026-10-18: Wave 2C historical viewing and analytics

Owner-authorized2C reuses2A records/ranges and2B rating lineages. History is selected per event/coverage, never via a lifetime-complete interpretation of history_version1. Explicit untracked legacy facts remain where not shadowed; unknown precision/scope/duration remains unknown. Canonical rewatch IDs shadow legacy projections. New scope is explicit, IDs/ranges/duration frozen; movie runtime and cached actual episode runtimes can be retained, TV medians are not invented exact evidence. Nullable duration totals retain known subtotals and visible uncertainty.

Rejected: reconstructing legacy dates from timestamp shapes, whole-title legacy rewatch inference, provider-current duration recomputation, rating confirmations as extra votes, current scope reassigning historical cour coverage, full event sourcing and schema/table proliferation. A local function-only migration is necessary for UNKNOWN-date rewatches and guarded snapshot context; remote application requires explicit approval. No production writes/deployment/Git commit. Reopen receipts/persistence only in explicit2D, reconciliation/backfill only in explicit2E. Implementation and evidence: [wave2c-history-consumption.md](wave2c-history-consumption.md).


## 2026-10-20: Wave2C targeted blocker corrections

Owner requires exact overlap subtraction, not a chronological frontier: first-watch ranges mask only their storage intervals, including intentional retraction tombstones; unrelated legacy estimates remain. Completion is fallback consumption only for frozen membership absent from first-watch/usable legacy/other completions; milestone duration stays unknown. Ambiguous completion retains uncertainty rather than inferred episodes. Cour conversion requires accepted season snapshot evidence with complete selected membership; aired counts alone never establish offsets. Missing/incomplete evidence blocks before RPC with an explicit refresh reason. Rejected: suffix cutoffs, globally ignoring/counting all completions, guessed cour offsets or broad reconstruction. Reopen reconciliation only with2E authorization; no SQL change or remote application in this block. Evidence: [wave2c-history-consumption.md](wave2c-history-consumption.md).

## 2026-10-21: Wave2D permanent receipts, independent live progress

Owner explicitly authorized the minimal Goals integration despite its general module freeze. Chosen:10 consumption milestones get append-once org/user/key receipts; Connoisseur/Hall of Fame remain current-opinion states. Eligibility uses fresh server-computed canonical proof, not client cache, legacy estimates, inferred dates or mutable episode-count projections. Reached-now receipts say FIRST_OBSERVED_SATISFIED, never invented historical unlock dates. Goals owns completion/date/receipt and independently displays current progress; ordinary regression never revokes completion. SQL owner/org isolation, service-only writers, source/definition validation and unique receipts enforce retries/concurrency. Explicitly retained legacy completion is not guessed into a new receipt.

Rejected: client-authorized metrics, automatic revocation, bulk reconstruction, a new rules/Goals engine or broad admin UI. Invalid-original-proof correction requires a separately reviewed controlled migration preserving audit; ordinary recomputation cannot delete/revoke. Reopen legacy reconciliation only on2E approval. Details and evidence: [wave2d-milestone-receipts.md](wave2d-milestone-receipts.md).

## 2026-10-21: accepted legacy dates and one canonical Marathoner metric

Owner explicitly accepts ALL stored legacy watch dates as usable calendar data, including approximate pre-2024 month/day; no cutoff, suppression, date correction or backfill. This supersedes prior rejection of legacy timestamp-shaped calendar attribution. Explicit partial components and per-season/cour year stamps remain authoritative where available; unrelated coverage survives partial history adoption.

Marathoner uses the SAME canonical mixed total-hours ledger as Stats, recomputed on the authenticated server's fresh fingerprinted snapshot. Accepted legacy runtime estimates count; unknown duration is not invented. Prospective receipt requires raw actual>=3000 and trusted observation time, never browser cache or rounding. Other milestone/Goal qualification remains unchanged. Rejected: Stats accepting hours while Marathoner silently discards them, guessed historical unlock dates or UI redesign around the temporary missing remote2D schema. Reopen only on explicit owner policy change. Evidence: wave2d-milestone-receipts.md targeted-corrections section.

## 2026-10-21: complete calendar axis and faithful rating distribution

Owner's2021/2022/2023/2026 reports reproduced. Stored title calendar facts survive conflicting annual scope-year stamps and later partial viewing_date adoption; this does not move annual hours/episode counts or invent monthly duration where attribution is unknown. Rejected: erasing valid stored calendar activity, shifting years to reconcile contradictory data, masking by UNKNOWN scope/year alone or fabricating zero-month consumption.12 UI month positions represent empty buckets as zero; a truly undated year retains the no-month-data state. Rating distribution now counts exact scores rather than rounded integers and shares numeric positions for points/ticks, with non-overshooting controls. Rejected: arbitrary axis padding beyond real extrema and a visual redesign. Reopen calendar reconciliation only on explicit owner authorization; no2E or DB correction here. Evidence: wave2d-milestone-receipts.md follow-up section.

## 2026-10-21: close final2D date/identity/organization findings

Owner authorized targeted fixes. Historical date selection follows existing scoped assertions instead of treating an equal year as proof that a projection belongs to the unrelated legacy remainder. Stable work identity survives retained local stubs and same-identity return; new UUIDs are not inferred duplicates. Stats and receipt metrics both use the selected organization, with owner filters, explicit organization-bound reads/cache and captured query closures. Rejected: global cross-org progress against org-specific receipts, dynamic-org reads stored under an older cache key, and losing local historical consumption when a library row is removed.235 tests and targeted re-review pass. No new schema, production operation or2E. Reopen only on a concrete failing case or explicit product-scope change.

## 2026-10-21: estimated legacy date proposal explicitly requested

Owner has no exports/backups and explicitly accepts an offline plausible-date proposal. This supersedes the prior refusal of arbitrary redistribution only for preparing a clearly labelled synthetic mapping:85 estimates across89placeholder candidates,4review cases,202other rows unchanged. Public release/airing facts supply bounds only; all proposed personal dates remain estimates without personal evidence. No production write or confirmed historical assertion authorized. Existing app/date/receipt rules remain unchanged. Rejected: calling these recovered true dates, shifting known years to hide contradictions, or applying without approval of the precise mapping. Reopen implementation only after owner reviews that mapping and estimated provenance.


## 2026-10-21 ? Rewatch scope is a frozen viewing event

Owner-approved targeted correction: full title by default; optional multiple DISPLAY seasons, translated through the existing anime lens. No episode-grid UX. Available membership is bounded by the selected day/period (end of month/year capped at today), retaining original watch precision. Episode IDs/ranges and accepted per-episode runtime are frozen; no later robot growth/remapping/runtime can enlarge an event. Representative TV runtime and browser-cache availability do not supply new canonical hours. Missing facts remain unknown; legacy rewatches require explicit scope confirmation, with atomic replacement and no automatic full-show assumption. Reopened only for an explicit future product request for episode-level rewatch tracking. Implementation/rollout evidence: rewatch-hardening-2026-10-21.md. Black Clover poster correction remains outside this task.


## 2026-10-22: accepted consumption is the achievement ruler

Owner explicitly approved implementation after the business audit. Achievement progress, explanations and server eligibility use the same accepted Stats ledger, including usable legacy viewing; Ever-Present uses Activity months including rewatches, Long Hauler uses finished frozen membership/accepted legacy completion. This supersedes the earlier strict-history-only achievement eligibility policy, not Goals' explicit-history completion contract. Permanent receipts remain immutable, first observed now, never backdated. Legacy first-watch coverage/runtime is captured once as an estimate, not promoted to confirmed assertions; canonical overlap replacement and frozen rewatch scopes retain precedence. Annual season artwork follows actual display-space contributions, not a minimum number of dated years. Rejected: implicit full-show legacy rewatches, future catalogue growth changing old hours, fabricated unlock dates, copying all global opinions into S1. Reopen only for explicit product policy or verifiable correction of an accepted estimate. Implementation, manifest and owner rollout: accepted-consumption-2026-10-22.md.


## 2026-10-28: completion date follows accepted terminal viewing

Owner explicitly authorized fixing the Bleach business bug and personal data. A deliberate new full-title completion may inherit the confirmed viewing date of its terminal aired episode; recorded_at, rewatches and inferred/imported dates are not substitutes. Explicit date input wins, undated retrospective corrections remain unknown, and already-dated canonical completion survives unrelated earlier-season edits. Old aggregate season years are separate historical facts: do not move all of S2 to the final-completion year. Before a compatibility watched_at update, the guarded command retains the prior legacy calendar value once (including NULL), preventing monthly activity from moving with the finish projection. No new schema objects or alternate status writer. Rejected: copying release years as personal watch dates, overwriting prior-season years, silently changing all multipart anime mappings, or backfilling other titles. Reopen only for explicit history correction or separately scoped multipart mapping work. Concrete owner-only repair/backup/proof: bleach-completion-diagnostic-2026-10-28.md, implementation section.


## 2026-10-28: viewing periods and nested provider parts before final 2D review

Owner explicitly reopened and authorized the multipart work with point 1. Hours follow recorded first-view periods, independent of finish and release dates; accepted legacy estimates remain where precision is absent. Multi-season anime keep TMDB seasons and storage coordinates, exposing verified AniList parts nested inside those seasons. No airing-gap heuristic, no mass recut or invented historical dates. Old opinions retain frozen membership and move behind history when their scope grows; unchanged completed parts remain independently rateable through the existing RPC. Qualified part votes supersede overlapping season aggregate votes, preventing duplicate annual scoring; global title opinion remains separate. New-year viewing and later growth cannot move old hours or enlarge rewatches. Incomplete provider membership/boundaries preserve accepted state. This supersedes the earlier exclusion of multipart mapping only for this approved nested representation; ordinary series without verified grouping fall back to episodes. No final2D review/2E started. Manifest, real Bleach replay, local SQL and UI checks: season-periods-and-parts-2026-10-28.md.


## 2026-10-28: unknown rating coverage is not an obsolete opinion

Owner explicitly approved the UI regression correction after discussion. Restore saved legacy season/cour ratings on posters and panels even without frozen membership. Display and write eligibility share identity checks but have distinct evidence requirements: readable does not mean verified or editable against an unknown catalogue. Keep known-remapping guards and reserve Previous rating for demonstrated growth. Replace duplicated technical warnings with one collapsed plain-language disclosure. Poster geometry is fixed2:3, independent of personal controls/history, with no content-driven artwork height. No data rewrite or rating qualification change. This supersedes the previous CURRENT-only score visibility rule. Reopen on a concrete identity/growth regression or owner design decision. Proof: season-panel-polish-2026-10-28.md.


## 2026-10-28: annual Top Picks require first-view consumption

Owner explicitly approved excluding rewatch-only titles from annual Top Picks. Eligibility comes from the existing accepted first-view ledger in the selected year, independently of favorite status or a high global score. Rewatches retain hours/activity/counts and all-time eligibility; a first viewing plus rewatch in the same year still qualifies once under the existing score/favorite rules. Unknown first-view dates cannot inherit a dated rewatch year. Rejected: moving first-view dates, clearing favorites/ratings, removing rewatches from Activity or maintaining a separate ranking source. Reopen only for explicit product policy. Evidence: top-picks-rewatch-fix-2026-10-28.md.


## 2026-10-28: Top Picks identifies scored subjects, not seasons touched

Owner explicitly chooses Hannibal S1–S2 without unrated S3 progress. Annual label/poster must consume the same selected vote scopes as its numeric score; S3 identifies one season, S1–S2 a range, S1, S3 separate seasons. A nested part label requires an independent part vote. Actual Bleach9 covers all50 S2 episodes, so display S2, not Part4. The owner separately confirms Part4 first viewing occurred in2026: add only missing E41–48 with YEAR precision, preserve existing E49/E50 days, exact frozen legacy duration and all ratings. Rejected: activity-derived rated labels, copying the S2 score into Part4, inventing months/days, overwriting season stamps or redistributing other titles. Controlled data-only owner migration20261028010000 prepared; later read-only evidence confirms it is already remotely applied (earlier not-applied assumption corrected2026-10-29). Proof: rated-scope-and-bleach-part4-2026-10-28.md.


## 2026-10-28: owner defers Bleach S2 part-year reconciliation

Owner now requests leaving current Bleach data unchanged until a later discussion of each part's personal viewing period. This supersedes the prepared Part4 year correction approval. At that time20261028010000 was mistakenly considered unapplied; fresh remote evidence later disproved this, and the exact migration file was restored2026-10-29. Its dedicated SQL rehearsal was archived under ignored scripts/.shots, outside the deployable migrations/test manifest; schema watermark restored to applied20261028000000. No remote rollback/data write. Previously applied completion-date repair and generic rated-scope UI remain. Reopen only after owner supplies/approves the part-by-part mapping; not an automatic2E backfill or a blocker to committing the validated2D code.


## 2026-10-28: Wave2E initial read-only audit approved

Owner approves controlled reconciliation framing and explicitly requires PROVEN/UNKNOWN/INFERRED classification, before/evidence/after review, and a stop before corrections. Unknown facts remain unknown; inferred dates stay inferred. Bleach part-year mapping remains excluded. Fresh audit discovers remote20261028010000 is already applied despite earlier documentation; report this evidence without reversing personal data or repairing history. No correction is authorized by this audit. Reopen implementation only after owner reviews concrete findings and approves the relevant correction block. Evidence: wave2e-reconciliation-audit-2026-10-28.md.


## 2026-10-29: restore already-applied migration without changing history

Owner explicitly authorizes exact repository restoration and local replay verification only. Restored original20261028010000 matches archived bytes and all remote stored statements;31versions align and watchdog watermark advances with unchanged object counts. No production mutation or migration repair. Literal fresh replay is not proven: native PostgreSQL lacks pg_cron and three existing data repairs reject missing personal prerequisites. Retain exact historical SQL; no fake records, empty replacement, silent skip or guard weakening. Discuss a reviewed fresh-bootstrap strategy before declaring full replayability. Evidence: migration-alignment-2026-10-29.md.


## 2026-10-29: empty-database reconstruction is separate from2E

Owner accepts migration alignment as resolved and explicitly defers full empty-database reconstruction as separate technical debt, nonblocking for2E. Do not modify already-applied historical migrations to make a clean bootstrap pass. Reopen only on an explicit dedicated reconstruction/bootstrap task.2E resumes read-only examination of Tokyo Ghoul, The Seven Deadly Sins and Fire Force; personal viewing assertions and all corrections require owner confirmation. Scope/year discrepancies are proven stored differences, not proof that the cursor or year map is the erroneous fact. See wave2e-three-anime-clarification-2026-10-29.md.


## 2026-10-29: generic backwards correction retires invalid active assertions

Owner authorizes generic implementation before any three-title production repair. Explicit first-view backwards corrections invalidate only contradicted identified scope, independently of Paused/Dropped. Frozen membership wins over ordinals; kept/unknown subjects and independent rewatches survive. The existing transactional personal command owns clipping/retraction, legacy map/membership/calendar cleanup, append-only score clears and immutable before/after audit. Removed the duplicate TypeScript backwards planner. No invented watch dates, blanket historical deletion, receipt revocation or automatic backfill. New progress_corrections is internal audit, never another analytical ledger. Previous leave-stamps-and-only-hide policy is superseded for explicit corrections. Applied migrations stay intact; new20261029000000 remains local. Three personal rows remain unchanged pending separate approval. Evidence: wave2e-backward-correction-2026-10-29.md.


## 2026-10-29: Wave2E locked / completed

Owner validates and locks the scoped reconciliation. Generic20261029000000 and separately authorized data-only20261029010000 are applied;33local/remote versions align, no pending. Only approved three-anime residuals retired with full audit; legitimate positions, posture, global ratings, annual/all-time hours and independent history retained. UNKNOWN stays unknown, Bleach stays excluded, empty-database reconstruction remains separate nonblocking debt. Do not reopen the reconciliation without new evidence or explicit owner scope. Final proof: wave2e-final-reconciliation-2026-10-29.md. Git commit/push remain owner actions; no next workstream authorized by closure.


## 2026-10-29: concise Top Picks and bounded native image sources

Owner approves hiding annual season labels only for complete first-view work and complete rated scope in that year; keep partial/multi-year/cour identifiers. Remove film release-year fallback in card/panel, including no fallback when a whole-work label is hidden. Scores/qualification remain unchanged. Reuse existing image helper for native density/width candidates under global unoptimized configuration, keep compact40pxdensity/blur and lazy loading, bound long poster rails, avoid eager fallback preloads. No universal native-detail enhancement claim; actual original resolution still limits quality. Evidence/manifest: image-quality-and-top-picks-implementation-2026-10-29.md. Reopen on contradictory viewing scope or measured image-quality/performance evidence.


## 2026-10-30: same opinion only for proven equivalent single scope

Owner approves global/single-season bidirectional edits only where accepted complete membership and actual progress prove equivalence; anime needs accepted mapping, with no independent parts. One append-only scoped opinion, compatibility projections synchronized inside existing transaction, no historical backfill. New season/part breaks equivalence; never average global opinion automatically or propagate it into new scopes. Preserve original opinion audit, unknown dates and existing annual attribution on scoped edits. Nested part controls retain stable-identity/snapshot guards; missing accepted evidence is fixed by robot bootstrap, never by trusting a legacy cache or copying a season grade into each part. Rewatch uses one visible entry and one editable-date form with shared compact controls. Reopen only on owner policy change or contradictory membership evidence. Migration20261029040000 and worker deployment are pending owner rollout. Evidence: single-scope-ratings-and-rewatch-2026-10-30.md.


## 2026-10-30: public viewing timeline and film default date

Owner explicitly rejects the raw persistence journal as Viewing history. Consumption rows and terminal completion context form the UI; rating milestones, superseded assertions and recording-date audits remain internal. No deletion of audit to improve presentation. Keep frozen rewatch scope/duration and edit/remove through one timeline; translate anime summaries through the existing lens. Owner explicitly changes the film claim default: marking a film watched now records today's local day unless a past/partial/unknown date was supplied. The existing atomic service path owns this rule for insertion and update. Existing rows and explicit historical corrections are not retrospectively dated. Reopen for contradictory personal evidence or explicit product change. Evidence: viewing-timeline-and-film-date-2026-10-30.md.


## 2026-10-30 — Personal state card and contextual viewing panel

Owner approves the revised visual direction: state/action on the petrol card, streaming providers separate, one panel for useful first-view part details and independent rewatches. Ordinary season dates stay on the season surfaces; no redundant history for an unconsumed or ordinary no-rewatch title. Explicitly reject episode-click journals, raw audit/recording/rating assertions and guessed dates for nested parts. Use the accepted Stats ledger for minutes/coverage; full-title completion is not another consumption. Preserve first-view unknowns and frozen independent rewatches. Implementation requires owner visual acceptance before declaring UX completed; reopen presentation on that feedback, not by adding another history model.


## 2026-11-02 - Watching pastel labels and stable hover

Owner chooses pastel pill badges inspired by the supplied reference across Watching; status colours carry semantic meaning. This supersedes earlier dark flag treatments in this module. Shared Badge gains an opt-in pastel variant; Watching consumes it through one adapter and globals.css palette. Keep scores/hearts/bookmarks/rank ribbons as distinct marks, not generic text pills. Last Watched date/caught-up label belongs beside the title. Zoom artwork only to keep labels stable. For You direct Want to Watch reuses the existing AddMark writer and remains keyboard/touch accessible. Reopen only on owner visual feedback or a measured contrast/readability problem.


## 2026-11-02 - Owner rejects pastel badge experiment

Supersedes the earlier same-day pastel-label/image-only-hover decision: restore the pre-experiment badge skin/positions and whole-card MediaCarousel hover. Keep independent For You add, keyboard access, cache consistency, cleanup and Stats memo optimization. Rail cards must have enough edge clearance to visibly expand past their original bounds while keeping content aligned. Reopen badge redesign only on a new owner request; do not retain the rejected adapter or alternate skin as dead code.


## 2026-11-02 - Dark status pills from owner reference

Owner explicitly reopens status presentation: dark tinted pills, coloured labels, left glyph and conditional black artwork veil. Share palette/geometry in globals.css between poster statuses and personal state card. Supersedes rejected pastel skin; preserve accepted whole-card hover, rails, distinct score/heart/bookmark marks and semantic classification. Reopen on owner visual feedback. Evidence: dark-status-ci-2026-11-02.md.


## 2026-11-02 - Artwork actions, status roles and Person sidebar

Owner approves discussed hierarchy: white media Add, dark person-plus Follow/dark Dismiss with shared geometry;20px compact status/26px regular status; inline Last Watched Caught Up; neutral11px timestamps; shared overlay trailer Button and quiet editing/subtle navigation. Person inherits card/header/counter presentation using Panel and shared AccoladesSummary while preserving distinct person award scope and journey metrics. Genre pills and accepted hover/rail alignment unchanged. Supersedes prior discussion-only note; reopen on visual feedback or proven accessibility issue. Implementation: design-harmony-2026-11-02.md.


## 2026-11-02 - Correct position is not Resume watching

Owner chooses consistent position/pencil access across active and suspended series/anime. Correction preserves Paused/Dropped/reason at all positions, even aired frontier or finished endpoint. Resume/Finish express explicit intent; correction does not stamp today. Preserve2E transactional backwards cleanup and independent history. Supersedes the old paused-frontier resume and correction-year stamping. Person highlights use80px full-width rows; merge identical highest/gem entries. Weekly Caught Up remains, sharing compact geometry with neutral position. Reopen only for explicit product change or contradictory regression evidence. See correction-stance-highlights-2026-11-02.md.


## 2026-11-03 - Broadcaster and animation studio are distinct facts

TMDB networks must display as Network, never Studio. Legacy studio column retains its documented compatibility source (TV network / film production company); animation credits belong separately to each stable AniList entry and include only primary animation studios. Unknown cache remains unknown. Reuse cron->cache->app and existing query key. Metadata enrichment must not create provider scope growth or touch personal records. See network-animation-studios-2026-11-03.md; owner rollout pending.


## 2026-11-04 - Consumption, correction and availability

Owner-approved: a correction fixes recorded progress without creating a viewing date/frontier timestamp; retain known historical year/precision, UNKNOWN when scope evidence conflicts. caught_up_at is never Last Watched recency. Newly released seasons/identified parts after a completed scope require explicit resumption; weekly active follow-up continues. resume_required is the persisted availability hold, with requiresResume and activityBucket as shared projections; consumed-library engagement is retained. Migration prospective only; no repairs to old data. Rejected: max(caught_up_at) sorting, automatic resume on release, date-gap part inference and automatic dating of corrections. Reopen only on a deliberate product decision; Updates will consume these rules rather than become a status writer. Local implementation/report: tracking-intent-2026-11-04.md. Remote rollout pending.

## 2026-11-05 — Initial image reveal belongs to each section

Owner-approved: coordinate visible native image loading/decoding independently for each hub section, keep exact card geometry under its skeleton, skip cache replay and retain visible content on refetch. A bounded2500ms deadline permits late artwork instead of blocking forever. Reject whole-page image barriers and per-card stagger; reconsider only if measured owner-visible behavior warrants it. Status-card changes retain a single content shell and animate height, not overlapping identities or scaled text.

## 2026-11-05 — Completed recency uses its completion projection

Supersedes unconditional max(watched_at,last_watched_at) for completed claims. watched=true ranks and labels only by valid watched_at; unknown completion remains unknown. Ongoing titles retain genuine forward-viewing recency. Raw legacy activity stamps may be historical claim dates, so equality with caught_up_at or recency alone is not proof of viewing. Reopen only with an explicitly modeled independent viewing requirement/evidence; do not discard real ongoing activity or invent a date to clean a rail.

## 2026-11-05 — Updates is personal release availability, not a consumption feed

Owner approved All/Series/Anime, Available now, Coming next by month and DateTBA, using existing Watching materials/size tokens. Paused/dropped are secondary opt-in. Existing started scopes do not emit weekly episode notifications. Exact accepted anime grouping only; absent dates remain unknown, and a returning lifecycle alone cannot invent a new season. Explicit Resume uses the single writer and opens the fiche without moving position or dating a viewing. No new robot/storage schema required for this version. Reopen feed/discovery/notification scope only with an explicit product request.


## Updates release news / visual revision — 2026-11-05

Owner approved replacing backlog-style Available now with dated release news and larger vertical Seasons-token posters. One evolving card per explicit season/accepted part; unseen content during confirmed airing, then a 30-day release window. Old unwatched seasons are not news; timestamps of edits/robot checks are not release evidence. Paused upcoming/TBA announcements stay visible, dropped requires opt-in. Continue is navigation; explicit Resume retains the single writer. Four columns on desktop, responsive fewer columns, unchanged 144px poster ladder. Initial horizontal full-width rows and anime-only catalogue evidence are superseded. Reopen the release window only on a product request; do not add historical viewing dates to improve news coverage. Implementation/evidence: updates-page-2026-11-05.md.


## Updates owner visual correction / Available now only — 2026-11-05

Plain poster-grid presentation rejected. Owner authorizes section-by-section work: Available now backdrop carousel first, future Coming next/TBA compact poster-left cards remain discussion-only. Include genuinely recent Want to watch series/anime releases, never old backlog or automatic resume. Hide empty Available section. Four desktop cards, mobile peek, aligned navigation and See all panel beyond four. Current implementation affects Available only; historical posters/other sections unchanged. Evidence and limitations: updates-page-2026-11-05.md, latest owner-review section.


## Updates announcement cards implemented — 2026-11-05

Owner authorizes both remaining sections: compact poster-left graphite cards for Coming next/TBA, shared visual family and full-card detail link, no notification/follow machinery. Existing date chronology and eligibility retained.80px/2:3 poster token,1/2/4columns and matching skeleton. Available carousel unchanged. Supersedes discussion-only implementation status above; visual acceptance still pending.


## Updates native carousel / full watchlist eligibility — 2026-11-05

Owner approves actual shared MediaCarousel reuse, removal of Updates heading/redundant View details, first-visible header filters,112px announcement posters with no image zoom, and Want to watch eligibility in ALL three sections. Shared carousel exposes typed custom rendering while retaining one layout/navigation/reveal. Native5desktop/mobile portrait geometry wins over the temporary4desktop/85%-mobile private rail. No automatic watchlist resume, invented scope/date or production data correction. Owner visual validation pending; prior first-section-only eligibility superseded.


## Updates announcement rails / default priority — 2026-11-05

Owner confirms four desktop announcement cards, horizontal rail for Coming next AND TBA, no repeated month heading, poster-derived ambient background matching Don't Miss,112px sharp posters with density-aware sources, and started-before-watchlist TBA order without a filter. Remove suspended opt-in control/state: paused announcements visible, dropped excluded by default. Available artwork unchanged. Supersedes the three-column announcement grid and suspended toggle. No data correction/schema/deploy.


## 2026-11-11: optional AnimeThemes lookup through HEGON

Owner explicitly requested a resilient same-origin boundary after browser CORS/503 failure, preserving favorites/player and early track display. Optional public catalogue discovery remains on-demand behind authenticated Next, validated successful-result caching and bounded failure handling; it does not become a status/date source or introduce a new DB catalogue. Saved favorites remain the durable personal collection. This is the narrowly approved enrichment exception to normal cron/table reads. Reopen a robot/catalogue persistence design only if cold-outage availability or exact provider-ID mapping is requested. Read-only audit and verification: anime-themes-audit-2026-11-11.md.


## 2026-11-17: deletion removes active consumption, not the audit

Owner explicitly rejects deleted test titles contributing to Hours Watched. Supersedes the prior retained-history synthetic-title policy for active analytics. Only history/ratings attached to a current owned row with matching work identity can contribute to Stats, calendars, goals and new achievement observations; re-adding a work cannot revive orphan records. Audit rows and already-issued permanent receipts stay intact. No production backfill. Reopen only for an explicit separate archive/history feature.

Season/cour ratings require the full announced scope to have aired and been watched; caught-up alone is insufficient. Independently complete parts remain rateable; missing membership blocks new assertions without erasing historical opinions. Prospective private SQL guard, no date invention. Owner confirms the current Dragon Ball Z (rather than Kai) identity is correct; retain it.
