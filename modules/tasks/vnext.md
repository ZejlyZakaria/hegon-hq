# HEGON Tasks — V2 Spec

> Source de vérité pour la V2 du module Tasks. Décisions prises pendant la session brainstorming du 2026-05-19 entre Zakaria et Claude (post-audit + Opus re-review).

---

## 1. Vision & positionnement

**Positionnement** : HEGON Tasks = le Linear personnel qui sait **pourquoi** tu fais ce que tu fais.

Là où Linear te montre un backlog, HEGON te dit "fais ÇA maintenant" — basé sur ton goal, ce qui est urgent, ce qui pourrit. Aucune app concurrente n'a un système cross-module aussi complet (tasks ↔ goals ↔ habits ↔ journal).

**Apps de référence** :
- **Linear** — référence UX / animation / polish / status icons
- **Jira** — référence complétude des features (mais on évite le bloat)
- **Notion** — référence flexibilité (mais on évite le piège de la flexibilité totale)
- **Sunsama** — référence "what should I work on now"
- **Things 3** — référence scheduled tasks (start_date)

---

## 2. État actuel (V1 — déjà construit)

**Foundation solide :**
- Hiérarchie : Workspaces → Projects → Statuses → Tasks
- 3 vues : Kanban, List, Calendar
- Drag & drop entre statuses + position (avec trigger DB atomique pour éviter race conditions)
- **Real-time multi-user — validé en prod 2026-05-19** ✅
- Workspace sharing avec invitations + rôles (owner/admin/member/viewer)
- Tags par workspace
- Assignees (people picker avec MemberAvatar)
- Priorité (critical / high / medium / low + colonne `priority_rank` GENERATED)
- Due date avec couleurs (overdue rouge, year-aware)
- Liens cross-module : Goals ↔ Tasks (bidirectionnel)

**Schema existant :**
- Tables : `tasks`, `projects`, `statuses`, `workspaces`, `workspace_members`, `tags`, `task_tags`, `org_invitations`
- Columns notables :
  - `start_date` (pas exposé UI — à exposer en V2)
  - `estimated_hours` / `actual_hours` (à drop — inutiles)
  - `priority_rank` GENERATED
  - `completed_at` (set par moveTask quand status `is_completed=true`)
  - `goal_id` (cross-module link)

---

## 3. Features V2 — lockées

| # | Feature | Effort | Wow | Signature |
|---|---|---|---|---|
| 1 | Sub-tasks hiérarchiques | Moy | Standard mais essentiel | — |
| 2 | ⌘K + inline create partout | Bas | Linear-level | — |
| 3 | Staleness visuel (conditional) | Bas | 🔥 | — |
| 4 | Comments + Activity log par task | Moy | Indispensable collab | — |
| 5 | Context section dans slide panel (cross-module) | Bas | 🔥🔥 | 🥈 |
| 6 | Smart "Now" view (algo de reco) | Moy | 🔥🔥🔥 | 🥇 |
| 7 | Custom columns + icon system | Moy | 🔥🔥 | 🥉 |
| 8 | Notifications collab (bell topbar) | Moy | Table stakes collab | — |
| 9 | Start date exposé + scheduled tasks | Bas | Différenciateur | — |

---

## 4. Top 3 features signature pour vidéo LinkedIn

### 🥇 Smart "Now" view

**Promesse** : "Le seul tool qui te dit quoi faire MAINTENANT, pas juste ce que tu as à faire."

**Concurrence** :
- Sunsama (16$/mois) le fait — mais moins bien et payant
- Aucun autre tool n'a ça

**Ingrédients de l'algo (à spécifier plus tard)** :
- Priorité (priority_rank)
- Proximité due_date
- Goal linkage strength
- Staleness (jours sans activité)
- Filtre `start_date <= today`

### 🥈 Context section dans slide panel

**Promesse** : "Voir d'un coup d'œil POURQUOI tu fais cette task — quel goal elle alimente, quels habits y contribuent."

**Concurrence** : Aucun. Linear, Notion, Jira sont silotés.

**UI** : Section dans le slide panel de détail de la task, sous les properties.

### 🥉 Custom columns + icon system

**Promesse** : "Configurer ton workflow exactement comme tu le penses, avec un design visuel cohérent (cercles à progression : empty → 1/4 → 1/2 → 3/4 → full)."

**Concurrence** : Linear fait ça basiquement. On peut faire mieux visuellement.

---

## 5. Décisions tranchées

