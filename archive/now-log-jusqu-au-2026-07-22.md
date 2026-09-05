# HEGON — NOW (état courant)

> Le seul fichier d'état lu au démarrage. Détails/décisions = `MEMORY.md`.

## 🎯 LA LISTE DE CLÔTURE DE WATCHING — établie le 21/07 16h45, à tenir jusqu'à la fin
> **Règle owner : on ne sort pas de cette liste.** Elle existe parce que chaque plan se faisait
> dérailler en cours de route. Tout ce qui surgit et n'est pas bloquant va au § BACKLOG, pas dans le fil.
> **Sport ne démarre que quand Fable a dit « il ne reste rien, Watching ≥ 90 % ».**

### 🔴 VERDICT OWNER DU 21/07 SOIR — **IL NE COMMITE PAS.** 4 défauts ouverts.
Le travail skeletons de Fable est **un progrès, pas le niveau voulu**. Reste, dans ses mots :
1. **Le LOADING, pas les skeletons.** « parfois je vois les mauvais skeletons au mauvais endroit ».
   Hypothèse : c'est le **routage** qui est fautif (quel skeleton pour quelle route), pas le dessin.
   → **OPUS** (frontières de route : `loading.tsx` vs rendu conditionnel).
2. **Le hero de la page Person est nul** — il doit être **celui de la fiche détail**, adapté. → **FABLE**.
3. **Le skeleton Episodes est trop chargé** sous l'image (bloc texte trop dense vs le vrai). → **FABLE**.
4. **✅ FAIT (22/07) — le modal Add attendait un corps qu'il ne dessine pas.** `AddMediaModal`
   demandait `append_to_response=credits,aggregate_credits` **avant de pouvoir remplir le formulaire**
   (saisons, épisodes, chips de position). Or `cast`/`directors` ne sont **affichés nulle part** : ils
   sont lus **une seule fois**, dans le payload de submit. Et `aggregate_credits` sur une série longue
   est énorme — **Jujutsu Kaisen : 422 KB contre 5 KB** pour les détails seuls.
   **Correctif :** 2ᵉ requête (`/credits` ou `/aggregate_credits`) **tirée au même instant mais jamais
   attendue** ; le submit `await` la promesse (une saisie rapide ne peut pas enregistrer un cast vide).
   **Mesuré à travers `/api/tmdb`, A/B alterné, médiane de 7 :** JJK **285 → 228 ms (−20 %)** · Game of
   Thrones **361 → 247 ms (−32 %)** · poids **−99 %**. ⚠️ 2 titres sur 5 : la session du harness a expiré
   en cours de run — ce sont les deux plus lourds, donc le pire cas.
   ⚠️ **Vérifié au niveau données seulement** (les 2 endpoints répondent 200, les 2 dialectes de
   `character` se replient bien) — **pas cliqué dans l'app.** Vient de `c0a5312`, pas de cette session.

### ✅ VÉRIFIÉ CLOS LE 22/07 (contrôlé, pas déduit)
- **C3 tooltips natifs** — **RIEN À FAIRE** : zéro `title=` sur un élément DOM dans `modules/watching`
  (les 20 occurrences de `title="` sont toutes des **props de composants** — `SectionHeader`, `Panel`,
  titres de dialogues). La migration vers `Hint` était déjà complète.
- **A4 prefetch à l'intention** — **branché** : `usePrefetchMedia` est câblé dans `MediaCarousel`.
- **`CLAUDE.md` disait Next.js 15** → corrigé en **16.1.6**.

### ⏳ RESTE À LA CHARGE DE L'OWNER
- ✅ **MIGRATION APPLIQUÉE — CONFIRMÉ EN BASE LE 22/07.** `20260721_cour_columns_series_only.sql` :
  `select conname from pg_constraint …` renvoie bien `media_items_cour_columns_series_only`.
  **Rien à relancer.** (Opus ne pouvait pas vérifier d'ici : `supabase migration list` →
  `Cannot find project ref`, projet non lié en local — c'est l'owner qui a exécuté la requête.)
  ⚠️ **Seul reliquat possible, à contrôler un jour :** si elle a été appliquée **à la main dans
  l'éditeur SQL**, `supabase_migrations.schema_migrations` ne l'a pas enregistrée et le **prochain
  `db push` la rejouera** → erreur 42710 (`add constraint` déjà posé). Contrôle :
  `select version from supabase_migrations.schema_migrations order by version desc limit 5;`

---

## 🔴 22/07 — LE VRAI RESTANT DE WATCHING, À REPRENDRE ICI DEMAIN

### ① LE FLASH DU SKELETON HOME SUR PERSON — **NON RÉSOLU, ET C'EST STRUCTUREL**
Owner (22/07, après le fix de Fable) : *« pour page person oui côté skeleton, mais je vois pour un
instant le skeleton de la main page, donc il reste probablement toujours un problème dans loading »*.
**Le skeleton lui-même est réparé** (Fable : le bloc contenu n'existait que dans la branche `lg:hidden`
⇒ hero **vide** sur desktop). **Le flash est un AUTRE bug.**

**⛔ PISTE MORTE, NE PAS LA REPRENDRE :** `key={pathname}` dans `watching/layout.tsx` — **retiré le
21/07**, c'est écrit dans le fichier. Vérifié le 22/07.

**🔍 CE QUE LA VÉRIFICATION A SORTI À LA PLACE :** `watching/loading.tsx` **EST le skeleton de la HOME**
(Don't Miss + carousels). Or dans l'App Router un `loading.tsx` de segment n'est pas la frontière de sa
page, c'est **la frontière de tout ce qui vit en dessous**. Donc le skeleton spécifique de la home sert
de repli d'ancêtre à toutes les routes enfants. **4 routes n'ont AUCUN `loading.tsx` propre et retombent
dessus : `movies`, `animes`, `tv-shows`, `discover/[type]/[tmdbId]`.**

**Correctif proposé (PAS un petit fix — restructuration de routes, à faire à froid) :** regrouper la home
dans `watching/(home)/page.tsx` + son `loading.tsx` propre, et rendre `watching/loading.tsx` **neutre**
(ou le supprimer et donner à chaque route la sienne). ⚠️ 4 routes s'appuient aujourd'hui sur ce repli.

**❓ QUESTION POSÉE À L'OWNER, SANS RÉPONSE — À LUI DEMANDER AVANT DE CODER :**
**d'où clique-t-il pour arriver sur Person ?** (casting d'une fiche · home · recherche). **Le chemin
détermine quelle frontière déclenche.** Fable a reproduit la *fidélité* du skeleton, **pas la séquence de
navigation réelle** — c'est ça qu'il faut rejouer demain, en partant du vrai clic.

### ② BASE DE TESTS PAS VERTE AVANT 3 AUDITS
**245/247.** Les 2 échecs sont `journal-utils.test.ts` (`…` vs `...`), **antérieurs au 18/07, hors
Watching**. Opus allait les corriger, l'owner a interrompu (fin de session). **À faire avant les audits :
un audit part d'une base verte.**

### ③ TROUVAILLE — LA CONNAISSANCE DU PROJET N'EST DANS AUCUN DÉPÔT (mesuré le 22/07)
**`CLAUDE.md` gitignoré (ligne 44) · `hq/` gitignoré (ligne 48, 47 docs) · `supabase/*` gitignoré
(ligne 70) → 90 fichiers de migration sur le disque, 0 versionné.**
Si ce disque lâche : le code survit (GitHub), la base survit (Supabase), **tout ce qui EXPLIQUE le projet
disparaît**. Le cap du mois est « état parfait, retour instantané après la pause » — c'est exactement ce
que ça rend impossible. **Premier point du nettoyage `hq/`.**

### ⛔ NE JAMAIS LANCER `npx supabase db push` SUR CE PROJET (règle posée le 22/07)
**L'owner n'a JAMAIS fait de `db push` : il applique toujours par copier-coller dans l'éditeur SQL de
Supabase.** Donc `supabase_migrations.schema_migrations` **ne connaît aucune** des 90 migrations ⇒ un
`db push` ne « sauterait » rien, **il tenterait de rejouer les 90 fichiers**.
⚠️ **Opus a donné deux fois l'instruction inverse le 22/07 ("relancer est sans risque") — c'était FAUX**,
déduit du comportement nominal de l'outil sans vérifier le flux réel de l'owner. Instruction retirée.
**Conséquence :** les fichiers de `supabase/migrations/` sont une **archive**, pas un mécanisme ; rien
(ni git, ni Supabase) ne sait quelle migration est appliquée. **La seule source de vérité est la base.**
Vérifier une migration = interroger l'objet qu'elle crée (ex. `pg_constraint`), pas la liste des fichiers.

### 🗺️ LA SÉQUENCE DE CLÔTURE — arrêtée le 21/07 soir (owner + contre-proposition Opus acceptée)
**Ordre non négociable, il existe parce que les audits ne sont PAS indépendants :**
0. **Les 4 défauts ci-dessus → commit.** Un audit décrit un état ; l'arbre doit être figé d'abord.
1. **AUDIT MÉTIER** (Fable) — logique contradictoire, deux règles qui s'annulent, dead code, propreté.
   ⚠️ **Le dead code passe par `knip` AVANT** : « cet export est-il utilisé » est mécanique, faire
   raisonner un LLM dessus est du gaspillage. Fable ne tranche que les **cas ambigus**.
2. **AUDIT DESIGN SYSTEM** (Fable) — après le métier, qui aura supprimé du code.
3. **AUDIT RAPIDITÉ** (Fable pour le constat, Opus pour la mesure) — **EN DERNIER** : il mesure la
   **forme finale**. Le placer avant, c'est mesurer du code sur le point de changer.
4. **NETTOYAGE** — `shots/` (trop de captures) + `hq/` : ne garder que le courant, **archiver le
   consommé** (ou son résumé) dans `hq/archive/`. À la fin, **une seule fois**, parce que les audits
   produisent des documents. *(Exception : `now.md` est dégraissé AVANT — 503 lignes lues à chaque
   démarrage = une taxe sur toutes les étapes qui suivent.)*
5. **GATE FINAL** — Fable : « il ne reste rien, Watching ≥ 90 % ». Puis Sport.

**Règle de répartition posée par l'owner (« je te laisse décider c'est qui le meilleur ») :**
**FABLE = juger et REGARDER** (fidélité visuelle, audits — il a trouvé ce qu'Opus a raté parce qu'il
**ouvre les captures**) · **OPUS = mesurer, structurer, corriger** (A/B chiffré, archéologie git,
frontières de route, application des correctifs). **Un audit est une note sur le travail d'Opus ⇒
c'est Fable qui le fait. L'indépendance EST la valeur du gate.** Voir [[feedback-fable-vs-opus-split]].

### 1 — PERF
- [x] ~~**SSR des 3 pages grille**~~ → **⛔ DÉCIDÉ LE 21/07 : ON NE LE FAIT PAS. On reste en CSR.**
  **Le gain mesuré** : `/library` (SSR) 1165 ms de contenu / 4 requêtes client · `/movies` (CSR)
  1607 ms / 14 requêtes → **~440 ms, 27 %, sur la PREMIÈRE visite seulement** (les visites suivantes
  sont déjà rapides grâce aux caches). ⚠️ Comparaison imparfaite : library affiche 1 liste, movies 7.
  **Le coût, trouvé par Fable** : *aucune* fonction de lecture de `service.ts` ne peut tourner côté
  serveur — toutes utilisent le `createClient()` de `infrastructure/supabase/client.ts`, module
  `'use client'` sans session cookie. Importée telle quelle dans un Server Component, la RLS renvoie
  **zéro ligne sans erreur** ⇒ une page qui affiche « tu n'as rien » avant que le client ne monte.
  Il faudrait donc **une 2ᵉ implémentation des lectures** — la maladie que ce module a déjà eue 3 fois.
  **Rapport jugé mauvais : 440 ms de première visite contre une duplication permanente.**
  **🔑 SI ON Y REVIENT UN JOUR, LA BONNE FORME N'EST PAS UN FORK** : **injecter le client Supabase en
  paramètre** (`getMediaItems(client, userId, …)`) — une seule implémentation, deux appelants,
  mécanique et vérifiable par le compilateur. Déclencheur possible : ouverture des inscriptions, ou
  une page qui redevient lente. ⚠️ `client.ts` et `server.ts` restent INTOUCHABLES (CLAUDE.md §8) —
  on les *appelle*, on ne les modifie pas.
  **Corollaire à traiter un jour (petit)** : l'app résout l'auth de **3 façons** (`getUser` dans
  library, `getSession` dans `(main)/layout.tsx`, `getClaims` dans le middleware), aucune enveloppée
  dans `cache()`.
