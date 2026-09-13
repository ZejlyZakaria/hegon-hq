# HEGON — LA DOCTRINE SYSTÈME

> **Livrable de la phase 2** ([`hq/audit/plan.md`](../audit/plan.md) §4). Ce fichier dit **ce que HEGON
> a le droit de faire**. La checklist par module (phase 3) ne fait ensuite que vérifier la conformité.
>
> ⭐ **Règle du chantier (amendement 1) : chaque règle nomme son cliquet, ou se déclare FRAGILE.**
> Une règle sans cliquet n'est pas une règle, c'est un vœu — elle est gardée, mais étiquetée.
>
> Établi le **2026-09-09**, à partir de preuves d'exécution, pas de lecture. Les écarts sont en §3.

---

## 0. LE CONTRAT (axe 1) — déjà tranché, ne pas rouvrir

**HEGON est l'app personnelle de l'owner.** L'architecture multi-tenant est conservée, le SaaS reste
une option jamais poursuivie. Décision du 2026-09-07, détail dans [`hq/product-strategy.md`](../product-strategy.md) §0.

> **La règle de décision, applicable à tout choix :**
> *« Est-ce que je construirais ça si j'étais le seul utilisateur pour toujours ? »*
> **Non** → ne le construis pas. **Oui** → construis-le, et mets `org_id` sur la table.

Garder l'architecture multi-tenant ne coûte rien ; construire des **fonctionnalités** multi-tenant coûte
cher. Tout le reste de cette doctrine en découle.

---

## 1. LA FORME DU SYSTÈME (ce qui est vrai, mesuré)

| axe | ce que HEGON EST |
|---|---|
| 2 · modèle de données | **68 tables · 206 policies · 35 fonctions · 138 index**, 3 schémas (`public` / `watching` / `sport`). RLS **activée sur les 68**. |
| 3 · chemin d'une requête | `middleware` (getClaims, en process) → page **client** → hook TanStack → `service.ts` → Supabase navigateur. **9 pages serveur sur 34.** |
| 4 · cache | **DEUX étages réels** : TanStack côté client, tables-cache en base côté serveur. |
| 5 · asynchrone | **15 crons** → `internal.call_edge()` → **20 edge functions** → 11 fournisseurs → écriture en base. |
| 6 · dépendances | 11 fournisseurs externes. Le plus fragile : **`tennisexplorer.com`, qui est du scraping HTML**, pas une API. |
| 7 · défaillance | L'app dégrade en silence par défaut. **C'est le mode de panne dominant**, pas l'exception. |
| 8 · observabilité | Sentry couvre **l'app** (client + serveur + edge Next). Il ne couvre **RIEN** des 15 crons ni des 20 edge functions. |
| 9 · déploiement | Résolu en phase 1 : baseline + crons versionnés, 106 migrations archivées. |
| 10 · secrets | Split par rôle : `HEGON_SECRET_KEY` (privilégié, 17 usages) vs clé publique de passerelle. Sain. |
| 11 · durabilité | Restauration **prouvée**. Sauvegarde **manuelle** → seul poste irréversible encore ouvert. |

---

## 2. LES RÈGLES

### R1 — Une page est CLIENTE par défaut. Elle ne devient serveur que pour tenir un secret.
9 pages serveur sur 34, et `createServerClient` ne vit que dans le middleware, 13 routes `/api/*` et
3 pages. Les routes API n'existent **que** parce qu'une clé ne peut pas descendre dans le navigateur.
C'est cohérent et c'est le bon choix pour un mono-utilisateur : le CSR évite les 150 ms d'auth serveur.
→ **Cliquet : mesuré, non bloquant.** `npm run map` publie `serverPages` ; un écart se voit. **FRAGILE.**

