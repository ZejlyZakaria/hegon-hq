# HEGON — Plan d'audit & de fondation

> **Document de pilotage du chantier.** Décidé avec l'owner le 2026-09-05. La conversation d'exécution
> démarre en lisant CE fichier, pas à froid. Il est mis à jour au fil des passes — c'est lui qui dit
> où on en est.
>
> 🟢 **Sauvegarde AUTOMATISÉE depuis le 2026-09-12** — `backup.yml`, hebdo, chiffrée, vers
> `hegon-backups`, 5 auto-vérifications, témoin healthchecks.io ; **run vert 12/12**. La structure
> (migrations) et les données sont couvertes. **R11 reste « run vert », pas « prouvé »**, tant que
> l'owner n'a pas ouvert un fichier à la main, à froid, avec la passphrase du cahier. *(Ce bandeau
> disait « 🔴 non automatisée » jusqu'au 13/09 — un jour de retard sur le code, corrigé.)*
>
> ⛔ **Gel des features — PAR MODULE, jusqu'à la fin de SON audit de phase 3** *(amendement 6,
> 2026-09-12 ; remplace « pendant tout le chantier »)*. Un module non audité est gelé. Un module
> audité (9 axes + tests) peut recevoir des features, dans une conversation dédiée, à **trois
> conditions** — voir Phase 3. Sport (page stats football, tennis, F1) reste gelé tant que son audit
> n'est pas fait. **Watching sera le premier dégelé.**

---

## 1. Pourquoi ce chantier

L'owner a formulé le problème ainsi : *« HEGON sort de ma main, trop de trucs que je ne comprends
pas. »* Ce n'est pas une impression, c'est mesurable. Constats faits le 2026-09-04/05 :

- **`CLAUDE.md` porte 6 blocs « ⚠️ Corrected » / « Superseded »** — le dashboard décrit des semaines
  après avoir été remplacé, la Sidebar documentée alors que le Dock l'avait tuée, la police annoncée
  en Geist longtemps après Inter, les couleurs Books et Journal fausses.
- **`MEMORY.md` prévient lui-même que 19 fichiers d'archive mentaient sur leur propre statut.**
- **`knip` trouve ~1 400 lignes d'UI orpheline dans le seul module football** (`FootballFanLog`,
  toute la chaîne `BestXI`/`Legends`, `useFanLog`, `useFootball`).
- **Une mémoire donne un chemin faux** (`shared/ui`, dossier vide ; le vrai est `shared/components/ui`).
- **Un skeleton fantôme** : `loading.tsx` rend `FootballBestXISkeleton` pour un bloc retiré de la page.
- **`hq/now.md` fait 555 lignes / 47 KB** alors que le session-protocol dit qu'il est lu EN ENTIER au
  démarrage et doit « rester court ».

**Diagnostic en une phrase : le code va plus vite que sa description.** C'est pour ça que l'owner
n'arrive pas à expliquer HEGON de A à Z — la description qu'il se réciterait est périmée.

---

## 2. Les règles du chantier (non négociables)

**⭐ Rien n'atterrit en prose seule.** Chaque passe finit en **code committé ou en fichier supprimé**.
Un audit qui ne produit que des `.md` aura donné trois semaines de confort avant de redevenir du bruit
— la preuve est au §1.

**⭐ Un mécanisme, jamais un état.** « Zéro dead code » est vrai un jour et faux le lendemain. Ce qu'on
installe, c'est un **cliquet** : `knip` en CI qui casse le build, `ds:coverage` en CI — comme le
pre-commit `--max-warnings 0` qui existe déjà. L'objectif n'est pas de nettoyer, c'est de rendre
l'accumulation impossible.

**⭐ Gel des FEATURES, pas gel du code.** L'audit produit des suppressions, des renommages, des gardes.
C'est du code, et c'est voulu.

**⭐ Un rapport = un écran.** Si un rapport de module ne tient pas sur un écran, c'est un roman, pas un
audit. Une note par axe + les écarts en une ligne chacun avec lien fichier. Pas de prose.

**⭐ Chaque règle doit être défendable seule.** Watching sert à TROUVER les règles candidates, pas à les
valider. Si la seule justification est « c'est comme ça dans Watching », on jette la règle — sinon on
canonise ses accidents.