### ❌ Coupé / refusé

| Feature | Raison |
|---|---|
| Milestones tasks | Redondant avec sub-tasks (breakdown) + goal milestones (checkpoints) + priority (importance) |
| Time tracking (estimated_hours / actual_hours) | Inutile pour usage perso. Drop UI + drop columns schema |
| Dependencies (blocked by / blocking) | Peu utilisé en pratique. Power-user feature Jira-style. À reconsidérer en V3 si besoin réel émerge |

### ⏭ Reporté ailleurs

| Feature | À discuter |
|---|---|
| Weekly auto-review | Plutôt dans module Dashboard (récap auto du dimanche) |

---

## 6. Cleanup à faire pendant l'implémentation V2

- [ ] Drop columns `tasks.estimated_hours` et `tasks.actual_hours` (migration SQL)
- [ ] Drop champ `estimated_hours` du type `CreateTaskInput`
- [ ] Drop champ `estimated_hours` de `CreateTaskModal` (si exposé) et `EditTaskModal`
- [ ] Vérifier que `start_date` est bien gardé et l'exposer dans les modals
- [ ] Renommer `EditTaskModal` → `EditTaskPanel` (sliding panel)

---

## 7. Notes d'architecture

### Activity log
Nouvelle table `task_activities` :
```sql
- id uuid
- task_id uuid (FK → tasks)
- user_id uuid (FK → profiles, qui a fait l'action)
- action text ("created" | "assigned" | "status_changed" | "commented" | ...)
- metadata jsonb ({ from, to, ... })
- created_at timestamptz
```
- RLS : visible aux membres du workspace de la task
- Indexes : task_id, created_at

### Notifications
Nouvelle table `notifications` :
```sql
- id uuid
- recipient_id uuid (FK → profiles)
- type text ("task_assigned" | "task_completed" | "comment_added" | ...)
- payload jsonb ({ task_id, assigned_by, ... })
- link text (URL relative, ex: "/pro/tasks?taskId=...")
- created_at timestamptz
- read_at timestamptz (NULL si non lu)
```
- RLS : visible uniquement par `recipient_id = auth.uid()`
- Index : recipient_id + read_at (pour count badge)
- Realtime : enable replication

**UI** : icône cloche 🔔 dans topbar global HEGON avec :
- Badge count des notifs non lues
- Dropdown au click avec liste des 10 dernières
- Click sur une notif → marquer lue + naviguer vers le link

**Hooks d'invalidation (V2 simple, hardcodé) :**
- `useUpdateTask.onSuccess` si `assignee_id` change → INSERT notification
- `useCreateTask` si `assignee_id !== current user` → INSERT notification
- `useDeleteTask` → pas de notif (le user a déjà la confirmation modal)
- Migration future : passer ça dans Phase 4 Automation Engine

### Start date
- Champ déjà dans schema (`tasks.start_date`)
- À exposer dans CreateTaskModal + slide panel d'édition
- Utilisé par algo Smart "Now" view (filtre les tasks où `start_date > today`)
- Indicateur sur TaskCard si `start_date > today` ("starts in 3d" + opacité 60%)
- Calendar view : afficher comme barre horizontale de `start_date` à `due_date`
- Animation : quand `start_date` arrive (frontière dépassée) → la task "se réveille" visuellement

### Custom columns
- Extension de la table `statuses` existante
- Ajouter colonne `icon_style` enum :
  - `empty_dashed` (Backlog)
  - `empty_solid` (To Do)
  - `quarter` (Started, jaune)
  - `half` (In Progress, bleu)
  - `three_quarter` (In Review, vert)
  - `full_check` (Done)
  - `cancelled` (X)
- UI dans Workspace settings ou Project settings : add / edit / delete columns
- Suggestions automatiques de couleur selon le nom (ex: "QA" → rouge par défaut, modifiable)
- Reorder par drag-drop dans la sidebar

### Sub-tasks
- Champ `parent_task_id` déjà dans le schema
- UI : sub-tasks affichées en indentation dans le slide panel détail
- Progress : "3/5 sub-tasks done" affiché sur la parent task card
- Création rapide : Tab + Enter pour ajouter une sub-task depuis le panel
- Reorder par drag-drop dans la liste de sub-tasks

---

## 8. Animations & UX details

### Staleness (conditional)
- Affiché **uniquement si `due_date IS NULL`** (sinon la couleur du due_date suffit, pas de double signal)
- Barre fine à gauche de la TaskCard, 2px
- < 3 jours : pas de barre
- 3-7 jours : jaune
- 7-14 jours : orange
- \> 14 jours : rouge + léger pulse