### R2 — L'identité se lit avec `useCurrentUserId()`. `auth.getUser()` est interdit sur un chemin de lecture.
`getUser()` est un aller-retour réseau vers Supabase Auth (~150-300 ms mesurés) ; côté navigateur,
`getSession()` lit le jeton **local** — et la base le vérifie de toute façon à chaque requête (RLS),
donc « vérifier » côté client est un aller-retour pour rien. Il restait **31 appels à `auth.getUser()`**,
dont **8 dans `tasks/service.ts`** et 4 dans `watching/service.ts` — sur des chemins de lecture.
**Corrigé le 13/09, sur le chemin critique** : `settings/service.ts` (`requireUserId`, `getIsDemo`) et le
helper partagé `getCurrentUserId` faisaient **deux `getUser()` en série montés sur chaque page** —
~290 ms avant la première vraie requête, que j'avais mis sur le compte du bundle. `getSession()` →
premier appel REST **819 → 631 ms**. Restent **~22 sites** dans les services, hors chemin critique.
→ **Cliquet : RÉEL, à poser.** Règle ESLint `no-restricted-syntax` interdisant `auth.getUser()` dans
`src/modules/**/service.ts`, avec une liste d'exceptions nommées (flux d'auth, écritures qui doivent
*vérifier* l'utilisateur, pas seulement le lire) — les ~22 restants annotés comme dette, comme le N+1.

### R3 — ⭐ Le cache a DEUX étages. Pas quatre. Et une requête ne fait jamais attendre une liste.
Le plan en supposait quatre. Vérifié : **`cache()` de React n'est utilisé nulle part**, et la
revalidation Next l'est **une seule fois** — un `revalidatePath("/tasks")` qui vise une route
**inexistante** (la vraie est `/pro/tasks`) et cliente de surcroît. Il reste :
- **TanStack** (client) — fraîcheur d'une donnée que l'utilisateur vient peut-être de modifier.
- **Une table-cache en base** (`for_you_cache`, `trending_cache`, `football_standings`…) — dès qu'un
  calcul est cher ou qu'un quota externe est en jeu. **Un cron remplit, l'app lit. L'app n'appelle
  jamais un fournisseur pendant un rendu.**

**La cascade — mesurée le 2026-09-12 sur `/perso/watching/movies`** (`scripts/measure-cache.mjs`,
horloge de la page, médiane sur 5) : **15 requêtes Supabase en 4 vagues séquentielles**. Trois
mécanismes distincts, et ils ne se détectent pas pareil :
1. **Vague 2 — dépendance réelle** : `useOwnedMediaIds` a besoin des `tmdb_id` de la vague 1. Légitime
   ; se supprime par une jointure ou une RPC. *Conception, pas règle.*
2. **Vague 3 — cascade de montage** : `getActiveWatchingGoals` ne dépend de rien, mais vit dans
   `useWatchActions`, qui ne monte qu'avec les cartes. *La règle : une requête ne conditionne pas une
   autre sans en consommer le résultat.* Non détectable statiquement — **FRAGILE, mesurée** par le
   script en phase 3.
3. **Vague 4 — N+1 de service** : `Promise.all(goals.map(async g => await count(...)))` — une requête
   **par objectif**.
   ⚠️ **4ᵉ forme, trouvée par l'audit Watching (13/09) : le N+1 de MONTAGE.** Un hook qui fait une
   requête, monté dans un composant rendu **par carte** (`MediaActionMenu`, caché par opacité, pas
   démonté) = N requêtes distinctes que TanStack ne déduplique pas (clé différente par carte).
   `/animes` partait à **60 requêtes** là où `measure-cache.mjs` en comptait 15 — 44 `anime_cours
   tmdb_id=eq.X`, dupliquant les 4 batches du rail. Invisible au lint et au compteur ; visible à la
   **liste** (`scripts/list-requests.mjs`). Règle : *un composant par carte ne fetche pas ; il reçoit
   la vue batch du rail.* FRAGILE, mesurée. Huit sites de cette forme (5 Goals, 2 par course F1, un appel iTunes **par
   morceau** dans Watching, un UPDATE par statut réordonné dans Tasks — celui-ci invisible à la
   première version de la règle, trouvé au contre-examen du 13/09).
→ **Cliquet POSÉ le 12/09, élargi le 13/09** (`eslint.config.mjs`, `NO_N_PLUS_ONE_IN_SERVICES`) —
**trois formes** dans `service.ts` et les routes API : `.map(async` avec un `await` (corps expression ou
bloc) · une boucle `for`/`for…of`/`while` avec un `await` · un `.map(` **non async** dont le callback
construit une requête (`.from(`) — la forme idiomatique avec `Promise.all`, que la première version
laissait passer. Les 8 sites existants portent un `eslint-disable` **qui dit pourquoi et quel audit
le corrige**. Détection, pas correction : `/movies` se répare en phase 3, axe 3.
**Ce qui passe encore, FRAGILE** : `Array.from(xs, …)`, une chaîne `.then(…)`, et un helper qui
encapsule la requête (`xs.map((x) => getOne(x))` — rien dans le callback ne dit « requête »). « Plus
aucune nouvelle n'entre » serait faux ; **plus aucune sous ces trois formes** est vrai.
→ Le 3e étage (`cache()` / `revalidatePath`) : **FRAGILE**, non gardé — un seul usage, mort.