**⭐ Toute règle nomme son cliquet, ou se déclare FRAGILE.** *(amendement 1, validé le 2026-09-06)*
Chaque règle de doctrine doit répondre à : **« quel test, quelle règle de lint ou quel script la rend
impossible à violer ? »** Celles qui n'ont pas de réponse sont marquées **fragiles**, explicitement —
on les garde, mais on sait qu'elles ne tiennent que par la discipline. Sans cette contrainte, la phase 2
violerait la règle « rien n'atterrit en prose seule » : une doctrine sans cliquet n'est pas une règle,
c'est un vœu. Le projet a déjà le bon modèle chez lui — `/settings/styleguide` ne peut pas mentir parce
qu'elle lit les vrais tokens à l'exécution, et `npm run map` régénère la carte.

---

## 3. Calibration des objectifs de l'owner (corrections assumées)

| Formulé | Recalibré | Pourquoi |
|---|---|---|
| « Zéro code inutilisé » | Un **cliquet** en CI | Zéro est un état, pas un processus ; du code meurt à chaque refactor |
| « On arrête de coder » | Gel des **features** | Un audit qui n'atterrit qu'en docs pourrit (cf. §1) |
| « Comprendre 100 % » | **Rien ne peut me surprendre + je sais où chercher** + je connais à fond les ~20 % qui portent le risque | 100 % n'est atteint par personne ; viser ça, c'est se garantir un échec |
| « Supprimer tout `.md` inutile » | Supprimer **sauf les résultats négatifs** | « On a testé, c'est payant, ne relance pas » n'est pas actionnable et vaut deux jours (cf. `reference_football_data_sources`) |

**⭐ L'angle mort à combler : les TESTS.** « Vibe-codé » ne veut pas dire *mal compris*, ça veut dire
**qu'on ne peut pas le modifier sans peur**. La compréhension s'évapore en trois semaines ; les tests
restent. Constat qui n'est pas une coïncidence : **14 fichiers de tests, dont 8 dans Watching** (+1
Tasks, Books, Habits, Journal) et **ZÉRO dans football / tennis / F1** — exactement les modules dont
l'owner est satisfait vs ceux dont il ne l'est pas. Les tests ont forcé la séparation logique/affichage,
et c'est cette séparation qu'il ressent comme de la qualité.

---

## 4. Les phases, dans l'ordre

### Phase 0 — LA CARTE (globale, mécanique, courte)

Constat pur, aucun jugement. Routes réelles · graphe de dépendances · surface partagée · tables DB
confrontées à leur usage réel dans le code · crons et edge functions confrontés à ce qui les appelle
vraiment · sortie `knip` complète.

**Livrable : UNE page que l'owner peut redessiner de mémoire.** C'est le « expliquer HEGON à
n'importe qui ». Si elle n'est pas redessinable, c'est un dépotoir, pas une carte.

### Phase 1 — LA PURGE ET LES CLIQUETS

On supprime ce que la phase 0 a **prouvé** mort, et on installe les gates pour que ça ne revienne pas.
Court, très satisfaisant, rétrécit immédiatement la surface à comprendre.

### Phase 2 — LA PASSE SYSTEM DESIGN (globale — produit LA DOCTRINE)

⚠️ **Vient AVANT les audits par module, et c'est délibéré.** Elle produit la doctrine ; la checklist
par module ne fait ensuite que **vérifier la conformité**. Sans elle, un axe « cache et état » n'a rien
contre quoi être jugé — on constaterait que Watching fait du client-side et Sport du server-side sans
pouvoir dire lequel a raison. Étant globale, elle ne souffre pas du problème des 40 pages.

⚠️ **PRÉALABLE — la session « contrat », à faire AVANT d'ouvrir la phase 2** *(amendement 3, validé le
2026-09-06)*. Ce n'est pas un item d'audit : tout le reste se découvre en lisant le code, ça non. C'est
une **décision qui n'appartient qu'à l'owner**, et tant qu'elle n'est pas prise, la moitié des autres
axes n'ont pas de critère. L'ambiguïté coûte déjà : **trois futurs cohabitent dans la base** — l'infra
multi-tenant (`org_id`, `my_orgs()`, partage de workspace), la garde read-only démo sur 6 modules, et
l'Access Control documenté en 6 étapes et jamais construit — pour un usage mono-utilisateur.

