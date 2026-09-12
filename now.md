# HEGON — NOW (état courant)

> Le seul fichier d'état lu au démarrage. **Il ne contient QUE ce qui est vivant.**
> Décisions Watching datées = `hq/modules/watching/decisions.md`. Reports = `.../backlog.md`.
> Archi = `hq/modules/watching/architecture.md`. Règles durables = `hq/rules/`. Journal = `hq/log/`.
> Historique consommé = `hq/archive/`.
>
> **Règle d'hygiène :** quand une entrée devient de l'histoire, elle descend dans l'archive ou le log.
> Ce fichier reste court parce qu'il est lu en entier à chaque démarrage.

**Date de maj : 2026-09-05**

---

## 🛑 FOCUS COURANT (2026-09-05) — CHANTIER AUDIT & FONDATION · **SPORT GELÉ**

> ⛔ **Gel des features — PAR MODULE** *(amendement 6, 2026-09-12)*. Un module est gelé **jusqu'à la
> fin de son audit de phase 3** ; audité, il peut recevoir des features (conversation dédiée, feature
> dans son `backlog.md`, conforme à la doctrine, sans toucher un module non audité). Sport reste gelé
> tant que son audit n'est pas fait. Tout ce qui suit dans ce fichier est de l'**histoire**.

**Le plan vit dans [`hq/audit/plan.md`](audit/plan.md)** — c'est LE document de pilotage, une conversation
d'exécution démarre en le lisant. Il contient les 6 phases, la checklist system design (10 axes), la
checklist par module (9 axes) et le format de rapport imposé.

**✅ Phase 0** → [`hq/audit/map.md`](audit/map.md) (`npm run map` régénère les chiffres).
**✅ Phase 1** → purge, cliquets CI, reconstructibilité, restauration prouvée. Chronologie : [`hq/log/2026-09.md`](log/2026-09.md).