### R4 — Un `staleTime` est un PALIER, jamais un nombre. Le défaut s'hérite.
**Avant** : 129 `useQuery`, **126 `staleTime`** dans 78 fichiers, **51 recopiaient le défaut** de 5 min,
la même minute s'écrivait de trois façons — et le réglage global de `QueryProvider` ne gouvernait que
**10 requêtes sur 129**. Il n'y avait pas de doctrine mal réglée : il n'y en avait **aucune**.
**Après (2026-09-12)** : [`src/shared/lib/stale.ts`](../../src/shared/lib/stale.ts) — 10 paliers
(`STALE.NONE` … `STALE.WEEK`), les 126 sites migrés **à comportement identique** (2 min reste 2 min),
les 51 redéclarations du défaut **supprimées** (elles héritent), `QueryProvider` seul écrit `STALE.DEFAULT` — et
`STALE.DEFAULT` est **refusé** partout ailleurs (`STALE_DEFAULT_IS_INHERITED`) : l'héritage est mécanique, pas discipliné.
→ **Cliquet POSÉ** : ESLint refuse tout `staleTime:` qui n'est pas `STALE.<palier>`
(`STALE_TIME_IS_A_TIER`). Vérifié : 0 violation, et un littéral ajouté à la main est refusé.

**Un palier ne dit sa durée que si `gcTime` suit** (contre-examen du 13/09). `staleTime` = « fraîche
pendant X » ; `gcTime` = « gardée en mémoire X **sans observateur** », 10 min par défaut. Une donnée
`STALE.DAY` sans `gcTime` est **ramassée après 10 min hors écran**, puis refetchée : « fraîche 24 h »
n'a jamais eu lieu. **21 sites mentaient ainsi**, dont `useTeamArtwork` commenté *« Cached a day »* avec
10 min de gcTime, et les fiches TMDB de Watching (`DAY` avec 1 h). Corrigé : `gcTime: STALE.<le même>`
sur les 21 — la seule modification de comportement de la migration, voulue (la donnée reste en mémoire
le temps qu'on lui avait promis).
→ **Cliquet POSÉ** : [`stale.test.ts`](../../src/shared/lib/stale.test.ts) — un palier > 10 min sans
`gcTime` ≥ échoue en nommant le site. Le lint ne sait pas comparer deux propriétés ; le test lit le code.
Pourquoi pas un `gcTime` global d'une semaine : tout resterait en mémoire une semaine dans un onglet
longue durée ; le couple explicite ne coûte que là où on l'a voulu.
⚠️ Ce passage est **mécanique** : les noms disent la durée, pas l'intention. **Phase 3, axe 4** décide
requête par requête si la donnée est *la mienne* (un seul écrivain → l'invalidation suffit, le palier
peut être long) ou *celle des robots* (palier = cadence du cron qui la remplit, **par table** — les
classements se rafraîchissent 10×/jour en fenêtre de matchs, les compétitions une fois par mois).

**Le multi-appareil n'est pas une question de durée.** Un film noté sur le téléphone n'apparaît sur
l'onglet PC que si la requête est périmée **et** remontée : `refetchOnWindowFocus` est `false`
globalement (surchargé `true` dans 2 fichiers sur 129) et le motif SWR `refetchOnMount: "always"`
existe dans **1 fichier** (`useDashboardData` ; `useJournalCalendar` dit `true`, pas `"always"`). C'est là qu'est la justesse, pas dans `staleTime`. **Tranché** : on ne
touche pas au global en phase 2 — passer `refetchOnWindowFocus` à `true` relancerait jusqu'à ~15 requêtes à
chaque retour d'onglet (les observateurs **montés et périmés**, pas les 129 — `queryObserver.js:450`), pour un gain de justesse que seul le multi-appareil réel justifie. C'est un
arbitrage **par module** en phase 3 (Watching : oui pour les statuts ; Journal : non). **FRAGILE.**

**La persistance sur disque — possible, chiffrée, non prioritaire.** Mesurée le 2026-09-12 sur
`/movies` : grille visible à **1 200 ms** sans, **874 ms** avec — **−326 ms (27 %)** pour 70 Ko de
`localStorage`. Le plancher est l'**hydratation (~820 ms en local, davantage en prod)** : le cache
n'est lu qu'après, il ne le franchit pas. Le squelette est visible **≈ 870 ms** (FCP 336 → grille 1 207) ;
seuls ~380 ms sont postérieurs à l'hydratation, c'est cette tranche que le cache attaque. Retirée du code ;
si un jour on la veut : `dehydrate`/`hydrate` de TanStack, hydrater **dans l'initialiseur de
`useState`** de `QueryProvider` (avant que les enfants montent), persister sur `getQueryCache().subscribe`
avec 1 s de debounce, en **SWR** (`refetchOnMount: "always"`) et **sélectif** (jamais `journal`/`takes`).
**Rejouable, pas en prose** : `git apply hq/archive/query-persist.patch` remet le persister à l'identique
(vérifié : apply → tsc → lint → `apply -R`), puis `node scripts/measure-cache.mjs /perso/watching/movies
--runs=5 --min-posters=40`.

**Le plancher d'hydratation — levier séparé, HORS phase 2 — et ce qu'il cachait.** J'avais écrit
« ~820 ms de JS + hydratation ». Faux d'un tiers : le contre-examen du 13/09 a lu la chronologie —
`/auth/v1/user` à 646 ms, **un second en série**, fin à 935, premier REST à 938. **~290 ms d'auth**,
pas de bundle : `settings/service.ts` (`requireUserId`, `getIsDemo`) faisait deux `auth.getUser()`
montés sur **chaque page**. C'était R2, corrigé le jour même (`getSession()` local — la base vérifie
le JWT à chaque requête de toute façon) : **premier appel REST 819 → 631 ms, grille 1 200 → 1 054 ms.**
Ce qui reste (~630 ms en local, davantage en prod) est le vrai plancher — JS + hydratation React —
et aucun cache ne le touche. Phase 5 (bundle, code-splitting par module), avec ce chiffre comme
référence. *Leçon : un « plancher » se décompose avant d'être déclaré incompressible.*