⭐ **LIVRABLE EN CODE — l'alerte cron** *(amendement 5, validé le 2026-09-06)*. Les axes 7 et 8 ne
doivent pas produire un paragraphe décrivant le trou : le livrable est **l'alerte qui manque**, celle
qui prévient qu'un cron a échoué en silence. C'est ce qui ferme la pire panne connue (40 h), c'est du
code, et ça satisfait la règle §2. Nommé ici pour qu'il ne se dissolve pas en observation.

### Phase 3 — LES AUDITS PAR MODULE (sérialisés, un par un)

Contre la doctrine de la phase 2 + la checklist du §6. **Watching en premier** (l'étalon).

⭐ **LES TESTS VOYAGENT AVEC CHAQUE MODULE — ils ne sont plus en phase 5** *(amendement 2, validé le
2026-09-06)*. Diagnostiquer les tests comme l'angle mort (§3) puis les programmer en dernier, après un
gel de features de six phases, était une contradiction — la passe la plus utile aurait été celle qui
avait le plus de chances de ne jamais arriver. Chaque audit de module se termine donc par **les tests de
sa logique pure**, écrits dans la foulée. C'est l'axe 5 du §6, et il devient bloquant.

⚠️ **Le deuxième ne doit PAS être le deuxième meilleur.** Prendre un module jugé raté **mais petit** :
la grille se fait mordre pour de vrai, la liste de corrections reste finie, et on obtient un
« avant/après » complet rapidement. Lequel : à choisir une fois la carte faite (mesurer, pas deviner).

⚠️ **Attente à poser** : la passe Watching produira surtout des **règles**, pas des corrections. C'est
normal — c'est l'étalon. Son livrable est la checklist elle-même.

⭐ **LE DÉGEL PAR MODULE** *(amendement 6, décidé le 2026-09-12)*. Le gel des features couvre chaque
module **jusqu'à la fin de son audit de phase 3** (9 axes + tests), pas jusqu'à la fin du chantier —
sinon la passe la plus utile pour l'owner (construire) attend six phases, et le chantier redevient le
tunnel qu'il devait éviter. Un module audité peut recevoir des features, **dans une conversation
dédiée**, à trois conditions, toutes bloquantes :
1. **La feature existe dans le `backlog.md` du module, avec son déclencheur** — pas d'idée du moment.
2. **Elle respecte la doctrine de la phase 2** (`hq/rules/system-design.md`) — c'est à ça que sert
   d'avoir audité le module d'abord.
3. **Elle ne touche aucun module non audité.** Si elle en touche un, elle attend l'audit de ce
   module. C'est la condition qui empêche le dégel de contaminer le gel.
L'audit d'un module ne le rend pas seulement conforme : il le **rouvre**. Watching sera le premier.

### Phase 4 — LA VÉRITÉ DES DOCS

Chaque `.md` du repo et chaque mémoire : **vivant / archivé / supprimé**. Résoudre les contradictions
connues (au moins une explicite : une mémoire déclare « prime sur CLAUDE.md §5 »). Traçabilité gardée
mais **dans `archive/`, jamais dans ce qu'on lit au quotidien**. Inclut la cure d'amaigrissement de
`hq/now.md` (555 lignes lues à chaque démarrage).

### Phase 5 — LES RÈGLES UI/UX ET LE DESIGN SYSTEM

En dernier **volontairement** : c'est la passe la plus coûteuse, et écrire des règles pour des modules
qu'on va restructurer, c'est les écrire deux fois. Écrites en **CODE** (primitifs + `ds:coverage`), pas
en prose. La base existe déjà : `npm run ds:coverage`, `/settings/styleguide`, Watching = pilote fini.

⚠️ **Les tests ne sont plus ici** — ils ont été remontés en phase 3, où ils voyagent avec chaque audit
de module *(amendement 2)*.

---

## 5. Checklist SYSTEM DESIGN (phase 2 — 11 axes)

Taillée sur ce que HEGON EST réellement.

⚠️ **Hors périmètre assumé** : sharding, réplication, load balancers, files distribuées, CAP, scaling
horizontal. Ce sont des sujets d'entretien pour des systèmes à millions d'utilisateurs ; HEGON est
mono-utilisateur sur serverless et n'en aura jamais besoin. Les apprendre demandera un autre véhicule.

1. **Le contrat** — ⚠️ **N'EST PLUS UN AXE D'AUDIT** *(amendement 3)*. C'est une **session de décision
   owner**, à tenir **avant** d'ouvrir la phase 2 : voir le préalable en §4. Ce que HEGON promet
   vraiment (mono-utilisateur ? démo publique ? multi-tenant via `org_id` ?) et ses cibles
   non-fonctionnelles. Jamais écrit ; tout le reste en découle. Sa réponse ne se trouve pas dans le code.
2. **Le modèle de données** — 3 schémas, préférence (`org_id`) vs référence globale, RLS, indexation.
3. **Le chemin d'une requête** — auth, middleware, frontière serveur/client, **cascades séquentielles**.
   C'est ici que vit la perf, donc ici que se répond la question « aussi rapide qu'Instagram ».
4. **La doctrine de cache** — les 4 étages (React `cache()`, TanStack, revalidation Next, tables-cache
   en base type `for_you_cache` / `football_competition_winners`) : qui possède quoi, TTL, invalidation,
   tolérance à la donnée périmée.
5. **Le travail asynchrone** — 14 crons, edge functions, idempotence, reprises, quotas externes.
6. **Les dépendances externes** — que se passe-t-il quand un fournisseur change de tier ou **meurt**
   (Ergast est mort → migration jolpica).
7. **Les modes de défaillance et le rayon d'explosion** — cas d'étude : **les 40 h de panne silencieuse**
   (`net.http_post` async, timeout 5 s, cron rapportant `succeeded`).
8. **L'observabilité** — ce qui est instrumenté, et surtout ce qui ne l'est pas (cf. axe 7).
9. **Déploiement, migrations, retour arrière** — le point le plus fragile : migrations appliquées **à la
   main**, `schema_migrations` vide.
10. **Secrets et moindre privilège** — le split de clés par rôle est bon ; l'owner doit savoir l'expliquer.
11. ⭐ **Durabilité & reconstructibilité** *(amendement 4, validé le 2026-09-06)* — **l'axe manquant, et
    probablement la question système n°1** pour une app qui accumule des années de journal, d'historique
    de visionnage et d'habitudes : *que se passe-t-il si je perds la base, et en combien de temps je
    reviens ?* Il était absent des 10 axes et traînait dans `project_ideas_backlog` comme une « idée » —
    le mauvais rayon. Couvre : sauvegarde, **restauration testée**, rétention, et la capacité du repo à
    reconstruire l'app entière (schéma + crons + edge functions + secrets).
    **Traité en phase 1 (05-09/09)** : sauvegarde faite, restauration **prouvée**, reconstructibilité
    fermée. 🔴 **IL RESTE UN TROU, ET C'EST LE DERNIER : la RÉGULARITÉ.**

    **La structure est résolue** — chaque changement passe par une migration versionnée dans git, donc
    le repo suit tout seul. ⚠️ Une seule condition, et c'est l'ancienne habitude qui est le risque :
    **tout changement DOIT passer par une migration**. Une colonne ajoutée à la main dans l'éditeur SQL
    et le repo redevient faux — c'est exactement ce qui avait produit les 28 tables sans migration.
    **Cette règle n'a AUCUN cliquet** (règle §2 ⇒ elle est **FRAGILE**, assumée).

    **Les données ne le sont pas** : `data.sql` est figé au 2026-09-06 et l'écart grandit chaque jour.
    La structure se reconstruit depuis le repo ; **les données ne se reconstruisent depuis rien.**

    ⭐ **Nuance qui change le calcul — toutes les données ne se valent pas** : les **1,3 Mo de
    référence** (matchs, classements, palmarès) sont **régénérables** par les crons ; les **13 Mo
    personnels** (takes, notes, journal, habitudes) sont **irremplaçables**. Une sauvegarde ciblée sur
    le personnel serait bien plus petite et plus rapide.

    **Options évaluées le 2026-09-09** :
    - ⭐ **Action GitHub programmée (recommandée)** — hebdomadaire, sur les tables personnelles, vers le
      repo privé `hegon-hq` ou en artefact. Tourne **sans le PC allumé, sans Docker**, gratuit.
      ⚠️ 2 points à régler : le runner a `pg_dump` **16** alors que le serveur est en **17.6** (installer
      le client PGDG 17), et choisir la destination + la rétention. Mot de passe en secret GitHub.
    - Script local + planificateur Windows — simple mais exige PC allumé + Docker. Fragile.
    - Manuel avec rappel (via le module Habits !) — repose sur la discipline seule, donc pourrira.
    - Supabase Pro (25 $/mois) — règle tout, mais coûte de l'argent.

---

## 6. Checklist PAR MODULE (phase 3 — 9 axes)

Grille **identique pour tous** : c'est ce qui rend les modules commensurables. Une note par axe.

1. **Structure & anatomie** — conformité au pattern (`service.ts`, `types.ts`, `hooks/query-keys.ts`,
   `components/`) ; chaque fichier a un rôle nommable et des connexions connues.
2. **Dead code** — zéro, et prouvé par `knip`.
3. **Couche données** — discipline du `service.ts` (aucune requête inline), forme des requêtes.
4. **Cache & état** — conformité à la doctrine de la phase 2 ; clés TanStack ; périmètre Zustand.
5. **Tests de la logique pure** — la logique est-elle séparée de l'affichage, et testée.
6. **Skeletons** — fidélité de forme au contenu réel + **colocation** (pas d'inline, pas de squelette
   qui promet un bloc absent).
7. **Design system** — tokens et primitifs, mesuré par `ds:coverage`.
8. **UI/UX** — composition (leçon du 19/08 : le problème n'est pas la déco, c'est la composition),
   4 devices (⚠️ iPad portrait), empty states.
9. **Documentation du module** — ce qui doit être écrit, et où.

---

## 7. État d'avancement

| Phase | Statut |
|---|---|
| 0 — La carte | ✅ **faite le 2026-09-05** → [`map.md`](map.md) · chiffres regénérables par `npm run map` |
| 1 — Réduire le risque (purge + cliquets) | ✅ **TERMINÉE le 2026-09-09** — voir §8 |
| 2 — System design (doctrine) | ✅ **FERMÉE le 2026-09-13.** **Mécanique** : doctrine [`system-design.md`](../rules/system-design.md) R1-R11 · watchdog (A · B **au 2ᵉ échec** · B-bis · C · dérive du schéma · témoin externe · `job_health`) · sauvegarde hebdo chiffrée + 5 vérifications · cache (paliers + ESLint + N+1 + `gcTime`) · R9 crons décalés + retry + `errMsg`. **Dette annotée, avec déclencheur** : R2 (~22 `auth.getUser()`, dont le double en série corrigé le 13/09) · R5 (`upsert` sans `onConflict`, 3 `insert` Tennis — protégés par des UNIQUE, 409 non retenté) · R8 (`.catch(() => {})` ×12) → **l'audit de chaque module ramasse les siens** ; R7 (4 routes football sans rate-limit, quota 10/min) → **audit Sport**. **Les 7 écarts §3 de la doctrine**, un par un : ① `revalidatePath("/tasks")` mort → dette, audit Tasks · ② rate-limit football → dette R7, audit Sport · ③ `enrich-team` à chaque montage → dette, audit Sport · ④ synchro compétition relancée tant que vide → dette, audit Sport · ⑤ `getUser()` : **chemin critique corrigé** (settings ×2 + helper partagé, REST 819 → 631 ms), ~22 restants → dette R2 · ⑥ `upsert`/`insert` Tennis → dette R5, audit Tennis · ⑦ `staleTime` redéclarés → **fait** (126 migrés, 51 supprimés). **R11 : run vert, ouverture manuelle par l'owner NON FAITE** — la ligne ne dira « prouvé » qu'après. Premier vrai test de R9 : 13/09 12:00 UTC. |
| 3 — Audits par module (Watching d'abord) | ▶️ **À OUVRIR — brief d'ouverture ci-dessous.** |
| 4 — Vérité des docs | ⬜ |
| 5 — UI/UX + design system | ⬜ |

**Reprise Sport après SON audit de phase 3** *(amendement 6 — plus « après le chantier »)* : page stats
football — la route n'existe pas. La logique manquante est le **record de prédictions** (les
prédictions sont stockées mais jamais confrontées au résultat). À inscrire dans
`hq/modules/sports/backlog.md` avec son déclencheur, sinon la condition 1 du dégel la bloque.

⚠️ **`FanLog` + `BestXI`/`Legends` ne sont PLUS dans le code** — supprimés par la purge du 2026-09-07
(voir §8). **1 395 lignes récupérables intégralement** depuis le commit **`2b0b3fb`** :

```bash
git show 2b0b3fb --stat                                    # voir les 8 fichiers
git checkout 2b0b3fb -- src/modules/sports/football/components/squad/     # BestXI + Legends (1 272 l.)
git checkout 2b0b3fb -- src/modules/sports/football/components/fanlog/    # FanLog (95 l.)
git checkout 2b0b3fb -- src/modules/sports/football/hooks/useFanLog.ts
```

Le service garde `getFanLog()` et `getBestXI()` — seuls les composants sont partis. ⚠️ Ils portaient
encore de l'`emerald` codé en dur (avant le passage au lime), donc une reprise telle quelle demandera
de toute façon une passe couleur.

---

## 7 bis. BRIEF D'OUVERTURE — Phase 3, audit Watching

> **Pour une conversation vierge.** Ce bloc est la première chose qu'elle lit après l'en-tête du plan.
> Écrit le 2026-09-13 à la clôture de la phase 2. Watching est **l'étalon** : sa passe produira
> surtout des règles ; c'est normal, son livrable est la checklist elle-même — et il sera le
> **premier module dégelé** (amendement 6).

**Lire, dans cet ordre, et rien d'autre au départ :**
1. **§6 ci-dessus** — la checklist des 9 axes. C'est la grille, identique pour tous les modules.
2. [`hq/rules/system-design.md`](../rules/system-design.md) — la doctrine R1-R11 : l'axe 4 (cache & état)
   se juge contre R3/R4, l'axe 3 (couche données) contre R2/R5, l'axe 2 (dead code) contre `knip`.
3. [`hq/modules/watching/backlog.md`](../modules/watching/backlog.md) — **sa première tâche est la
   première tâche de l'audit** : réconcilier les 4 fichiers du dossier (`backlog`, `vnext`,
   `architecture`, `decisions` — 1 098 lignes) **contre le code**, par `git log` et l'arborescence,
   jamais par lecture des `.md` entre eux. `vnext.md` a menti trois mois et demi. Ce qui existe est
   rayé ; ce qui reste porte un déclencheur ; **un seul fichier vivant survit**.
