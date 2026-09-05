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
| 1 — Purge + cliquets | 🟡 **EN COURS** — voir §8. Liste d'entrée : [`map.md` §5](map.md) |
| 2 — System design (doctrine) | ⬜ |
| 3 — Audits par module (Watching d'abord) | ⬜ |
| 4 — Vérité des docs | ⬜ |
| 5 — UI/UX + design system | ⬜ |

**Reprise après le chantier** (gelé, ne pas y toucher avant) : page stats football — la route n'existe
pas, mais `FanLog` + `BestXI`/`Legends` sont **déjà construits et orphelins** ; la seule logique
réellement manquante est le **record de prédictions** (les prédictions sont stockées mais jamais
confrontées au résultat). Puis tennis, F1, cleanup.

---

## 8. ÉTAT DE SESSION — 2026-09-05 (soir), phase 1 en cours

### ✅ Décidé et fait

**`hq/`, `memory/` et `CLAUDE.md` restent gitignorés.** Décision owner assumée : le repo principal est
**public sur GitHub** et sert de vitrine pour des recruteurs. `hq/` est donc devenu **son propre dépôt
privé** — `ZejlyZakaria/hegon-hq`, commit `908c135`, push vérifié.

⚠️ **ZONE SANS CLIQUET, assumée explicitement.** Aucune CI ne pourra vérifier la fraîcheur de `hq/` ni
de `memory/` — la CI clone le dépôt public, ces fichiers n'y sont pas. Leur exactitude repose sur la
**discipline seule**. C'est le seul endroit du projet où le principe « un mécanisme, jamais un état »
ne s'applique pas, et il faut s'en souvenir.

**Destination visée (phase 4) : découper par VOIX, pas par dossier.** `blueprint.md`, `rules/`,
`modules/*/decisions.md` n'ont aucune empreinte IA — réécrits en voix neutre ils deviennent le
**meilleur morceau** d'un repo public (peu de projets perso montrent des décisions écrites avec leurs
alternatives rejetées). Ce qui décrit la *collaboration* (`CLAUDE.md`, `memory/`, session-protocol)
reste privé.

**Supabase = plan Free ⇒ AUCUNE sauvegarde** (vérifié sur la page Backups : « Free Plan does not
include project backups »). Le dump manuel sera donc **la seule copie existante**. Rien d'autre ne
protège les données aujourd'hui.

**CLI opérationnelle** : `supabase login` ✅ · `supabase link --project-ref femvhonlpafdajyamvcu` ✅.
⚠️ Le `Cannot find project ref` venait de l'absence de `supabase/config.toml` — le `linked-project.json`
du 16/07 n'est plus le format lu par la CLI 2.x. Le `link` l'a régénéré. CLI **2.116.0**, Docker **27.4.0**.

### ⏸️ Bloqué ce soir — réseau

`supabase db dump` exécute `pg_dump` **dans un conteneur** dont la version doit correspondre exactement
au serveur (Postgres 17.6.1) — d'où Docker, obligatoire, pas contournable proprement. Le pull de
`supabase/postgres:17.6.1.063` (~1 Go, ~90 couches) échoue sur un wifi instable (`Retrying in…`).
**Docker conserve les couches déjà téléchargées** ⇒ relancer la même commande reprend où ça s'était
arrêté. Rien n'est perdu, il n'y a rien à réparer.

### ▶️ REPRENDRE ICI — ce que l'owner fait, dans l'ordre

⚠️ **Depuis le dossier du projet**, jamais depuis `~` : `--linked` a besoin du projet pour savoir de
quelle base on parle.

1. Les 3 dumps vers **OneDrive** (`C:/Users/ADMIN/OneDrive/hegon-backups/`) : `--role-only`, schéma,
   `--data-only`. ⚠️ **Jamais dans le repo** — le dump de données contient son journal, son historique
   et ses habitudes, et le repo est public. OneDrive est choisi exprès : le Desktop n'est **pas**
   synchronisé (vérifié), donc un dump posé là vivrait sur le même disque que ce qu'il protège.