### R5 — Un robot écrit avec `upsert`, jamais avec `insert`.
Un cron peut être rejoué : un `net.http_post` asynchrone ne dit pas s'il a abouti, donc **toute écriture
doit pouvoir être répétée sans dégât**. Aujourd'hui : 14 `upsert` (dont **8 seulement avec
`onConflict`**) contre **3 `insert`**, tous les trois dans Tennis.
→ **Cliquet : RÉEL, à poser.** Test qui échoue sur un `.insert(` dans `supabase/functions/**`, et sur
un `.upsert(` sans `onConflict`.

### R6 — L'app ne parle jamais à un fournisseur externe pendant un rendu.
Onze fournisseurs, et le plus fragile n'est pas une API : **`tennisexplorer.com` est scrapé en HTML**
— il casse sans prévenir au premier changement de balisage. Corollaire de R3 : le chemin normal est
*cron → table → app*. Les routes `/api/*` sont l'exception assumée (elles tiennent les clés).
→ **Cliquet : partiellement STRUCTUREL.** Les clés ne sont pas dans le bundle, donc le navigateur ne
*peut* pas appeler la plupart d'entre eux. **FRAGILE** pour ceux qui passent par `/api/*`.

### R7 — Toute route `/api/*` qui consomme un quota externe porte un rate-limit.
5 routes sur 13 en ont un. ⚠️ **La protection est inversée par rapport au risque** : TMDB, dont le
quota réel est ~3 000/min, est plafonné à 120/min — tandis que les **4 routes qui appellent
`football-data.org`, dont le palier gratuit est de 10 requêtes/minute, n'ont aucune limite.**
→ **Cliquet : RÉEL, à poser.** Test qui parcourt `src/app/api/**/route.ts` et échoue si un fichier cite
un hôte externe sans importer un limiteur.