4. La carte : [`map.md`](map.md) §1 pour la forme, `npm run map` pour les chiffres du jour.

**Ne PAS lire au départ** : `hq/now.md` en entier (592 lignes, histoire), `hq/log/`, les archives.

**Le format du rapport — UN ÉCRAN.** Une note par axe (9 lignes), puis les écarts, **une ligne
chacun avec le lien fichier:ligne**. Pas de prose. S'il ne tient pas sur un écran, c'est un roman,
pas un audit (§2). Il vit dans `hq/modules/watching/audit-2026-09.md` et remplace toute description
d'état antérieure.

**L'axe 5 — les tests — est le DERNIER et il est BLOQUANT** (amendement 2). L'audit ne se ferme pas
tant que la logique pure de Watching n'est pas séparée de l'affichage **et testée** ; Watching a déjà
9 fichiers de tests, la passe les complète là où l'audit trouve de la logique nue. Un module « audité »
sans ce dernier axe n'est pas audité — et ne se dégèle pas.

**Ce que la phase 2 lui laisse en entrée, déjà trouvé, à vérifier et à classer :**
- `watching/service.ts` = 1 787 lignes, le plus gros fichier du repo (axe 1) · 4 `auth.getUser()`
  (axe 3, R2) · 1 N+1 annoté (iTunes par morceau, axe 3, R3) · 2 hooks à `refetchOnWindowFocus`
  `false` explicite (axe 4, R4 : arbitrage multi-appareil par module).
