# Watching — Backlog

> ⛔ **PREMIÈRE TÂCHE DE L'AUDIT WATCHING (phase 3), avant tout le reste — décidé le 2026-09-12 :**
> réconcilier les 4 fichiers de ce dossier (`backlog.md`, `vnext.md`, `architecture.md`,
> `decisions.md`) **contre le code**, ligne par ligne, par `git log` et l'arborescence — jamais par
> lecture des `.md` entre eux. `vnext.md` a dit « V2 pas commencée » pendant 3 mois et demi après sa
> construction, et une conversation l'a recopié comme backlog. Ce qui existe est rayé ; ce qui reste
> porte un déclencheur ; un seul fichier vivant survit (celui-ci), les autres sont marqués historiques
> ou fusionnés. Le statut d'une feature se prouve, il ne se lit pas.

> **Chaque ligne porte son DÉCLENCHEUR.** Sans lui, un backlog devient un cimetière : on relit une
> ligne, on ne sait plus si elle compte encore, et on la relit à la session suivante.

| Ce qui est différé | Pourquoi pas maintenant | Ce qui le réveillerait |
|---|---|---|
| ⭐ **PRIORITÉ — Attribution d'année d'une saison RÉ-OUVERTE / multi-années.** Une saison finie (stampée son millésime dans `season_years`/`cour_years`) qui **grandit** (nouveaux ép une autre année) ne peut porter qu'**UNE** année. Le `+1` mid-saison date le viewing (`last_watched_at`) mais ne re-stampe rien (par design : on stampe l'année d'une saison **quand on la finit**, `watch-status.ts` « THE TWO LAWS »). Ex. **Bleach** : saison finie en 2023 (40 ép), +10 ép en 2026 → les ép vus en 2026 restent attribués à 2023 → **absent de la vue Stats 2026** (présent en all-time). Le stamp 2023 « fait de l'ombre » à l'activité 2026. | Le fix honnête = attribution **par cour / par date de visionnage** (sous-saison), sur le système d'année qui est **« un seul écrivain », délicat**. Un `+1` qui écraserait 2023→2026 ré-attribuerait les 40 ép de 2023 à 2026 = **bug inverse**. Vrai chantier, pas un patch de fin de session. | Un **2ᵉ titre** concerné (saison finie qui rouvre sur 2+ ans), OU le chantier **anthologie / position non-linéaire** déjà listé ci-dessous (même racine : le modèle stampe au niveau saison, pas épisode) — faire les deux ensemble. |
| Table de référence partagée `series_facts(tmdb_id)` — sortir les 9 colonnes volatiles/synchronisées (`status`, `season_aired`, `season_end_dates`…) de `media_items` vers une table `tmdb_id`-only, pour que 2 comptes qui possèdent la même série ne stockent plus 2 copies qui peuvent diverger. Design fait + revu par Fable (~85 %→solide après 6 corrections), doc archivé `hq/archive/series-facts-design.md`. | Différer ne coûte rien à 1-3 utilisateurs (backfill trivial) ; c'est un vrai chantier (migration + refactor service). | **AniList v2** (le prochain chantier anime touchera de toute façon `tmdb_id`-scoping) **ou** l'ouverture des inscriptions (le vrai coût de la duplication apparaît avec plusieurs comptes qui partagent des séries). Le premier des deux. |
| `theme_favorites` pèse ~8 KB sur **chaque** page Watching, pas seulement `/themes` — le `ThemePlayer` (et sa playlist complète) vit dans le layout, donc la liste part même si l'utilisateur n'ouvre jamais le lecteur. Seul l'état des ♥ est réellement nécessaire avant l'ouverture. | Poids mesuré mais pas prioritaire — pas dans le chemin critique des pages grille. | Une passe perf qui rouvre le layout Watching, ou si le poids grossit avec le nombre de favoris. |
| Contrainte type-conditionnelle manquante en base : rien n'empêche aujourd'hui d'écrire `cour_years`/`cour_ratings` sur un film ou une série non-overlay. | Petit trou, pas encore exploité (aucune écriture erronée observée) ; la lentille `MediaView` route déjà correctement en écriture normale. | Une régression observée qui écrit `cour_years` hors contexte anime-overlay, ou l'ajout d'une contrainte DB dans une prochaine migration de toute façon prévue sur `media_items`. |
| L'app résout l'authentification de **3 façons différentes** (`getUser` dans `library`, `getSession` dans `(main)/layout.tsx`, `getClaims` dans le middleware), aucune enveloppée dans `cache()`. | Fonctionne aujourd'hui, coût mesuré comme secondaire face aux autres chantiers perf. | Une prochaine passe perf CSR, ou une incohérence de session observée entre deux surfaces. |
| Deux irrégularités de chargement repérées le 22/07, **cause non établie** (pas d'A/B avant/après fait) : `library` — son skeleton **disparaît** alors que les données chargent encore (93→0 pulsations avant les données) ; `movies` — **~400 ms de vide** avant que son skeleton n'apparaisse. | Pas encore diagnostiqué — `library` est une route qui n'a pas été touchée récemment, donc probablement pas une régression introduite. | La prochaine fois qu'on touche au chargement de ces deux routes, ou si l'audit rapidité (Audit 3) les fait remonter avec des chiffres. |
| Anthologies (ex. True Detective) : le modèle de position est **linéaire** (un curseur saison/épisode), il ne peut pas exprimer « j'ai vu la S1 et la S4, pas la S2/S3 ». | Trou de modèle identifié mais pas urgent — une seule série connue concernée dans la library actuelle. | Une 2ᵉ anthologie ajoutée à la library qui rend le curseur linéaire visiblement faux à l'usage. |
| Position à corriger manuellement pour 2 titres hérités d'une affirmation invérifiable du script de repair (`One-Punch Man`, `Silo` — `True Detective` déjà classé comme cas d'anthologie ci-dessus). | Correction manuelle ponctuelle, pas un chantier. | La prochaine fois que l'un de ces titres est ouvert/édité. |
| Remède « titre du vrai cour dans le popover » pour Re:Zero et les 3 autres titres cour≠saison (voir `decisions.md`, entrée 2026-07-20) — accepté mais pas codé. | Décision figée, implémentation non prioritaire tant que ça ne « regratte » pas. | Une remarque owner sur la confusion des numéros de saison d'un de ces 4 titres. |
| Achievements **cliquables → sliding panel de détail**. **Front (quick win, validé brainstorm 01/08)** : panel **lecture seule** qui porte le **POURQUOI** — titres contributeurs (« Marathoner → débloqué par X, Y, Z »), **calcul exact** de progression (« 14/20 séries, encore 6 : candidats… »), **date de déblocage**. Sinon = chrome vide (HEGON déteste). Coût : `computeStats.ts` doit **exposer les items contributeurs**, pas juste un booléen. **Back (vrai chantier, différé)** : rendre le set **éditable/créable** depuis l'UI — table DB `achievements` + moteur de règles **data-driven** remplaçant les 12 compteurs **codés en dur** (règle+seuil+couleur+icône). | Les 12 viennent d'être calibrés sur la vraie biblio — justes tels quels. Le panel read-only est petit et isolé ; l'éditabilité est le gros morceau (moteur data-driven, pas juste le panel). | **Front** : envie de rendre les badges vivants / cohérence sliding-panels OS — petit, à prendre quand on repasse sur Stats. **Back** : envie de tweaker un seuil / ajouter un achievement **sans redéploiement**, ou l'ouverture des inscriptions (chaque compte voudra ses objectifs). |
| ⭐ **MUSÉE — page History / Awards (Oscars ↔ Emmys).** Pousser Watching du *tracker* vers le *musée* : un lieu où **ta collection rencontre le canon**. **Reframe (pas un almanach Wikipédia) :** la donnée awards tisse **3 surfaces** — ① **page dédiée** = frise d'années scrubbable + 2 modes (*Explorer une année* : lauréats/nommés, **ceux que tu possèdes s'allument** anneau teal, le reste « ＋ watchlist » ; *Ton panthéon* : étagère à trophées de TA biblio, compteur canon « 12/96 Best Pictures vus » branché sur les achievements) · ② **fiche** = section *Accolades* factuelle · ③ **page Person** = timeline de récompenses de l'acteur/réal. Source **Wikidata** (SPARQL), **join direct → TMDB via `P4947` film / `P4983` série / `P4985` personne** (pas de matching par titre). Donnée **historique figée** → table `watching.awards`, ingestion 1×, **cron annuel** post-cérémonie. | **Mini-projet** (table + ingestion Wikidata + cron + nouvelle page + 2 plaques). Pas un patch. La disponibilité data est OK (Wikidata gratuit, join propre) ; le boulot = ingestion + surfaces. Watching est stable → pas urgent. | La volonté d'attaquer la **vision musée** pour de vrai (après Goals publié / passe Sport), ou l'envie d'un des sous-morceaux isolé (la vue *Panthéon* seule est self-contained et branche direct les achievements). |
| **Thèmes de films/séries — les DEUX (miroir du modèle anime), validé brainstorm 01/08.** Grand player → bouton **« Watch video »** = **iframe YouTube** ; scrubber → **mp3 self-hosté** (Supabase Storage), **réutilise le `ThemePlayer` existant**. **Image = TMDB** (déjà stockée, non-problème contrairement à l'anime). Data/thème : `{ label, audioUrl (self-host), youtubeId (iframe), tmdbImage }` — label « Main Theme »/« Score »/nom du morceau (pas d'OP/ED pour un film). **Blocage résolu :** un lien YouTube **n'est pas** un fichier audio → `<audio>` ne peut pas le lire ; d'où le self-host. DL gratuit : **yt-dlp** (`yt-dlp -x --audio-format mp3`) ou **cobalt.tools**. | Petit code (le player existe), mais = data model + Storage + **gate owner sur la démo** (redistribution d'audio sous copyright = le vrai risque, tolérable en perso mono-user). Manuel, peu de thèmes → pas pressé. | Zakaria a **téléchargé les mp3** d'un thème qu'il aime et veut l'ajouter, OU une passe qui rouvre le `ThemePlayer` de toute façon. |

---

## 🐛 2026-09-12 — `series-sync` dé-vue une série terminée sur un hoquet de TMDB (Haikyu!!)

**Symptôme** : Haikyu!! (vu intégralement, terminé en 2020, marqué vu en 2022) est réapparu dans
*In Progress* avec sa position conservée (S4 E25). Aucun nouvel épisode. L'owner a re-cliqué
« mark as watched ».

**Cause** : `supabase/functions/watching-series-sync/index.ts`, **chemin 1** (~l. 300) :
*« A row marked `watched` on a show that is NOT over → it means "I'm caught up" »* ⇒ écrit
`watched: false, in_progress: true`. Règle écrite pour une **migration** (l'époque où « watched »
était le seul statut), mais qui tourne encore **toutes les 4 h contre des données vivantes**.
TMDB est édité par des bénévoles ; Haikyu!! a eu 2 films en 2024-25 ⇒ un éditeur a très probablement
passé le statut à « Returning Series » puis un autre l'a remis à « Ended ». La synchro est passée
entre les deux. TMDB est revenu ; la ligne, non. *(Non prouvable : la trace vivait dans la réponse
HTTP, conservée ~6 h. Mais TMDB aujourd'hui = base de mardi : `Ended`, 4 saisons, 85 épisodes.)*

**Le vrai bug** : **une seule lecture d'un tiers instable peut écraser la déclaration de l'utilisateur.**
Peut refrapper n'importe quelle série terminée, n'importe quand.

**Faute structurelle en plus** : viole ⭐⭐ *« UN SEUL écrivain de statut »* — la fonction edge écrit
`watched`/`in_progress` **directement**, en contournant `lib/watch-status.ts`. → **axe 3 de l'audit
Watching (phase 3).**

**Correctif** : le chemin 1 ne doit se déclencher que s'il y a **réellement du nouveau à regarder**
(`airedTotal > total(m.season_aired)`), jamais sur un simple changement d'étiquette de statut. Ça le
fusionne dans l'exigence de preuve du chemin 2 (*« growth is a COMPARISON, not a deduction »*).
Implique un `supabase functions deploy watching-series-sync` (main de l'owner).

**Sans rapport avec la fausse alerte du watchdog du même jour** (autre job, `anime-cours-sync`).

**💡 Idée owner née de l'incident** : une notification dashboard *« nouvelle saison / nouvel épisode
disponible »* pour les titres vus ou en *Waiting for*. C'est le chemin 2 fait **correctement** :
détecter contre la référence, **montrer**, ne jamais réécrire le statut en silence. → gelé, à cadrer.

---

## 🔍 2026-09-10 — Diagnostic Watching (lecture seule, 4 points owner) — GELÉ, à traiter en phase 3

### 1. For You « toujours les mêmes, et ça ne matche pas mon goût » — 4 causes distinctes

Le code est bon (rotation, MMR anti-mur-de-genre, dédup franchise, cron OK — dernier calcul vérifié).
Le problème est ailleurs :

- **Le dismiss ne remonte jamais.** `localStorage` uniquement (`hegon_dismissed_foryou_*`).
  L'algorithme ne sait pas qu'un titre a été rejeté ⇒ il le resélectionne à chaque calcul. Vider le
  cache navigateur les fait tous revenir. **⭐ Le système n'a AUCUN signal négatif.** → 1er geste.
- **La rotation ne tourne presque pas pour les séries.** `start = (index×20) % pool`. Avec les
  chiffres réels (220 films / 40 séries / 50 animes favoris notés) : films = 11 fenêtres disjointes ✅ ;
  **séries = 2 fenêtres qui alternent indéfiniment** ; animes = 5 fenêtres à 50 % de recouvrement.
- **`MIN_VOTE_AVERAGE = 7.0` sur `/recommendations` TMDB = le consensus, pas le goût.** Le vivier est
  dominé par les mêmes titres célèbres. Le système optimise « bon », pas « toi ».
- **Le badge NEW est faux par construction** : `is_new = !prevIds.has(id)` compare **uniquement à la
  rotation précédente**. Un titre revenu après 3 rotations est re-marqué NEW. → historique des ids.

*(Les 20 stockés / 10 affichés sont un choix délibéré et juste : réserve pour glisser le 11e sans
refetch quand on en ajoute un.)*

**⭐ Direction v2 (owner, 10/09)** : ce n'est PAS « un LLM qui recommande » (hallucine, ne connaît pas
les sorties récentes, biais d'entraînement). C'est **un LLM qui lit les takes / notes / rewatches /
highlights et produit un PROFIL DE GOÛT structuré** stocké dans une table dédiée — puis un moteur
déterministe interroge `/discover` avec ce profil, et le LLM au plus réordonne et explique.
**Le modèle est la bouche, pas le cerveau.** Bonus produit : le profil devient **lisible et
corrigeable** (« voilà ce que HEGON pense de ton goût »). Signaux uniques non exploités aujourd'hui :
**`rewatches`** (le signal de goût le plus fort qui existe), `episode_highlights`, `My Take`.
⚠️ IMDb n'a pas d'API gratuite — seuls ses **datasets non commerciaux** sont utilisables. ⚠️ Les takes
partiraient chez un tiers — décision owner en connaissance de cause. ⚠️ 10e dépendance externe ⇒ si
l'appel LLM échoue, For You continue avec l'algo actuel (R8).

### 2. Want to Watch : 24 titres INVISIBLES, pas « 50 partagés »

`WantToWatchSectionClient` et `WaitingForSectionClient` appellent **la même requête**
`useMovies({ wantToWatch: true, limit: 50 })` puis se la partagent côté client via `isAwaitingRelease`.
Or il y a **74 films en want_to_watch** ⇒ 24 ne sont **jamais chargés**, nulle part.
**Correctif** : rail limité à ~20 (un carrousel de 74 est inutilisable), « See all » qui ouvre le
sliding panel avec **sa propre requête complète**, tri et filtres. Et les 2 sections ne partagent plus
une requête.

### 3. Priorité de want-to-watch non modifiable — et un piège de nommage

Deux colonnes voisines : **`priority` (int 1-10) = rang du Top 10** ; **`priority_level`
(high/medium/low) = priorité want-to-watch**. `priority_level` n'est écrit qu'**à l'ajout**
(`QuickAddPanel`, `useQuickAdd`) — la page discover **force « medium » en dur** — et n'est lu qu'à
**un seul endroit** (`MediaCarousel`). Aucun chemin de mise à jour n'existe.

### 4. La « bande » de priorité — pourquoi elle ne sera jamais parfaite telle quelle

`PriorityMark` = icône Bookmark rouge / ambre / zinc. **Les 3 états ne diffèrent que par la teinte**,
à 17 px, sur une affiche, avec ombre portée — aucun libellé, aucune forme, aucune taille. Ambre/rouge
est la paire la plus dure. **L'encodage n'a aucune redondance** ⇒ aucun réglage de couleur ne sauvera.
**Options** : (a) quantité — bookmark rempli 1/3, 2/3, plein ; (b) ⭐ **n'encoder qu'UN état** : seuls
les « high » portent une marque, les autres rien. Rare = remarquable. Les 3 niveaux restent en base
pour le tri. Owner veut **voir un rendu** avant de trancher.

### 🏷️ Top 10 — badge façon Netflix, idée owner validée

Dans la **section Top 10** : afficher le **rang réel** (1, 2, …) — le badge y serait du bruit.
Dans la **bibliothèque** : un badge « TOP 10 » façon Netflix (ruban à coin replié, **accent Watching**)
= l'**appartenance**, l'info utile quand on balaye. ⚠️ Doit entrer dans la **grammaire des marques**
(2 clusters max par affiche) — vérifier `Marks.tsx` avant.

### Ordre recommandé (valeur/effort)
dismiss persistant → `is_new` sur historique → requête complète du want-to-watch (24 titres perdus
= perte de données perçue) → priorité éditable depuis la fiche → puis le reste.

---

## 🎬 2026-09-12 — Best Episodes : 3 bugs (autorisés pendant le gel) + panneau (feature, gelée)

**Bugs, causes lues dans le code :**

1. **Le « … » reste affiché sur un best ep.** `Episodes.tsx` ~l. 268 : `highlighted ? "opacity-100" : …`
   — choix délibéré (*« or when the best-ep star is active »*), refusé par l'owner : l'étoile dit déjà
   tout. **Fix : supprimer la branche.**
2. **Hover pas smooth vs `MediaCarousel`.** Dans le carrousel **toute la carte** est le `group` et scale
   1.04 d'un bloc, marques incluses. Dans `Episodes` seul le **cadre image** scale (1.03, 300 ms) et le
   cluster étoile + « … » est **hors du cadre**, immobile, `transition-all` 150 ms → deux horloges, un
   bouton qui flotte. Le commentaire *« matching every other card's menu »* est faux. **Fix : cluster
   dans le cadre qui scale + `transition-opacity duration-150 ease-out`.**
3. **Vue Highlights sans date ni synopsis.** Elle rend depuis `episode_highlights`, qui n'a que `title` +
   `still_path` (`types.ts` l. 94-106). **Fix : colonnes `air_date` + `overview` écrites au moment où on
   étoile (photo du moment) + backfill des lignes existantes. Migration.**

**Trouvaille :** `episode_highlights.note` existe en base et **n'est affichée nulle part**.

**Feature validée en direction (owner + moi), GELÉE — déclencheur : fin du gel ou audit Watching :**
**l'épisode devient un objet → clic = sliding panel.** Image en grand, `S4.E25 · date`, synopsis,
ta note, best ep, et **`note`** (la colonne orpheline trouve sa maison). Le panneau règle le bug 3 par
construction. Règles : **supprimer le « … »** (étoile au hover = toggle en un geste, clic carte =
panneau ; deux affordances, pas trois) · lecture d'abord, formulaire ensuite · **même Select de note**
qu'ailleurs (cohérence ; s'il change, il change partout).
