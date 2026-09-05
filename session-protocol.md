# HEGON — Session Protocol (routeur de contexte)

> Lis ce fichier en début de session. Il dit **quoi charger** — et surtout **quoi NE PAS charger** par défaut.

---

## 🟢 START — contexte minimal (budget serré)

**Au démarrage, charge UNIQUEMENT :**
1. `CLAUDE.md` — auto-chargé (règles + archi + pointeurs)
2. `MEMORY.md` — auto-chargé (mémoire évolutive : décisions/feedback)
3. **`hq/now.md`** — l'état courant (focus + next steps). ⚠️ **PAS `progress.md`** (c'est l'archive, 70 KB).

C'est tout. Le reste se lit **à la demande**.

## 🗺️ À LA DEMANDE — la carte « si tu fais X → lis Y »

Ne lis un fichier `hq/` **que** si la tâche le concerne, **et seulement la section pertinente** (Grep/Read ciblé) :

| Tu fais… | Lis (la section pertinente) |
|---|---|
| Design / UI / passe module | `hq/rules/design-system.md` + [[reference_graphite_palette_and_material]] |
| Sécurité / RLS / auth / headers | `hq/rules/security.md` |
| Performance / DB perf | `hq/rules/performance.md` |
| Tests | `hq/rules/testing.md` |
| Navigation / sliding panels | ⚠️ pas de fichier courant — `hq/archive/navigation.md` (probablement périmé) |
| Décision archi / scope / stack | `hq/blueprint.md` (le § concerné) |
| Construire un module neuf | `hq/rules/new-module.md` + spec `hq/futur/{module}-v1.md` si elle existe |
| Prochaine version d'un module existant | `hq/modules/{module}/vnext.md` (brainstorm) → `decisions.md` une fois tranché |
| Anti-patterns à vérifier | `hq/rules/anti-patterns.md` |
| Historique complet d'une phase | `hq/log/{mois}.md` (rare — c'est le journal) |

### ⚠️ Règle budget contexte (la plus importante)
- **Jamais lire un gros `hq/*.md` en entier.** Cible la section (Grep le titre, Read avec offset).
- Gros fichiers connus : `hq/log/2026-04-a-06.md` (~1100 lignes), `rules/design-system.md`, `rules/testing.md`, `rules/performance.md`. Toujours par section.
- Ne re-lis pas un fichier déjà en contexte.

---

## 🟡 END — update ce qui a changé (et SEULEMENT ça)

> **Cadence (règle owner)** : on update **uniquement** quand le **focus change** ou qu'un **jalon significatif** est atteint. **PAS** de maj pour des micro-changements — la traçabilité sert les grandes choses.

| Tu as fait… | Update… |
|---|---|
| **Changé de focus** ou atteint un **jalon significatif** | **`hq/now.md`** (état courant + next steps) |
| Implémenté une feature/module commité | `hq/log/{mois}.md` (append au journal) |
| Tranché un arbitrage sur un module existant | `hq/modules/{module}/decisions.md` (avec son « refusé » et « ce qui la rouvrirait ») |
| Ajouté une migration SQL | mentionner dans `now.md` (à appliquer) + `hq/log/{mois}.md` |
| Décision structurante (archi/scope) | `hq/blueprint.md` § Décisions |
| Touché auth/RLS/headers | `hq/rules/security.md` |
| Anti-pattern découvert | `hq/rules/anti-patterns.md` |
| Décision utile dans 3+ semaines | mémoire auto (`memory/` + ligne dans `MEMORY.md`) |
| Renommé/déplacé un `hq/*` | grep le nom partout, fixer les refs — **une restructuration n'est finie que quand plus aucun fichier ne pointe vers un chemin mort** |

**Règle d'or :** rien changé dans une catégorie → ne touche pas le fichier.

---

## Division des rôles (pas de doublon)
- **CLAUDE.md** = règles + archi + pointeurs (statique, court).
- **hq/rules/** = savoir **statique** détaillé (design-system, sécurité, perf, conventions) — à la demande.
- **hq/now.md** = état **courant** (tiny, lu au start).
- **hq/log/** = **journal/archive**, un fichier par mois (rarement lu).
- **hq/modules/{module}/decisions.md** = arbitrages **datés qui ne périment jamais** (le « pourquoi »).
- **MEMORY.md (+ memory/)** = faits/décisions **évolutifs** cross-session.

## Quand updater CE fichier
Tu changes la structure `hq/`, ou tu trouves une catégorie d'update manquante/inutile. Ne PAS y mettre de règles de code/design (→ `workflow.md`/`design-system.md`).