- La cascade mesurée sur `/perso/watching/movies` : 15 requêtes en 4 vagues, dont **une cascade de
  montage** (`useWatchActions` → `useWatchingGoals`) et une dépendance réelle (`useOwnedMediaIds`)
  — axe 3, à corriger ici, pas en phase 2. Instrument : `node scripts/measure-cache.mjs /perso/watching/movies --min-posters=40`.
- Le bug Haikyu!! (`series-sync` écrit `watched`/`in_progress` en direct, viole ⭐⭐ « un seul
  écrivain de statut ») — axe 3, déjà diagnostiqué dans le backlog.
- Le diagnostic du 10/09 (For You sans signal négatif, 24 titres want-to-watch invisibles,
  `priority_level` jamais éditable) — axe 8, déjà dans le backlog.

**Cette conversation n'ajoute AUCUNE feature.** Le dégel (amendement 6) prend effet à la clôture de
son rapport, pas à son ouverture : le panneau saison et le reste du backlog attendent une **conversation
dédiée** (« Watching v4 »), ouverte après. Une conversation qui audite et construit en même temps
n'audite plus.

**Règle de conduite** : l'audit **constate et classe**, puis **corrige ce qui tient dans la passe** ;
ce qui ne tient pas devient une ligne de backlog **avec son déclencheur**. Chaque correction commitée
passe les six portes en clone propre. Le contre-examen (`/challenge`) vient sur le rapport écrit,
pas sur les intentions.

