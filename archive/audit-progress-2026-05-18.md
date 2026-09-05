# HEGON — Audit Progress Tracker

> Suivi d'application de [digger-audit-2026-05-18.md](digger-audit-2026-05-18.md).
> À utiliser par Opus pour re-vérifier après application complète.
> Mis à jour au fil de la session.

---

## Score global cible

| Axe | Score audit v1 | Cible post-fixes |
|---|---|---|
| Crashes & erreurs silencieuses | 5/10 | 8/10 |
| Flash & flickering | 6/10 | 8/10 |
| Performance perçue | 6.5/10 | 8/10 |
| Qualité du code | 6/10 | 8/10 |
| Sécurité & données | 6/10 | 8/10 |
| UX & cohérence | 6/10 | 7/10 |

---

## Sprint 1 — Critiques bloquants démo ✅ TERMINÉ (2026-05-18)

| # | Titre | Statut | Fichiers touchés | Vérifié |
|---|---|---|---|---|
| §4.2 | Middleware code mort (84 lignes commentées) | ✅ Fait | `src/middleware.ts` | typecheck + lint |
| §6.2 | Toast spam autosave Tasks | ✅ Fait | `src/modules/tasks/hooks/useTasks.ts` | UX validée par Zakaria — "on garde sans toast" |
| §1.1 | Dashboard invité cassé (`created_by` au lieu de `created_by OR assignee_id`) | ✅ Fait | `src/modules/dashboard/service.ts` (`getTodayTasks` + `getTasksActivityByDay`) | Testé en compte invité — dashboard affiche bien les tasks assignées |
| §1.2 | `getCurrentOrgId` non-déterministe en multi-org | ✅ Fait | `src/shared/utils/getOrgId.ts` (+ `.order("created_at")`) + `src/app/invite/[token]/page.tsx` (auto-set org après accept) | Indirectement testé via dashboard invité |
| §5.1 | Profiles RLS trop ouverte (GDPR) | ✅ Fait + corrigé après bug | `supabase/migrations/20260518_audit_fix01_profiles_rls.sql` | Testé en double compte (zejly12 + zejlyz123) — picker assignee montre bien les 2 |

### Bug découvert & corrigé pendant le Sprint 1

**§5.1 v1 cassée** : ma première version de la RLS utilisait uniquement `my_workspace_ids()` (membres only). L'owner d'un workspace n'étant pas dans `workspace_members`, le picker ne montrait plus que soi-même.

**v2 (déployée)** : ajout de 2 cas supplémentaires :
- Membres visibles via `my_workspace_ids()` **OU** `my_org_workspace_ids()` (couvre cas owner)
- Owner visible aux membres via `workspaces` join

Fichier maintenant **idempotent** (DROP IF EXISTS pour toutes les policies).

### Bonus collatéral (non planifié)

| # | Titre | Statut | Note |
|---|---|---|---|
| §4.1 | `eslint-disable @typescript-eslint/no-explicit-any` file-wide dans `tasks/service.ts` | ✅ Fait | Tombé pendant le grand fix TypeScript du commit Phase 2.5 (cause : 38 erreurs implicit any non détectées par le hook lint-staged local mais bloquantes en CI) |

---

## Sprint 2 — Fix flash & flickering ✅ TERMINÉ (2026-05-18)

| # | Titre | Statut | Fichier | Vérifié |
|---|---|---|---|---|
| §1.4 | Optimistic update préserve les jointures (status/assignee résolues depuis cache) | ✅ Fait | `src/modules/tasks/hooks/useTasks.ts` (`useUpdateTask.onMutate`) | typecheck + lint clean — **test UX en attente** |
| §2.3 | `useRealtimeTasks` chirurgical (setQueryData sur UPDATE/DELETE + invalidate seulement sur INSERT) | ✅ Fait | `src/modules/tasks/hooks/useRealtimeTasks.ts` | typecheck + lint clean |
| §2.2 | `useRealtimeTags` + `task_tags` ciblé (invalide uniquement caches contenant le tag/task) | ✅ Fait | `src/modules/tasks/hooks/useRealtimeTags.ts` | typecheck + lint clean |

