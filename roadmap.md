# HEGON — Roadmap (l'ordre du travail, avec les déclencheurs)

> Écrit le 2026-09-17 pour que n'importe quel agent (Codex dès le jour 1, Claude Code au retour)
> sache **par quoi commencer, dans quel ordre, et pourquoi**. Chaque ligne dit ce qu'elle débloque.
> L'état du jour est dans `now.md` ; le pilotage détaillé du chantier audit dans `audit/plan.md`.
> **Jour 1 = lire `AGENTS.md` → `session-protocol.md` → `now.md` → ce fichier → commencer le §1.**

## Le principe qui ordonne tout

Depuis le 2026-09-05 le projet est en **chantier audit & fondation** : chaque module est **gelé**
jusqu'à la fin de son audit de phase 3, puis se rouvre sous trois conditions (feature au backlog
avec son déclencheur · respect de la doctrine R1-R11 · aucun module non audité touché). Watching a
été le premier — audité le 13/09, puis v4 construite en trois jours **sans une régression**, parce
que l'audit avait d'abord rendu le module vrai (md réconciliés contre le code, une seule source
d'état, tests sur la logique pure). C'est la méthode : **auditer, puis construire**. Ne pas
l'inverser pour aller plus vite — c'est ce qui avait produit 28 tables hors migration.

## 1. Phase 3 — les audits, dans cet ordre

| # | Module | Pourquoi cet ordre | Brief |
|---|---|---|---|
| 1 | ✅ **Watching** (13/09) | l'étalon — sa passe a produit la checklist | `modules/watching/audit-2026-09.md` |
| 2 | ▶️ **dashboard-os** (2 693 l.) | « petit et jugé raté » ; **débloque les deux tuiles que Watching a déjà préparées** (§3) | `audit/plan.md` §7 bis (le modèle), §6 (la grille des 9 axes) |
| 3 | **Journal** (2 715 l.) | petit ; débloque « mentionner un film/série dans une entrée » | idem |
| 4 | **Books** (4 385 l.) | débloque `media_items.based_on_book_id` + « Adapted into… » | idem |
| 5 | **Sport** (9 998 l., 0 test) | le plus gros et le moins testé — en dernier, avec le temps qu'il faut ; plan de refonte prêt : `modules/sports/football-ultimate-plan.md` | idem |
| 6-8 | Habits, Goals, Tasks | V1/V2 stables ; audits courts | idem |

Méthode d'un audit (identique pour tous) : lire §6 de `audit/plan.md` (9 axes) → doctrine
`rules/system-design.md` → réconcilier les md du module **contre le code** (jamais entre eux) →
instruments `scripts/list-requests.mjs` puis `scripts/measure-cache.mjs` → rapport **un écran** dans
`modules/{module}/audit-{date}.md` → l'axe tests en dernier, **bloquant** → un contre-examen avant le
commit. Sur un chemin d'écriture, l'audit corrige un bug, jamais une optimisation.

## 2. Watching — ce qui reste (module dégelé, `modules/watching/backlog.md`)

Rien d'urgent : le module tourne seul (6 robots, watchdog, sauvegarde). Par valeur :
1. **§9 For You v2** — à cadrer dans une conversation dédiée avant une ligne de code.
2. **§10 Thèmes films/séries** — déclencheur : l'owner a des mp3.
3. **Universes** (non tranché) · **Achievements créables** (gabarits) · dettes mesurées (page
   Awards 730 KB, `getPersonUpcoming` en `in(...)`, pochettes iTunes → table).

## 3. Inter-modules — voulus, bloqués par la condition 3 (s'ouvrent après l'audit partenaire)

| Lien | Ce qu'il fait | S'ouvre après |
|---|---|---|
| **Watching → Dashboard** | tuile « nouvelle saison / nouvel épisode disponible » + tuile « Oscars dans N jours, N nommés à voir » (la moitié Watching existe : `award_ceremonies`, `person_upcoming`, `series-state`) | audit **dashboard-os** |
| **Journal ↔ Watching** | mentionner un film / une série dans une entrée | audit **Journal** |
| **Books ↔ Watching** | `based_on_book_id` + badge « Adapted into… » sur la fiche livre | audit **Books** |

## 4. Transverse, après les audits

- **Skeletons homogènes** sur tout Watching puis les autres modules : un primitif, mêmes rayons,
  même pulsation, fidélité à la surface au pixel (les rails People sont le modèle : 10 cellules
  aux breakpoints du rail). Deux irrégularités connues : le skeleton de `library` disparaît avant
  les données ; `movies` a ~400 ms de vide avant le sien.
- **Design system → les 7 autres modules** (`project_design_system_code` : Watching est le pilote
  fini ; `npm run ds:coverage` mesure la propagation).
- **Access control** (approbation manuelle des comptes) — toujours pas construit, à faire avant
  toute ouverture d'inscriptions.
- **Nouveaux modules** (`futur/`) : Culture, Tech, Travel — specs v1 prêtes, après les audits.
- **Stratégie produit** (`product-strategy.md`, décision du 07/09) : HEGON est une COLLECTION, le
  différenciateur est l'agrégation transversale ; le partage est le canal de distribution.

## 5. Ce qu'un agent ne fait JAMAIS ici

Toucher un module gelé « juste pour une ligne » · exécuter un commit ou un `db push` (l'owner le
fait) · écrire une migration sans `expected-objects.json` · appeler un fournisseur pendant un rendu
· annoncer un gain sans A/B · déclarer « fait » ce qui n'a pas été vu à l'écran ou en base.