**🔵 Phase 2 OUVERTE le 2026-09-09 → LA DOCTRINE : [`hq/rules/system-design.md`](rules/system-design.md).**
Les 11 axes sont passés. Ce qu'elle établit, en bref : le contrat est tranché (app perso, multi-tenant
gardé, jamais construit sans usage) · **le cache a DEUX étages, pas quatre** (`cache()` de React = 0
usage, revalidation Next = 1 usage et il vise une route inexistante) · **l'échec silencieux est le mode
de panne dominant**, prouvé deux fois (40 h, puis 28 jours) · **Sentry ne couvre pas les robots** —
0 ligne dans les 20 edge functions.
**Livré en code** : le **watchdog** ([`.github/workflows/watchdog.yml`](../.github/workflows/watchdog.yml)) —
alerte cron + garde anti-dérive du schéma, toutes les 6 h (la cadence est structurelle :
`net._http_response` n'est gardé que ~6 h).
**🟢 Sauvegarde : RUN VERT 12/12 le 2026-09-12 (17:50 UTC)**, push vers `hegon-backups` et round-trip
inclus → [`backup.yml`](../.github/workflows/backup.yml). Hebdo, chiffrée, 5 auto-vérifications, témoin
healthchecks.io sur les 2 workflows. Deux contre-examens avant le run, puis le **piège `pg_wrapper`**
au premier run réel (`/usr/bin/pg_dump` routait vers PG 16 ; `--version` *affichait* 16.15 et passait
au vert — corrigé dans les 2 workflows, version *exigée*). **R11 reste « run vert », pas « prouvé »**,
tant que l'owner n'a pas ouvert un fichier à la main, à froid, avec la passphrase du cahier.
**Première vraie alerte du watchdog le 12/09** (500 Gateway Timeout de `football_sync_standings`) →
deux défauts corrigés le jour même : `internal.call_log` nomme la fonction (migration
`20260912000000_call_log.sql`, **appliquée par `npx supabase db push --linked`** — la voie normale
depuis le repair du 09/09, l'historique est tenu) et la section B ne regarde plus que la *dernière*
réponse. **Watchdog vert après migration.** Ajouts owner : **section B-bis** (appel dans `call_log`
sans réponse depuis > 10 min = requête disparue, worker pg_net mort — le cas qu'aucune section ne
voyait) · **grâce healthchecks du watchdog 1 h → 3 h** (GitHub a livré un créneau avec 74 min de
retard ; ⚠️ à changer **à la main** dans healthchecks.io, le paramètre d'URL ne s'applique qu'à la
création du check).
**✅ Doctrine de cache écrite et POSÉE EN CODE le 12/09** (R3/R4 de la doctrine) :
`src/shared/lib/stale.ts` (10 paliers) + 2 règles ESLint dans `eslint.config.mjs` — `staleTime` doit
être un palier (126 sites migrés **à comportement identique**, 51 redéclarations du défaut supprimées,
0 violation) et **N+1 de service** interdit (8 dettes annotées avec leur audit : 5 Goals, 1 F1, 1
Watching, 1 Tasks). Mesure A/B refaite sur **ce que l'utilisateur voit** : grille `/movies` à 1 200 ms
sans persistance, 874 avec → **−326 ms, non prioritaire**, retirée du code — **rejouable** par
`git apply hq/archive/query-persist.patch`. ⚠️ Piège trouvé : en flat config, `no-restricted-syntax`
ne fusionne pas — un bloc `src/**` effaçait le garde des coordonnées Watching ; corrigé par des
sélecteurs partagés listés dans chaque bloc.
**Contre-examen complet passé le 13/09 — 7 points, tous traités** : leçon esquery corrigée (`>` =
enfant direct, sémantique normale) · règle N+1 élargie à 3 formes (boucle avec await, `.map(` sync
qui construit une requête) → `reorderStatuses` attrapé, 8e dette · `STALE.DEFAULT` **refusé** hors
`QueryProvider` (l'héritage est mécanique) · **`gcTime` suit le palier** : 21 sites corrigés + test
`stale.test.ts` (un palier > 10 min sans `gcTime` ≥ est un mensonge — TanStack ramasse avant que ça
périme) · **le « plancher d'hydratation » contenait ~290 ms d'auth** : deux `auth.getUser()` en série
dans `settings/service.ts` montés sur chaque page → `getSession()` local ; **premier appel REST 819 →
631 ms, grille 1 200 → 1 054 ms** (c'était R2, pas le bundle) · chiffres de récit corrigés (51, 10/129,
≤ 15 requêtes au focus, squelette ≈ 870 ms dont 380 post-hydratation).
**▶️ Restent en phase 2** : 4 cliquets (R2 `getUser` — 22 sites restants dans les services —, R5
`upsert`, R7 rate-limit — corriger football d'abord —, R8 `.catch(() => {})`) et les **7 écarts** de
la doctrine §3.

**Pourquoi** : l'owner ne tient plus la codebase (« HEGON sort de ma main »). Mesuré, pas ressenti —
CLAUDE.md porte 6 blocs « Corrected », 19 archives mentent sur leur statut, `knip` trouve ~1 400 lignes
orphelines dans le seul football, ce fichier fait 555 lignes alors qu'il est lu en entier au démarrage.
**Diagnostic : le code va plus vite que sa description.**

**Ordre** : carte → purge + cliquets CI → **system design (produit la doctrine)** → audits par module
(Watching = étalon) → vérité des docs → règles UI/UX + design system.

**Règle du chantier** : rien n'atterrit en prose seule — chaque passe finit en code commité ou en
fichier supprimé. Un mécanisme (gate CI), jamais un état.

**▶️ Reprise Sport après le chantier** : la page stats football n'a pas de route, mais `FanLog` +
`BestXI`/`Legends` sont **déjà construits et orphelins** ; seule logique manquante = le **record de
prédictions** (stockées, jamais confrontées au résultat). Bug à corriger au passage : `loading.tsx`
rend `FootballBestXISkeleton` pour un bloc retiré de la page.

---

## ⚽ SESSION 04-05/08 — SPORT / FOOTBALL DÉMARRÉ (la passe craft du roadmap §2)

> Watching est **clos** → tout ce qui suit = **histoire** (candidat archive `hq/archive/`). Le chantier
> VIVANT = **Sport, 4→9**, en commençant par Football. Détail durable → mémoires
> [[project_football_card_and_branding]] + [[project_football_redesign_chantier]].

**✅ Fait & commité :**
- **Carte `FootballMatchCard`** — gradient-top **100% CSS, pas d'image** (perf ; `--competition-color` =
  glow haut à la couleur de la compète), mark blanc, **ticket doré si stade**. Branding par compète
  (`brand_color`+`logo_url` sur `football_competitions`, logos recolorés blanc/gris DANS le fichier).
  Câblée dans **Goals « Counting toward »** (`getGoalContributingMatches` enrichi : score/crests/venue/
  `watched_where`).
- **Perf Finding 1** — `auth.getUser()` (~150-300 ms en TÊTE du chargement) retiré des **3 modules Sport**
  (football/tennis/f1) → `useCurrentUserId()` synchrone + `enabled:!!userId`, comme Watching.

**Migrations :** branding `20260804` + follow-compétitions `20260805` = **appliquées + vérifiées en base**.

**MAJ 07/08 marathon — détail complet + fichiers dans [[project_football_redesign_chantier]].** L'essentiel :
- **Couleur = LIME `#B6FF2E` + GRAPHITE** (appliquée partout, y compris `dashboard-os/config.ts` `OS_APPS.sport`
  — le Dock rend la tuile via OS_APPS, pas `NAV_GROUPS.accent`). Build-first assumé, le "cool" = à LA FIN.
- **✅ MAIN PAGE COMPLÈTE** (sections indépendantes DB) : Following · **Upcoming · Recent** (menu-select
  `SectionTeamSelect`, 4 cartes/rangée) · **Standings** (filtre compétition, table depuis matchs) · **Fan Log**.
  + **PAGE compétition** (tabs, tout dérivé DB, bandeau off-season) + **PAGE équipe** (stats profondes) +
  **logos cliquables → team page** (match panel + standings). `computeStandings`/`StandingsTable` partagés.
- **Cleanup fait** : déchet 2025 supprimé (CL 189 gardée) + **17 fichiers morts** (via knip ; gardés :
  `FootballMatchCard`=Goals, `supabase/functions/*`=live). Commits `f594eb8` + fan-log + cleanup.
- **Perf** : `football_matches` bien indexé (vérifié) · `getFootballMatch` DB-first · sections indépendantes.

**MAJ 07/08 soir — suite marathon (détail → [[project_football_redesign_chantier]]) :**
- ✅ **Unfollow-équipe rebranché** (bouton Follow/Following/Unfollow sur la Team page ; main team = pill).
- ⛔ **Squad réel ABANDONNÉ** — aucune source gratuite fiable (football-data free `squad:[]`, endpoint teams pollué/faux, TheSportsDB plafonné). Construit puis **revert complet** + table droppée. Best XI manuel reste. Leçon → [[reference_football_data_sources]].
- ✅ **Season rollover** : edge fn `football_sync_season` (full-season compètes+équipes suivies) + cron hebdo. Lecture déjà prête (`maxSeason` + bandeau). Appliqué à la main par l'owner (peuplé, vérifié 1635 matchs).
- ✅ **Wikidata past-winners** : onglet **Honours** (Most titles + Roll of honour) sur la competition page, source Wikidata cachée en base (edge fn `football_sync_wikidata`, cron mensuel). **Palmarès d'équipe abandonné** (Wikidata trop sale : titres de ligue éclatés). Fix bug 1929 (clé = QID de saison). ⚠️ infra à (re)déployer à la main.

**MAJ 19/08 — pages ÉQUIPE et COMPÉTITION refaites et COMMITÉES** (détail → [[project_football_redesign_chantier]]) :
- **⭐ La leçon du lot** : le problème n'était pas la déco des cartes, c'était la **composition**. Une pile de bandes pleine largeur sur du contenu clairsemé = du vide. Réponse : grille **`[2fr_1fr]`** (celle de la fiche Watching) sur la page équipe, **`max-w-6xl`** sur la page compétition, et **rien ne s'affiche s'il n'a rien à dire**. À appliquer d'entrée à tennis/F1.
- **Page équipe** : hero cinématique + picker Artwork (~9 backdrops, **venue fanart** inclus) + **résolveur TheSportsDB** (corrige suffixe FC / trait d'union / équipe féminine) + trophy cabinet 6/row avec 30 vrais logos + carte Season 2 colonnes + `FeatureMatchCard`.
- **Page compétition** : hero à la **brand_color** + logo filigrane + Back, onglets scrollables, **Fixtures = une journée à la fois ouverte sur la journée en cours** (c'était 380 lignes), Summary/Honours en 2 colonnes.
- **Fix prod iOS** : `lang="fr"`→`"en"` + `translate="no"` — l'auto-traduction cassait React (`removeChild`).
- ⚙️ **Mode de travail** : Opus décide et implémente, l'owner ne fait que la **vérification UI** + migrations/commits/déploiements (groupés en une liste).

**▶️ REPRENDRE ICI (19/08)** : page stats football (Fan Log filtrable + record prédictions + Best XI/Legends re-parqués) → tennis/F1 (même moulinette, composition d'entrée) → cleanup → design-system. Ordre owner : audit✅ → UI/UX(en cours) → design-system.

---

## ✅ SESSION 30/07 — WATCHING CLÔTURÉ (histoire — module clos, à archiver)

Watching est **bon**. Il ne reste que le **backlog différé** — chaque ligne derrière un déclencheur (voir `modules/watching/backlog.md`), rien d'actif.

- **Overlays sur artwork = 1 seul cran 8px** — `OVERLAY_CLUSTER` (Marks.tsx) en `top-2`/`left-2`/`right-2` ; tous les outliers (InList, SeasonHistory, Don't Miss) alignés ; héros (Media/PersonHero) exemptés (grande surface = grande marge).
- **CaughtUpBadge unifié** — 1 composant (`flag` teal + `CheckCheck`) partout (In Progress, Last Watched desktop top-left, mobile ligne méta). Tooltip **dérivé de `caughtUpOn`** (épisode vs saison), plus de « next season » codé en dur.
- **Cran on-artwork déterministe** — `flag sm` figé à **`h-5` (20px)** dans `badge.tsx` = LE standard backdrop ; chip position "S03 E12" + boutons +1/Finish alignés dessus. ⚠️ Le micro-jitter des composants au *settle* du scale = sub-pixel **inhérent** (tentatives transform-gpu + hauteurs uniformes = sans effet) → abandonné, on ne complexifie pas sans gain.
- **Trailer HD** — `pickTrailer` trie par **résolution (`size`) puis récence** (The Dark Knight prenait un 240p plus récent qu'une HD).
- **Don't Miss** — split rail/accordéon **par pointeur** (`lg:can-hover:`), pas par largeur → iPad tactile ne reçoit plus un accordéon mort. Tablette (md+) : **grille de 6 sans scroll** ; `DontMissSkeleton` miroir des 3 layouts.
- **Saut au switch de saison** (Episodes) — vraie cause trouvée : `showChevrons` dépendait de `episodes.length` (=[] pendant le fetch) → les flèches `CarouselNav` (h-8 = l'élément le + haut du header) clignotaient → hauteur du header changeait → tout le bloc sautait. Fix : `isLoading || episodes.length > 3`. (Ce n'était PAS la hauteur des cards.)
- **Search Watching = titres + PERSONNES** — `searchCatalogue` (`/search/multi`, ordre de pertinence : « Nolan » → personne en tête, « Dune » → films) ; personne → `/perso/watching/person/[tmdbId]` (page qui rend n'importe quel id). `searchTmdbForList` (ajout aux listes) **intact**, titres-only.

### ✅ Lot 2 — polish device (test mobile / PC / iPad, 1 commit « op A »)
- **Don't Miss** — la carte *owned* (« In your library ») ouvre la **fiche** directement (map `useOwnedMediaIds` → id interne ; fallback discover). Carte entière cliquable.
- **ThemePlayer** — (a) **scrub** : `time` optimiste au relâchement → la barre ne revient plus en arrière ; (b) **durée** : `onDurationChange` (gardé, affine). ⚠️ L'API AnimeThemes n'expose PAS de durée ; le « un peu trop long » restant = silence en fin de fichier (non résoluble sans analyse audio).
- **Stats — logique** : `caught up` (via `isAwaitingNextSeason`) **séparé** de `in progress` (« 1 in progress · 3 caught up »). 28 tests verts.
- **Responsive 4-devices** (le trou = **iPad portrait** lumpé avec mobile) :
  - Rails **MediaCarousel + For You** : `w-[42%]` → `md:w-[28%]` (posters plus denses sur iPad, plus géants) + skeletons.
  - **More Like This** → pattern **Watch History** : rail scroll partout, rung fixe `--poster-lg`/`--rail-peek` (plus de grille figée). Seul poster hors-famille, corrigé.
  - **Stats** : 3 graphes en échelle `lg:grid-cols-2 xl:grid-cols-3` (Rating span pleine largeur sur iPad paysage — plus de légende tronquée) ; **AchievementGrid** (partagé) `lg:grid-cols-4 xl:grid-cols-6` (4 tuiles iPad, 6 desktop) ; skeletons alignés.
- 📌 **Backloggé PRIORITÉ** (`modules/watching/backlog.md`) : **attribution d'année d'une saison ré-ouverte/multi-années** (Bleach : saison finie 2023 → +10 ép 2026 → les ép 2026 restent en 2023, absent de Stats 2026). Racine = le modèle stampe l'année **au niveau saison, pas épisode** ; le `+1` mid-saison date le viewing sans re-stamper (par design). Un `+1` qui écraserait l'année = bug inverse. Vrai chantier (attribution par cour/date), pas un patch.

**⏭️ Prochain (plan owner) : discuter GOALS** (le module qu'il va construire) → **audit simple** → **Sport** (la vraie passe craft, 4→9).

---

## 🌐 SESSION 29/07 — MIGRATION CLOUDFLARE + LOT POLISH WATCHING (tout commité)

### 🔴 Prod injoignable depuis Orange Fibre → Cloudflare devant Vercel
- **Symptôme :** `hegon.fr` = `ERR_CONNECTION_TIMED_OUT` **sur le WiFi Orange Fibre uniquement** (marchait en 5G mobile, sur `hegon.vercel.app`, en localhost). Cause = **trou de routage Orange↔nouvelles plages IP Vercel** (`64.29.17.0/24`, `216.198.79.0/24`). **Rien à voir avec le code.**
- **Fix permanent :** **Cloudflare en proxy devant `hegon.fr`** (registrar reste IONOS ; nameservers `graham/opal.ns.cloudflare.com`, SSL/TLS **Full**, records migrés — dont **2 DKIM `s1/s2-ionos._domainkey` ratés à l'import auto**, rajoutés à la main). Zone **Active**. Détails → [[infra-dns-cloudflare-in-front-of-vercel]].
- ⚠️ **Mauvais diagnostic assumé :** j'avais d'abord accusé l'auth (`getClaims → getUser` sur middleware + 7 routes API + 2 pages) → **reverté**, `getClaims` restauré (c'était le réseau, pas l'auth). Leçon : « inaccessible » qui dépend du **réseau** = DNS/FAI, jamais le code.
- **`useInitOrg`** durci en passant : l'`AbortError`/steal du verrou Supabase → 1 retry silencieux, plus de toast « Please refresh ».

### ✅ Lot polish Watching (29/07, commité)
- **Hover iPad** — variante `can-hover` (`@media (hover: hover)`) dans `globals.css` ; les 5 usages (MediaCarousel, Episodes, LibraryCard, ListDetail ×2) passent de `sm:` → `can-hover:`. Boutons d'action **toujours visibles sur tactile** (iPad/mobile), révélés au survol **seulement sur souris**. Le `sm:` (largeur) était un mauvais proxy pour « a une souris ».
- **ThemePlayer** (✅ **VALIDÉ iPhone**) — le **mini-player** affiche le **restant qui décrémente** (avant : total figé, incohérent avec le grand player). Sur **iOS**, Safari mé-lit la durée Ogg/WebM d'AnimeThemes (garbage, « 400 min ») → on **cache le total/restant** (juste l'écoulé qui défile), **le son joue**. La vraie durée iOS est **irrécupérable sans transcodage** (infra, abandonné). ⚠️ Un 1er fix par seek cassait la lecture (seek→`ended`→skip) → reverté.
- **OP/ED — vérif d'identité** (`searchAnimeThemes`) — la recherche AnimeThemes renvoyait un look-alike pour un titre absent (Idaten Jump → « Heion Sedai no Idaten-tachi »). On **vérifie nom + synonymes** : titre absent → **rien** (pas de faux) ; cross-langue OK via synonymes (Attack on Titan → Shingeki no Kyojin). ⚠️ Compromis : un anime légitime dont le titre ne matche aucun nom/synonyme n'affiche plus de thèmes (rare).
- **Posters card ListDetail** — la grille (vue card) utilise enfin **la même échelle de colonnes que Library** (`grid-cols-3 sm:4 md:6 lg:8 xl:10`, gap-3) → posters **même taille** (avant 5/8/10 = plus petits). Poster lui-même déjà identique.

### ✅ Points « encore ouvert » du 29/07 — TOUS résolus le 30/07 (voir section du haut)
- **#5 Trailer 240p** → résolu (tri par `size`).  **#6 Don't Miss iPad** → résolu (pointeur + grille de 6).
- **Boutons hover radius** → tranché : `rounded-full` gardé, inset unifié à 8px (`OVERLAY_CLUSTER`).

---

## 🌙 POLISH UI WATCHING (post-clôture) — TOUT COMMITÉ & POUSSÉ (au 28/07)

> Watching était clos (gate 91 %). L'owner a rouvert une liste de petits chantiers UI, un par un.

### ✅ COMMITÉ & VALIDÉ
- **poster-xs** (`55a9423`) — radius via token `--radius-thumb`, `MediaRow` = LA row poster-xs unique, 6 sites.
- **Achievements** (`c5836de`) — 12 binaires 1 niveau, 6/ligne, sans glow, calibrés (paliers Bronze/
  Silver/Gold ABANDONNÉS, infra `AchievementTiers` retirée). Backlog : achievements **cliquables →
  panel éditable + ajout depuis l'UI** (codés en dur `computeStats.ts`).
- **Rang person page** (`0f406f9`) — règle `hasWatchTime`, un seul nombre vérifiable (journey « Watched »
  = rang, compte affiché à côté de « tied with »), **casting COMPLET** (plus de cap — Truman #78/80 dans
  Oppenheimer + backfill `scripts/backfill-cast.mjs`), **split acteurs live / voice actors** par type
  (anime → voix ; Rikiya Koyama #1 voix, Brad Pitt #2 acteur). `getPeopleCounts` → `{actors, voiceActors,
  directing}`, la page place la personne dans son seau dominant.
- **QuickAddPanel** (`5d29ca9`, `f6eae7f`) — panneau glissant d'ajout rapide, câblé PARTOUT :
  search-first (héros), destinations en **onglets-texte discrets**, **priorité contextuelle** (Want-to-
  watch seul, rouge/ambre/gris), **recherches récentes** (localStorage), **owned-aware**, **reste
  ouvert** (bulk), **scoping par type de page** (Animes ajoute des animes ; Library = tous types). Câblé
  au « + » **Library + 3 sections** ; **Top 10 = mode finder** (pick → fiche pour ranger). Hook
  `useQuickAdd` (fetch bundle+credits+cours → payload discover → `useAddMedia`). **Éligibilité Top 10
  corrigée** sur la fiche : `watched||in_progress||paused||dropped` (tout sauf want-to-watch) — **dropped
  INCLUS** (Prison Break après S4, True Detective S1 = arrêts satisfaits aimés ; avant `media.watched`
  seul cassait caught-up + in-progress).
- **AddMediaModal RETIRÉ** (`6d8cd2d`) — recos (DontMiss/ForYou/MoreLikeThis/discover) + crédits person
  **routés vers discover** ; modal + `HowFarDidYouGet` supprimés ; exports morts nettoyés
  (`getTakenPriorities`, `getExistingMediaEntry`, `TmdbModalResult`). **Gardé `uploadCustomPoster`** (la
  galerie en aura besoin). Le contexte Watching expose `openTitle(item)` (→ discover) au lieu de
  `openModalWithItem`.
- **Menu épisode refait** (commité, VALIDÉ) — le « … » d'un épisode regardé est **visible sur mobile**
  (plus de hover-only) ; menu en **Popover** : ligne « Best episode » (star **pleine or**, distinction
  au texte) + rating = **le Select de valeurs de Watch History** (`—` + 10→5, le `—` fait le clear, pas
  de bouton Clear séparé — l'ancien slider RatingPicker abandonné ici). Width `w-56`. **Menu décorrélé
  du scale** : sorti du div qui zoome (frère `z-20`) → au hover l'image scale, le menu reste fixe et le
  popover Radix ne dérive plus (comportement de tous les autres menus).
- **Menus poster mobiles** (commité, **VALIDÉ EN PROD**) — `MediaActionMenu` (carousel) et `LibraryCard`
  ne débordent plus à gauche sur le 1er item (clamp horizontal du portal, 8px des deux bords) ; le
  bouton « … » de Library est **visible au touch** (`opacity-100 sm:opacity-0 sm:group-hover`).
- **Stats** (commité) — Year Wrapped affiche enfin le **poster de cour AniList** (l'allowlist de
  `/api/proxy-image` bloquait `*.anilist.co` → image blanche ; suffixe ajouté) ; **heures arrondies**
  (`171.566h` → `171.6h`).

### 🔴 HOTFIX PROD (27/07, commité `next.config.ts`) — l'optimiseur Vercel brûlait le quota
- **Symptôme prod (owner) :** beaucoup d'images ne chargent pas → `GET /_next/image… 402 (Payment
  Required)`. Le quota d'**Image Optimization** de Vercel était épuisé.
- **Cause chez nous :** un changement du 21/07 avait retiré le `unoptimized` par-carte pour gagner
  l'AVIF/WebP → toutes les `<Image>` repassaient par `/_next/image`, ce qui a fait exploser le quota.
- **Correctif :** `images.unoptimized: true` **global** dans `next.config.ts` (`formats`/`minimumCacheTTL`
  retirés, morts sans optimiseur). On route AUTOUR de l'optimiseur : chaque image est déjà dimensionnée
  à la source (`tmdb-image.ts` demande la variante TMDB exacte, uploads downscalés). **On perd** juste la
  conversion de format (marginal) ; **on garde** taille + vitesse.
- 🔑 **Leçon :** faire repasser des images déjà dimensionnées par l'optimiseur Vercel = coût pur (quota)
  pour un gain marginal. Le vrai levier format = un redimensionneur hors-Vercel (Cloudflare Images…),
  c'est un projet à part (voir roadmap images ci-dessous).

### ✅ GALERIE + CAUGHT-UP + FIXES (session 27-28/07, COMMITÉ & POUSSÉ)
- **Galerie d'artwork — en SLIDING PANEL** (la route `/{id}/images` a été **supprimée**). `SlidingPanel`
  gagne `width="gallery"` (640px = 4 posters / 2 backdrops) et **se portale dans `document.body`** — fix
  d'un vrai bug : l'overlay se recalait sur un ancêtre transformé par framer-motion → bande basse
  cliquable ; corrigé pour **tous** les panels. `ImageGallery` : onglets Posters/Backdrops, « Current »
  = ring teal, hover « Set as… », **Upload** posters (`uploadCustomPoster`), **limite 24**.
  `include_image_language=en,null` (sinon seulement les images anglaises = titre incrusté ; les
  backdrops **textless** remontent maintenant) + **dedup** par `file_path`. Backend :
  `poster_url`/`backdrop_url` déclarés dans `updateMediaSchema` (sinon strippés), `getMediaImages` +
  `useMediaImages` sous `TMDB_KEYS`. Entrée = icône Images sur le poster du hero. **Temps réel** : un
  edit d'artwork invalide `stats` + `by-person` (`useUpdateMedia`). **2 décisions non tranchées** (non
  bloquantes) : upload backdrop aussi ? · bloquer le set-as sur les animes ?
- **Menus épisode + Watch History** — ferment au **scroll** (comme le menu poster) + look de
  `MediaActionMenu` (surface-3, border-default, rows px-3 py-2, icônes 13).
- **caught-up → Last Watched** — ✅ **c'est l'ancien vnext « caught-up PAR SAISON », FAIT.** Un titre en
  attente de sa **prochaine saison** quitte In Progress → Last Watched (badge « Caught up » teal), et y
  revient seul à la sortie d'un ép. Clé (insight owner) : `isAwaitingNextSeason` = caught-up **ET**
  `caughtUpOn === "season"` → une série qui sort **chaque semaine** (Bleach) **reste In Progress**. Pas
  de « 2 years ago » sur un caught-up (on masque `watchedAgo`, basé sur un `watched_at` périmé).
- **Heros plus nets** (`283a351`) — backdrop desktop `original`, poster/profil `dpr 3` (fiche + person).
  ⚠️ Le backdrop desktop `original` a un léger compromis LCP ; si trop lent, revert **ce point** en `w1280`.
- **ThemePlayer** — thumb de seek draggable façon Spotify sur les 2 barres. Commité & validé.
- **`useInitOrg`** (hors Watching) — le verrou d'auth Supabase transitoire (`AbortError`/steal) ne
  déclenche plus le toast « Please refresh » : 1 retry silencieux, `console.warn` si ça persiste.

### 🖼️ Roadmap IMAGES — reste le projet à part
- **📥 BACKLOG — Cloudflare Images** (ou équiv.) devant le domaine : AVIF/WebP partout sans toucher le
  quota Vercel → remonter la qualité des **rails** sans payer la vitesse. Infra, pas une ligne. (Le
  « bump hero » 1+2 est fait et commité.)

### ⏭️ Encore ouvert
- **Topbar reveal-au-scroll** : testé le 28/07, **ABANDONNÉ** (owner : « aucune utilité »). Ne pas reproposer.
- Rien d'autre d'actif : le module reste **clos (gate 91 %)**. Backlog parké → `hq/modules/watching/backlog.md`.

---

## 🎯 LA CLÔTURE DE WATCHING — la seule liste en cours

> **Règle owner : on ne sort pas de cette liste.** Elle existe parce que chaque plan se faisait
> dérailler. Ce qui surgit et n'est pas bloquant va au BACKLOG, pas dans le fil.
> ~~**Sport ne démarre que quand Fable a dit « il ne reste rien, Watching ≥ 90 % ».**~~
> ✅ **Gate passé le 24/07 : 91 %.** La condition devient : **Sport démarre quand les 3 correctifs
> issus du gate sont faits** (section « 🔧 LES 3 CORRECTIFS »).
> **⚠️ Après cette clôture, on NE REVIENT PLUS sur Watching** — sauf nouvelle feature ou modification
> d'une existante. D'où le tri systématique « corriger maintenant » vs « backlog ».

### ✅ SESSION DU 23/07 — FAIT ET COMMITÉ (les 3 audits Watching sont passés + correctifs appliqués)
- **Skeletons** (Fable puis Opus) : flash du mauvais skeleton résolu (suppression de
  `watching/loading.tsx`, chaque route a sa frontière) · Person hero desktop réparé · Episudes et
  My Take **allégés** (barres plus courtes que la ligne, hiérarchie par la hauteur, pas l'opacité).
- **AUDIT 1 MÉTIER (Fable)** → 2 récidives corrigées : Discover applique enfin la lentille des cours
  (`shouldOverlay`) ; `library/page.tsx` importe `LIBRARY_COLUMNS` au lieu de recopier (avait déjà
  dérivé de 5 colonnes). Reste du module jugé sain.
- **AUDIT 2 DESIGN SYSTEM (Fable)** → verre sur artwork figé remplacé par `on-artwork` (8 sites) ·
  genres = **une seule** variante `Badge overlay` en pilule, poids réglé pour ne jamais dépasser le
  titre (0,45) · échelle d'affiches documentée dans `globals.css` · radius : `xs`→`chip`, le reste
  →`tile`, 0 radius brut · scale de hover ne déborde plus.
- **AUDIT 3 RAPIDITÉ (Fable)** → 3 gains : poster Library avec état de chargement (grille figée →
  pulsante) · `getClaims()` sur `library`+`lists` (78 ms → 0,5 ms) · `theme_favorites` **5 fetch → 0**
  sur les pages sans thème.
- **Remarques UI/UX owner (toutes validées à l'écran) :** Don't Miss refait — carte ouverte à largeur
  fixe (le contenu ne se recompose plus pendant l'ouverture), courbe iOS, **cartes = leurs affiches**
  (0 bande morte de 1536px à 2560px), animation portée par le flex-basis (plus de saut) · priorité =
  marque-page coloré (rouge/ambre/gris, même code que la modale d'ajout) · sélecteur de rang **Top 10
  sur la fiche** (swap, pas cascade) qui retire l'exception de la modale · rang affiché en sur-titre
  du hero (teal = c'est toi) · stat de rating corrigé (les égalités comptent, phrase qui se retourne
  dans le bas de classement) · **navigation clavier** dans la recherche globale (↓↑ + Entrée).
- **NETTOYAGE** : 20 fichiers morts supprimés · 3 deps retirées + 3 radix déclarées (ne marchaient que
  par transitivité de `cmdk`) · `scripts/.shots` **−438 MB** · `proxy-image` exige un appelant
  authentifié · `cv`+`linkedin` sortis vers `personal/` (gitignoré).
- **`hq/` RESTRUCTURÉ** : `rules/` (ne périme pas) · `modules/<m>/{architecture,decisions,backlog,vnext}`
  + `_template/` · `log/` (journal par période) · `archive/`. Audit de contenu par Fable : constitution
  d'archi remise en accord avec le code, ~15 réfs mortes corrigées, `decisions.md`+`backlog.md` écrits.
- **MÉMOIRE dégraissée** : 88 → **42 vivants**, 46 archivés (dont 19 qui mentaient « NON COMMITÉ »).

### ✅ SESSION DU 24/07
- **🐛 Bug carousel Don't Miss — CAUSE TROUVÉE ET CORRIGÉE** (détail au point 0 ci-dessous).
  ⚠️ **Vérifié en build local seulement — le déploiement doit confirmer.**
- **❌ Mon diagnostic du 23/07 au soir était un FAUX POSITIF** — j'avais mesuré des éléments
  `display:none` (qui renvoient `top=0`). **Leçon : filtrer `width>0 && height>0` avant toute mesure
  de position.**
- **FLUX DISCOVER refait** (décision owner, après mesure) :
  · **Plus de modal pour les 3 actions** — le titre est déjà choisi, il n'y a rien à demander.
    *Want to watch* → medium · *Start watching* → **S1E1** ("start" veut dire commencer ; la
    StatusCard corrige) · *Mark as watched* → **dernier épisode DIFFUSÉ**.
  · **Le blocage « série en cours » est LEVÉ, et renommé** : le bouton dit **« I'm caught up »**.
    L'ancienne règle bloquait l'ACTION alors que le défaut d'origine était la PHRASE. `addStatusPatch`
    dérive le statut depuis la position → une série en cours atterrit *caught up*, jamais *watched*.
    **2 tests inversés** (ils verrouillaient l'ancien blocage).
  · **Navigation instantanée** : on utilise l'id que la mutation RETOURNE déjà, au lieu de le jeter et
    d'attendre que `useOwnedMediaId` le redécouvre. **Mesuré : ~1650 ms de détection → 0.** Test réel :
    aucun modal, fiche atteinte en **1072 ms**, titre peint au même instant (cache semé).
  · **`season_aired` ajouté à `mapTmdbDetails`** — il manquait à la ligne virtuelle, donc discover
    aurait raisonné depuis l'ANNONCÉ. La lentille (`buildMediaView`) est branchée sur ce chemin pour
    que l'année d'un anime à cours aille dans `cour_years`.
  · **Code mort supprimé** : `openAdd`, `addContext`, `selfItem`, `closeAdd`, `tmdbPathFromUrl`.
    Le modal ne sert plus qu'aux **recommandations** (et reste intact sur ses 5 autres surfaces).
- **SKELETON DISCOVER — pari inversé.** Il dessinait la forme POSSÉDÉE tant que la possession était
  inconnue, au nom du « superset ». Or sur cette route le cas majoritaire est **non possédé** :
  mesuré **619 → 879 px** de saut (+260) dans la colonne droite, à chaque chargement. Désormais
  l'inconnu dessine la forme **invité** ; le lien direct vers un titre possédé garde la forme fiche.
  Vérifié : une seule forme (85 pulsations constantes), et la page invité n'apparaît **jamais** sur
  un titre possédé. ⚠️ Reste ~168 px d'écart skeleton↔page finale (fidélité, pas changement de forme).
- **`CLAUDE.md` corrigé (4 points)** : police **Inter** (disait Geist) · **le pattern de card ne montre
  plus de hairline décoratif** (il enseignait ce que la préférence owner interdit) · radius = tokens ·
  **⚠️ `SUPABASE_SERVICE_ROLE_KEY` : le nom couvre DEUX valeurs** — auto-injectée dans les edge
  functions = **morte**, mais celle de **Vercel/.env.local est la VRAIE clé** posée à la main par
  l'owner. Les 2 routes tennis qui la lisent **fonctionnent** ; un audit les avait signalées à tort.
- **🏁 GATE FINAL PASSÉ — 91 %** (Fable, périmètre complet + `AddMediaModal` nommé suspect n°1).
  3 findings, **aucun bloquant**, tous re-vérifiés par Opus dans le code → « 🔧 LES 3 CORRECTIFS ».
  **Ce que le gate CONFIRME comme sain — à ne pas casser dans Sport :** les 5 invariants d'archi
  tiennent **dans le code, pas dans le commentaire** (aucun `watched: true` assemblé à la main dans
  tout le module ; `findOwnedMediaId` sans `.eq("type")` ; garde ESLint `no-restricted-syntax` sur
  les coordonnées brutes bien en `error`, exceptions toutes justifiées in-file) · **0 radius brut**
  sur une vraie surface · **0 hairline décoratif** sur 15+ écrans · `extendTailwindMerge` déclare
  bien **tous** les tokens custom utilisés · grammaire des marques cohérente hub/library/fiche/stats ·
  mobile 390 px propre partout. **`AddMediaModal` méritait son statut de suspect : le seul vrai trou
  d'écriture du module y était.**
  🔎 **Il a écarté 2 faux positifs tout seul** (une fiche « introuvable » = RLS qui fonctionne sur un
  id d'un autre compte · 3 posters « manquants » = course d'affichage, `img.complete` vrai 1 s après).
  Le réflexe `width>0 && height>0` a été transmis et appliqué.

### ⏭️ RESTE
0. **✅ 24/07 — CORRIGÉ ET COMMITÉ (`ef0b740`) : DÉPENDANCE CIRCULAIRE. Reste à CONFIRMER EN PROD.**
   **Le symptôme réel** (photo de l'owner, en PROD sur `/animes`) : le titre et le sous-titre de
   Don't Miss s'affichent, **mais pas ses cartes** — puis elles arrivent en retard. Dev OK, prod KO.
   **La cause :** hier j'ai exporté `ROW_VARS` depuis `DontMissSectionClient`, que `WatchingSkeletons`
   importait — or `DontMissSectionClient` importe `DontMissSkeleton` de `WatchingSkeletons`.
   **Cycle.** Dev le tolère ; le bundle de prod évalue dans un autre ordre ⇒ `ROW_VARS === undefined`
   ⇒ `className="flex gap-4 undefined"` ⇒ **ni `h-60` ni `--dm-poster`** ⇒ cartes sans largeur ni
   hauteur. La VRAIE section, elle, lisait la constante depuis son propre fichier → elle s'affichait
   normalement une fois les données là, d'où la « latence » ressentie.
   **Correctif :** `ROW_VARS` sort dans **`lib/dont-miss-layout.ts`** (module neutre, sans import) —
   une constante partagée par deux modules n'appartient à aucun des deux. Cycle éliminé (0 import
   croisé), `tsc` 0, `eslint` 0.
   **Vérifié en BUILD DE PRODUCTION** (`next start` sur :3100) : rangée **240 px**, cartes
   **586×240 + 5× 160×240** — les cartes existent.
   ⚠️ **Ce qui n'a PAS été fait : la contre-épreuve.** J'allais rebuilder SANS le fix pour prouver
   le lien de causalité ; interrompu. Le cycle est un fait objectif et sa disparition est vérifiable,
   mais **le lien cycle → symptôme reste une déduction, pas un A/B**. À refaire si le bug persiste
   après déploiement.
   🔑 **Leçon durable : un `export const` ajouté "juste pour partager" peut créer un cycle qui ne
   casse QU'EN PROD.** Vérifier le sens des imports avant d'exporter depuis un composant.

   ~~À REPRENDRE EN PREMIER — flash carousel en haut de Don't Miss.~~
   Signalé par l'owner (23/07 soir) : **à la PREMIÈRE visite seulement**, pendant que Don't Miss
   charge, un skeleton de carousel s'affiche en haut de la page à sa place.
   ❌ **MA « REPRODUCTION » DU 23/07 AU SOIR ÉTAIT UN FAUX POSITIF — corrigé le 24/07.**
   J'avais annoncé un `aspect-video` à `top=0` au-dessus de Don't Miss. C'était une **erreur de
   mesure** : un élément `display:none` renvoie un rect à `top=0`, et `CarouselSkeleton` a **15
   variantes de breakpoint cachées**. En ne comptant que les éléments VISIBLES : premier
   `aspect-video` à **479 px**, Don't Miss à **169 px** — l'ordre est correct, rien ne le survole.
   ⚠️ **Leçon : filtrer `width>0 && height>0` avant toute mesure de position.**

   **NON REPRODUIT après 3 méthodes** (chargement direct · navigation client depuis /lists ·
   `trending_cache` retardé de 5 s) : le `DontMissSkeleton` s'affiche correctement en haut dans les
   trois cas, captures à l'appui. L'ordre du DOM est bon (`WatchingPageWrapper` : DontMiss d'abord).
   **➡️ EN ATTENTE D'UNE PRÉCISION DE L'OWNER** (question posée le 24/07) : les cartes qu'il voit en
   haut sont-elles **larges/paysage 16:9** (= vrai `CarouselSkeleton`) ou **verticales/affiches**
   (= le `DontMissSkeleton` lui-même, qu'il prendrait pour un carousel) ? Cette réponse discrimine
   les deux causes possibles. ⚠️ **NE PAS bricoler** le code Don't Miss avant de savoir.
1. ~~**GATE FINAL — Fable**~~ ✅ **PASSÉ le 24/07 : 91 %.** Aucun BLOQUANT. `eslint --max-warnings 0`
   et `tsc --noEmit` verts, repo laissé intact, ~20 captures ouvertes (desktop + mobile 390 px).
   **Les 3 findings sont détaillés dans la section « 🔧 LES 3 CORRECTIFS » ci-dessous — c'est LA
   liste de demain.** Sport démarre quand elle est vide.
2. ~~**À corriger dans `CLAUDE.md`**~~ ✅ **FAIT le 24/07** (voir la session ci-dessus). Ancien texte :
   §1 dit « Geist »
   → c'est **Inter** (cf. `rules/performance.md` §5.5) · §6 bis dit que `SUPABASE_SERVICE_ROLE_KEY`
   contient le JWT mort → **faux, l'owner y a mis la vraie clé `hegon_server`** (le nom trompe, les 2
   routes tennis marchent) · §5 montre encore un hairline top décoratif (interdit) · §4 Next 15 → 16.

---

## 🔧 LE LOT DE CLÔTURE — ✅ **FAIT le 25/07, NON COMMITÉ** (12 fichiers, 4 nouveaux)

> **Tout ce qui suit est appliqué.** `vitest` **276/276** · `tsc --noEmit` **0** · `eslint
> --max-warnings 0` (la porte du pré-commit) **0**. HEAD toujours `9533f4d` — **rien n'est commité**.
> 13 fichiers touchés, 4 nouveaux.
>
> **Vérifié en EXÉCUTANT :** ⓪ de bout en bout (base + chiffres réels) · ⑤ par **contre-épreuve**
> (les 2 lignes remises dans leur état d'avant → exactement 3 tests rouges, puis restaurées) ·
> ②③ + RankMark à l'écran, captures ouvertes.
>
> **① : la règle est extraite et TESTÉE, le clic ne l'est pas.** `claimableEpisodes` /
> `claimableSeasons` vivent dans `lib/media-view.ts` avec 6 tests (dont le cas HotD : 8 annoncés,
> 5 diffusés → 5). Ce qui n'a pas été fait, c'est de voir « max 5 » à l'écran : **5 tentatives
> perdues sur les sélecteurs**, arrêt volontaire (règle « ne pas se battre avec le harness »).
> 📍 **La porte est `/perso/watching/tv-shows` → rangée « In Progress » → bouton `Add`**
> (`WatchingClient.tsx:71`). Celle de la Library est `listContext="library"` et ouvre en mode FILM :
> c'est pour ça que la recherche « House of the Dragon » n'y renvoyait rien.
>
> **RankMark : fait (demandé par l'owner le 25/07).** Et pas seulement la pastille — la carte de
> `ListDetail` passe sur la grammaire complète : `OVERLAY_CLUSTER` à gauche (identité = `RankMark`,
> le même lockup que le rail Top 10 et le hero) et à droite (actions, `OVERLAY_CIRCLE`). Les deux
> clusters s'alignent **par construction** au lieu de deux `top-*` qu'il fallait garder d'accord.

### ⓪ 🔴 LE RUNTIME NE SURVIT PAS À DISCOVER — *trouvé par l'OWNER le 25/07, mesuré, le plus grave*

**Le symptôme observé :** « mark as watched depuis discover → les heures n'augmentent pas dans Stats
pour une série, mais ça marche pour les animes et les films ».

**La cause, mesurée (sondage direct de TMDB, pas une déduction) :** `episode_run_time` est **VIDE**
pour **10 séries sur 10** (HotD, The Boys, Severance, TLOU, Silo, Breaking Bad, GoT, Dark, Squid
Game, Arcane). ⚠️ **Et ce n'est pas une affaire de type** — Attack on Titan, Demon Slayer et Blue
Lock renvoient `[]` aussi. Les animes de l'owner marchaient **par chance** (One Piece/JJK/Frieren ont
encore la valeur), ses films lisent `d.runtime` qui est vivant.

**Le vrai défaut est chez nous — DEUX cascades divergentes pour la même valeur :**
- `AddMediaModal.tsx:272-280` → **3 niveaux** : film `d.runtime` → moyenne d'`episode_run_time` →
  **`last_episode_to_air.runtime`**.
- `service.ts:726` (`mapTmdbDetails`, **le seul chemin de discover**) → **2 niveaux**, il s'arrête à
  `d.episode_run_time?.[0] ?? null`. **Pas de repli.**

**Vérifié sur la ligne réellement cassée (Elite, ajoutée le 24/07) :** `episode_run_time = []`,
`last_episode_to_air.runtime = 57`. → via discover **0,0 h** · via la modale **60,8 h**.

**Et rien ne répare :** ni le cron horaire (il ne touche que `status`/`season_*`), ni aucun script,
ni les seeds. `runtime = null` est **définitif** pour la ligne.
**Dégâts en base (mesurés) :** 1 seule ligne — **Elite**. Le reste de la bibliothèque est saine
parce qu'elle est passée par la modale. Le trou date de l'ouverture de discover à l'écriture (24/07).

**Le correctif : sortir la dérivation, ne pas la recopier.** Une fonction `runtimeFromTmdb(d, isFilm)`
partagée, consommée par `mapTmdbDetails` ET `AddMediaModal` — la valeur dérivée d'une API externe
appartient à **un** endroit. Recopier la cascade dans le mapper marcherait aujourd'hui et
redivergerait au prochain enrichissement. Puis **réparer Elite** (script ponctuel ou re-sync).
🔑 Voir [[reference_tmdb_episode_runtime_dead]]. **Pourquoi ça a survécu à 3 audits + le gate : un
`null` de dérivation ne lève rien, il se propage en `0`, et `0` ressemble à un chiffre.**

### ① La porte « In Progress » borne sur l'ANNONCÉ — *correctness, le plus important*

**Où :** `components/modals/AddMediaModal.tsx:483-488` (`getMaxEpisode`) et `:481` (`maxSeason`).

**Ce qui est faux :** `getMaxEpisode` lit `overlayCours[].episodes` ou
`selectedItem.seasons[].episode_count` — **l'annonce TMDB**. Jamais `addView.seasons[i].aired`, que
`lib/media-view.ts:42-43` désigne pourtant comme *« the only source for progress decisions »*.
La même modale fait pourtant les choses correctement **60 lignes plus haut** : `HowFarDidYouGet`
reçoit `addView.seasons.map(s => s.aired)` (`:946`). **Une porte sur quatre est passée à travers**
— les 3 autres (library / recentlyWatched / topTen) sont saines.

**La barrière ne rattrape rien :** le refinement Zod de `schemas/media.schema.ts:104-110` et
`:173-189` vérifie qu'une écriture de position **recalcule `caught_up_at`** — jamais qu'elle est
physiquement possible. Aucun contrôle `episode ≤ aired` nulle part.

**⚠️ LE VRAI SCÉNARIO DE CASSE — Fable l'avait sous-estimé.** Il a écrit « la fiche affichera
S3 E8 ». C'est vrai mais anodin. Le vrai problème est **différé** :
1. House of the Dragon S3 : 8 annoncés, 5 diffusés. Porte « In Progress » → saison 3 → épisode 8.
   Le label affiche même « max 8 », c'est-à-dire le mauvais chiffre. Aucune erreur. Submit.
2. `claimedStatus` (`watch-status.ts:293-294`) écrit `current_episode: 8` **tel quel**.
3. **Aujourd'hui c'est contenu** : `watchedCount` **clampe à la lecture**
   (`series-state.ts:88`, `Math.min(current_episode, aired[season-1])`) → tes 8 comptent pour 5.
4. **Demain le clamp se relâche tout seul.** Quand les épisodes 6, 7, 8 diffusent, `aired` monte,
   le `Math.min` ne mord plus → **tu es marqué comme ayant vu 3 épisodes que tu n'as jamais
   regardés**, et le badge « New episodes » ne se déclenchera jamais pour eux.

C'est exactement ce que `season_aired` existe pour empêcher, et ça rentre par la seule porte qui ne
le lit pas.

**Le correctif :** `getMaxEpisode(season)` lit `addView.seasons[season-1].aired`, avec **repli sur
l'annoncé uniquement si `aired` est absent ou 0** (un titre ajouté avant que la sync ait tourné ne
doit pas se retrouver borné à 0 — sinon on casse l'ajout au lieu de le protéger). Même logique pour
`maxSeason` : la dernière saison ayant `aired > 0`. Les messages d'erreur (`Max: N ep in Sx`)
suivent mécaniquement.
**Vérification :** ouvrir la modale sur une série en cours et constater que le max proposé = le
diffusé. Plus les tests de `watch-status.test.ts` qui doivent rester verts.

### ② Du verre sur une affiche figée — 4 fichiers, *règle explicite violée*

**Où :** `components/lists/ListDetail.tsx:663` (pastille de rang), `:679` et `:688` (boutons note /
supprimer au survol) · `components/lists/ListsClient.tsx:135` (badge « Ranked ») ·
`components/sections/MyThemesSectionClient.tsx:70` · `app/(main)/perso/watching/person/[id]/page.tsx:281`.

Tous : `rounded-full bg-black/70 backdrop-blur-sm` sur une image **qui ne bouge jamais**.

**Pourquoi c'est net et pas une opinion :** `shared/components/ui/button.tsx:40-45` **écrit lui-même
la règle** — le variant s'appelait `glass`, il a été renommé `overlay` parce que *« the glass was a
lie: a control on a STATIC image has nothing moving behind it to refract »*. Le primitif qui règle
le problème **existe déjà** (`on-artwork`, plat). Dans `ListsClient`, le bouton supprimer
(`:140-148`) utilise correctement `Button variant="overlay"` **cinq lignes** sous un badge qui blur
encore : les deux traitements coexistent sur la même image statique.
Rappel : 8 sites avaient déjà été traités le 23/07 — ce sont les 4 restants.

**Le correctif :** `Button variant="overlay"` pour les boutons, `on-artwork` (ou `Badge
variant="flag"`) pour les badges. Bénéfice secondaire réel : plus de `backdrop-blur` par carte dans
une grille qui scrolle.

**❓ LA SEULE QUESTION POUR L'OWNER :** dans `ListDetail`, `Marks.tsx` fournit **`RankMark`**, un
lockup plat déjà construit pour numéroter un rang. L'utiliser **change l'allure de la grille des
listes**. Le matériau est tranché par la règle ; **la forme du lockup, c'est l'owner qui décide** :
→ (a) garder la pastille actuelle, simplement mise à plat, ou (b) passer sur `RankMark` ?
**Ne pas trancher seul.**

### ③ Le bouton « Add » n'a aucun hover — *NIT, correctif d'une ligne*

**Où :** `components/modals/AddMediaModal.tsx:1089-1093`.

`variant="legacy"` + `style={{ backgroundColor: "var(--color-accent-watching)" }}`.
`button.tsx:12-18` qualifie lui-même `legacy` de *« a call site awaiting review »*. Mécanique du
défaut : `legacy` répond en `hover:bg-primary/90`, **une règle de classe** — le `backgroundColor`
inline la bat **toujours**. Le CTA principal du flux d'ajout est **le seul du module sans retour
visuel au survol**.

**Le correctif :** `variant="accent"` + `style={{ "--btn-accent": "var(--color-accent-watching)" }}`
— `accent` répond en `hover:brightness-110`, un `filter`, qui n'entre jamais en collision avec un
`background-color` inline. Le pattern est déjà utilisé pour ce même CTA teal dans
`LibraryClient.tsx:196` et `:225`.

### ⑤ AUDIT STATS (demandé par l'owner le 25/07) — 2 incohérences réelles + 1 manque

La logique n'a **pas** cassé sous les changements récents : `seasonContributions` lit bien
`season_aired`, le cas *caught-up* (watched=false + in_progress=true) contribue correctement ses
heures, et `countsTitle`/`contributionYears` partagent bien leur source. Mais :

- **`computeStats.ts:240` — le rewatch lit l'ANNONCÉ.**
  `totalEps = i.season_episodes?.reduce(...) || i.episodes` → **`season_episodes`**, alors que le
  même fichier a été corrigé pour ça 170 lignes plus haut (`:68`, avec un long commentaire sur « la
  plus vieille maladie du module »). Un rewatch d'une série en cours **facture des épisodes qui
  n'existent pas**. Doit lire `season_aired ?? season_episodes`, comme `seasonContributions`.
- **`computeStats.ts:343` — l'activité exclut `paused`/`dropped`.**
  `else if (item.watched || item.in_progress)`, alors que les heures ET les comptes passent par
  `hasWatchTime` (qui **inclut** paused/dropped). Le commentaire de `contributionYears` (`:97-99`)
  affirme pourtant « Same source as the hours, so counts/activity always agree ». Une série dropée
  compte ses heures et son titre, mais **produit zéro événement** dans la timeline.
- **`computeStats.ts` n'a AUCUN test.** 458 lignes d'arithmétique pure ; les 7 fichiers de test du
  module couvrent `lib/`, pas celui-ci. C'est précisément la surface où **un mauvais résultat
  ressemble encore à un résultat** (cf. le runtime ci-dessus). Candidat n°1 à des tests.

### ④ À TRANCHER — les ~168 px du skeleton discover

Fable les a **mesurés proprement** (délai réseau forcé à 1200 ms, `main.scrollHeight`, filtré
`width>0 && height>0`, 1440×900) : **film → 0 px · série → +17 px · anime → +113 px.**
**La cause anime est identifiée et précise :** `DiscoverSkeleton`
(`components/shared/WatchingSkeletons.tsx:542-576`) **ne réserve aucune place pour `<AnimeThemes>`**,
qui est pourtant monté **sans condition** dans la page chargée
(`discover/[type]/[tmdbId]/page.tsx:411`) et lance sa **propre** requête indépendante. Ce n'est donc
pas un écart diffus de fidélité : c'est **une section entière absente du skeleton**, dont la hauteur
grandit avec le nombre de thèmes trouvés.
**Décision owner attendue :** corriger dans la foulée des 3 autres, ou envoyer au backlog ?



---

## ⚠️ RÈGLES DE TRAVAIL QUI ONT COÛTÉ CHER (ne pas les réapprendre)

- **⛔ `npx supabase db push` : JAMAIS sur ce projet.** L'owner applique tout par **copier-coller dans
  l'éditeur SQL**. La table `schema_migrations` ne connaît **aucune** des 90 migrations ⇒ un push les
  **rejouerait toutes**. Les fichiers de `supabase/migrations/` sont une **archive, pas un mécanisme** :
  ni git ni Supabase ne savent ce qui est appliqué. **La seule source de vérité est la base** —
  vérifier une migration = interroger l'objet créé (`pg_constraint`…). Voir
  [[reference-migrations-applied-by-hand]].
- **Mesurer, pas déduire.** Le serveur de dev dérive (~1900 → ~4000 ms sous charge Playwright) ⇒ seuls
  valent les **A/B dos à dos dans la même exécution**, échauffés. Voir [[feedback-measure-speed-claims]].
- **Regarder, pas compter.** Un CLS bas ne prouve pas qu'un skeleton est fidèle ; des captures non
  ouvertes ne prouvent rien. **Fable regarde et audite, Opus mesure et structure** —
  [[feedback-fable-vs-opus-split]].
- **Répartition confirmée par les faits du 21-22/07 :** ce que j'ai raté (fidélité visuelle) Fable l'a
  trouvé en ouvrant les captures ; ce que j'ai trouvé (la lentille, les collisions d'ids TMDB, les
  cascades) demandait de la mesure et de la structure.
- **Ne pas se battre avec le harness.** 4 tentatives de sélecteurs perdues le 22/07 sur le repro du
  flash. Passer par les **réponses réseau de l'app** pour obtenir un id, pas par des sélecteurs CSS.
  Le harness : `node scripts/auth-setup.mjs` (session) · `MSYS_NO_PATHCONV=1 node scripts/shoot.mjs
  <route> out.png` · ⚠️ **la page scrolle dans `main`, pas dans `body`** (`--full` ne descend pas).

---

## 📥 BACKLOG WATCHING → **`hq/modules/watching/backlog.md`** (7 reports, chacun avec son déclencheur)
Les décisions datées (SSR refusé, Re:Zero, la lentille, le swap Top 10…) sont dans
`hq/modules/watching/decisions.md`. On ne duplique plus ici.

---

## 🗺️ ROADMAP DU MOIS — LE CAP
Cadre : *possiblement le dernier mois avant une pause budget → laisser HEGON dans un état parfait,
retour instantané.* Notes owner : Watching 9.5 · Goals/Tasks/Habits/Books 9 · Journal 8.5 ·
Dashboard 8 · **Sport ~4, c'est le chantier.**

1. **Clore Watching** — ✅ quasi fait : 3 audits + gate final passés (91 %), correctifs + features.
   **Reste les 3 correctifs sortis du gate**, puis c'est clos.
2. **SPORT = l'investissement** — le seul travail que l'owner ne peut pas faire seul : tester les vraies
   APIs, mesurer les rate-limits, proposer des features, retoucher les edge functions. **Passe craft
   complète.** 4 → 9.
3. **Audit Fable du socle partagé** — la colonne vertébrale **une seule fois** (RLS `org_isolation`,
   couche service, hooks TanStack) **+ Sport à part**. Pas 8 audits.
4. **Handoff = assurance-retour** — Settings (⚠️ à scoper : l'owner doit lister les douleurs concrètes)
   · vue observabilité **faisable** (ce qui est sauvé et où — pas de compteur de tokens live) ·
   **dégraisser les docs** (fait pour `now.md` ; `progress.md` = 72 KB, à faire).

**Au fil de l'eau :** workspace « HEGON Dev » dans Tasks (dogfood, pas de backfill) · post LinkedIn à
chaque module fini.

**⛔ COUPÉ (décidé) :** automatisation type Zapier · backfill Tasks · compteur de tokens en direct ·
thèmes séries/films (**point 9 : recherché et ABANDONNÉ, ne pas reposer la question** — aucune source
légale ne donne l'audio complet ; AnimeThemes n'a pas d'équivalent pour les B.O. commerciales).

*Distinction craft : **check craft** (revue rapide) pour un module ≥ 8.5 · **passe craft** (redesign)
réservée à Sport et à ce qui est sous 8.5.*