### Status icons
- Système de cercles avec progression visuelle (inspiré Linear, mais customisable)
- 7 styles d'icônes, couleur configurable par status
- Animation transition de status : l'icône morph d'un état à l'autre (rotation + fill)

### Sliding panel (remplace EditTaskModal)
- Slide depuis la droite, animation spring < 280ms
- Largeur conforme `w-120` (480px, comme WorkspaceInvitePanel)
- Le kanban reste visible derrière
- Switch entre tasks : slide content sans fermer le panel (slide gauche/droite)
- Esc pour fermer
- Click backdrop pour fermer

### Delete confirmation
- Pattern : sliding panel ouvert + petit modal centré de confirmation
- Reuse `DeleteTaskModal` existant

### Animation cross-module (start_date passage)
- Quand `start_date` arrive : la task "se réveille" visuellement
- Opacity 60% → 100% + léger pulse (1 cycle, 600ms)

### Shared element transitions (Linear-style)
- TaskCard kanban ↔ TaskPanel : `layoutId` partagé sur le titre
- Quand on clique une card, son titre "vole" vers le header du panel

---

## 9. Cross-module connections

### Existant (V1)
- Tasks ↔ Goals (bidirectionnel)
  - Via `tasks.goal_id`
  - Pickers : "Link task" dans GoalDetailPage + select "Goal" dans CreateTaskModal/EditTaskModal
  - Recalcul auto goal progress (`recalc_goal_progress` RPC v2)
  - Section "Fueling this Goal" sur la GoalDetailPage

### À étendre / clarifier en V2
- **Tasks ↔ Habits** : connexion via Goals (déjà fait) OU direct ?
  - À voir lors du brainstorm Habits V2
- **Tasks ↔ Journal** : mention de tasks dans journal entries ?
  - À voir avec brainstorm Journal V2
- **Tasks ↔ Dashboard** :
  - Smart "Now" view = widget principal Dashboard
  - Stats : "X tasks completed this week" dans Right Panel
  - Activity feed cross-module ?

---

## 10. Angle vidéo LinkedIn — "HEGON Tasks"

### Hook (3-5 sec d'accroche)
> "You've used Linear. You've used Jira. You've used Notion. They all give you the same thing: a backlog.
> But what if your task tool told you what to do **right now** — based on your goals, your week, your energy?"

### Démo flow (60-90 sec)
1. Open HEGON dashboard → see **Smart "Now" view** widget
2. Click on the suggested task → **slide panel** opens with full context
3. Show **Context section** : "This task feeds 🎯 'Launch HEGON V1' (67%) and 🔁 habit 'Code 1h/day' (🔥 12 days)"
4. Mark task done → animation + **goal progress bar climbs in realtime**
5. Back to kanban → show **custom columns** with progression icons + **stale tasks** with orange/red bars
6. Bonus : drag-drop with smooth spring animation

### Message clé final
> "**The only personal task manager that knows why you're doing what you're doing.**"

### CTA
- Link en bio / commentaire vers landing page HEGON
- Suite : 1 vidéo / module / semaine

---

## 11. Wrapped & Achievements (cf. `achievements-v2.md`)

### Cards fournies au Wrapped annuel
- Tasks completed this year
- Most productive month
- Completion rate global
- Longest daily-completion streak
- Top project (par tasks done)

### Achievements du module
| Achievement | Type | Paliers |
|---|---|---|
| Getting Started (1ère task) | événementiel | — |
| Productive (tasks complétées) | tiered | 100🥉 / 500🥈 / 1000🥇 / 5000💎 |
| Finisher (compléter toutes les tasks d'un projet) | événementiel | — |
| Consistency (≥1 task complétée N jours d'affilée) | tiered streak | 7🥉 / 30🥈 / 100🥇 / 365💎 |
| Organizer (projets créés) | tiered | 5🥉 / 15🥈 / 30🥇 |

---

## Status & next steps

- ✅ Brainstorm session : 2026-05-19
- ✅ Décisions lockées
- ✅ Section Wrapped + Achievements ajoutée (2026-05-20)
- ⏭ Algo Smart "Now" à spec quand on implémentera
- ⏭ Implémentation V2 : pas commencée
- ⏭ Prochains brainstorms : Habits, Goals, Books, Journal, Dashboard, Sports