2. Vérifier que les fichiers ne sont pas vides (`ls -lh`) — un dump de 0 octet est le piège classique.
3. Le baseline dans le repo : `db dump --linked -f supabase/migrations/20260906000000_baseline.sql`.
   **NE PAS COMMITER** avant relecture (vérifier qu'aucun corps de fonction n'expose quoi que ce soit).
4. Dans le SQL Editor : `select jobid, schedule, command, jobname, active from cron.job order by jobid;`
   et coller le résultat.

### Puis moi

Relire le baseline · archiver les 105 migrations dans `supabase/migrations/_archive/` · transformer les
crons en migration versionnée · `supabase migration repair --status applied 20260906000000` (⚠️ **écrit
une ligne de suivi, n'exécute PAS le SQL** — c'est ce qui rend `db push` sûr ensuite) · vérifier avec
`npx supabase db diff --linked`, qui doit ne **rien** afficher = preuve mécanique que le repo décrit la
base.

⛔ **Ne JAMAIS lancer `supabase db push` avant le repair.** La table de suivi est vide : il rejouerait
les 105 migrations sur une base qui a déjà les objets, et celles contenant un `drop`/`truncate` feraient
de vrais dégâts.

### 📦 Déménagement de dossier (décidé, à faire à une pause propre — PAS maintenant)

Cible : `Desktop/MyDocuments/Hegon/hegon-app` (l'actuel `digital-second-brain`), backups dans OneDrive.

**Aucun risque GitHub** — l'URL du remote vit dans `.git/config`, pas dans le nom du dossier ; les deux
remotes et celui de `hq/` suivent. Pas de `.vercel` local ⇒ le déploiement passe par l'intégration
GitHub et ignore le disque.

⚠️ **Le piège** : la mémoire de Claude Code est indexée **par le chemin**. Après le déplacement, renommer
`C:\Users\ADMIN\.claude\projects\c--Users-ADMIN-Desktop-MyDocuments-digital-second-brain`
en `…-Hegon-hegon-app`, sinon toute la mémoire et l'historique disparaissent de sa vue.
Puis supprimer `.next/` et relancer `npm install`.

### ⚠️ AMENDEMENTS PROPOSÉS AU PLAN — EN ATTENTE DE GO OWNER

Issus de la relecture croisée du plan (conversation system design, 2026-09-05). **Non appliqués.**

1. **La doctrine (phase 2) doit nommer son cliquet.** Chaque règle indique le test ou la règle de lint
   qui la rend impossible à violer ; celles qui n'en ont pas sont marquées **fragiles**. Sans ça, la
   phase 2 viole la règle §2 « rien n'atterrit en prose seule ». *(La phase 0 a déjà résolu sa moitié :
   `npm run map` régénère la carte.)*
2. **Les tests remontent de la phase 5 vers la phase 3** — ils voyagent avec chaque audit de module.
   Diagnostiquer les tests comme l'angle mort puis les programmer en dernier est une contradiction.
3. **« Le contrat » sort de la checklist** et devient une **session de décision** avant la phase 2 :
   ça ne se découvre pas dans le code, ça n'appartient qu'à l'owner. Trois futurs cohabitent
   aujourd'hui (infra multi-tenant `org_id`/`my_orgs()`, garde démo read-only sur 6 modules, Access
   Control documenté jamais construit).
4. **Nouvel axe 11 — durabilité & reconstructibilité.** Absent des 10 axes, alors que c'est
   probablement la question système n°1. C'est précisément ce que la phase 1 est en train de traiter.
5. **La leçon des 40 h doit produire du code, pas un paragraphe** : l'alerte sur cron échoué devient un
   **livrable nommé** de la phase 2.

Nuance honnête retenue de la relecture : la corrélation « tests ↔ satisfaction » du §3 est réelle mais
présentée trop fort — Watching a aussi eu dix fois plus de passes que Sport.