### Test UX à faire après Sprint 2

- **§1.4** : Ouvrir EditTaskModal, changer status/assignee → la carte ne doit PAS flicker avec status/assignee vide pendant 200-500ms
- **§2.3** : Compte A drag une task, compte B doit voir le déplacement instantanément (sans re-render global du kanban)
- **§2.2** : Compte A ajoute un tag à une task, compte B doit voir le chip apparaître sans repaint du kanban entier

### Bug découvert pendant tests Sprint 2 (hors audit, fixé)

**Tags unique constraint pas migré en Phase 7**

| Symptôme | Cause | Fix |
|---|---|---|
| Erreur 23505 sur `tags_user_id_name_key` quand on crée un tag avec le même nom dans 2 workspaces différents | La migration `20260518_phase7_tags_per_workspace.sql` a ajouté `workspace_id` mais n'a pas droppé l'ancien constraint `(user_id, name)` | Nouvelle migration `20260518_audit_fix02_tags_unique_per_workspace.sql` — drop ancien constraint + create `(workspace_id, name)` |

**Note Sonnet (correction apportée à l'audit)** : §3.6 staleTime → 10 min (pas Infinity).

---

## Sprint 3 — Quick wins UX ✅ TERMINÉ (2026-05-18)

| # | Titre | Statut | Fichier |
|---|---|---|---|
| §2.6 | TopBar avatar : `getSession()` (0ms localStorage) au lieu de `getUser()` (~150ms) | ✅ Fait | `src/shared/components/layout/TopBar.tsx` |
| §6.6 | `TaskCard` due_date affiche l'année si différente de l'année courante | ✅ Fait | `src/modules/tasks/components/kanban/TaskCard.tsx` |
| §2.4 | TasksTopbar search → `Input variant="tasks"` (pattern cohérent avec Goals/Habits/Books) | ✅ Fait | `src/modules/tasks/components/layout/TasksTopbar.tsx` |
| §2.5 | `onMouseEnter`/`Leave` inline → Tailwind `hover:` (7 occurrences supprimées) | ✅ Fait | `src/modules/tasks/components/layout/TasksTopbar.tsx` + `TasksSidebar.tsx` |
| §4.5 | `TaskCard` grip handle + assignee fallback : inline `style` → Tailwind | ✅ Fait | `src/modules/tasks/components/kanban/TaskCard.tsx` |
| §2.7 | `useCurrentUserId` lit localStorage **synchrone** dans `useState` initializer | ✅ Fait | `src/shared/hooks/useCurrentUserId.ts` |

### Bonus collatéral

| # | Titre | Note |
|---|---|---|
| §6.5 | TaskCard drag handle invisible au touch | ✅ Résolu pendant §4.5 — ajout `[@media(hover:none)]:opacity-50` au grip handle |

---

## Sprint 4 — Performance perçue ✅ TERMINÉ (2026-05-19)

### Faits (6/6)

| # | Titre | Statut | Fichier |
|---|---|---|---|
| §3.3 | `getCurrentOrgId` in-flight memoization (dédup queries parallèles cold mount) | ✅ Fait | `src/shared/utils/getOrgId.ts` (+ export `resetOrgIdCache`) |
| §3.4 | Dashboard `getTodayTasks` inner join + filter `statuses.is_completed=false` server-side | ✅ Fait | `src/modules/dashboard/service.ts` |
| §3.5 | `getMediaItems` limit 100 par défaut | ✅ Fait | `src/modules/watching/service.ts` |
| §3.1 | Cookie middleware `hegon_has_workspace` — 5 garde-fous | ✅ Fait | `src/middleware.ts` + `actions.ts` + `invite/[token]/page.tsx` + `useWorkspaces.ts` |
| §3.2 | Goal progress → RPC `recalc_goal_progress` + helper `recalcGoalIfAuto` | ✅ Fait | `supabase/migrations/20260519_audit_fix03_goal_recalc_rpc.sql` + `goals/service.ts` + `useTasks.ts` |
| §3.7 | Bundle analyzer | ⏭ Reporté | Avant vidéo LinkedIn — pas urgent |

### (Archivé) À faire (3/6) — décisions prises avec Opus 2026-05-18

#### §3.1 Middleware cookie — GO (5 garde-fous)

**Insight Opus crucial** : le gain réel est **30-50 ms par nav**, pas 200-400 ms comme suggéré dans l'audit initial. Le `getUser()` reste obligatoire, le cookie skip uniquement la query `workspaces`. **À mesurer après application** : si gain < 30 ms, drop le cookie et garde le middleware simple.

**Garde-fous validés :**
1. Clear automatique dans signOut server action
2. Clear côté `/onboarding/complete` (au cas où user revient)
3. Durée 7 jours (pas 30, pas 1)
4. Set cookie après `acceptInvitation` réussie côté `/invite/[token]/page.tsx` — sinon invité paye le coût à la 1ère nav
5. Clear cookie dans `useDeleteWorkspace.onSuccess` quand `workspaces.length === 1` avant la mutation — sinon user fantôme bloqué

**Fichiers à toucher :**
- `src/middleware.ts` (cookie check + set après workspace check OK)
- `src/infrastructure/auth/actions.ts` (signOut → clear cookie)
- `src/app/invite/[token]/page.tsx` (set cookie après acceptInvitation)
- `src/app/onboarding/page.tsx` ou complete handler (clear cookie au retour onboarding)
- `src/modules/tasks/hooks/useWorkspaces.ts` (useDeleteWorkspace → clear si dernier)

#### §3.2 Goal progress — PIVOT vers RPC (option 3, pas trigger)

**Opus a pivoté** sur la reco initiale après débat. Validé : "explicite > magique". L'audit a été patché par Opus (§3.2 + §A.5 reflètent maintenant RPC, pas trigger).

**Plan validé :**
- Migration SQL : fonction `recalc_goal_progress(p_goal_id uuid) RETURNS void` (PLPGSQL, corps identique à l'ancien trigger)
- Callable via `supabase.rpc('recalc_goal_progress', { p_goal_id })` depuis service.ts
- Refactor `useTasks.ts` : extraire helper `recalcGoalIfAuto(queryClient, goalId)` qui appelle la RPC + invalide les caches → supprime la duplication 4×
- **Réserve pour plus tard** : RPC combinée `move_task_and_recalc(p_task_id, p_new_status_id, p_new_position)` qui ferait drag + recalc en 1 round trip. **Mesurer d'abord avec la simple, optimiser si nécessaire.**

**Gain attendu :** 4 round trips par mutation → 2 round trips (mutation + recalc). Pas 1 round trip à moins de faire la combinée plus tard.

**Fichiers à toucher :**
- Nouvelle migration `supabase/migrations/20260519_audit_fix03_goal_recalc_rpc.sql`
- `src/modules/goals/service.ts` (ajouter wrapper `recalculateProgress` via RPC)
- `src/modules/tasks/hooks/useTasks.ts` (extraire helper, simplifier 4 onSuccess)

#### §3.7 Bundle analyzer — REPORTÉ avant vidéo LinkedIn (pas urgent)

---

## Sprint 5 — Tasks design system retrofit ✅ TERMINÉ (2026-05-19)

| # | Titre | Statut | Fichiers |
|---|---|---|---|
| §6.1 | Design pass complet Tasks — onMouseEnter/Leave → Tailwind, inline styles → tokens | ✅ Fait | `KanbanColumn.tsx` + `ListRow.tsx` + `CalendarDayCell.tsx` (Sprint 3 avait déjà fait Topbar + Sidebar + TaskCard) |

---

## Sprint 6 — Sécurité restante ✅ TERMINÉ (2026-05-19)

| # | Titre | Statut | Fichiers |
|---|---|---|---|
| §5.2 | Middleware defense-in-depth : `checkUserHasWorkspace` explicite (owned + members en parallèle) | ✅ Fait | `src/middleware.ts` |
| §5.3 | `createWorkspaceInvitation` owner-only + migration RLS `org_invitations` | ✅ Fait | `src/modules/tasks/service.ts` + `20260519_audit_fix04_invitations_rls.sql` |
| §5.4 | Rate limit 10 invitations/heure — DB count (`org_invitations` WHERE `invited_by + created_at`) | ✅ Fait | `src/modules/tasks/service.ts` |
| §5.5 | Messages d'erreur génériques + `console.error` pour Sentry (4 hooks) | ✅ Fait | `src/modules/tasks/hooks/useTasks.ts` |
| §5.6 | Token regex `[A-Za-z0-9_-]{20,128}` avant RPC `accept_invitation` | ✅ Fait | `src/modules/orgs/service.ts` |
| §5.7 | `lh3.googleusercontent.com` ajouté aux `remotePatterns` (CSP déjà OK, pas de wildcard `**`) | ✅ Fait | `next.config.ts` |

**Migration à appliquer dans Supabase Studio :** `20260519_audit_fix04_invitations_rls.sql`

---

## Sprint 7 — Crashes restants ✅ TERMINÉ (2026-05-19)

| # | Titre | Statut | Fichiers |
|---|---|---|---|
| §1.3 | Race condition sur `position` à la création | ✅ Fait | `20260519_audit_fix05_task_position_trigger.sql` + `tasks/service.ts` (SELECT MAX supprimé, position omise → trigger atomique) |
| §1.5 | `pickPriorityTask` perd les CRITIQUES sans due_date | ✅ Fait | `20260519_audit_fix06_task_priority_rank.sql` + `dashboard/service.ts` (order priority_rank → due_date) |
| §1.6 | `getInvitationByToken` swallow erreur réseau | ✅ Fait | `invite/[token]/page.tsx` (state `network_error` + retry button + retryKey) |
| §1.7 | `formatEventDate` casse au passage DST | ✅ Fait | `dashboard/service.ts` (diff via dates Paris YYYY-MM-DD, pas ms wall-clock) |
| §1.8 | `useInitOrg` fail silencieux | ✅ Fait | `useInitOrg.ts` (console.error + toast.error) |

### Migrations à appliquer dans Supabase Studio
- `20260519_audit_fix05_task_position_trigger.sql` — triggers position atomique tasks + projects
- `20260519_audit_fix06_task_priority_rank.sql` — colonne priority_rank GENERATED + index

---

## Sprint 8 — Qualité code restante ✅ TERMINÉ (2026-05-19)

| # | Titre | Statut | Fichiers |
|---|---|---|---|
| §4.3 | `getOrgMembers` exposé renvoie TOUTE la table profiles | ✅ Fait | `tasks/service.ts` (supprimé) + `useOrgMembers.ts` (useOrgMembers + ORG_MEMBER_KEYS supprimés) |
| §4.4 | Helper `recalcGoalIfAuto` extrait (DRY) | ✅ Déjà fait Sprint 4 | — |
| §4.6 | `WorkspaceInvitePanel` bypasse `useRealtimeSync` | ✅ Fait | `WorkspaceInvitePanel.tsx` (raw channel → 2× useRealtimeSync) |
| §4.7 | `useTasks.ts` types `any` dans setQueryData | ✅ Fait | `useTasks.ts` (setQueryData&lt;Task[]&gt; + callbacks typés) |
| §4.8 | `dashboard.service` `Promise.all` sans tolérance d'erreur | ✅ Fait | `dashboard/service.ts` (Promise.allSettled + fallback par section + console.error) |

---

## Sprint 9 — UX & cohérence restante ✅ TERMINÉ (2026-05-19)

| # | Titre | Statut | Fichiers |
|---|---|---|---|
| §6.3 | Pas d'état d'erreur sur les listes | ✅ Fait | `KanbanBoard.tsx` + `ListView.tsx` (isError + retry) |
| §6.4 | `WorkspaceInvitePanel` width `w-110` non-spec | ✅ Fait | `WorkspaceInvitePanel.tsx` (w-110 → w-120 = 480px) |
| §6.5 | TaskCard drag handle invisible au touch | ✅ Déjà fait Sprint 3 | — |
| §6.7 | Empty state dashboard pour zéro orgs | ✅ Fait | `DashboardContent.tsx` (Refresh + lien onboarding) |
| §6.8 | z-index `z-9999` non standard | ✅ Fait | `TopBar.tsx` + `Sidebar.tsx` (×2) + `Dock.tsx` → z-20 |

---

## Gaps audit v1 — à traiter séparément

| Gap | Décision |
|---|---|
| Tests coverage | À mesurer (`vitest --coverage` 1 min). Améliorer = sessions multiples séparées. |
| A11y | Planifié avant prospection LinkedIn (pas pour la 1ère vidéo). |
| Bundle size réel | À mesurer + optimiser **avant** vidéo LinkedIn. |
| DB indexes audit | À faire ici via grep migrations + cross-check services. ~15-20 min. |

---

## État global au 2026-05-19 23:30 (AUDIT + RE-REVIEW OPUS FIXES ✅)

- **Sprints terminés** : 9/9
- **Items audit fixés** : ~45 critiques/élevés/moyens sur ~50 total (~90%)
- **Opus re-review fixes appliqués** (2026-05-19 23:30) :
  - §3.6 staleTime tasks 3min → 10min ✅
  - §4.1 eslint-disable file-wide retirés (tasks/types.ts + dashboard/service.ts + useTasks.ts bonus) ✅
  - §3.2 v2 RPC `recalc_goal_progress` `RETURNS integer` + check mode dans SQL + `SECURITY INVOKER` ✅ → migration `_fix07_goal_recalc_returns_int.sql`
  - §2.1 KanbanBoard skeleton chirurgical (workspaces/projects ne bloquent plus) ✅
  - WorkspaceInvitePanel retrofit complet (5 onMouseEnter → hover Tailwind, native inputs → Input variant tasks, boutons unifiés) ✅
- **Aucun commit** — working tree complet prêt pour commit unique
- **Aucun commit** — tout reste en working tree jusqu'à fin d'application

### Bonus hors-audit fixés cette session
- `ListRow` bouton `...` était vide → DropdownMenu Edit+Delete connecté
- `ListGroup` : nettoyage onMouseEnter/Leave
- `DeleteTaskModal` partout : TaskCard + ListRow (avant : delete direct sans confirmation)
- `§4.4` (recalcGoalIfAuto) : déjà fait en Sprint 4 §3.2

### Migrations appliquées dans Supabase Studio ✅
- `20260519_audit_fix03_goal_recalc_rpc.sql` — RPC recalc_goal_progress
- `20260519_audit_fix04_invitations_rls.sql` — RLS invitations owner-only

### Audit + Opus fixes terminés — prochaine étape : migration `_fix07_` à appliquer, puis commit unique, puis brainstorm modules

---

## Pour Opus (re-vérification)

Quand tu re-audit après application complète :
1. Lire ce fichier en entier pour voir ce qui est fait
2. Vérifier que les `Statut: ✅ Fait` sont réellement dans le code (pas juste documentés)
3. Re-noter chaque axe (0-10) et comparer avec les scores cibles
4. Flag tout regression ou nouveau problème introduit par les fixes
5. Confirmer ou challenger les "corrections apportées à l'audit" (notes Sonnet)