### R8 — Un échec silencieux est un BUG, pas une dégradation gracieuse.
C'est le mode de panne signature de HEGON, prouvé deux fois : les **40 h** de cron mort, et les
**28 jours** de football absent du dashboard — masqués par un `Promise.allSettled` qui faisait
exactement son travail. `.catch(() => {})` apparaît **12 fois**.
**La règle** : on n'avale que le cosmétique (`localStorage`, un log d'activité). Tout ce qui change ce
que l'utilisateur voit doit remonter à Sentry — dégrader l'affichage **et** signaler.
→ **Cliquet : RÉEL, à poser.** Lint interdisant `.catch(() => {})` et `catch {}` hors d'une liste
d'exceptions nommées (accès `localStorage`, télémétrie best-effort).

### R9 — ⭐ Les robots doivent être surveillés. Aujourd'hui personne ne les regarde.
**Zéro ligne de Sentry dans les 20 edge functions.** Elles écrivent 55 `console.error` dans les logs
Supabase, que personne n'ouvre. C'est exactement le trou des 40 h, et **il est toujours ouvert**.
Rappel de mécanique : `net.http_post` est **asynchrone** — un cron qui rapporte `succeeded` prouve que
le SQL s'est exécuté, pas que la fonction a réussi. La vérité vit dans `net._http_response`, conservé
~6 h.
→ **Cliquet : LE LIVRABLE EN CODE de la phase 2** (amendement 5) — l'alerte cron, §4.

**Ce que la première panne RÉELLE a appris (12-13/09)** — le watchdog a trouvé une panne récurrente
(12/09 12:00, 13/09 00:00 et 04:00) : `football_sync_standings` en 504 **de l'API REST Supabase** (pas
de football-data — lecture d'abord fausse, corrigée en lisant le code : `fetch(${SUPABASE_URL}/rest/v1/…)`),
et `watching-series-sync` mort sur un select `media_items` avec une erreur illisible (`String(e)` sur
une `PostgrestError` = `[object Object]`). Plus aucune série synchronisée pendant 12 h.
**Cause probable : la collision** — **12 crons sur 15 partaient à la minute :00** ; à 00:00 et 12:00,
trois fonctions edge la même seconde contre une base Free ; le 1er du mois à minuit, quatre.
Quatre règles en sont sorties, toutes en code :
1. **Jamais deux robots la même minute.** Migration `20260913000000_stagger_crons.sql` — 15 horaires
   distincts, cadences inchangées, :20 réservé au watchdog. Vérifié par script : aucune paire
   (minute, heure) partagée.
2. **Un robot réessaie une fois avant de mourir.** `supabase/functions/_shared/retry.ts` —
   `fetchWithRetry` (2 s, sur réseau/408/5xx) injecté dans `createClient({ global: { fetch } })`
   pour les 9 fonctions supabase-js, et à la place de `fetch(` pour les 13 appels REST bruts des 6
   fonctions football. Sans dégât au rejeu parce que les robots écrivent en `upsert` (R5).
3. **Une erreur se lit.** `errMsg(e)` partout où il y avait `String(e)` — la prochaine panne
   dira son nom.
4. **Une alerte au 2e échec consécutif, pas au premier.** `internal.job_health` (migration
   `20260913000100`), mise à jour par `job-health.sql` à chaque passage du watchdog ; le 1er échec
   s'imprime en information, la section B n'alerte qu'à partir de 2. Un cron à 6 h qui rate une
   fois = 12 h de données au pire, pas une alerte.
⚠️ **Vérifié par `deno check`** (Deno installé le 13/09 pour ça) : 13 fonctions sur 15 sans erreur ;
les 2 de Tennis portent **6 erreurs TS pré-existantes** (`deno_dom` typé `Node`), identiques avant/après
— le déploiement Supabase ne typechecke pas, elles tournent quand même. Dette pour l'audit Tennis.

### R10 — Tout changement de schéma passe par une migration versionnée.
C'est l'habitude inverse qui avait produit 28 tables hors migration. Règle **FRAGILE** aujourd'hui.
⚠️ **`db diff --linked` ne convient pas** (testé le 06/09 : diff massif et bruité → il finirait
désactivé). Le garde retenu est le **comptage d'objets**.
⚠️ **Piège vérifié le 2026-09-09, à ne pas reproduire** : le baseline contient **138** `create index`
alors que la base réelle en déclare **250** — l'écart, ce sont les index implicites des clés primaires
et des contraintes `unique`. **Comparer le fichier à la base rend le garde rouge dès le premier jour**,
c'est-à-dire exactement le piège qu'on cherchait à éviter. ⇒ **Le garde interroge la base des deux
côtés** : il compte dans Postgres, et compare à des valeurs de référence **stockées dans le repo**.
→ **Cliquet : à construire, §4.**