---

## 8. Décisions vivantes issues de la phase 1

> 📖 **La chronologie complète de la phase 1 est dans [`hq/log/2026-09.md`](../log/2026-09.md)** — ce
> qui a été fait, dans quel ordre, et les pièges rencontrés. Ne restent ici que les décisions qui
> **gouvernent encore** le travail à venir.

**⚠️ `hq/`, `memory/` et `CLAUDE.md` restent gitignorés du repo public** (décision owner : le repo
principal est PUBLIC et sert de vitrine). `hq/` est son **propre dépôt privé** `ZejlyZakaria/hegon-hq`.
⇒ **ZONE SANS CLIQUET assumée** : la CI clone le dépôt public, elle ne pourra donc **jamais** vérifier
la fraîcheur de ces fichiers. Leur exactitude repose sur la discipline seule. *(Destination visée en
phase 4 : découper **par voix** — `blueprint`, `rules`, `decisions` réécrits en voix neutre → publics,
c'est un atout recruteur ; `CLAUDE.md`, `memory/`, session-protocol → privés.)*

**⛔ Tout changement de schéma DOIT passer par une migration.** C'est l'ancienne habitude — modifier à
la main dans l'éditeur SQL — qui avait produit les 28 tables sans migration. **Règle FRAGILE au sens du
§2** : aucun cliquet ne la protège aujourd'hui.

⚠️ `db diff --linked` ne convient **PAS** comme garde — testé le 06/09, il sort un diff massif et
bruité pour des objets **tous présents** dans le baseline, donc il finirait désactivé, exactement comme
le piège du `generatedAt`. **Le garde qui marche est le comptage d'objets** (tables / policies /
fonctions / index / crons) confronté à la base réelle : mesuré pendant le test de restauration, il
donne des chiffres nets et stables — **68 / 206 / 35 / 250 / 15**. ⇒ à construire à l'**axe 11**.

**📦 Déménagement de dossier — décidé, PAS encore fait.** Cible : `Desktop/MyDocuments/Hegon/hegon-app`.
Sans risque pour GitHub (le remote vit dans `.git/config`, et il n'y a pas de `.vercel` local).
⚠️ **Le piège** : la mémoire de Claude Code est indexée **par le chemin** — après le déplacement,
renommer le dossier `.claude/projects/c--…-digital-second-brain` en `…-Hegon-hegon-app`, sinon toute la
mémoire disparaît de sa vue. Puis supprimer `.next/` et relancer `npm install`.

**⭐ Les 3 jobs de l'axe 11 sont la MÊME plomberie** — *« un job programmé qui prévient que quelque
chose a cassé en silence »* : la **sauvegarde hebdomadaire** des tables personnelles, l'**alerte cron**
(la pire panne connue, 40 h de silence), et le **garde anti-dérive du schéma** ci-dessus. Les
construire ensemble rend la phase 2 nettement moins chère — c'est l'argument le plus juste de la
relecture croisée du 2026-09-09.
