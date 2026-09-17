# HEGON — NOW (état courant)

> Réécrit le 2026-09-17. Ce fichier dit **où on en est et quoi faire ensuite** — rien d'autre.
> Un jalon fini sort d'ici (il vit dans `git log` et dans `modules/*/decisions.md`). Le plan de
> travail ordonné est dans `hq/roadmap.md` ; le pilotage du chantier audit dans `hq/audit/plan.md`.

## 🛑 Le cadre en vigueur — CHANTIER AUDIT & FONDATION (depuis le 2026-09-05)

- **Gel par module** (amendement 6, 12/09) : un module reste gelé jusqu'à la fin de SON audit de
  phase 3. **Watching = audité → dégelé** (`modules/watching/audit-2026-09.md`). **Sport, Books,
  Journal, dashboard-os, Habits, Goals, Tasks = gelés** : on n'y touche pas, même pour une ligne.
- Phases 0-2 ✅ (carte du repo, purge + cliquets CI, doctrine R1-R11, watchdog, sauvegarde).
- Phase 3 = un audit par module, « petit et jugé raté » d'abord : **prochain = dashboard-os**
  (2 693 l.) — il débloque les deux tuiles que Watching attend. Brief dans `audit/plan.md` §7 bis.

## ✅ Watching v4 — FINI le 2026-09-17

§1→§8 (panneau saison, watchlist, dismiss persistant, priorité éditable, ruban TOP 10, le Musée
Oscars/Emmys, achievements « pourquoi », fondu des affiches), **§11 Follow people** (suivi, Cast &
Crew paginé, Library › People avec rails de 20 et panneaux paginés par 50 en base, robot
`watching-people-sync` hebdo + à chaud), **§12 le « + »** d'ajout direct, la grammaire des affiches
(bookmark à 5 états, cercles de statut, cœur à gauche, « + » à droite). Bloc « autonomie » du 17/09 :
les robots répondent 500 quand tout échoue, fraîcheur awards / people dans le watchdog, R7 sur tous
les appels tiers, titres pas encore sortis rafraîchis chaque semaine, Dock peint côté serveur,
squelettes homogènes (un `Pulse`, cellules de rail partagées `RAIL_CELLS`, plus de doublon More Like
This). Deux irrégularités de chargement (`library` / `movies`, notées le 22/07) restent **non
mesurées en prod** — en dev la compilation masque tout ; à mesurer sur hegon.fr avant de toucher.

**Reste (avec déclencheurs) → `modules/watching/backlog.md`** : §9 For You v2 (à cadrer), §10
thèmes (attend un mp3), Universes (non tranché), dettes mesurées.

## 🤖 Ce qui tourne tout seul (20 crons, watchdog toutes les 6 h, sauvegarde hebdo)

Watching : `series-sync` 4 h · `anime-cours` 6 h · `trending` quotidien · `for-you` 5 j ·
`awards` Oscars/Emmys (2×/jour en saison, mensuel sinon) · `people` mardi 05:35 (+ titres non
sortis). Un robot mort se voit dans `Watchdog` (GitHub Actions) et sur healthchecks.io — pas
besoin d'ouvrir l'app. Clés : `AGENTS.md` §Keys.

## 📏 Mesuré le 17/09 (dev, 2ᵉ chargement, octets réels)

Hubs 51-92 KB · Library 0 KB (semée serveur) · Lists 25 KB · Stats 399 KB (a besoin de tout) ·
People **102 KB** (index lean) · **Awards 980 KB** — le poids est le canon (`award_entries` 730 KB
pour 20 cartes), levier = requête dédiée, chiffré dans `modules/watching/backlog.md`.

## ▶️ PROCHAINE SESSION

0. **Squelette de la fiche** (`DetailSkeleton`) : l'owner le voit « hero vide, My Take et Details
   saturés, Top 10 / In list / Accolades absents » — impossible à capturer ici (la ligne arrive
   avant l'interception). **Partir de SA capture** au reload d'une fiche (film + série), puis
   corriger en une passe. Ensuite, si voulu, la requête dédiée d'Awards (~1 h).

1. Lire `AGENTS.md` (Codex) ou `CLAUDE.md` (Claude) → ce fichier → `hq/roadmap.md`.
2. **2ᵉ audit de phase 3 : dashboard-os** (brief `audit/plan.md` §7 bis). Ne pas construire de
   feature dessus avant la fin de l'audit — c'est la règle qui a rendu Watching fiable.
3. Après l'audit : les deux tuiles Watching → Dashboard (« nouvelle saison / épisode », « Oscars
   dans N jours, N nommés à voir ») — la moitié Watching est prête.
