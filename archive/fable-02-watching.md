# Audit produit — Watching

> Fable · 2026-07-13 · Regard produit, pas code. Aucun fichier modifié hors celui-ci. `hq/` non lu (consigne).

---

## ⚠️ INCIDENT PENDANT L'AUDIT — à lire avant le reste

**J'ai supprimé ta liste « GOAT » par accident, et elle est irrécupérable par l'app.**

- Comment : pour ouvrir le détail d'une liste, j'ai utilisé un `--click` avec un sélecteur générique (`div[class*='grid'] a, …button`). Le premier bouton de la grille était **l'icône corbeille au hover de la carte GOAT**. Un clic → toast « List deleted. » — pas de confirmation, pas d'undo.
- Vérifié en base immédiatement : `media_lists` ne contient plus GOAT et **0 ligne orpheline** dans `media_list_items` → le FK a cascadé, les **9 items sont détruits** (hard delete, pas de soft-delete).
- Ce que je sais de son contenu : 9 titres, « Updated today » (13/07), couverture = **Game of Thrones, Breaking Bad, Attack on Titan** (les 3 seuls visibles sur la mosaïque, capture `w2-lists.png` prise AVANT l'incident — elle fait foi).
- **Récupération possible côté toi uniquement** : Supabase Dashboard → Database → Backups (backup quotidien ou PITR selon ton plan) → restaurer les lignes `media_lists` + `media_list_items` d'aujourd'hui avant ~l'heure de l'incident. Je n'ai **pas** tenté de recréer la liste moi-même : j'aurais fabriqué 6 titres sur 9.
- Je n'ai plus utilisé de sélecteur générique ensuite (clics par texte exact uniquement).

L'incident est aussi un constat produit (voir C-11) : **supprimer une liste entière = 1 clic sur une icône au hover, sans confirmation, sans undo, avec cascade** — alors que supprimer un simple titre passe, lui, par un modal de confirmation.

---

## 1. Les écrans que j'ai réellement vus

| Écran | Viewport | Capture (`scripts/.shots/`) |
|---|---|---|
| Home Movies | mobile 390 | `w2-home-m.png` |
| Home Movies | desktop | `audit-watching.png` (prise le même jour, audit précédent) |
| Home TV Shows | desktop | `w2-tv.png` |
| Library | desktop + mobile | `w2-lib.png`, `w2-lib-m.png` |
| Library — select de tri ouvert | desktop | `w2-add.png` (clic raté devenu utile : options de tri visibles) |
| Modal « Add to Library » (état vide) | desktop | `w2-add2.png` |
| Lists (index) | desktop | `w2-lists.png` (avant incident) + `w2-list-detail.png` (après — toast visible) |
| Détail de liste « Mind Blowing Movies » | desktop | `w2-listdetail2.png` |
| Stats (année 2026) | desktop + mobile | `w2-stats.png`, `w2-stats-m.png` |
| Fiche détail Attack on Titan (watched) | mobile 390 | `w2-detail-m.png` |
| Personne — Brad Pitt | desktop | `w2-person.png` |

---

## 2. Constats (faits vérifiables sur les captures)

- **C-1** · Les trois homes (Movies/TV/Animes) s'ouvrent sur **deux rails de découverte** (« Don't Miss », « For You ») avant tout contenu personnel. Sur mobile, « In Progress » (TV) arrive après ~2 écrans de scroll.
- **C-2** · Sur mobile, les onglets sont `Movies · TV Shows · Animes · More ⌄` — **Library, Lists et Stats vivent derrière « More »** (2 taps).
- **C-3** · La carte « In Progress » (TV home) affiche la position `S03 E03` + barre de progression, mais **aucune action** : avancer d'un épisode = ouvrir la fiche.
- **C-4** · Le tri Library = Date added / Rating / Title / Year / Favorite. **Pas de filtre genre, année ou plateforme.** Recherche = plein texte titre.
- **C-5** · Le détail de liste est riche : mosaïque + description + stats (15 titres · 14 watched · 8.4 avg) + Rank + recherche interne + table double note (TMDB or / MY RATING teal) + vue liste/grille.
- **C-6** · La fiche mobile : poster centré, meta avec badges TMDB **et** IMDb, synopsis clampé + « More », « Watch trailer », puis la carte de statut (Watched · Finished · 2017–2021 · Log rewatch · Where to watch). L'action d'état est au-dessus du pli.
- **C-7** · Stats : chips d'années 2013→2026 + All time, **Export**, **« 2026 Wrapped »**, KPIs avec sous-répartitions, goals inline, donut heures (dont part rewatch), activité par an, distribution des notes, Top Picks, Top Genres.
- **C-8** · Page personne : « Your titles with Brad » (notes perso + rôles), toggle **Timeline**, tri par ta note, carte « Your journey » (14 vus · 8.1 avg · « 2 of your Top 10 are theirs » · filmographie 14/87), « Seen with Brad » (collaborateurs).
- **C-9** · Modal Add : recherche TMDB, état vide correct (icône + « Find a movie · Search to import from TMDB »). Placeholder « Search for a **movie**… » depuis la Library onglet « All » — le type visé n'est pas explicite.
- **C-10** · Le breadcrumb (`Watching / Movies`) et la barre d'onglets disent la même chose, l'un au-dessus de l'autre, sur tous les écrans desktop.
- **C-11** · Suppression d'une **liste** : icône au hover, 1 clic, hard delete cascadé, toast sans undo. Suppression d'un **titre** : modal de confirmation. Deux politiques pour deux destructions comparables.
- **C-12** · La grammaire couleur des notes est appliquée partout : **or = le monde (TMDB/IMDb), teal = toi**. Cohérente sur home, library, listes, stats.

