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
`getUser()` est un aller-retour réseau vers Supabase Auth (~150-300 ms mesurés) ; `getClaims()` vérifie
le JWT en process. Il reste **31 appels à `auth.getUser()`**, dont **8 dans `tasks/service.ts`** et 4
dans `watching/service.ts` — sur des chemins de lecture.
→ **Cliquet : RÉEL, à poser.** Règle ESLint `no-restricted-syntax` interdisant `auth.getUser()` dans
`src/modules/**/service.ts`, avec une liste d'exceptions nommées (flux d'auth, écritures qui doivent
*vérifier* l'utilisateur, pas seulement le lire).

### R3 — ⭐ Le cache a DEUX étages. Pas quatre.
Le plan en supposait quatre. Vérifié : **`cache()` de React n'est utilisé nulle part**, et la
revalidation Next l'est **une seule fois** — un `revalidatePath("/tasks")` qui vise une route
**inexistante** (la vraie est `/pro/tasks`) et cliente de surcroît. Les deux étages du milieu n'existent
pas. Il reste :
- **TanStack** (client) — fraîcheur d'une donnée que l'utilisateur vient peut-être de modifier.
- **Une table-cache en base** (`for_you_cache`, `trending_cache`, `football_standings`,
  `football_competition_winners`, `football_team_honours`, `tennis_players_cache`) — dès qu'un calcul
  est cher ou qu'un quota externe est en jeu. **Un cron remplit, l'app lit. L'app n'appelle jamais un
  fournisseur pendant un rendu.**
→ **Cliquet : RÉEL, à poser.** Un test qui échoue si `cache(` de React ou `revalidatePath/Tag`
réapparaît sans être ajouté à la doctrine — un troisième étage doit être une décision, pas une dérive.

### R4 — Le `staleTime` par défaut est de 5 minutes. Ne le redéclare pas.
126 `staleTime` déclarés, **52 répètent exactement le défaut** de [`QueryProvider`](../../src/shared/providers/QueryProvider.tsx).
Ce bruit cache les 74 vraies décisions (0 · 30 s · 2 min · 10 min · 30 min · 1 h · 24 h · 7 j). Une
valeur non-défaut est une **affirmation sur la tolérance à la donnée périmée** et doit porter sa raison.
→ **Cliquet : RÉEL, à poser.** `no-restricted-syntax` sur `staleTime: 1000 * 60 * 5` (et ses écritures
équivalentes). Le commentaire de justification reste **FRAGILE**.

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
irremplaçables. Une sauvegarde **ciblée sur le personnel** est donc petite et rapide.
→ **Cliquet : à construire, §4.** Aujourd'hui chaque dump est manuel ⇒ **la pente n'est pas corrigée**,
seul le compteur l'est.

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
silence »*), donc **un seul workflow**, trois jobs, **un seul secret** (chaîne de connexion Postgres) :

| job | ferme | cadence |
|---|---|---|
| **sauvegarde** des tables personnelles → dépôt privé | R11 · le seul poste irréversible | hebdomadaire |
| **alerte cron** — lit `cron.job_run_details` + `net._http_response`, signale un échec ou un silence | R9 · la panne de 40 h | quotidienne |
| **garde anti-dérive** — compte tables/policies/fonctions/index/**colonnes** dans la base, compare aux valeurs de référence du repo | R10 · la règle fragile | hebdomadaire |

⚠️ **Le garde doit compter les COLONNES.** Compter tables/policies/fonctions/index ne détecte **pas**
une colonne ajoutée à la main — or c'est précisément le geste que la règle R10 interdit. Sans la
mesure colonne, le garde est aveugle à son propre cas d'usage.

**B. Les 5 cliquets de lint/test** nommés en R2, R3, R4, R5, R7, R8.

**C. Restent FRAGILES, assumées** : R1 (mesurée, non bloquante), R6 pour les routes `/api/*`, la
justification en commentaire d'un `staleTime` non-défaut, et l'exactitude de `hq/` + `memory/` (dépôt
privé séparé, la CI publique ne peut pas les voir).
