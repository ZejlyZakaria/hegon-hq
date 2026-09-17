# HEGON — Session Protocol (routeur de contexte)

> Lis ce fichier en début de session. Il dit **quoi charger** — et surtout **quoi NE PAS charger**.
> Vaut pour tout agent : Codex lit `AGENTS.md`, Claude Code lit `CLAUDE.md` — même contenu.

---

## 🟢 START — contexte minimal

1. `AGENTS.md` / `CLAUDE.md` — architecture, règles non négociables, commandes (auto-chargé).
2. **`hq/now.md`** — l'état courant et la prochaine étape (court, réécrit à chaque jalon).
3. **`hq/roadmap.md`** — l'ordre du travail et les déclencheurs de chaque ligne.
4. (Claude Code seulement) `MEMORY.md` — mémoire cross-session, auto-chargée.

C'est tout. Le reste se lit **à la demande**, la section pertinente seulement (Grep le titre).

## 🗺️ À LA DEMANDE — « si tu fais X → lis Y »

| Tu fais… | Lis |
|---|---|
| Design / UI / passe module | `hq/rules/design-system.md` (+ `DESIGN.md` = les tokens) |
| Sécurité / RLS / auth / headers | `hq/rules/security.md` |
| Performance / DB perf | `hq/rules/performance.md` |
| Tests | `hq/rules/testing.md` |
| Crons, robots, migrations, cliquets CI | `hq/rules/system-design.md` (R1-R11) |
| Décision archi / scope / stack | `hq/blueprint.md` (le § concerné) |
| Construire un module neuf | `hq/rules/new-module.md` + `hq/futur/{module}-v1.md` si elle existe |
| Toucher un module existant | `hq/modules/{module}/backlog.md` (ce qui reste, avec déclencheurs) → `decisions.md` (ne pas rouvrir) → `architecture.md` (invariants) |
| Auditer un module (phase 3) | `hq/audit/plan.md` §7 bis (le brief) + `hq/audit/map.md` (la carte du repo) |
| Anti-patterns à vérifier | `hq/rules/anti-patterns.md` |
| Leçons chèrement apprises (pièges concrets) | `hq/rules/lessons.md` |

**Règle budget** : jamais un gros `hq/*.md` en entier (`design-system.md`, `testing.md`,
`performance.md` font 500-1 500 lignes) — Grep le titre, Read la section.

## 🟡 END — mettre à jour ce qui a changé, et SEULEMENT ça

| Tu as… | Mets à jour… |
|---|---|
| Changé de focus ou atteint un **jalon** | `hq/now.md` (court : état + prochaine étape ; un jalon fini SORT du fichier) |
| Tranché un arbitrage sur un module | `hq/modules/{module}/decisions.md` (daté, avec le « refusé » et « ce qui le rouvrirait ») |
| Reporté quelque chose | `hq/modules/{module}/backlog.md` — **avec son déclencheur**, sinon la ligne pourrit |
| Ajouté une migration | `supabase/checks/expected-objects.json` (`_migrations_through` + compteurs) **dans le même commit** — cliquet CI |
| Touché un composant | `npm run ds:coverage` et commiter `coverage.generated.json` — cliquet CI |
| Décision structurante | `hq/blueprint.md` § Décisions |
| Découvert un piège qui coûtera cher la prochaine fois | `hq/rules/lessons.md` (une ligne : le piège, la preuve, le remède) |
| Renommé / déplacé un `hq/*` | grep le nom partout — une restructuration n'est finie que quand plus rien ne pointe dans le vide |

**Règle d'or** : rien changé dans une catégorie → ne touche pas le fichier. Pas de journal de
séance : le récit vit dans `git log` (un commit = une ligne qui dit le quoi et le pourquoi).

## Division des rôles (pas de doublon)
- `AGENTS.md` / `CLAUDE.md` = règles + archi + pointeurs (statique, court, **vrai**).
- `hq/rules/` = savoir statique détaillé — à la demande.
- `hq/now.md` = état courant (court). `hq/roadmap.md` = l'ordre et les déclencheurs.
- `hq/modules/{module}/` = `backlog.md` (vivant) · `decisions.md` (daté, ne périme pas) · `architecture.md` (invariants).
- `hq/audit/` = le chantier audit & fondation (plan + carte).
- `memory/` (Claude Code seulement) = faits cross-session ; ce qui y est vital est aussi dans `hq/rules/lessons.md`.