- [ ] **Images : 18 fichiers restants** avec `unoptimized` — fiche détail (`MediaHero`, `CastCrew`,
  `MoreLikeThis`), pages Person, `LibraryCard`, Lists, Stats, dashboard-os, Goals. Même helper
  `lib/tmdb-image.ts`, mécanique. (Les 3 pages grille sont faites : 22,4 MB → 0,62 MB.)
- [ ] **Posters uploadés** (bucket Supabase) : redimensionner côté client à l'upload (~780 px). Aucun
  helper d'URL ne peut les corriger — signalé par Fable.
- [ ] **A4 — ouverture instantanée d'une fiche** : prefetch à l'intention (survol/clic d'une carte).

### 2 — DETTES TECHNIQUES
- [ ] **B2** préremplissage `AddMediaModal` depuis la lentille (documenté in-file, demande un effet).
- [ ] **B3** contrainte type-conditionnelle : rien n'empêche d'écrire `cour_years` sur un film.

### 3 — EN ATTENTE D'UNE DÉCISION DE L'OWNER
- [ ] **C1** badge d'année **affiché** sous une règle, **éditable** sous une autre → 1 ligne, attend son go.
- [ ] **C3** les autres `title=` natifs → `Hint` (il devait les repérer à l'usage).

### 4 — CRAFT
- [ ] **D1 skeletons.** 🔔 **LUI DEMANDER D'ABORD : il a quelque chose à me dire sur cette partie.**
  Déjà instruit : **Stats = CLS 0,0739** (le plus haut des 9 pages, seul à approcher 0,1) ·
  **discover : 13 px résiduels** · `Lists` = 0 px, la référence.

### 5 — DOCS & PROCESS
- [ ] `CLAUDE.md` dit **Next.js 15**, c'est **16.1.6**.
- [ ] Nettoyage `hq/` : archiver les 4 docs d'audit vers `hq/archive/`.
- [ ] **Fable — la revue adversariale de `hq/watching-architecture.md` lancée le 20/07 : est-elle
  revenue ?** Question posée 3 fois, jamais répondue. Bloque le nettoyage ci-dessus.

### 6 — LE GATE FINAL (avant Sport)
- [ ] **Audit Fable de clôture** : « il ne reste rien, Watching est à ≥ 90 % ». Tant qu'il ne l'a pas
  dit, on ne passe pas à Sport.

### 📥 BACKLOG (hors périmètre de clôture — ne pas y toucher maintenant)
`theme_favorites` = 8 KB sur chaque page Watching (le `ThemePlayer` est dans le layout) · Re:Zero
cour ≠ saison (remède accepté : titre du cour dans le popover, zéro migration) · `series_facts`
(déclencheur : AniList v2 ou ouverture des inscriptions).

---

## 🔴 AU RÉVEIL, LIRE CECI D'ABORD — session du 21/07 (arrêt 2h40 du matin)

### ⛔ L'OWNER N'A RIEN TESTÉ. TOUT LE TRAVAIL DU 21/07 EST COMMITÉ MAIS NON VALIDÉ À L'USAGE.
Il a commité sur la foi de `tsc` + `eslint` + tests + mes mesures. **Aucun clic humain.** À faire tester
en priorité : la **recherche** (marqueur « In your library » + le clic doit mener direct à la fiche) ·
le **badge Now** sur HotD · l'**Undo** du « +1 » sur une carte · **Where to watch sur discover** ·
la **fiche** en général (j'ai touché `InList`, `QuickStats`, `StatusCard`, les hooks TMDB).

### ✅ FAIT ET COMMITÉ LE 21/07 (4 commits)
1. **Badge « Now »** → règle sortie en `isSeasonLive` (`lib/series-state.ts`), 6 tests. *Prouvé écran.*
2. **Undo restaure `last_watched_at`** (`undoPosition`) + `last_watched_at` ajouté à `SECTION_COLUMNS`.
   *Prouvé en base via l'UI : Widow's Bay restauré à l'identique.* ⚠️ piège trouvé : `undefined` (la
   surface ne sait pas) ≠ `null` (il n'y avait rien) — sinon l'Undo effaçait une vraie date.
3. **Recherche : la possession est résolue AU CLIC**, plus après navigation. Lecture groupée
   `findOwnedMediaIds` + marqueur « In your library ». **A/B dos à dos : 2 pages d'attente → 1,
   ~35-52% plus rapide** selon les runs. Règle de collision d'ids TMDB en `lib/possession.ts` (6 tests).
4. **TMDB : 6 appels → 1** (`getTitleBundle`, append_to_response) ; `useTmdbDetails` cesse de refetcher
   la même fiche nue ; providers **trimés aux régions lues** (49→22 KB) ; `PROVIDER_REGIONS` = MA, FR, US
   (GB retiré, ne servait à rien) ; **Where to watch ajouté sur discover** (extrait en composant partagé).
5. **Garde de coordonnées passé en `error`**, 26 sites annotés (11→3 sur `[id]/page.tsx` en rassemblant
   les lectures dans `storedPosition()`). **Lint : 155 → 0.** `.claude/**` sorti du périmètre ESLint.
6. **A2 entamé** : doublon `media_lists` supprimé (`InList` appelait 2 hooks pour la même chose) et
   3 requêtes remontées au niveau page → **17 → 16 appels**, la vague 2 fond dans la vague 1.

### ⏭️ REPRENDRE ICI DEMAIN
- **✅ A2 CLOS (21/07).** La cascade et le doublon étaient les vrais gains. Les `select=*` ont été
  **mesurés puis écartés** : 17,8 KB pour les 5, moins que le bundle TMDB de la même page ; la ligne de
  fiche fait 4,1 KB dont 46 % de `cast_members`, qu'on utilise. ⚠️ **Je m'étais trompé en écrivant que
  `CLAUDE.md` mentait** — l'affirmation est dans `hq/audit/db-schema-audit.md`, et elle qualifiait déjà
  `getMediaItemById` de « légitime ». La mesure lui a donné raison. Doc corrigée (inventaire complété).
  **Backlog, pas urgent :** `theme_favorites` = 8 KB sur *chaque* page Watching (le `ThemePlayer` est
  dans le layout, donc la playlist complète part même si tu ne l'ouvres jamais ; seul l'état des ♥ est
  nécessaire avant ça).
- Puis **A3** (`key={pathname}` = remount complet à chaque navigation — **décision motion à trancher
  par l'owner** : on garde le fondu ou pas) · **A4** (ouverture instantanée par prefetch à l'intention).
- Reste du plan Watching : B2 (préremplissage `AddMediaModal`), B3 (contrainte type sur `cour_years`),
  C1 (badge d'année affiché vs éditable — **1 ligne, attend son go**).
- **D1 — SKELETONS. 🔔 AVANT DE COMMENCER, DEMANDER À L'OWNER : il a quelque chose à me dire sur
  cette partie** (demandé le 21/07 : « rappelle-moi de te dire un truc lorsqu'on arrivera à la partie
  des skeletons »). **Ne pas démarrer D1 sans lui avoir posé la question.**
  Déjà instruit le 21/07 (audit de layout-shift sur les 9 pages Watching, aucune dans le rouge) :
  · **Stats = CLS 0,0739**, le plus haut, seul à approcher le seuil de 0,1 → à regarder en premier
  · **discover : 13px résiduels** (le rail Episodes, après être passé de 95px à 13px)
  · `Lists` est à 0px, c'est la référence · les autres pages dérivent en hauteur mais sous la ligne
  de flottaison (grandir ≠ sauter — c'est le CLS qui mesure le mal, pas la dérive de hauteur).
- **E : Fable est-elle revenue ?** Toujours sans réponse — bloque le nettoyage `hq/` (archiver les 4
  docs d'audit vers `hq/archive/`).

### ❓ QUESTION OUVERTE DE L'OWNER (21/07, 2h40) — À INSTRUIRE, NE PAS DEVINER
**« Le cast : plus rapide depuis notre base ou depuis l'API TMDB ? »** Née du constat que `cast_members`
est une des colonnes jsonb lourdes de `media_items`, et que la fiche fait `select=*`.
**Ce qu'on sait déjà :** la fiche lit le cast **depuis la DB** quand il est stocké (`hasStoredCast`) et
ne tombe sur TMDB (`useMediaCredits` → `credits,aggregate_credits`) que sinon · **mesuré le 21/07 :
`credits`+`aggregate_credits` = +165 KB** sur House of the Dragon, ce qui a justifié de les EXCLURE du
bundle TMDB · l'appel TMDB est **rate-limité par IP** chez nous, pas la DB.
**Comment trancher (méthode, pas opinion) :** mesurer les deux — poids réel de la colonne `cast_members`
en base vs poids de la réponse TMDB, ET le temps de réponse des deux, **dos à dos dans la même
exécution**. ⚠️ Ce n'est pas qu'une question de vitesse : la DB survit hors-ligne et sans quota, TMDB est
toujours à jour. **Ne pas répondre sans chiffres.**

### ⚠️ NOTE DE MÉTHODE (vraie leçon du 21/07)
**Le serveur de dev a dérivé de ~1900 à ~4000 ms** pendant la session à force de chargements Playwright.
⇒ **Les chiffres absolus ne sont PAS comparables d'une heure à l'autre.** Seuls valent les A/B **dos à
dos dans la même exécution** (rejouer l'« avant » en bloquant une requête plutôt qu'en revertant le code).
**Redémarrer `npm run dev` avant tout jugement au ressenti.** Voir [[feedback-measure-speed-claims]].

**Date de maj :** 2026-07-21

## 🗺️ ROADMAP DU MOIS (abonnement ~28 j restants, décidé le 17/07) — **LE CAP ACTUEL**
Cadre : *possiblement le dernier mois avant une pause budget → laisser HEGON dans un état parfait, retour instantané.* Notes owner : Watching 9.5 · Goals/Tasks/Habits/Books 9 · Journal 8.5 · Dashboard 8 · **Sport ~4 (le chantier)**.

**Vagues, par valeur décroissante (si les jours manquent, ils manquent sur le bas) :**
1. **Clore Watching** — remarques + propositions owner → **check craft** (léger, pas une passe) → post LinkedIn. But : 10/10. **EN COURS ⬇️**

### 🎬 V1 Watching — session 17-18/07 (avancement)
**Commité :** batch craft #1 — More Like This titre 1 ligne, cœur grand player OP/ED rouge, barre mini player blanche.
**Codé + lint/tests verts, À COMMITER par owner (2 cmds données) :**
- **Règle de statut (fix + 5 tests, 32/32 verts)** : `positionPatch` — atteindre le dernier ép. diffusé fait sortir de **paused → Caught up** ; **dropped survit** (décision de ne pas continuer, cas HotD vécu owner) ; complétion l'emporte. La seule ligne de comportement qui change, tout le reste était déjà juste.
- **Signal Fallout** (`SeasonHistoryStrip`) : saison diffusée en avance de la position → **œil barré** (pas vu, à réclamer) au lieu du **cadenas** (« interdit ») ni coche (« fait »). Cadenas gardé pour les saisons vraiment inatteignables. Copie popover « You haven't marked… » → « Mark it watched to rate this season ».
- **Point 2** : `title=` natif → `Hint` shadcn (strip).
- **Point 5** : tooltip age rating (`MediaDetails`) + dictionnaire US/UK/FR, curseur help.
- **Point 7** : genres carousel **et** Don't Miss → pastille douce du hero (au lieu de `Badge overlay` trop clair). ⚠️ décision DS : le hero devient le canon des genres, `overlay` inutilisée pour ça → à noter pour l'audit Fable.
**RESTE sur Watching :** points 2 (autres tooltips natifs — owner cherchera) + 7 déjà faits ; **8 skeletons au millimètre** (surtout hero Person) ; **10 recherche globale IMDb-style** (fin de vague, 1 post LinkedIn). Détail page : gel LEVÉ, mais valider chaque modif avant.

### 🔴 18/07 — POINT 10 CONSTRUIT PUIS **INVALIDÉ PAR LE TEST**. ⛔ NE RIEN COMMITER.
**Tout est dans l'arbre de travail, NON COMMITÉ, et doit y rester tant que ça ne marche pas.**

**Construit (Opus) :** `discover/[type]/[tmdbId]/page.tsx` (fiche complète d'un titre non possédé, bâtie sur un `WatchingMedia` « virtuel » via `getTmdbDetails`) · `WatchingSearch.tsx` (recherche catalogue TMDB dans le header, dropdown desktop / plein écran mobile) · prop `actions` sur `SectionHeader` partagé · `useTmdbDetails`, `useOwnedMediaId`, `tmdbPathFromUrl`, `tmdbResultType`, `findOwnedMediaId` · patch de l'overlay cours dans `AddMediaModal` · fix `<img src="">` dans `AddMediaModal`.

**❌ CE QUI NE MARCHE PAS (testé par l'owner) :** « Want to watch » et « Start watching » **n'atterrissent pas** dans le bon rail (invisible dans le carousel) · la StatusCard change « in a bad way » · **l'overlay AniList n'est appliqué ni dans la StatusCard ni dans les cartes du carousel** · **rechargement complet de page** alors que seule la StatusCard devrait changer · lenteur ressentie.

**⚠️ MA FAUTE, ET LA LEÇON EST DÉJÀ ÉCRITE DANS `CLAUDE.md` :** j'ai livré en vérifiant **types + lint** et annoncé « c'est prêt ». **« Posé » ≠ « fonctionne »** — je ne peux pas cliquer dans l'app, donc la discipline doit être : **construire par petits morceaux et faire valider chacun**, jamais livrer une feature entière comme prête.

**DEUX FAMILLES DE PROBLÈMES (ne pas confondre) :**
- **A — sémantique de l'ajout, mal comprise par moi.** `AddMediaModal` ~L101-103 : *« For a SERIES, the position is the only honest input — the list you came from no longer decides your status. »* ⇒ pour série/anime le statut vient de **`claimedStatus(media, position, stance)`**, PAS de `listContext`. Mes 3 boutons reposent sur une prémisse fausse.
- **B — faille STRUCTURELLE, antérieure à moi.** L'overlay de cours (`lib/anime-overlay.ts`) est appliqué **à la main** sur 2 surfaces (`SeasonHistoryStrip`, `Episodes`) et **oublié partout ailleurs** (StatusCard, cartes carousel, modal). **Aucun point d'application unique ⇒ toute nouvelle surface l'oublie par défaut.** C'est la maladie que `watch-status.ts` a guérie pour les STATUTS, encore vivante pour les COORDONNÉES.

**✅ AUDIT FABLE TERMINÉ ET VÉRIFIÉ PAR OPUS → `hq/audit/watching-architecture.md` (LE document, lire avant d'agir).**
**🔴 LA CAUSE RACINE, CONFIRMÉE PAR LECTURE DU CODE (antérieure à mon travail) :** `useAddMedia.ts:123` + `:132` — `claimedStatus` **retourne `null` quand `position` est nulle** (juste pour un UPDATE, **fatal pour un INSERT**), et la porte « In Progress » **ne collecte jamais `position`** ⇒ **la série s'insère avec TOUS les drapeaux de statut faux** = ligne « limbo », invisible dans tous les rails, StatusCard affiche « Want to Watch ». Mes 3 boutons n'ont fait que déclencher un bug existant.
**Autres causes confirmées :** `key={pathname}` dans `watching/layout.tsx` **remonte tout l'arbre** à chaque navigation (= le « rechargement complet ») · `useMediaItems` `gcTime: 5min` (⇒ skeleton après 5 min d'inactivité) · `useMediaItem` sans graine ni staleTime (⇒ fiche toujours à froid) · `library/page.tsx` a une **copie périmée** des colonnes. **Les skeletons ne sont PAS la cause** — c'est la fréquence à laquelle l'app décide de les montrer.
**➡️ ORDRE D'EXÉCUTION (plan §7 du doc) :** 1) verrou du statut à l'ajout (`insertMediaSchema` + `addStatusPatch`) — **vérifiable par tests unitaires purs, à faire AVANT tout** · 2) commiter le travail non commité avec 4 retouches · 3) perf quick wins · 4) la lentille `MediaView` (le fix structurel) · 5) `append_to_response`.

### ✅ ÉTAPE 1 — **VÉRIFIÉE DE BOUT EN BOUT (20/07, Opus a enfin l'accès app)**. Toujours **NON COMMITÉE**.
Verdict owner (19/07) : « les actions dans les médias ajoutés marchent bien ». Le bug limbo est **mort**.
**Accès app obtenu (20/07)** : session harness rafraîchie (`node scripts/auth-setup.mjs`) → Opus screenshote (`scripts/shoot.mjs`, `MSYS_NO_PATHCONV=1` sur Git Bash) **et lit/écrit la base** en réutilisant le JWT de la session (cookies SSR `sb-…-auth-token.0/.1`, base64, RLS respectée). ⚠️ la session expire (~10 j) → relancer `auth-setup.mjs` quand un screenshot retombe sur le login. ⚠️ `SUPABASE_SERVICE_ROLE_KEY` de `.env.local` est la clé MORTE (rotation) ; `HEGON_SECRET_KEY` **absente** de `.env.local` → pas de lecture service-role locale.
**Preuves réelles (pas déduites) :** œil barré fixé (screenshot HotD, S3=« Now ») · affichage années non-overlay OK · « +1 » piloté sur HotD → `caught_up_at` posé, `season_years` **inchangé** (garde anti-stamp d'une saison 5/8 diffusée), puis **restauré à l'identique** · **fix 1 prouvé** : ajout Andor via porte « In Progress » (S2E1) → `season_years:{"1":2026}`, `in_progress:true`, un seul statut, **pas de limbo**, puis **ligne supprimée**. Reste unitaire-seulement : fix 2 positif (même code de stamp que fix 1).

### 🟡 CE QUI A ÉTÉ TROUVÉ PENDANT SES TESTS — 4 correctifs, **1 validé écran, 3 NON TESTÉS**
**⛔ L'owner est reparti sans rien tester des 3 derniers. NE PAS COMMITER SANS SON TEST.**

1. **✅ VALIDÉ ÉCRAN — œil barré sur la saison en cours** (`SeasonHistoryStrip`). Ma faute, batch craft #1. Le masque ne connaissait que **2 états** (pas vue / finie) alors qu'il y en a **3** : *devant toi* = pas vue (à réclamer) · **sous toi = en cours** · *derrière toi* = vue. Le milieu tombait dans « pas vue » et sautait direct à « finie ». Fix = `unseenAhead = canJump && !editable && s !== currentSeason`. **Le clic reste** (« Watched through S » est légitime sur la saison en cours).
2. **🟡 NON TESTÉ — une porte sans date parle du PRÉSENT.** `addStatusPatch` ne stampait que si une date était fournie, et **seule la porte `library` en fournit une** ⇒ « Start watching » Blue Lock à S2E4 laissait S1 en placeholder « Year ». Incohérent avec le « +1 », qui stampe l'année courante. Désormais `claim.watchedAt ?? now`. ⚠️ **le test que j'avais écrit la veille verrouillait le mauvais comportement — inversé, pas contourné.**
3. **🟡 NON TESTÉ — la saison qu'on FINIT se date.** `positionPatch` stampait `seasonRange(from, season-1)` = ce qu'on **quitte**, jamais ce qu'on **termine**. Conséquence : une série sur laquelle on est **à jour** avait sa dernière saison éternellement vide (impossible à « quitter »). Ajout de `landed` (la saison courante si elle vient de devenir `isSeasonDatable`).
4. **🟡 NON TESTÉ — le cas d'étude de l'owner : le stamp qui ressuscite.** Blue Lock terminé → S2 mis à 2025 à la main → retour « through S1 » (S2 devient non vue) → re-marqué terminé → **2025 revenait**. Cause : `stampSeasons` **n'écrasait jamais**. **⚠️ Sa règle (« une saison non-vue perd tout ») a été REFUSÉE et il a accepté le contre-exemple** : S2 vue en 2019 et corrigée à la main serait réécrite en 2026 en finissant S5. **La vraie ligne de partage = RÉCLAMES-TU CETTE SAISON MAINTENANT ?** stamp derrière ta position → **protégé** · stamp devant ta position (déjà non lu par `reachedEntries`) → **reliquat, re-daté**. `stampSeasons(…, isStale?)` + `staleStamp(m) = !hasReachedSeason(m, s)`. **Rien n'est jamais supprimé** (une *correction* ne stampe pas ⇒ aller-retour par erreur = zéro perte). Les **2 cas sont testés côte à côte** pour qu'on ne répare pas l'un en cassant l'autre.

**Vert : `tsc` 0 · `eslint` 0 · 129/129 tests watching** (5 neufs). **Et ça ne prouve toujours rien pour 2-3-4.**
**À TESTER EN PRIORITÉ AU RETOUR :** ① « Start watching » Blue Lock S2E4 → S1 doit lire **2026** · ② terminer une saison au stepper → elle se date seule · ③ le scénario exact de l'owner (terminé → 2025 à la main → through S1 → re-terminé) → doit lire **2026**.

**🔲 OUVERT, NON TRANCHÉ PAR L'OWNER :** le badge d'année est **affiché** sous `year != null || datable` mais **édité** sous `editable = !locked && datable` ⇒ la saison où tu te tiens peut **afficher une année que le popover refuse de modifier ou d'effacer**. Les fix 2-3-4 la rendent rare, **ne la ferment pas**. Correctif proposé = aligner l'affichage sur la règle d'écriture (`!locked && datable`), 1 ligne. **Attendre son go.**

### 🔭 ÉTAT AU 20/07 SOIR — LA LENTILLE `MediaView` EST CONSTRUITE ET ADOPTÉE SUR LA FICHE
**2 commits owner** : `c45d048` (recherche catalogue + discover + add path status-locked) puis le commit de la lentille. **Le reste ci-dessous est NON COMMITÉ.**

**✅ FAIT & VÉRIFIÉ (base + écran, par Opus lui-même) :**
- **A — barrière update dans le tuyau** : `updateMediaItem` parse `updateMediaSchema` en interne (validation seule : on écrit `data` brut, donc les faits-du-monde passent intacts). Ferme le 🔴 #1 de Fable (`addTmdbItemToList` contournait). 4 tests.
- **(f) possession par `tmdb_id`** : `findOwnedMediaId` sans `.eq("type")`, renvoie `{id,type}`, exclut les stubs → plus de doublon anime. **Logique seule, pas testé écran** (il faudrait une ligne mal-typée).
- **`useTmdbDetails` → `TMDB_KEYS.details`** : sort de `WATCHING_KEYS.all` (un ajout ne refetch plus tous les détails TMDB).
- **B1 `lib/media-view.ts`** — LA lentille (11 tests). Lit en espace AFFICHAGE (`position`, `seasons`, `yearMap`, `seriesFacts`), écrit via `toStorage`/`writeYear`/`writeYears` qui **routent vers la bonne colonne**. Fallback sûr : source non-anilist / série / cours absents / anime déjà multi-saisons TMDB → identité.
- **B2 — 3 fonctions de stamp routées** (`markWatchedPatch`, `markCaughtUpPatch`, `positionPatch`) via le seam `stampSpace`. **Clé qui a réduit le risque : le STATUT est invariant par espace** (les totaux vus/diffusés sont identiques) ⇒ `seriesState`/`caughtUpAt` intouchés ; seules les **frontières de saison** diffèrent. Sans lentille = byte-identique (les 144 tests d'avant sont restés verts).
- **B3.1 câblage dans `useWatchActions`** (l'écrivain unique) ⇒ **toute surface hérite du fix sans changer de call site**. Colonnes en trop **optionnelles** sur son type d'entrée → adoption sans jour de bascule.
- **B3.2 StatusCard + collapse de la fiche** : `hideSeasonStepper` **supprimé**, `handleCourSetPosition` **supprimé**, 4 handlers année/note → **2**, forest de ternaires `overlay ? … : …` → `view.*`.

**🧪 PREUVE RÉELLE DU FIX (Blue Lock, en base, via l'UI) :** position mise à la fin du cour 1 (ép. plat 24) → « +1 » piloté → `cour_years` **{"2":2025} → {"1":2026,"2":2025}**, `season_years` **jamais touché** → **ligne restaurée à l'identique**. Écran : le cour 1 porte le badge « 2026 ». **Avant, ce clic ne stampait RIEN** (test unitaire qui reproduit le bug conservé à côté du test du fix).
**🧪 ÉCRAN :** JJK affiche **Season 3/3 · Episode 12/12** (au lieu du plat 59/59 + stepper masqué) ; **HotD pixel-identique** (non-régression). Owner a confirmé Blue Lock = 2 saisons bien affichées.

**🔧 L'ACCÈS APP D'OPUS (à réutiliser) :** `node scripts/auth-setup.mjs` (owner, interactif, ~30 s) → session dans `scripts/.auth/storageState.json`. Screenshot : `MSYS_NO_PATHCONV=1 node scripts/shoot.mjs <route> out.png` puis Read le PNG. **Lecture/écriture base** : reconstituer le JWT depuis les cookies SSR `sb-…-auth-token.0/.1` (base64) → REST avec `apikey: ANON` + `Bearer <jwt>` + `Accept-Profile: watching` (RLS respectée). **JWT expire en ~1 h** → relancer un chargement de page via Playwright puis `ctx.storageState({path})` pour le rafraîchir **sans** consommer le refresh-token (sinon la session du harness casse). ⚠️ `SUPABASE_SERVICE_ROLE_KEY` de `.env.local` = clé MORTE ; `HEGON_SECRET_KEY` absente.

**✅ B3.4 (cartes) FAIT + VÉRIFIÉ ÉCRAN** : `getAnimeCoursMany` (lecture **groupée**, 1 requête par rail) + `useMediaViews(items)` ; `MovieCard` prend `view`, `NextEpisodeButton` dit la position via `actions.view` (exposée par `useWatchActions`). **Preuve : BLUE LOCK S01 E37 → S02 E13 · JUJUTSU KAISEN S01 E59 → S03 E12 · BLEACH inchangé** (vraies saisons TMDB → pas d'overlay).
**⭐ REMARQUE OWNER → `pending`, l'état « JE NE SAIS PAS ENCORE » (20/07).** Il a vu la carte afficher **S01 E59 puis devenir S03 E12** : les cours arrivent en async, et pendant ce temps la lentille **confondait « pas d'overlay » et « pas encore chargé »** → elle affichait le plat puis se corrigeait. **C'est la maladie du module reproduite DANS l'outil censé la guérir** (un mot pour deux faits). Fix = `MediaView.pending` explicite (`undefined` = non résolu ≠ `null` = résolu sans cours) + **règle : aucune surface n'affiche une position tant que `pending`** (même loi que « on ne stampe pas une année sur une saison en cours »). Cartes → pas de puce ; fiche → le skeleton existant tient jusqu'à résolution. Un non-anime n'est **jamais** pending. ✅ **Owner a jugé à l'écran : « rien » est BON, on le garde.** Il réclamera un skeleton discret plus tard s'il change d'avis — ne pas reposer la question.

**🟡 REMARQUE À VOIR (PAS MAINTENANT) — cour ≠ saison, écart avec IMDb.** Re:Zero affiche **5 saisons**, IMDb en dit **4**. Cause : **AniList segmente par COUR** ; une saison diffusée en deux blocs (« Season 2 » puis « Season 2 Part 2 ») = **2 entrées AniList = 1 saison**. Notre resolver les numérote séquentiellement sans fusionner. **Encore la même maladie** : on appelle « Season » une unité qui est un **cour**. **⚠️ Rien n'est cassé** (position, années, posters, stats justes) — c'est un **libellé**, pas une donnée fausse.
**Ampleur mesurée : 4 titres / 56** résolus anilist — AoT (6 cours → 4), Re:Zero (5 → 4), Haikyu (5 → 4), Fire Force (4 → 3). **Le signal de fusion est DÉJÀ stocké** : le titre AniList dit « … Season 2 Part 2 ».
**⛔ Décision owner (20/07) : ON NE LE FAIT PAS MAINTENANT** — coût/risque disproportionné (les numéros de cours sont les **CLÉS** de `cour_years`/`cour_ratings` ⇒ re-segmenter impose une **remigration des clés**, qui peut corrompre les années). **À reconsidérer un jour SI on veut être aligné à 100 % avec IMDb.**
**Remède alternatif sans toucher aux données (le préférer si ça regratte) :** afficher le **vrai titre du cour** (déjà stocké) dans le popover du strip → le « 5 » cesse d'être trompeur, on voit que 2 blocs forment 1 saison. Zéro migration.

**🔴 PIÈGE À RETENIR :** ça ne marchait pas au premier essai parce que **`SECTION_COLUMNS` ne sélectionnait pas `tmdb_id`** → `useMediaViews` filtrait tout (pas d'id → pas de cours → pas d'overlay). **Une surface ne peut parler les bonnes coordonnées que si sa requête porte la clé qui y donne accès.** Colonne ajoutée. À vérifier pour chaque nouvelle surface adoptée.

**🔲 RESTE SUR B (owner attend que je TERMINE avant de tester globalement) :**
- **✅ B3.3 FAIT (20/07)** — `addStatusPatch` prend la lentille (**la 4ᵉ et dernière fonction de stamp routée**) ; `AddStatus` porte `cour_years` ; `useAddMedia` transporte `view` et spread un `yearPatch` opaque (aucun call site ne sait quelle colonne) ; **`AddMediaModal` construit une vue SYNTHÉTIQUE** (`buildMediaView` sur un source fabriqué depuis le payload TMDB + cours, puisque la ligne n'existe pas encore) → `HowFarDidYouGet` reçoit `addView.seasons` donc **ses chips sont par COUR**, et la position réclamée est convertie via `addView.toStorage` au submit. **Prouvé par test** : addStatusPatch avec lentille → `cour_years:{"1":2024}` · sans → **rien**. **Écran** : porte Last Watched sur un anime 4-cours → « Through S1..S4 » (⚠️ non isolé formellement : si TMDB donnait nativement 4 saisons on verrait pareil — c'est le test unitaire qui isole la branche overlay).
- **AUDIT DES SURFACES FAIT** — il en reste **moins que prévu**. Celles qui lisent une position : `SeasonHistoryStrip`/`StatusCard`/`MediaCarousel`/`NextEpisodeButton` **✅ adoptées** · **Dashboard n'affiche AUCUNE position** (rien à faire) · **pas de `LibraryCard` distincte** (la bibliothèque passe par `MediaCarousel`) · **`computeStats` lit le STOCKAGE et c'est CORRECT** (totaux invariants par espace + Stats indexe par saison TMDB → ⛔ NE PAS convertir).
- **✅ B4 GARDE ESLint POSÉ (20/07)** — `no-restricted-syntax` sur `.current_season|.current_episode|.season_episodes|.season_aired|.season_years|.season_ratings|.cour_years|.cour_ratings` dans `components/**` + `app/(main)/perso/watching/**`. **`components/stats/**` EXEMPTÉ** (c'est du CALCUL : totaux invariants par espace + indexation par saison TMDB voulue ; le passer par la lentille changerait ses chiffres, pas les corrigerait).
  **⚠️ EN `warn`, PAS `error`** — les 3 vrais trous sont fixés, mais **~25 lectures LÉGITIMES** restent à annoter d'une raison chacune (états locaux de la fiche qui miroitent la ligne, fallbacks `view ? … : media.…`, totaux invariants). **À passer en `error` une fois annotées** — c'est ça qui rend l'invariant (c) vraiment mécanique.
  **🔴 CE QUE LE GARDE A TROUVÉ IMMÉDIATEMENT (ce que l'audit manuel avait raté) :** ① `handleWatchedYearChange` écrivait `season_years["1"]` brut → année invisible sur un anime overlay — **fixé** (passe par `view.writeYear`) · ② `watchedLabel`/`watchedYearOf` (StatusCard) lisaient `media.season_years` → libellé retombait sur `watched_at` au lieu de « 2023 – 2024 » — **fixé** (prennent la carte de la lentille) · ③ `AddMediaModal` préremplit les champs (espace **cour**) avec `existing.current_season/episode` (espace **stockage**) → **NON FIXÉ, documenté in-file** : impossible de convertir à cet endroit (les cours sont fetchés depuis l'id qu'on est en train de sélectionner, la lentille est encore vide). **Le fix = préremplir depuis un effet quand la lentille se résout.**
- Reste aussi : contrainte type-conditionnelle sur `cour_years`/`cour_ratings` (rien n'empêche de les écrire sur un film).

### 📝 TEST GLOBAL OWNER (20/07) — 4 remarques, analysées, NON CODÉES
1. **🔴 Flash de la page discover** avant la vraie fiche quand le titre est déjà possédé. **Même maladie que le `pending`** : `useOwnedMediaId` est async et « pas encore résolu » est traité comme « pas possédé » → discover se rend puis se ravise. **Fix = skeleton tant que la possession n'est pas résolue.** ⚠️ **Règle owner posée : un écran qui se ravise est une FAUTE, pages comme skeletons — le loading doit être parfait ET rapide.** À faire **avec l'étape perf** (même sujet : quoi afficher pendant qu'on ne sait pas).
2. **`-1` ne rembobine pas `last_watched_at`.** ⛔ **Ma lecture, acceptée : PAS d'historique jsonb.** Le stamp en-avant-seulement est JUSTE (reculer = correction, pas visionnage). Le vrai cas owner = **+1 par erreur puis annulation** — et là **seul l'Undo du toast sait** qu'il annule l'action qui vient d'avoir lieu. **Fix = l'Undo restaure le `last_watched_at` précédent** (valeur déjà connue du patch). Un `-1` plus tard reste une correction et laisse la date. Zéro colonne nouvelle.
3. **More Like This sans interaction sur discover** — incohérence pure, doit être identique à celui de la fiche. Petit.
4. **Badge « Now » perdu quand on est caught up** (HotD S3). Règle actuelle trop grossière : `currentEpisode < aired` — écrite pour le cas FROM (saison entièrement diffusée, tu attends la suivante). **Mais HotD S3 n'a pas fini de diffuser (5/8)** : tu es encore DEDANS. **La bonne question n'est pas « as-tu tout vu ? » mais « cette saison a-t-elle fini de diffuser ? »** — si non, le badge reste.

**➡️ ORDRE PROPOSÉ : étape PERF d'abord** (plaintes 4 et 5 d'origine, **jamais traitées** : `key={pathname}` = le « rechargement complet », `gcTime` 5 min, pas de `placeholderData`, fan-out TMDB → `append_to_response`) **+ le flash discover**. Puis les 3 petits (badge Now · Undo · More Like This). Puis les dettes : garde ESLint en `error`, préremplissage modale.

### 🏛️ CONSTITUTION D'ARCHITECTURE — ÉCRITE (20/07) = `hq/watching-architecture.md`
**LA source de vérité d'archi Watching, une seule et finale** (décision owner : plus d'audits éparpillés). 5 invariants, chacun avec son **point d'étranglement** + la faute évitée : (a) statut = dérivation pure (`watch-status.ts`) · (b) écriture = parse qui refuse (`media.schema.ts`) · (c) **coordonnées = lentille `MediaView` unique — À CONSTRUIRE, étape 4** · (d) un fait = un propriétaire · (e) sécurité = RLS. Le §2 pose le design `MediaView` **avec le routage `season_years` vs `cour_years`** qui ferme le bug des années overlay du 20/07.
**⏳ FABLE EN REVUE ADVERSARIALE** (lancé 20/07, en arrière-plan) — brief : trouver un invariant sans chokepoint · deux sources de vérité résiduelles · couplage inter-module qui touche les tables d'un autre · **attaquer le design MediaView** (cycle de dépendance pur→vue ? cas de stamp à cheval sur 2 cours ? `writeYear` qui contourne la barrière ?) · exclusions du §5 à tort · un chemin d'écriture qui saute la barrière. **⚠️ NE PAS toucher au code du module ni archiver `hq/audit/watching-architecture.md` tant que Fable lit.**

### 🧹 NETTOYAGE hq — À FAIRE APRÈS FABLE (demandé owner)
Une seule vérité d'archi. Après la revue : **archiver** `hq/audit/watching-architecture.md` + `db-schema-audit.md` + `series-facts-design.md` + `db-diagnostic-watching.md` → `hq/archive/` (garder le raisonnement, retirer du « courant ») ; **purger** la prose d'archi de `now.md` (pointer vers la constitution). Règle : archi = 1 fichier · `now.md` = état courant · `progress.md` = journal.

### 🟡 ÉTAPE 1 — détail technique du lot initial (18/07 soir)
**Vert : `tsc` 0 · `eslint` 0 · 124/124 tests watching (49 sur `watch-status.test.ts`, 32 avant).**
**⚠️ ET ÇA NE PROUVE RIEN — c'est exactement la faute de la veille. L'owner n'a encore cliqué sur RIEN.**
(Échec de test restant : 2 dans `journal-utils.test.ts`, `…` vs `...` — **antérieur, hors périmètre**, à traiter séparément.)

**Les deux verrous (purs, testés) :**
- **`addStatusPatch(door, claim, type)`** dans `lib/watch-status.ts` — le frère côté ajout de `claimedStatus`. **Film → la porte décide** (honnête, c'est binaire) ; **série → la POSITION décide**, la porte ne dit plus que si une réclamation est faite. **Retourne `null`** quand une porte qui a besoin d'une réclamation n'en reçoit pas ⇒ refus au lieu d'invention. C'était ça, la ligne limbo.
- **`insertMediaSchema`** dans `schemas/media.schema.ts` — 3 invariants : **exactement UN** statut vrai (`is_reference` compte : un stub de liste est une déclaration) · une **position porte** son `caught_up_at` · **une série que tu regardes A une position**. Un test parcourt **5 portes × 3 types** et prouve que les deux verrous s'accordent.

**Le câblage :**
- `useAddMedia` ne monte plus les drapeaux à la main : tout vient de `addStatusPatch`.
- **`insertMediaItem` parse le schéma → la barrière est DANS LE TUYAU** (couvre aussi la porte « ajouter à une liste custom » et les portes pas encore écrites).
- **Les 3 branches `update:*` passent par la barrière.** `update:inProgress` forçait `in_progress:true` quoi qu'on dise **et** écrivait une position sans `caught_up_at` : les deux sont morts.
- **La porte « In Progress » route enfin ses champs comme `position`** (mêmes champs, mêmes chiffres, entrée qui décide).
- **Submit désactivé** sur série + porte de réclamation sans position (« Tell us how far you got first. »).

**3 bugs trouvés en chemin (au-delà du plan) :**
1. **Ré-ajouter un titre qui n'était qu'un stub de liste** (`is_reference`) ne nettoyait pas le drapeau — et `deriveWatchStatus` le lit **EN PREMIER** ⇒ le titre restait « Unwatched », dans aucun rail. `is_reference:false` est maintenant **énoncé, pas omis**.
2. **`updateMediaSchema` est un parse QUI STRIPPE** : y faire passer les branches `update:*` aurait **supprimé en silence** `season_episodes`/`season_posters`/`season_air_dates`/`priority`/`rating`. Déclarés.
3. Le chemin d'ajout avait une **4ᵉ copie** de « quelle saison peut porter une année » : il stampait contre le **DIFFUSÉ** au lieu de l'**ANNONCÉ** (HotD S3E3, 3 ép sur 8 ⇒ datait une saison en cours). Pointé sur **`isSeasonDatable`**, la règle exportée. ⚠️ **changement de comportement pris seul — à valider par l'owner.**

**À TESTER AU RETOUR, dans cet ordre :** ① série **jamais possédée** via « In Progress » S2E4 → doit atterrir dans le rail, fiche « Watching » (LE bug) · ② même geste sur une série **déjà possédée** (c'était « même geste, résultat opposé ») · ③ Library/Last Watched/Top 10 sur une série → Add **grisé** sans réponse · ④ **Top 10 sur une série en cours** → rangée mais **pas** watched · ⑤ les **3 boutons discover** (ils passent par le modal, donc héritent du fix) · ⑥ **non-régression** : un film par chaque porte + ajout d'un titre TMDB à une **liste custom**.

**Cas de bord assumé, signalé :** une série dont TMDB ne connaît **aucune saison** ne peut pas se voir poser la question ⇒ Add reste grisé. Avant, elle produisait une ligne limbo. Arbitrage, pas évidence.

**Fichiers touchés (lot complet, étape 1 + les 4 correctifs) :** `lib/watch-status.ts` · `lib/watch-status.test.ts` · `lib/season-years.ts` · `schemas/media.schema.ts` · `hooks/useAddMedia.ts` · `service.ts` · `components/modals/AddMediaModal.tsx` · `components/detail/SeasonHistoryStrip.tsx`.

### ⏭️ SUITE DU PLAN (inchangée) — l'owner sait que 4 phases restent
**1 problème sur 5 traité.** Reste : **étape 2** (commiter le non-commité + 3 retouches — dont 🔴 `findOwnedMediaId` qui filtre encore `.eq("type")` [service.ts:601], **2ᵉ cause possible du « want to watch n'ajoute pas l'anime »**) · **étape 3 = PERF, la plus rentable** (`key={pathname}` toujours en place [layout.tsx:47] = le « rechargement complet » · `gcTime`/`staleTime` · `placeholderData`) · **étape 4** = la lentille `MediaView` (l'overlay AniList absent de StatusCard + cartes carousel) · **étape 5** = `append_to_response`.
**L'owner attend l'étape 3 en priorité** (rechargement + lenteur = ses 2 plaintes les plus visibles).

### ⛔ POINT 9 (thèmes séries/films) — RECHERCHÉ ET ABANDONNÉ (18/07). NE PAS REPOSER LA QUESTION.
**Règle owner : « soit la MÊME expérience que les OP/ED anime, soit rien. »** Impossible à tenir — vérifié par exécution réelle, pas supposé :
- iTunes/Apple, **Deezer**, Spotify → **extrait 30 s** uniquement (Deezer : 469 Ko de MP3 sur un titre de 78 s ; Spotify retire même le preview aux nouvelles apps).
- Apple Music / Spotify **complet** → exige l'abonnement **de l'utilisateur** + SDK (MusicKit / Web Playback) : pas universel, pas gratuit.
- YouTube → son complet mais **iframe vidéo** (≠ le ThemePlayer audio), clé + quota, matching peu fiable.
- **Pourquoi AnimeThemes marche et n'a pas d'équivalent :** c'est une archive communautaire qui **héberge les fichiers**, spécifique à l'anime. Rien de tel n'existe légalement pour les B.O. commerciales. ⛔ Aucune solution de contournement (scraping/piratage) — exclu.

**Ce qui a QUAND MÊME été appris et reste réutilisable :** la recette qui marche = **compositeur TMDB** (`job: "Original Music Composer"`, même pattern que les réalisateurs [service.ts:1252](src/modules/watching/service.ts#L1252)) **+ recherche iTunes ancrée sur son nom + filtre `artistName` contient le compositeur** → élimine les covers (« Geek Music », pianistes) qui polluent sinon le top des résultats. Testé OK sur Interstellar/Inception/GoT/Succession/Stranger Things. Si un jour une source complète existe, le matching est résolu.
2. **SPORT = l'investissement** — le seul travail que l'owner ne peut pas faire seul : je teste les vraies APIs, mesure les rate-limits, propose des features, retouche les edge functions (toutes en local). **Passe craft complète.** 4 → 9.
3. **Audit Fable** — ✅ décidé : **la colonne vertébrale partagée UNE fois** (RLS `org_isolation`, couche service, hooks TanStack) **+ Sport à part**. PAS 8 audits (redondant sur le partagé). Sécu/perf/structure → appliquer.
4. **Handoff = assurance-retour** — Settings (**⚠️ à scoper : owner doit lister les douleurs concrètes**) · **vue observabilité version FAISABLE** (ce qui est sauvé et où : `hq/`, mémoire, tailles, dates — PAS de compteur de tokens live, non faisable proprement) · **dégraisser les docs** (`progress.md` 68 Ko, `now.md` 36 Ko = trop gros pour un boot rapide).

**Au fil de l'eau, zéro jour dédié :** workspace **« HEGON Dev » dans Tasks** (dogfood en marchant, PAS de backfill depuis le début) · posts LinkedIn à chaque module fini.

**⛔ COUPÉ (décidé) :** automatisation Phase 4 (moteur type Zapier — le blueprint lui-même dit « seulement si >10 règles utilisées » : pas prouvable aujourd'hui) · backfill Tasks depuis le début · compteur de tokens en direct.

**Distinction craft clé :** *check craft* (revue rapide, 3-4 détails) pour un module ≥8.5 · *passe craft* (redesign) réservée à Sport et à ce qui est sous 8.5.

---

## ✅ CLOS — ROTATION `service_role` : **TERMINÉE LE 17/07. LA CLÉ FUITÉE EST MORTE (`curl` → 401).**
**Les règles durables vivent maintenant dans `CLAUDE.md` §6 bis.** Historique : `hq/audit/service-role-rotation-plan.md`. **Plus rien à faire ici.**

Soak vert (21 runs, 7 crons, aucun échec ; 5×200 dans la fenêtre observable) · les 3 dernières fonctions jamais exécutées sur la nouvelle clé testées avant le bouton (`f1_sync_race_results`, `f1_sync_constructor_standings`, `for-you-refresh` → 200) · **« Disable JWT-based API keys »** (pas *delete*, pas *generate a new JWT secret* → aucune session perdue) · smoke test OK (login owner + démo + page démo + ajout joueur tennis) · **preuve finale : vieille clé → `401`**.

**L'architecture obtenue :** la **plomberie ne transporte plus AUCUN secret** — les 14 crons et le trigger signup passent par `internal.call_edge(fn, payload)` qui porte la **clé PUBLIQUE** (vérifié : publishable → 200, le Bearer n'est qu'un laissez-passer de gateway). La clé privilégiée `hegon_server` (`sb_secret_`, **41 car.**) ne vit que dans **3 endroits privés** : `HEGON_SECRET_KEY` (secrets des functions), Vercel, `.env.local`.

**Fait au passage (bonus) :** `timeout_milliseconds := 300000` dans le helper → **l'aveuglement pg_net des 14 crons est mort** (on voyait les 2 crons football se faire trancher à 5 s toutes les heures, en direct) · **3110 lignes** de `job_run_details` contenant la clé purgées · webhook signup ne transmet plus `encrypted_password` · check `CRON_SECRET` mort supprimé de `for-you-refresh`.

**⚠️ LA LEÇON : « posé » ≠ « fonctionne ».** 3 fois ce soir une clé a été « enregistrée » sans l'être (collée **devant** l'ancienne dans `.env.local` → 224 car. ; secret Supabase ≠ vraie valeur). L'UI disait *Finished*/*Updated*. **Seule une exécution réelle a dit la vérité.** → vérifier par **empreinte sha256** (`supabase secrets list` la publie ⇒ comparaison locale **sans exposer la valeur**, méthode validée sur `TMDB_API_KEY`) **+ une invocation qui écrit**.

**Trouvé pendant, à traiter APRÈS (ne pas mélanger) :** ① **les edge functions sont invocables par n'importe qui** (une clé publique ouvre le gateway — vrai depuis toujours, la service_role n'y changeait rien) → **robinet à coût**, pas une fuite de données ; ② `f1_sync_race_results` **ne touche pas** aux classements (ce sont les 3 crons F1 qui appellent les 3 fonctions) ; ③ le **CI ne teste rien** : les clés n'y servent qu'à faire passer `npm run build` — un CI vert **ne prouve pas** qu'une clé est valide ; ④ noms des functions incohérents (`football_sync_teams` vs `watching-series-sync`) → backlog nettoyage ; ⑤ secrets inutilisés sur le projet : `BALLDONTLIE_API_KEY`, `RAPIDAPI_KEY`.

## ✅ AUTRE TRAVAIL DU 16/07 (matin) — FAIT
- **🔴 `watching-series-sync` était MORT depuis le 15/07 16:00** : son `SELECT` lisait `recently_watched` (colonne droppée) → chaque run mourait, mais **`net.http_post` est async donc le cron rapportait « succeeded »** (= la racine du piège en mémoire). **Mon oubli** (j'avais droppé la colonne sans toucher l'edge function). Corrigé dans l'edge function **et** `scripts/sync-series.mjs`, redéployé → **a immédiatement rattrapé « Silo: aired 22 → 23 »**.
- **🎌 ANIME V2 = CLOS** : cron `watching-anime-cours-sync` déployé + planifié `0 */6 * * *` + **vérifié 200** (avec `timeout_milliseconds := 60000`). Seul cron des 14 à être observable.
- **Commits** : tout est commité (owner était 6 commits en avance) SAUF `scripts/sync-series.mjs` → `git add scripts/sync-series.mjs && git commit -m "fix(watching): drop recently_watched from the series sync script"`.

## 🔶 SESSION 16/07 (veille) — commitée
Grosse journée. Dans l'ordre :
1. **✅ Section « Waiting for »** (films non sortis) — dérivée, zéro trigger. `release_date` (colonne, migration `20260715_media_release_date.sql`) = **UNE seule source TMDB** (owner a tranché : fini OMDb pour la date sur la fiche). `isAwaitingRelease` = LE prédicat unique (release_date futur → en attente ; fallback statut/année), `canComplete` en découle (bouton « Mark as watched » et rail ne peuvent plus se contredire). Section `WaitingForSectionClient` (badge compte à rebours, caché si vide) ; Want to Watch ne montre que les sortis. Fiche : « Not released yet · <date> » + ligne « Released » lit notre `release_date`. ⚠️ **owner a lancé `scripts/fix-release-dates-primary.mjs --apply`** (remis les dates primaires TMDB après un aller-retour théâtral). Vérifié : Spider-Man/Doomsday en Waiting for, The Odyssey (sorti) en Want to Watch.
2. **✅ Passe craft Waiting for** (`/impeccable`) : badge priorité « MEDIUM » retiré des cartes Waiting for (compte à rebours = seule marque), contraste date `white/40→50`. Vérifié écran desktop+mobile.
3. **✅ Anime v2 complet** (voir § dédié ⬇️) — fondation + UI + rate/date par-cour + fix plancher d'année.
- Feedback owner enregistré : **commits en anglais** ([[feedback_git]]).

## ✅ LA LISTE DU JOUR (15/07) — 13/13 FAITS & COMMITÉS
Bugs (search genres, film non-sorti non-visionnable), quick wins (First watched, More Like This année, Trending possédé « In your library », badge watched-ago compact mobile, chip Caught up flex-wrap), **date-picker partagé** (`lib/watched-date.ts` + `components/shared/WatchDatePicker.tsx`, année req + mois/jour opt, `buildWatchedAt` unique), **Search/Add « See details »** (titre possédé → fiche ; sauf contexte Top-10 ; film non-sorti = « Not out yet » inclickable), et **le gros chantier Last Watched** ⬇️.

### ⭐ LAST WATCHED — refonte (ex « Recently Watched »)
Trois bugs, une cause : **`watched_at` d'une série n'était PAS la date de visionnage** (= le jour où on l'a marquée). Corrigé :
- **Tri client par `priority` RETIRÉ** de `MediaCarousel` (il mettait les Top-10 en tête du rail). Ordre = serveur `watched_at DESC`.
- **`watched_at` honnête** : `markWatchedPatch`/`positionPatch(completes)`/édition d'année dérivent de `max(season_years)` (via `buildWatchedAt`, année courante → now). Films = date du picker.
- **Section dérivée** de `watched=true` ordonné par `watched_at` (plus de filtre booléen). **Rename « Last Watched »** partout (section, modal, bandeaux).
- **Script `scripts/repair-watched-at.mjs`** (dry-run montré, ⚠️ **owner doit lancer `--apply`**) : Tier 1 (season_years max) 6 lignes + Tier 2 (dates impossibles < fin de diffusion, ex Blue Lock → 2024) 8 lignes. Chernobyl 2023 laissé (plausible). Règle conservatrice.

### ⭐ DIAGNOSTIC DB #1 + cleanup `recently_watched`
Rapport : **`hq/audit/db-diagnostic-watching.md`**. Verdict : structure saine, 1 colonne morte (`recently_watched`), 1 grosse redondance (`series_facts`, backlog), 2 dénormalisations mineures (`seasons`/`episodes`). **`recently_watched` SUPPRIMÉE** (code + tests + migration `20260715_drop_recently_watched.sql`).

## 🎌 ANIME V2 (AniList overlay) — ✅ COMPLET (fondation + UI + rate/date par-cour), NON COMMITÉ
**But :** TMDB regroupe un anime en 1 saison plate (JJK = « S1, 59 ép ») ; on veut les vraies saisons (S1 24 / S2 23 / S3 12) avec leurs **posters AniList**. TMDB reste la colonne vertébrale, AniList = overlay (découpage + posters + titres/années par saison).

**✅ FAIT & SÛR (l'app ne lit pas encore `anime_cours` → zéro impact) :**
- **Table `watching.anime_cours(tmdb_id pk, cours jsonb, source, resolved_at)`** — migration `20260716_anime_cours.sql`. Référence PARTAGÉE façon `trending_cache` (RLS public read, service-role write, **pas d'org_id**). `cours` = `[{season, anilist_id, title, poster_url, year, episodes, start_episode, end_episode}]` ordonné par date.
- **Resolver `scripts/resolve-anime-cours.mjs`** : **Fribb anime-lists** (`themoviedb_id.tv → anilist_ids`, curé, 8435 TV) → **AniList GraphQL** (épisodes/date/poster/titre) → segmente la liste plate TMDB par cumuls. **Validation** : somme AniList == total TMDB, sinon `source='mismatch'` (fallback plat). Gère le rate-limit AniList (429 + Retry-After).
- **⚠️ OWNER A LANCÉ `--apply` → table peuplée** : **62 résolus (56 propres `anilist` + 6 `mismatch`), 7 non-mappés `none`**. Fallback plat garanti pour les 13. Ex propres : Bleach 5 saisons (TYBW), My Hero Academia 8, AoT 6, Kingdom 6, Demon Slayer 5 (arcs nommés), JJK/Blue Lock. Mismatches (fallback) : One Piece (AniList=0 ongoing), JoJo (Stone Ocean absent), DBZ Kai, 7 Deadly Sins, Mushoku Tensei ±1, Crush Gear.
- **`next.config.ts`** : `*.anilist.co` ajouté à CSP `img-src` + `images.remotePatterns`. ⚠️ **owner doit REDÉMARRER le dev server** (sinon posters bloqués).
- **⚠️ Migration `20260716_anime_cours.sql` : owner doit `npx supabase db push`** (le `--apply` a marché donc la table existe déjà chez lui — mais à versionner).

**✅ UI FAITE + VÉRIFIÉE ÉCRAN (JJK : 3 posters AniList S1/S2/S3 dans Watch History + boutons S1/S2/S3 dans Episodes) :**
- **`lib/anime-overlay.ts`** (helper pur) : `shouldOverlay` (anime + source anilist + **TMDB single-season** `season_episodes.length<=1` → sinon fallback), `flatEpisode`/`tmdbFromFlat`/`flatToCour`/`courToFlat` (transfo bidirectionnelle), `buildOverlay(media, cours)` → tableaux découpés (`seasonEpisodes/seasonAired/seasonPosters absolus/seasonEndDates`) + position remappée en coord cour. **Storage reste TMDB** (ép 47 = ép 47), l'overlay n'est qu'une LENTILLE d'affichage.
- **Service `getAnimeCours` + hook `useAnimeCours`** (query-key `animeCours`, staleTime 1h).
- **`SeasonHistoryStrip`** : prop `overlay` (poster absolu AniList vs path TMDB ; year/rating par-cour DÉSACTIVÉS — stockés par saison TMDB, incompatibles cour ; « Watched through S{s} » gardé, traduit cour→flat→TMDB dans `page.tsx` via `handleCourSetPosition`).
- **`Episodes`** : prop `cours` (boutons S1/S2/S3, fetch TMDB season 1 + slice par plage de cour, **marques en coord TMDB** season=1+ép flat, numéro affiché par-cour).
- **`StatusCard`** : prop `hideSeasonStepper` (masque le « Season 1/1 » redondant/contradictoire pour un anime overlay ; garde le stepper Episode flat). ⚠️ **PAS touché aux dérivations series-state** (elles restent TMDB — c'était le risque).
- **`next.config.ts`** : `*.anilist.co` (CSP + images). tsc/lint 0 erreur.
- **✅ RATE/DATE PAR-COUR (remarque owner)** : le year/rating par saison était désactivé en overlay (`season_years` indexé par saison TMDB, incompatible cour). Fix propre = **2 colonnes dédiées `cour_years`/`cour_ratings`** (migration `20260716_media_cour_years.sql`, indexées par cour, **séparées de season_years → Stats intactes**) ; le strip travaille déjà en coord cour → j'ai retiré le blocage `!overlay` de `datable` et pointé le year/rating vers `media.cour_years`/`cour_ratings` (handlers `handleCourYearsChange`/`RatingsChange` dans `page.tsx`). Ajouté au schéma zod + baseline + types. Le prop `overlay` du strip est devenu inutile (poster absolu détecté via `startsWith("http")`) → retiré.
- **✅ FIX PLANCHER D'ANNÉE (remarque owner)** : le picker d'année reposait sur l'année de **début** du cour → une saison oct.2023→mars2024 se laissait dater 2023 (illogique). Fix = **`end_year`** (année de fin, via `endDate` AniList) ajouté au type `AnimeCour` + au resolver ; `buildOverlay` floore sur `end_year ?? year`. **Vérifié : JJK S1 = 2020→2021 → plancher 2021.** ⚠️ **nécessite un re-run du resolver `--apply`** pour peupler `end_year` (sinon fallback année de début).

**⏭️ RESTE Anime v2 :**
- **🕒 CRON de re-résolution** : transformer `scripts/resolve-anime-cours.mjs` en edge function + cron (comme `watching-series-sync`) → auto-intègre les nouvelles saisons. **Staleness déjà sûre** (59≠71 quand S4 diffuse → mismatch → fallback plat auto).
- **Différé possible** : StatusCard position en coord cour (risqué, entangled avec series-state) ; titre du cour dans le popover du strip (ex « Mugen Train Arc »).
- **⚠️ NON COMMITÉ — owner teste demain d'abord.** Actions owner demain : (1) `npx supabase db push` (migrations `media_cour_years` + `anime_cours` + `media_release_date` + baseline) ; (2) `node scripts/resolve-anime-cours.mjs --apply` (re-peuple avec `end_year`). Puis test écran → commit du lot (message EN anglais).

## ⚠️ ACTIONS OWNER EN ATTENTE
1. **`node scripts/repair-watched-at.mjs --apply`** (répare les watched_at).
2. **`npx supabase db push`** (applique `20260715_drop_recently_watched` + `20260714_season_end_dates` si pas déjà fait).
3. **🔴 Rotation clé `service_role`** (2× fuitée) — dès la fin des modules.

## ⏭️ RESTE À FAIRE
- **🏗️ Section « Waiting for »** (films non sortis) — voir backlog §.
- **🎨 Passe `/impeccable craft`** (en dernier) — absorbe « modal d'ajout bon mais pas excellent ».

## ✅ AUDIT SCHÉMA `watching` — FAIT + CORRIGÉ EN LIVE (rapport = `hq/audit/db-schema-audit.md`)
Verdict : **structure fondamentalement saine et honnête** (RLS org_isolation cohérent + garde démo sur 6 tables, zéro `select(*)` sur les chemins chauds, cascades FK propres, commentaires de colonnes exemplaires).
⚠️ **CORRECTION MAJEURE** : la v1 de l'audit inférait les index depuis les migrations et criait « index manquants ». **L'owner a lancé `pg_indexes` en live → démenti.** `media_items` a en réalité **8 index dont un UNIQUE `(user_id, tmdb_id, type)`** — mais **6 + l'unique n'existent QUE dans la DB live** (créés à la main, jamais en migration). Donc :
- **A (index user_id/type/tmdb_id) → DÉJÀ RÉSOLU.** Composites `(user_id,type,watched,current_episode)` + `(user_id,tmdb_id,type)` + `(watched_at DESC)` couvrent toutes les requêtes chaudes.
- **C (unicité `(user_id,type,tmdb_id)`) → DÉJÀ RÉSOLU.** L'index UNIQUE protège réellement le `maybeSingle()` du code.
- **B 🟠 `series_facts(tmdb_id)` = LE seul gros chantier** (faits-du-monde ×1.95 → millions de lignes + divergence à 10k ; ancre AniList). À froid. **Candidat 2ᵉ avis Fable.**
- **D 🟠 (requalifié IMPORTANT) : le repo a DIVERGÉ de la DB.** 6 index + l'unique + le `CREATE TABLE` de media_items n'existent nulle part en fichier → une recréation/restauration les perd en silence. **Correctif = migration baseline** capturant l'état réel (colonnes + 8 index en `IF NOT EXISTS`). Petit, à faire tôt.
- **E 🟡 optionnel** : index `(media_item_id)` sur media_list_items (table 166 lignes, micro-gain). + `season_air_years` probablement morte.
- **Plan révisé** : ✅ baseline (D) FAITE ; P1 restant = series_facts (B) — **DESIGN FAIT + revu Fable, à DIFFÉRER** ; P2 = index E, tz-cleanup, idiomes RLS.

## ⭐ SERIES_FACTS — DESIGN FAIT + REVU FABLE, IMPLÉMENTATION DIFFÉRÉE (doc = `hq/audit/series-facts-design.md`)
Table de référence partagée `watching.series_facts(tmdb_id)` : sort les **9 colonnes volatiles/synchronisées** de `media_items` (status, seasons, episodes, season_episodes, season_aired, season_air_dates, season_end_dates, season_posters, last_synced_at). Identité statique (titre/poster/rating/cast) + films **restent inline** (pas de divergence, jointure inutile). RLS = référence globale façon `trending_cache` (tout le monde lit, service role écrit, **pas d'org_id**). Lecture = merge applicatif (helper `hydrateSeriesFacts`). Rollout 4 phases réversibles.
- **Fable a validé (~85%→solide après corrections)** et **recommande de DIFFÉRER**. Déclencheur = **AniList v2 OU ouverture des inscriptions, le 1er des deux**. Différer ne coûte rien (backfill trivial ≤3 users).
- **6 corrections Fable intégrées au doc** : 🔴 ordre phases inversé (dual-WRITE avant bascule lecture) · 🔴 garde `type<>'film'` au merge (collision ids movie/tv) · 🟠 split types TS (compilo force l'hydratation) · 🟠 DDL à compléter (index sync sur series_facts + updated_at upsert + drop media_items_sync_idx orphelin phase 4) · 🟠 `status` FILM à auditer avant drop · 🟡 orphelins (`WHERE EXISTS` possesseur) + gardes backfill + CHECK tmdb_id.
- **Mon filtre (anti-over-eng)** : j'accepte TOUT — zéro suggestion Fable n'est du gold-plating, il a lui-même écarté vue SQL/matérialisée/FK-embed/identité-statique/person_credits. Le design est prêt à dormir jusqu'au déclencheur.
- **✅ BASELINE FAITE (non appliquée)** : `supabase/migrations/20260715_baseline_media_items.sql` — CREATE TABLE IF NOT EXISTS (colonnes/types RÉELS via information_schema, réordonnées important→secondaire) + 8 index IF NOT EXISTS (via pg_indexes) + comments. **100% idempotent/inerte** sur la DB actuelle. ⚠️ owner : `npx supabase db push` un jour (ne change rien, documente). `season_air_years` **n'existe pas** (faux problème). Nouvelle dette repérée : `watched_at/created_at/updated_at` en `timestamp` sans tz vs le reste en `timestamptz` (nettoyage futur, migration à part).

## 🔬 AUDIT SCHÉMA `watching` — voir §« AUDIT SCHÉMA WATCHING — FAIT » ci-dessus. Rapport complet : `hq/audit/db-schema-audit.md` (9 axes couverts, plan P0/P1/P2, annexe SQL read-only).

## ✅ FAIT & COMMITÉ (chantier séries + refactors structurels)
- **Recently Watched dérivé de `watched_at`** (plus de booléen figé) + **badge « watched ago »** sur la carte. Le booléen `recently_watched` ne décide **plus rien** (marqué `@deprecated`, colonne à droper au diagnostic #11). Commité.
⚠️ **4 migrations appliquées** (`media_lists_soft_delete`, `series_aired_state`, `last_watched_at`, `season_end_dates`) · **Edge Function `watching-series-sync` DÉPLOYÉE** · **cron `0 */4 * * *`**.

- **Porte « Recently Watched » refuse une série en cours** (`resolveTransition`, juge unique) ; série terminée déjà In Progress → « marks it as finished » ; **Top 10 reste OUVERT** à une série en cours (*ranker ≠ regarder*). Commité.
- **Tout est commité, working tree propre. Rien d'inachevé fonctionnellement.**

## ⭐ LE CHANTIER : le modèle métier des séries (doc = mémoire `reference_series_aired_vs_announced`)
**Le fil rouge de tout : l'app avait des MOTS qui MENTAIENT, et elle forçait l'owner à mentir avec.**
1. `watched` confondait **« à jour »** et **« terminé »** → 25 lignes fausses. Réparé : l'état est **CALCULÉ** (vu vs **DIFFUSÉ**) dans `lib/series-state.ts`.
2. **`in_progress` avait la MÊME maladie** : il confondait **« je regarde en ce moment »** et **« je me suis arrêté en route »**. L'app possédait déjà `paused`/`dropped` — elle ne les **demandait** jamais. Réparé dans la modale (« And now? »).
3. **DIFFUSÉ ≠ ANNONCÉ** : `season_aired` (le réel) vs `season_episodes` (TMDB annonce). **Rien ne décide depuis l'annoncé** — mais l'annoncé peut s'AFFICHER (« 4 / 8 » + bouton `+` mort : les deux vérités à la fois).
4. **La liste est une CONSÉQUENCE, pas une donnée.** La ligne `watched: listContext === "library" || "topTen" || "recentlyWatched"` **fabriquait** les lignes fausses. Morte : le statut est dérivé de la POSITION par `seriesState()`, donc « watched » **ne peut plus être produit** sur une série en cours. Les **3 portes** fermées d'un coup (l'owner n'avait vu que Library).

## ✅ FAIT AUJOURD'HUI (14/07)
- **Garde-fous** : `+1` bloqué au dernier épisode **diffusé** (→ « Caught up ») · steppers bridés · **épisode non diffusé = ni notable ni étoilable** (backdrop flouté + « Airs in 3 days », seuil **14 j** sinon date sèche) · « Mark as completed » caché si en cours · badge « Now » = la saison qu'on **TRAVERSE**.
- **`last_watched_at`** (migration) — stampé **en avant UNIQUEMENT** (reculer = correction, pas visionnage). Mais **l'année d'une saison finie BAT le timestamp** (corriger sa position au stepper est un mouvement avant → ça datait la correction d'aujourd'hui). Voir `lastWatched()`.
- **UNE seule règle de « saison atteinte »** (`hasReachedSeason`) : Watch History et Quick Stats en avaient **deux différentes** → le badge 2017 s'affichait et Quick Stats l'ignorait.
- **Stamps fantômes** : `season_years`/`season_ratings` ne font que GROSSIR → on **cesse de les LIRE** pour une saison non atteinte (on ne les supprime pas : elles reviennent si tu ré-avances).
- **Modale d'ajout refaite** : « How far did you get? » (bande de saisons, **même instrument** que Watch History) → « And now? » (watching/paused/dropped) → la date **revient** et **stampe les saisons revendiquées**. **Plancher d'année = fin de diffusion de la saison choisie** (AoT « Through S4 » → 2022 mini, plus 2013). `airedFromTmdb()` lit `last_episode_to_air` **dans l'appel déjà fait** → un titre naît en sachant ce qui est diffusé.

## ✅ LES PORTES DE SERVICE (fermées 14/07) — *verrouiller l'entrée ne suffit pas*
Le bouton principal refusait de terminer une série en cours… et **4 autres chemins l'offraient quand même**. Réflexe à garder : **quand on ferme une règle, chercher toutes les portes.**
1. **Menu « … » de la StatusCard** → devient **« Mark as caught up »** si la série n'est pas finie. Et `handleMarkWatched` écrivait `watched:true` **sans poser la position** = la fabrique même des 23 lignes fantômes. Il pose maintenant la position et ne date que les saisons **entièrement diffusées**.
2. **`Set all year`** → règle **`isSeasonDatable`** (entièrement diffusée **ET** entièrement vue), **exportée** — la Watch History en avait une **3e** privée (« derrière ta position ») qui rendait indatable une saison finie où l'on se tient (FROM S4).
3. **Onglet Highlights** → cesse de **LIRE** les étoiles de saisons jamais atteintes (True Detective = anthologie). Rien n'est supprimé, ça revient quand on avance.
4. **`caught_up_at`** n'était écrit **qu'à l'ajout** → arriver à jour via `+1` ne le posait jamais → la fiche **n'aurait jamais pu s'allumer en « New »**. Recalculé à **chaque écriture de position**, effacé si on recule.
5. **« Watched through S3 »** (clic saison, **dans les 2 sens**) : le statut se **re-dérive**, les années **ne sont pas tamponnées**. StatusCard : « **Stopped after** · Season 3 ».
6. **Rewatch débloqué sur une série « à jour »** — la condition n'était pas `watched` mais **« as-tu tout vu ? »**. (Recently Watched : **rien à faire**, une série à jour n'est pas *terminée* — c'est le comportement juste.)
7. 🔑 **LE MOTEUR EST EN PLACE** : `supabase/functions/watching-series-sync` + **cron horaire**, par **lots de 25** (les plus périmées d'abord), ne redemande à TMDB que **ce qui peut avoir changé** (grâce à `season_end_dates`). Invocation manuelle vérifiée : `{ok:true, scanned:25, changed:0, failed:0}`.

## 🔭 DIFFÉRÉ — décidé, à faire plus tard (dans cet ordre)
1. **Table de référence partagée `series_facts(tmdb_id)`** — *l'owner a adoré l'idée, à garder.* La vérité du monde (`status`, `season_aired`, `season_end_dates`…) est stockée **par ligne, donc par utilisateur** : 239 lignes pour **124 séries distinctes** (3 comptes, dont le démo). Elle doit être stockée **UNE SEULE FOIS**, `media_items` ne gardant que ce qui est **à toi** (position, stance, années). Le groupement par `tmdb_id` du job (fait) en est le **1er pas**. = migration + refactor service, **à froid**.
2. **🔴 Rotation de la clé `service_role`** (2× fuitée) — **dès la fin du travail sur les modules**, décision owner. Voir mémoire `project_supabase_cron_auth`.
3. **3 positions à corriger à la main** : True Detective (anthologie), One-Punch Man, Silo.
4. **🎌 Anime v2 — saisons & jaquettes réelles (source AniList, OVERLAY sur TMDB).** *Owner tient à ça (JJK, Blue Lock : TMDB les fusionne en 1 saison de 38/59 ép ; il veut 3 boutons S1/S2/S3 + la S4 à sa sortie, avec les vrais posters par cour).* **TMDB reste la colonne vertébrale** (identité `tmdb_id`, épisodes/stills/dates, backdrop, cast, sync). **AniList ne possède QUE** : le découpage en cours (« ep 25-47 = S2 »), le poster de chaque cour, le titre/année du cour. S'accroche au `tmdb_id` dans `series_facts` ; **fallback gracieux** sur la structure plate TMDB si absent. ⚠️ **PAS de merge live TMDB↔IMDb** (aucun mapping épisode↔épisode fiable, IMDb sans API). Le suivi fonctionne DÉJÀ (ep 71 s'allume quand la S4 diffuse) — ce chantier n'ajoute que le **découpage + l'illustration**. **Résout AUSSI les anthologies** (True Detective) : même carte de saisons curée/dérivée par-dessus TMDB. = gros chantier, à froid.

## ⚙️ LE JOB DE SYNCHRO — ce qu'il fait (l'owner a challengé, et il avait raison)
- **Il compte des SÉRIES, pas des LIGNES** : 1 fetch TMDB par `tmdb_id`, écrit sur **toutes** les lignes qui le partagent → **~50 % d'appels en moins**, et surtout **deux copies d'une série ne peuvent plus DIVERGER** (avant : ta ligne synchro à 10h, celle du démo à 18h = 2 vérités dans la même base pendant 8h).
- **Il trie par URGENCE** : **180 des 239 lignes sont TERMINÉES** (75 % !) → elles ne sont revues qu'**1×/semaine**. Les **31 séries en cours** (dont 15 qui diffusent) sont revues **~3×/jour**. → cron descendu de **horaire à `0 */4 * * *`** (4× moins de réveils, meilleure fraîcheur là où ça compte).
- Réponse type : `{ ok, shows, rows, changed, repaired, failed, log }`. `changed: 0` après un backfill = **bon signe** (idempotent).

## 🧭 LE TROU DE MODÈLE À TRANCHER (pas urgent)
Le modèle **linéaire** ne sait pas dire « j'ai vu la S1 **et** la S4, pas la S2/S3 » → **les anthologies** (True Detective est dans la bibliothèque). Il faudrait un **ensemble** de saisons vues, pas un curseur. **Décision séparée.**

## ⚠️ 3 positions à corriger à la main (le script a hérité d'une affirmation invérifiable)
True Detective (S4E6 alors qu'il n'a vu que la S1 — **anthologie**), One-Punch Man, Silo.

---

# HISTORIQUE

**Person pages — livré + vérifié écran (`scripts/.shots/person-*.png`, acteur 20738 Song Kang-ho, réal 138 Tarantino) :**
- Route `/perso/watching/person/[id]` = **miroir de la fiche détail** : PersonHero (backdrop du titre signature = ton mieux-noté, portrait en poster **cadre gloss**, bio via `HeroDescription` **factorisé** partagé avec MediaHero) · **PersonJourney** = carte brandée teal (pendant StatusCard) : 2 mini-cartes Watched|Avg + Your #1 (poster `h-14 w-9 rounded-md` = Stats Top picks) + barre complétion `owned/total`. **Stat heures supprimée** (indécidable). · « Your titles » (ta note + rôle + **badges statut** Want/Watching/Paused/Dropped) · « Not seen yet » (filmo dé-bruitée, **search + Load more** client — `combined_credits` = 1 seul appel, pas de pagination backend) → clic = AddMediaModal want-to-watch · Details (bio/born/from/filmo).
- **Cast & Crew cliquables** (acteurs **et** réals, id>0 → person page).
- Plumbing : `getPersonBundle` (profil+filmo, 1 appel TMDB `append_to_response`). Hooks `usePerson`. Query-keys `person`/`titlesByPerson`.
- **Matching (fix majeur `052b96c`)** : « tes titres avec X » = **filmo TMDB de la personne (credits) ∩ ta library par `tmdb_id`**, PAS le `cast_members` stocké (plafonné à 12 → ratait les rôles non/mal crédités, ex Kevin Spacey/Se7en « John Doe »). Normalisation film→movie / serie+anime→tv ; rôle attaché depuis les credits ; `.or(...favorite)` inclut Top 10. **`getTitlesByPerson(userId, creditTmdbIds[])`.**
- Cast/réal cliquables utilisent l'id stocké (backfill `scripts/backfill-people.mjs` 689 rows + migration `directors json→jsonb` **appliquée** — plus utilisés pour le matching, gardés pour la clickability). Piège supabase mémorisé : [[reference_supabase_jsonb_containment]].

## ⏭️ NEXT
- ⚠️ **Petites remarques owner sur les Person pages** (non listées, « pas de grandes choses ») → à demander/régler au retour.
- **Time Capsule** (dernier morceau Couche 3) : « il y a N ans tu regardais X » sur la **page d'accueil Watching** (query `watched_at` ± années). Reste à trancher : jusqu'à combien d'années.
- Reliquats détail (bas priorité) : carousel In List **multi-listes** jamais vu écran ; contradiction rythme blueprint 32/24 vs design-system 16 (détail-page = exception assumée).

## Détail Watching — redesign TERMINÉ (`b5b3e2b`) + P2/P1 hygiène (`2b38bb8`)
StatusCard état-aware (providers in-card, rewatch Y+M, delete) · hero zéro-badge (status en meta, trailer slot réservé, ratings inline SVG, desc troncature-JS, fix casse titre) · Details purgé+enrichi OMDb (awards/country/language/votes) · Episodes readOnly want_to_watch + saisons>10→Select · In List éventail 5 posters 2/3-width scroll · slider knob-0 · colonnes réordonnées · P2 tokens (rounded-card/tile, surface-0, orphelin EpisodeHighlights supprimé). 2 réserves levées. Spec `hq/watching-detail-blueprint.md`.

## Passe Stats — ✅ COMMITÉE (64a94b0) + note par épisode ✅ COMMITÉE (d2a18cf)

## AUDIT MÉTIER — interactions statuts (07-07) — TOUT RÉGLÉ (1→3 commité 954b868)
- ✅ **Faille 1** — `lib/status-flags.ts` `RESET_STATUS` spread sur toutes les transitions positives.
- ✅ **Faille 2** — `useUpdateMedia` invalide `[...all,"library"]` `refetchType:"all"` (Library temps-réel).
- ✅ **Faille 3 (Option B)** — paused/dropped reconnus dans `resolveTransition` (existingLists + **bloqué + bandeau** « déjà dans ta collection ») ; selects `getExistingMediaEntry`/`getExistingMediaItem` + `MediaStateFlags` enrichis ; `RESET_STATUS` défensif dans `useAddMedia` ; 2 tests ajoutés (15/15).
- ✅ **Bonus** — badge "Now" de Watch History réservé au vrai `in_progress` (`inProgress` vs nouveau `incomplete` dans `SeasonHistoryStrip`).
- ⛔ **Faille 4 (BASSE) — LAISSÉE** : attribution année saison non-tamponnée paused/dropped = best-effort (`updated_at`) ; pas de vraie date/saison stockée → fix propre = **stampage** (déjà supporté). Pas de bricolage.
- ✅ **Faille 5** — retrait optimiste ciblé dans `useUpdateMedia.onMutate` (drop/pause → quitte In Progress instantanément).
- ✅ **Vérif 6** — `MyTakeRecord` : chips **Paused/Dropped** ajoutés.
- ✅ **Note par épisode** — table `episode_highlights` étendue (`rating numeric` + `highlighted`), menu `…` unique (best-ep étoile jaune + grille de notes 5-10 teal, affichage note bottom-right). Migration `20260707_episode_ratings.sql`.

## Fix Stats heures (07-07, commité) — paused/dropped comptent
`hasWatchTime` (watched||in_progress||paused||dropped) dans `computeStats` : heures + titres comptés pour paused/dropped. "currently watching" reste in_progress seul.

## Focus actuel — Watching V3 : Couche 1 TERMINÉE + COMMITÉE, Couche 2 EN COURS
Spec vivante = **`hq/watching_v3.md`**. Couche 2 resserrée (owner) = **Paused ✅ + Rewatch (Phase 1 ✅ + Phase 2 ✅ non-commité) + note/épisode (RESTE)**. Mood/context/source **écartés** (anti-formulaire).

## Thèmes anime (OP/ED) — CLOS + COMMITÉ (07-05/06)
- **Player** 3 états : mini (⤢/✕) ↔ card desktop / fullscreen mobile (⌄/✕) ; progress bar sous le contenu ; reset seek (events audio), spinner buffering, skip-on-error, Fisher-Yates, **Media Session API** (lockscreen/casque).
- **Pochettes** = **iTunes carré PARTOUT** (détail + My Themes + player), résolu en direct via `searchItunesArtwork` + `useThemeCovers` + **cache localStorage Map débatché** (`hegon-itunes-art`). Deezer + AniList banner **testés puis rejetés** (owner veut UNE pochette carrée cohérente, pas de switch entre surfaces). `anime_poster` (colonne) = dernier fallback.
- **Favoris en DB** : `watching.theme_favorites` (org_id + RLS org_isolation + demo_readonly_*). Migrations `20260706_watching_theme_favorites.sql` + `20260706_theme_favorites_anime_art.sql` (**hors-git, déjà appliquées**). Hook `useThemeFavorites` (query + mutation optimiste). Store Zustand localStorage supprimé.
- **My Themes** : rail sur page animes (titre cliquable ›) + **vue dédiée** `/perso/watching/themes` (hero mosaïque 2×2, Play/Shuffle, liste complète, ♥ retirer, empty state).
- CSP (`next.config.ts`) : itunes.apple.com + *.mzstatic.com + animethemes (Deezer/AniList retirés).

## Couche 1 — reste (commité avant les thèmes)
- Trailer, Watch Providers, IMDb chip, Age rating, OMDb (notes/awards/box office).
- Episodes = stills + Best Ep fusionné (max 20). Heatmap MASQUÉE (`HEATMAP_ENABLED=false`) — OMDb a des trous, réactiver après ingest datasets IMDb (Option B).

## Couche 2 — le moat (EN COURS)
- ✅ **Dropped + pourquoi** COMMITÉ (07-06) : colonnes `dropped`/`drop_reason` (migration `20260706_media_dropped_status.sql`), `deriveWatchStatus`, déclencheurs fiche (*Currently Watching* → "Drop this series") + menu `…` carte, `DroppedNotice` (Resume), guard démo hérité.
- ✅ **Pattern de capture réutilisable** = `components/shared/CaptureSheet.tsx` (chips **1-tap**, skip). Vocab drop = `lib/drop-reasons.ts`. **Réutiliser pour mood/source/who-where** (mood = étendre en multi-select).
- ✅ **Library = collection complète** COMMITÉ (07-06) : `getLibraryMedia` (watched/in_progress/paused/dropped), **badges** Watching (`Watching` + `S·E`)/Paused/Dropped + scrim, **filtre statut** (All/Watching/Paused/Completed/Dropped) persisté.
- ✅ **Paused (on-hold)** COMMITÉ (07-07) : instantané (pas de raison), migration `20260707_media_paused_status.sql`, `PausedNotice` + Resume, badge bleu. Cohérence pause/drop/resume s'effacent mutuellement.
- ✅ **Fix Stats COMMITÉ (07-07)** : paused/dropped comptent leurs heures + titres (`hasWatchTime` dans `computeStats.ts`, fetch stats élargi). "currently watching" reste in_progress seul.
- 🔨 **Rewatch Phase 1 (NON COMMITÉ)** : table `watching.rewatches` (migration `20260707_watching_rewatches.sql`), service/hook/section fiche (items watched : pills dates + "Log rewatch" 1-tap + date picker antidate). **Phase 2 = Stats** (stat Rewatches + heures, PAS +titres uniques, PAS retour Recently Watched) à faire.
- ⏳ **Reste Couche 2** : Rewatch Phase 2 (Stats) · **Note par épisode**. (Mood/context/source **écartés** par owner.)

## NEXT Watching V3
1. **Reste UX player (→ passe design-system)** :
   - ⚠️ **Overlap mini-player ↔ FloatingSaveBar** sur la fiche détail : les deux sont fixés en bas (save bar `bottom-6 left-1/2` centrée, mini-player `bottom-4 right-4`). Quand les 2 sont visibles ensemble (thème en lecture + modif non sauvegardée) sur **mobile/écran étroit**, ils se chevauchent. Fix = signal partagé (flag store posé par la fiche, lu par le player) pour décaler le mini-player quand la save bar est là. Cas de bord → à faire à la passe design-system.
   - Episodes tap-to-expand résumé ; skeletons.
2. **Couche 2 (moat)** : Dropped+why, watch context (qui/où), why-this-film (source), mood tags, rewatch, note/épisode. Défi = UX de capture sans friction.
3. **Couche 3** : Person pages (acteur/réal), Time Capsule, stats Couche 2.
4. Nettoyage : supprimer `EpisodeHighlights.tsx` (orphelin) + design-system pass + audit métier/sécurité/perf.

## Autres chantiers (hors V3)
- **LinkedIn** : post Watching prêt (publier + vérif login démo `demo@hegon.fr`/`HegonDemo2026`), seed des takes, relancer candidatures (CVs `hq/cv/` + photo).
- ⚠️ Rotation **service_role key** (fuité). [[project_supabase_cron_auth]]
- Option B dashboard (3 états) · versionner seeds démo.

## À ne pas oublier
- Commits : owner valide d'abord, **single-line**, **zéro mention IA**. [[feedback_git]]
- Changements CSP (`next.config.ts`) = **redémarrer le dev server**.
- `supabase/` est **gitignoré** → migrations appliquées à la main, hors commits.

## ⏸️ HORS-CHANTIER — à faire dans une pause dédiée (ne pas noyer dans les modules)
- **Candidature BESOIN 14 (ONEY Bank — Dev NTIC full-stack)** : owner a postulé (17/07). À préparer plus tard : le poste + la liste des connaissances à réviser (Java 25 / Spring Boot 4, Elastic Search, Oracle, monétique — foregrounder le paiement via Newteam + l'IA). Meilleur fit des 4 offres. 2ᵉ = BESOIN 11 (même boîte, Concepteur). Écartés : 04 (C# secondaire), 05 (C++/quant/Scala).