### R11 — Les données personnelles doivent survivre à la perte de la base.
La **structure** se reconstruit depuis le repo (phase 1). Les **données** ne se reconstruisent depuis
rien. Nuance qui change le coût : **1,3 Mo de référence** (matchs, classements, palmarès) sont
régénérables par les crons ; **13 Mo personnels** (journal, takes, habitudes, notes) sont
irremplaçables. *(Nuance retirée à la construction : la référence ne pèse que 10 %, l'exclure
n'économise rien et une liste d'exclusion peut rouiller — on sauvegarde tout.)*
→ **Cliquet : ÉCRIT le 2026-09-12, RUN VERT le jour même (12/12 étapes, push et round-trip inclus) —
pas encore « PROUVÉ »** : il le sera quand l'owner aura ouvert un fichier **à la main, à froid, avec
la passphrase du cahier**. Un round-trip dans le runner prouve que la machine sait déchiffrer ; il ne
prouve pas que l'humain saura, un jour de panique. [`backup.yml`](../../.github/workflows/backup.yml),
hebdomadaire, vers le dépôt privé `hegon-backups`, gzip + AES-256. Deux contre-examens avant le premier
run : **rouge par construction** dans sa première version (`pg_dump` n'écrit pas de guillemets sans
`--quote-all-identifiers` ; `[a-z_]+` excluait les 7 tables `f1_*`), puis le piège `pg_wrapper` au
premier run réel (voir §4A). Il vérifie sa sortie **cinq fois** (chaque table avec son
nombre exact de lignes · plancher de taille et jamais < 70 % de la précédente · gzip valide ·
**round-trip déchiffrement + recomptage avant le push** · aucune table n'a perdu > 20 % de lignes
depuis la sauvegarde précédente) et échoue bruyamment sinon. ⚠️ **Seul le comptage protège le
personnel** : `media_items` = 85 % du dump, `public.*` ≈ 200 Ko — la taille ne verrait pas un journal
vidé. RPO assumé : 7 jours. Procédure dans [`supabase/backup/RESTORE.md`](../../supabase/backup/RESTORE.md)
— qui inclut **le re-pointage de `internal.call_edge`** (URL et clé du projet codées en dur : sans ça,
les 15 crons d'un projet restauré appellent un projet mort, en silence).
**La vérification n°5 ne regarde que les tables PERSONNELLES** — règle calculée depuis le schéma :
référence = `sport.*` sans `user_id`/`org_id` + les 2 caches `watching` ; tout le reste est personnel.
Ni « tout `sport.*` » (abandonne `user_favorites`, `football_watched_matches`…) ni « a un `user_id` »
(abandonne `habit_completions`, rattaché par `habit_id`) ne suffisait seul. Validée sur les comptages
réels du 09/09 : la fin de tournoi tennis (table plafonnée, −25 % légitime) est ignorée, un journal
−67 % est détecté.
**Le témoin externe** : la **règle des 60 jours** (GitHub désactive les `schedule` d'un dépôt public
sans commit, sans run donc sans mail) est couverte par un **ping healthchecks.io** en fin des deux
workflows — il alerte sur l'*absence* de ping, un observateur qui ne dépend pas de ce qu'il observe.
Un secret de plus (`HEALTHCHECKS_PING_KEY`), décision owner du 2026-09-12.
**FRAGILE, assumé** : la passphrase (deux emplacements) · les fichiers Storage (hors Postgres).
OneDrive n'est **pas** un second emplacement : c'est un dump manuel figé, rien ne l'alimente.

---

## 3. LES ÉCARTS TROUVÉS PAR CETTE PASSE

Une ligne chacun, tous vérifiés par exécution. Ils alimentent la phase 3 et le §4.

1. **`revalidatePath("/tasks")` vise une route qui n'existe pas** — la vraie est `/pro/tasks`, et elle
   est cliente. Le seul usage de la revalidation Next dans tout HEGON est un no-op.
   [`tasks/actions/onboarding.ts:105`](../../src/modules/tasks/actions/onboarding.ts#L105)
2. **Les 4 routes `/api/football/*` tapent le quota le plus serré (10 req/min) sans rate-limit**,
   alors que TMDB (~3 000/min) est plafonné à 120. La protection est inversée. (R7)
3. **`FootballTeamPage` appelle `/api/football/enrich-team` à CHAQUE montage**, sans condition — le
   commentaire dit « no-op after the first visit », mais ce no-op est décidé **côté serveur**, donc
   l'aller-retour est payé à chaque ouverture, erreurs avalées.
   [`FootballTeamPage.tsx:140`](../../src/modules/sports/football/components/team/FootballTeamPage.tsx#L140)
4. **`FootballCompetitionPage` relance la synchro tant que la compétition est vide** — si le
   fournisseur échoue, la condition reste vraie et l'appel se rejoue à chaque visite, sans plafond ni
   signal. [`FootballCompetitionPage.tsx:51`](../../src/modules/sports/football/components/competition/FootballCompetitionPage.tsx#L51)
5. **31 `auth.getUser()` subsistent**, dont 8 dans `tasks/service.ts` — la règle CSR existe depuis
   juin et n'a jamais eu de cliquet. (R2)
6. **6 `upsert` sur 14 n'ont pas d'`onConflict`, et 3 `insert` subsistent** dans les edge functions
   Tennis — un rejeu peut dupliquer. (R5)
7. **52 `staleTime` sur 126 ne font que répéter le défaut** — bruit qui masque les 74 vraies
   décisions. (R4)

---

## 4. CE QUE LA PHASE 2 DOIT LIVRER EN CODE

> Règle §2 du chantier : rien n'atterrit en prose seule.

**A. Les 3 jobs programmés — la même plomberie** (*« un job qui prévient que quelque chose a cassé en
silence »*). *(Écrit d'abord « un seul workflow, un seul secret » ; faux à la construction : **deux
workflows** — le watchdog toutes les 6 h, la sauvegarde hebdomadaire, deux horloges ne se mélangent
pas — et **trois secrets** : connexion, passphrase, deploy key.)*

| job | ferme | cadence | état au 2026-09-12 |
|---|---|---|---|
| **sauvegarde** → dépôt privé `hegon-backups`, chiffrée, 5 auto-vérifications | R11 · le seul poste irréversible | hebdomadaire | **run vert 12/12** (12/09 17:50 UTC), push et round-trip inclus — **PROUVÉ le 2026-09-13** : ouverture manuelle par l’owner (Git Bash, pinentry, passphrase du cahier) → 17 Mo, 7 399 `INSERT` |
| **alerte cron** — `cron.job_run_details` + `net._http_response` + `internal.call_log` | R9 · la panne de 40 h | toutes les 6 h | vert depuis le 10/09 ; **première vraie alerte le 12/09** (500 de `football_sync_standings`) — elle a révélé deux défauts, corrigés le jour même (ci-dessous) |
| **garde anti-dérive** — tables/policies/fonctions/index/**colonnes**, comparés aux valeurs de référence du repo | R10 · la règle fragile | toutes les 6 h | vert, référence semée (68 · 684 · 206 · 35 · 250 · 15) |

Plus le **témoin externe** (healthchecks.io) sur les deux workflows — il alerte sur l'*absence* de run.

⚠️ **Le garde doit compter les COLONNES.** Compter tables/policies/fonctions/index ne détecte **pas**
une colonne ajoutée à la main — or c'est précisément le geste que la règle R10 interdit. Sans la
mesure colonne, le garde est aveugle à son propre cas d'usage.

**Ce que la première vraie alerte a appris (12/09)** — deux défauts de l'alerte cron, traités en
phase 2 parce qu'un garde qui crie au loup finit désactivé :
1. `net._http_response` **ne garde pas l'URL** → l'alerte disait « fonction non identifiée ». Corrigé :
   `internal.call_log`, écrit par `call_edge` à chaque appel (migration `20260912000000`), joint sur
   `request_id`. ⚠️ La procédure de restauration re-pointe **cette** version de la fonction.
2. Toute réponse non-2xx des 6 h alertait, même dépassée par un succès. Corrigé : seule la **dernière**
   réponse de chaque fonction compte — l'état courant, pas l'historique.

**Le piège `pg_wrapper` (12/09, premier run de la sauvegarde)** : `/usr/bin/pg_dump` sur le runner est
l'aiguilleur Debian et route vers le cluster **16** de l'image, même après `apt install
postgresql-client-17`. `pg_dump --version` **affichait** 16.15 et l'étape passait au vert — **afficher
n'est pas vérifier** (R8). Le watchdog tournait en psql 16 sans le savoir depuis deux jours. Remède
dans les deux workflows : `/usr/lib/postgresql/17/bin` en tête du `PATH` + version **exigée** par `grep`.

**B. Les 6 cliquets de lint/test** nommés en R2, R3, R4, R5, R7, R8 :

| règle | cliquet | état |
|---|---|---|
| R3 · N+1 de service | ESLint `NO_N_PLUS_ONE_IN_SERVICES` — 3 formes : `.map(async` avec await · boucle avec await · `.map(` sync qui construit une requête | **POSÉ 12/09, élargi 13/09** — 8 dettes annotées ; passent encore `Array.from`, `.then`, helper (FRAGILE) |
| R4 · `staleTime` = palier | ESLint `STALE_TIME_IS_A_TIER` (**`gcTime` aussi depuis l'audit Watching**) + `STALE_DEFAULT_IS_INHERITED` + `stale.ts` + test `stale.test.ts` (gcTime) | **POSÉ 12/09, durci 13/09, élargi le soir** — 126 sites migrés, 21 gcTime alignés, puis **10 `gcTime` bruts** trouvés par l'audit Watching (le sélecteur ne lisait que `staleTime`) → migrés, refus vérifié |
| R2 · `auth.getUser()` hors lecture | ESLint sur `service.ts`, exceptions nommées | à poser — **Watching : 4 → 0** (audit 13/09, `getCurrentUserId()`) |
| R5 · `upsert` + `onConflict` dans les edge functions | test sur `supabase/functions/**` | à poser |
| R7 · rate-limit sur toute route qui cite un hôte externe | test sur `src/app/api/**` (corriger football d'abord, sinon rouge jour 1) | à poser |
| R8 · pas de `.catch(() => {})` hors exceptions | ESLint, liste d'exceptions (`localStorage`, télémétrie) | à poser — **l'ancrage existe** : `shared/utils/report-error.ts` (audit Watching, 3 avaleurs branchés) ; le lint exigera « un `catch` appelle `reportError` ou est dans la liste » |

⚠️ **Composition en flat config** (trouvé en posant R3/R4) : `rules` ne fusionne pas — le dernier bloc
qui matche un fichier **remplace** `no-restricted-syntax` en entier. Un bloc `src/**` effaçait le garde
des coordonnées Watching en silence. Chaque bloc liste donc **tous** ses sélecteurs (constantes
partagées en tête de `eslint.config.mjs`), du plus large au plus précis.
**Leçon esquery, corrigée le 13/09** — j'avais écrit « un `>` en tête d'un `:has()` ne matche rien » :
faux. `>` veut dire *enfant direct*, sémantique normale : `:has(> AwaitExpression)` matche
`async (x) => await x` (le corps EST l'await) et ne matche pas `async (x) => { … await … }` (l'enfant
direct est un `BlockStatement`). Écrire `Parent > Callback:has(AwaitExpression)`. **Vérifier un
sélecteur sur les deux formes de corps avant de le poser.**

Ajoutés au contre-examen du 13/09 : **`STALE_DEFAULT_IS_INHERITED`** (R4 — `STALE.DEFAULT` refusé hors
`QueryProvider`, qui porte l'unique `eslint-disable`) et **`stale.test.ts`** (R4 — un palier > 10 min
sans `gcTime` ≥ est un mensonge, le lint ne sait pas comparer deux propriétés, le test lit le code).

**C. Restent FRAGILES, assumées** : R1 (mesurée, non bloquante), R6 pour les routes `/api/*`, la
cascade de montage (R3, mesurée par `measure-cache.mjs`, non détectable statiquement), le 3e étage de
cache (`cache()`/`revalidatePath`, un usage mort), l'arbitrage `refetchOnWindowFocus` par module (R4),
et l'exactitude de `hq/` + `memory/` (dépôt privé séparé, la CI publique ne peut pas les voir).
