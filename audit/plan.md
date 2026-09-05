# HEGON — Plan d'audit & de fondation

> **Document de pilotage du chantier.** Décidé avec l'owner le 2026-09-05. La conversation d'exécution
> démarre en lisant CE fichier, pas à froid. Il est mis à jour au fil des passes — c'est lui qui dit
> où on en est.
>
> ⛔ **Gel des features pendant tout le chantier.** Sport (page stats football, tennis, F1) est
> volontairement suspendu. On ne reprend le build qu'une fois la fondation posée.

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

### Phase 3 — LES AUDITS PAR MODULE (sérialisés, un par un)

Contre la doctrine de la phase 2 + la checklist du §6. **Watching en premier** (l'étalon).

⚠️ **Le deuxième ne doit PAS être le deuxième meilleur.** Prendre un module jugé raté **mais petit** :
la grille se fait mordre pour de vrai, la liste de corrections reste finie, et on obtient un
« avant/après » complet rapidement. Lequel : à choisir une fois la carte faite (mesurer, pas deviner).

⚠️ **Attente à poser** : la passe Watching produira surtout des **règles**, pas des corrections. C'est
normal — c'est l'étalon. Son livrable est la checklist elle-même.

### Phase 4 — LA VÉRITÉ DES DOCS

Chaque `.md` du repo et chaque mémoire : **vivant / archivé / supprimé**. Résoudre les contradictions
connues (au moins une explicite : une mémoire déclare « prime sur CLAUDE.md §5 »). Traçabilité gardée
mais **dans `archive/`, jamais dans ce qu'on lit au quotidien**. Inclut la cure d'amaigrissement de
`hq/now.md` (555 lignes lues à chaque démarrage).

### Phase 5 — LES RÈGLES UI/UX ET LE DESIGN SYSTEM

En dernier **volontairement** : c'est la passe la plus coûteuse, et écrire des règles pour des modules
qu'on va restructurer, c'est les écrire deux fois. Écrites en **CODE** (primitifs + `ds:coverage`), pas
en prose. La base existe déjà : `npm run ds:coverage`, `/settings/styleguide`, Watching = pilote fini.
Inclut l'ajout de tests sur la logique pure là où il n'y en a pas, en commençant par les modules Sport.

---

## 5. Checklist SYSTEM DESIGN (phase 2 — 10 axes)

Taillée sur ce que HEGON EST réellement.

⚠️ **Hors périmètre assumé** : sharding, réplication, load balancers, files distribuées, CAP, scaling
horizontal. Ce sont des sujets d'entretien pour des systèmes à millions d'utilisateurs ; HEGON est
mono-utilisateur sur serverless et n'en aura jamais besoin. Les apprendre demandera un autre véhicule.

1. **Le contrat** — ce que HEGON promet vraiment (mono-utilisateur ? démo publique ? multi-tenant via
   `org_id` ?) et ses cibles non-fonctionnelles. Jamais écrit ; tout le reste en découle.
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
| 1 — Purge + cliquets | ⬜ **démarrer ici** — la liste d'entrée est [`map.md` §5](map.md) |
| 2 — System design (doctrine) | ⬜ |
| 3 — Audits par module (Watching d'abord) | ⬜ |
| 4 — Vérité des docs | ⬜ |
| 5 — UI/UX + design system | ⬜ |

**Reprise après le chantier** (gelé, ne pas y toucher avant) : page stats football — la route n'existe
pas, mais `FanLog` + `BestXI`/`Legends` sont **déjà construits et orphelins** ; la seule logique
réellement manquante est le **record de prédictions** (les prédictions sont stockées mais jamais
confrontées au résultat). Puis tennis, F1, cleanup.