---

## 3. Réponses — jugements assumés (mes opinions, classées par impact)

### Q1 · Affiché mais inutile — ce que je supprimerais

1. **« For You » en l'état** (les 3 homes). Deux rails de découverte par home, c'est un de trop : « Don't Miss » (trending + gems) couvre déjà la découverte. For You = du TMDB-similar sans signal d'intention, posé au-dessus de TON contenu. Je le supprimerais des homes — ou je le fusionnerais dans Don't Miss. Personne ne le pleurera : c'est le rail qu'on scrolle pour atteindre le sien. *(Si tu y tiens : en dernière section, jamais deuxième.)*
2. **Le breadcrumb desktop** (C-10) — redondant avec les onglets à 40px près. Personne ne le remarquera.
3. **« My Top 10 Series/Films » sur les homes** — c'est une belle donnée au mauvais endroit : elle ne change presque jamais, donc elle rallonge un écran quotidien avec du contenu statique. Sa place est dans Stats (ou une liste auto). La home d'un tracker doit être ce qui BOUGE.
4. **L'année sur les cartes Library** — 4 métadonnées par carte (badge statut, cœur, note, année) ; l'année est la moins actionnable pour une bibliothèque que tu connais. À contraster : la note, elle, mérite sa place.
5. **« Updated 1w ago » sur les cartes de listes** — information d'ambiance, aucune décision ne s'y prend.
6. Ce que je ne supprimerais PAS malgré la tentation : Export (c'est ton backup), l'attribution JustWatch (contractuelle TMDB), les deux notes or/teal côte à côte (c'est ta grammaire, elle est lisible).

### Q2 · Ce qui manque et qui paraîtra évident une fois dit

1. **+1 sans ouvrir la fiche.** L'app connaît `S03 E03` sur la carte In Progress mais l'action la plus fréquente du produit — « j'ai fini un épisode » — exige home → carte → fiche → stepper. Un bouton `+1` sur la carte (ou long-press → quick actions) transforme l'usage quotidien. C'est LE manque n°1.
2. **« Up Next » et le reste-à-voir.** Nulle part : « il te reste 5 épisodes dans S3 », « prochain : S03E04 — *titre* ». La donnée existe (position + season_episodes). C'est ce qui distingue un tracker d'une liste.
3. **Le radar nouveautés.** FROM est « Ongoing », HotD attend une S3 — rien ne te préviendra qu'une saison est sortie (l'info n'existe que dans les masques « Coming soon » du Watch History, en pull). Une pastille « New season » sur les cartes Paused/Watching + une ligne sur la home = la raison de REVENIR dans l'app.
4. **Noter au moment de marquer vu.** Marquer « watched » déclenche le ripple Goals mais ne demande jamais « c'était comment ? ». C'est LE moment de capture (tu l'as prouvé avec le drop-reason). Un slider dans le toast/sheet post-watched remplirait tes stats sans friction.
5. **Le journal chronologique.** Tu as les données (watched_at, années par saison, rewatches datés) mais aucune vue « qu'est-ce que j'ai regardé en mars ». La Timeline de la page personne prouve que le pattern existe déjà — il manque la version globale. (C'est le cœur de Letterboxd, et tu as DÉJÀ la donnée qu'eux n'ont pas : les saisons datées.)
6. **Filtrer la watchlist par plateforme.** « Ce soir, Netflix : qu'est-ce que j'ai en Want to Watch ? » — les providers sont déjà sur chaque fiche ; il manque le filtre dans Library/Want to Watch. C'est LA question du vendredi soir.

### Q3 · Le parcours — où est le frottement

- **Ajouter un titre** : Library/rail → Add → recherche → choisir → statut. ~4 interactions, modal propre — **pas de frottement notable**. Deux réserves : le placeholder « Search for a movie… » depuis « All » (quel type vais-je créer ?), et l'ajout depuis mobile home qui dépend du bouton Add des rails.
- **Mettre à jour où j'en suis** : le pire ratio fréquence/coût de l'app (cf. Q2.1) : 4-5 taps dont un scroll sous deux rails de découverte. Fiche ouverte, en revanche, c'est propre (carte d'état au-dessus du pli, steppers directs).
- **Noter** : fiche → slider, deux taps + un geste, très bien. Le manque est en amont (Q2.4 : personne ne t'y amène au bon moment).
- **Retrouver quelque chose** : Library fait le travail à 344 titres (recherche + type + statut + tri). Le frottement est **mobile** : Library derrière « More » (C-2) + ~3 rangées de filtres avant le premier poster. Et il manque le filtre plateforme (Q2.6).

### Q4 · États vides et de chargement — ce qu'ils devraient dire

Vu : le vide du modal Add (bon : icône + instruction), « No media found » (nu), « No episodes found. » (nu), le vide des Highlights (bon : « Star an episode in All to pin… » — il APPREND le geste). Les chargements : skeletons artisanaux par écran, corrects sur la fiche.

La règle que j'appliquerais (et qui mérite ton composant partagé) — **un état vide répond à trois questions : qu'est-ce qui vivra ici, pourquoi c'est vide, quel est LE geste** — avec trois natures distinctes :
1. **Premier usage** (la section n'a jamais eu de contenu) : vendre + enseigner. « Your rewatches will live here — log one from any watched title. » + bouton.
2. **Vide filtré** (il y a du contenu, tes filtres l'excluent) : « No paused anime match “fire” » + **Clear filters**. Jamais le même écran que le premier usage — c'est le bug classique de « No media found » qui ment quand c'est juste le filtre.
3. **Erreur/réseau** : dire que c'est réparable + Retry. Jamais un vide silencieux qui ressemble à « tu n'as rien ».
Chargement : le skeleton doit mimer la géométrie de l'écran final (posters → rectangles 2/3, rails → rangée avec peek) — jamais un spinner central ; et une seule vague, pas de pop par section.

### Q5 · Mobile — le regard réel

Ce qui marche : le peek de rail (2.6 cartes = « ça scrolle »), la carte d'état sous le hero de la fiche (l'action au pouce), la grille 3 colonnes, la fiche mobile globalement excellente (C-6).

Ce qui coince, par impact :
1. **La hiérarchie de la home** : deux rails de découverte avant ton contenu (C-1). Sur desktop c'est discutable, au pouce c'est cher. Ordre que je défendrais : **In Progress → Recently Watched → Want to Watch → (une) découverte**.
2. **Library derrière « More »** (C-2) : la destination la plus utilisée à 2 taps. Sur mobile, l'ordre des onglets devrait changer : `Movies · Library · More(TV, Animes, Lists, Stats)` — ou mieux, une vraie bottom-bar module.
3. **La rangée de filtres Library** : type + statut + recherche + tri + compteur ≈ 3 rangées avant le premier poster. Une rangée (recherche + bouton filtres → sheet) rend un écran entier.
4. Pas de quick action sur les cartes (long-press) — tout passe par la fiche (recoupe Q2.1).

### Q6 · La comparaison

**En retard sur** : Trakt pour tout le « moteur télé » — up next, calendrier des sorties, notifications, historique horodaté fin ; Letterboxd pour le journal chronologique et le réflexe « je note en loggant ». C'est le même trou vu de deux apps : **HEGON mémorise magnifiquement le passé mais ne pousse pas le présent** (rien ne te dit quoi regarder ce soir ni ce qui vient de sortir pour toi).

**À égalité ou mieux exécuté** : stats (ton Stats + Wrapped est au niveau du Letterboxd Pro payant, en plus beau), listes (Rank, double note, recherche interne — au-dessus de Trakt), fiches (la carte d'état état-aware est plus claire que les deux).

**Ce que tu as et qu'aucun n'a** — à protéger, c'est ton moat :
1. **La mémoire par saison datée** (« S1 en 2022, S2 en 2024 ») — Letterboxd ignore les séries, Trakt ne date pas comme ça.
2. **La page personne subjective** (« 2 of your Top 10 are theirs », ta filmographie 14/87) — Letterboxd a les stats par acteur en Pro, sans cette voix.
3. **Le player d'openings d'anime** — personne, nulle part.
4. **Les fils croisés** (un film compte dans un Goal, ripple à l'appui) — impossible ailleurs par construction : eux n'ont pas le reste de ta vie.
5. La grammaire or/toi (C-12) — une cohérence de langage qu'aucun des quatre n'a.

**Priorité si je devais ordonner le backlog produit** : 1) +1 sur carte & Up Next (Q2.1-2) · 2) hiérarchie home mobile + Library hors de « More » (Q5.1-2) · 3) noter au mark-watched (Q2.4) · 4) confirmation/undo sur la suppression de liste (C-11 — j'en suis la preuve vivante) · 5) radar nouveautés · 6) journal chronologique · 7) filtre plateforme.

---

## 4. Ce que je suppose (non prouvé)

- **Ton usage** : quotidien, mobile d'abord, « je viens de finir un épisode » comme geste dominant — tout Q3/Q5 pèse ses recommandations là-dessus.
- La valeur faible de « For You » : jugée sur sa position et sa nature (TMDB-similar), pas sur des données de clic — si tu y découvres réellement des titres, ma reco Q1.1 s'adoucit (déplacer, pas supprimer).
- Le modal Add pré-scoperait le type selon l'onglet courant (placeholder « movie » vu depuis All) — non testé au-delà de l'ouverture.
- L'app ne vérifie pas les nouvelles saisons en push (déduit de l'UI vue, pas du code serveur).
- ⌘K : je n'ai pas vérifié s'il cherche dans TA bibliothèque ou seulement navigue.

## 5. Ce que je n'ai pas pu voir

- **Le flux Add complet** (résultats de recherche, écran de choix statut/note, création effective) — je n'ai pas voulu créer de données après l'incident ; jugé sur l'état vide seulement.
- Le mode **Rank** des listes, l'édition d'une liste, la création de liste.
- **Animes home** (supposé identique à TV), la home Movies desktop au-delà du haut de page, Stats mobile en détail (capturé mais non analysé finement : `w2-stats-m.png`).
- La page personne en **mobile** ; le player d'openings en lecture ; le détail « 2026 Wrapped ».
- Les états d'erreur réseau et les skeletons en conditions réelles (cache chaud partout).
- Tout parcours d'écriture de bout en bout (noter, marquer vu, logger un rewatch) — audit en lecture seule après l'incident, à l'exception du clic destructif involontaire documenté.
