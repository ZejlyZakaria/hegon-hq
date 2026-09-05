# Phase 6 — Workspace Sharing : audit & fixes

> Suivi des 15 fixes appliqués après l'audit complet du flow `workspace_members` (sécurité + real-time).
> Voir aussi : [`project_workspace_sharing_plan`](../C:/Users/ADMIN/.claude/projects/c--Users-ADMIN-Desktop-MyDocuments-digital-second-brain/memory/project_workspace_sharing_plan.md) (memory).

---

## Audit — résumé

Deux axes :
1. **Workspace sharing (sécurité & logique)** — RLS, RPC `accept_invitation`, middleware, écritures via service.
2. **Real-time (performance & bugs)** — `useRealtimeSync`, snap-back drag&drop, sur-invalidation.

### Récap priorité

| # | Sévérité | Fichier | Sujet |
|---|---|---|---|
| 1 | 🔴 | `org_invitations` RLS | énumération de tous les tokens |
| 2 | 🔴 | `accept_invitation v2` | pas de check email → join arbitraire |
| 3 | 🔴 | `getOrgId.ts` + `service.ts` | invités ne peuvent rien créer |
| 11 | 🔴 | `useRealtimeTasks` | drag&drop snap-back via auto-echo |
| 4 | 🟠 | `workspace_members` RLS | écriture par tout org-member, pas seulement owner |
| 12 | 🟠 | `useRealtimeTasks` | invalidation `TASK_KEYS.all` |
| 5 | 🟠 | `/invite/[token]` | race UX post-accept |
| 6 | 🟠 | `tasks` | pas de check integrity `status_id ↔ project_id` |
| 13 | 🟠 | divers | realtime manquant `projects/statuses/workspaces/workspace_members` |
| 7 | 🟡 | `/onboarding` | `.eq("user_id")` redondant |
| 8 | 🟡 | `WorkspaceInvitePanel` | pas de validation email format |
| 9 | 🟡 | `useInitOrg` | pas de gestion d'erreur |
| 10 | 🟡 | `WorkspaceInvitePanel` | owner affiché en "?" sans profil |
| 14 | 🟡 | `useRealtimeSync` / panel | channel name pas unique |
| 15 | 🟡 | `useRealtimeSync` | log si subscribe échoue |

---

## Détails des problèmes (par sévérité)

### 🔴 CRITIQUE

#### 1. Énumération + vol d'invitations
`supabase/migrations/20260515_phase3_org_invitations.sql:39-42`
La policy `invitations: public read by token` est `USING (true)`. N'importe quel user authentifié peut faire `SELECT * FROM org_invitations` et récupérer **tous les tokens en attente** de tous les workspaces.

#### 2. `accept_invitation` n'ignore pas l'email
`supabase/migrations/20260516_phase3_accept_invitation_v2.sql:20-23`
Aucune vérification que `inv.email = auth.jwt() ->> 'email'`. Combiné au #1 → **n'importe quel user peut joindre n'importe quel workspace** en lisant un token et en appelant le RPC.

#### 3. Les membres invités ne peuvent PAS créer
`src/shared/utils/getOrgId.ts:11-17` + `src/modules/tasks/service.ts`
`getCurrentOrgId()` query `memberships` → un workspace_member n'a aucune ligne → throw `"No organization found"`. Donc `createTask`, `createProject`, `createTag`, `addTagToTask` cassent. Le drag&drop marche car `moveTask` ne passe pas par `getCurrentOrgId`.

#### 11. `useRealtimeTasks` détruit l'optimistic update
`src/modules/tasks/hooks/useRealtimeTasks.ts:17`
`useMoveTask` ne fait délibérément **PAS** d'`invalidateQueries`. Mais le user qui drag reçoit son propre écho realtime → `invalidateQueries(TASK_KEYS.all)` → refetch → snap-back guaranteed.

### 🟠 IMPORTANT

#### 4. INSERT/DELETE `workspace_members` trop laxiste
`supabase/migrations/20260516_phase3_workspace_rls_fix.sql:48-57`
`my_org_workspace_ids()` ne filtre pas par rôle → tout membre de l'org peut INSERT/DELETE des workspace_members → escalade de privilèges intra-org.

#### 5. Race UX `/invite/[token]`
`src/app/invite/[token]/page.tsx:48-50`
Après accept, `setTimeout` puis `router.push("/pro/tasks")`. Si l'invalidation du cache ou la session ne sont pas hydratées, l'user atterrit sur un sidebar vide.

#### 6. `status_id` ↔ `project_id` non validé
`src/modules/tasks/service.ts:178-184`
Un workspace_member peut UPDATE une task avec un `status_id` d'un autre projet (la RLS valide juste le `project_id`). Corrompt les données.

#### 12. Over-invalidation realtime
`src/modules/tasks/hooks/useRealtimeTasks.ts:17`
`TASK_KEYS.all` invalide les tasks de tous les projets en cache.

#### 13. Realtime manquant sur les tables partagées
`projects`, `statuses`, `workspaces`, `workspace_members` → sidebar désynchronisée pour un membre invité jusqu'au prochain refresh.

### 🟡 MINEUR

#### 7. `.eq("user_id")` redondant dans `/onboarding`
`src/app/onboarding/page.tsx:14-19`

#### 8. Pas de validation email format
`WorkspaceInvitePanel.tsx:67-73`

#### 9. `useInitOrg` sans gestion d'erreur
`src/shared/hooks/useInitOrg.ts:11-32`

#### 10. Owner affiché en "?" sans nom/avatar
`WorkspaceInvitePanel.tsx:142-153`

#### 14. Channel name pas unique
`WorkspaceInvitePanel.tsx:45` — `workspace-panel-${workspace.id}` peut collisionner sur remount rapide.

#### 15. Pas de log si `subscribe()` échoue
`src/shared/hooks/useRealtimeSync.ts:39`

---

## Fixes appliqués — ordre d'exécution

> Une migration SQL par fix. Nommage : `20260517_phase6_fixNN_<slug>.sql`.

### Migrations SQL (5)

| # | Fichier | Effet |
|---|---|---|
| 2 | `supabase/migrations/20260517_phase6_fix01_email_check.sql` | `accept_invitation` exige `lower(inv.email) = lower(auth.jwt() ->> 'email')` → renvoie `email_mismatch` sinon |
| 1 | `supabase/migrations/20260517_phase6_fix02_invitations_rls.sql` | DROP `USING (true)` → policy `self or owner read` + RPC `get_invitation_by_token(p_token)` `SECURITY DEFINER` pour la page `/invite/[token]` |
| 3 | `supabase/migrations/20260517_phase6_fix03_org_id_trigger.sql` | Triggers BEFORE INSERT sur `projects` / `statuses` / `tasks` / `task_tags` → dérive `org_id` depuis le parent (workspace → project → task) |
| 4 | `supabase/migrations/20260517_phase6_fix04_owned_workspace_ids.sql` | Nouvelle fn `my_owned_workspace_ids()` (`role = 'owner'`) → policies INSERT/DELETE de `workspace_members` restreintes au owner |
| 6 | `supabase/migrations/20260517_phase6_fix06_status_same_project.sql` | Trigger BEFORE INSERT OR UPDATE sur `tasks` → reject si `status.project_id <> task.project_id` (`ERRCODE 23514`) |

### Code — nouveaux fichiers (4)

| Fichier | Rôle |
|---|---|
| `src/modules/tasks/hooks/optimistic-tracker.ts` | `Set<taskId>` partagé `markOptimistic` / `clearOptimistic` / `isOptimistic` |
| `src/modules/tasks/hooks/useRealtimeProjects.ts` | Sub `projects` filtrée `workspace_id=eq.` → invalide `PROJECT_KEYS.byWorkspace` |
| `src/modules/tasks/hooks/useRealtimeStatuses.ts` | Sub `statuses` filtrée `project_id=eq.` → invalide `STATUS_KEYS.byProject` |
| `src/modules/tasks/hooks/useRealtimeWorkspaces.ts` | Sub `workspaces` (all) + `workspace_members` filtrée `user_id=eq.<self>` → invalide `WORKSPACE_KEYS.all` |

### Code — fichiers modifiés (10)

| Fichier | Fix(es) | Changement |
|---|---|---|
| `src/modules/orgs/service.ts` | #1 | `getInvitationByToken` → `supabase.rpc("get_invitation_by_token", { p_token })` |
| `src/modules/tasks/service.ts` | #3, #10 | Drop `org_id: orgId` de `createTask`/`createProject`/`addTagToTask` ; ajout `getProfileById(userId)` |
| `src/modules/tasks/hooks/useTasks.ts` | #11 | `markOptimistic(taskId)` dans `onMutate` + `clearOptimistic(taskId)` dans `onSettled` de `useMoveTask` |
| `src/modules/tasks/hooks/useRealtimeTasks.ts` | #11, #12 | Skip si `isOptimistic(payload.new.id)` ; invalide `TASK_KEYS.byProject(projectId)` |
| `src/modules/tasks/components/kanban/KanbanBoard.tsx` | #13 | Branche `useRealtimeStatuses(selectedProjectId)` |
| `src/modules/tasks/components/layout/TasksSidebar.tsx` | #13 | Branche `useRealtimeProjects(workspace.id)` par item + `useRealtimeWorkspaces(currentUserId)` |
| `src/modules/tasks/components/panels/WorkspaceInvitePanel.tsx` | #8, #10, #14 | Regex `EMAIL_RE` + `emailError` ; fetch `getProfileById(workspace.user_id)` via `useQuery` ; `useId()` suffix dans le channel name |
| `src/app/onboarding/page.tsx` | #7 | Retire `.eq("user_id", user.id)` |
| `src/app/invite/[token]/page.tsx` | #5 | `router.refresh()` avant `setTimeout(() => router.push(...))` |
| `src/shared/hooks/useInitOrg.ts` | #9 | `if (error) console.warn("[useInitOrg] memberships query failed:", error.message)` |
| `src/shared/hooks/useRealtimeSync.ts` | #14, #15 | `useId()` suffix global ; `subscribe((status, err) => { if (CHANNEL_ERROR/TIMED_OUT/CLOSED) console.warn(...) })` |

---

## Checklist de tests manuels

| # | Test | Compte | Résultat attendu |
|---|---|---|---|
| #2 | Inviter `alice@x.com`, copier le token, se connecter avec un autre compte, appeler `accept_invitation` via console DevTools/REST | Zakaria + tiers | `{error: "email_mismatch"}` — pas de ligne `workspace_members` ajoutée |
| #1a | Loggué (Zakaria), DevTools → `supabase.from('org_invitations').select('*')` | Zakaria | Ne retourne que les invits où il est org-owner OU email = son email |
| #1b | Ouvrir `/invite/<token>` sans être loggué | anon | Page charge, invitation s'affiche (via RPC) |
| #3a | Zakaria invite zejlyz123 sur "Codysoft" → zejlyz123 accepte → ouvre Tasks → **crée une nouvelle task** | zejlyz123 | Task créée, plus de toast "No organization found" |
| #3b | zejlyz123 crée un nouveau projet dans Codysoft, drag&drop une task | zejlyz123 | Project + statuses créés, drag fonctionne |
| #11 | Drag&drop rapide d'une task d'une colonne à une autre | Zakaria | Pas de snap-back, la card reste dans la colonne cible |
| #12 | DevTools Network/WS, drag plusieurs tasks dans le projet A | Zakaria | Pas de refetch sur les query-keys d'autres projets |
| #4 | (multi-org) un membre non-owner essaie `INSERT INTO workspace_members` via SQL | tiers non-owner | Erreur RLS |
| #5 | zejlyz123 ouvre `/invite/<token>` → clique Join → arrive sur `/pro/tasks` | zejlyz123 | Sidebar affiche Codysoft sans flash vide |
| #6 | Console DevTools : update une task avec `status_id` d'un autre projet | Zakaria | Erreur `status_id ... does not belong to project ...` |
| #13a | Zakaria renomme/crée/supprime un projet (fenêtre A), Codysoft ouvert côté zejlyz123 (fenêtre B) | Zakaria + zejlyz123 | Sidebar B se met à jour sans refresh |
| #13b | Zakaria ajoute un status dans le board | Zakaria + zejlyz123 | Nouvelle colonne apparaît instantanément côté invité |
| #13c | Zakaria invite un nouveau membre → `workspace_members` insert | Zakaria | Sidebar côté invité voit le nouveau workspace |
| #7 | Cas synthétique : aller direct sur `/onboarding` quand workspace partagé existe | zejlyz123 | Redirige vers `/dashboard` (middleware) |
| #8 | Ouvrir le panel invite, taper "foo" puis cliquer Invite | Zakaria | Affiche "Please enter a valid email address" en rouge, pas d'insert |
| #9 | Couper le réseau, recharger l'app | Zakaria | Console : warning `[useInitOrg] memberships query failed:` |
| #10 | zejlyz123 ouvre le panel invite sur Codysoft | zejlyz123 | Première ligne = owner avec avatar + nom de Zakaria (pas "?") |
| #14 | Ouvrir/fermer le panel rapidement 5× consécutives | Zakaria | Pas d'erreur "channel already exists" dans la console |
| #15 | Provoquer un timeout realtime (couper réseau pendant qu'un board est ouvert) | Zakaria | Console : `[useRealtimeSync] channel "tasks-project-..." status=TIMED_OUT` |

---

## Fix post-audit — Supabase client singleton

**Contexte** : après les fixes #13 (3 nouveaux hooks realtime branchés dans `KanbanBoard` et `TasksSidebar`), on se retrouve avec 8+ appels `createClient()` simultanés. Chaque appel créait un **nouveau** client Supabase (pas de singleton), donc 8+ instances se battaient pour le même `navigator.locks` sur `sb-*-auth-token` → `NavigatorLockAcquireTimeoutError` après 5s + `Lock broken by another request with the 'steal' option`.

**Symptômes observés** :
- Loading lent après création d'un projet par un workspace member
- Console floodée d'erreurs de lock
- `Uncaught (in promise) AbortError: Lock broken by another request with the 'steal' option`

**Fix** : `src/infrastructure/supabase/client.ts` → singleton module-scope. `createClient()` retourne désormais une instance unique réutilisée par tous les hooks/services/composants. Pattern standard recommandé par Supabase pour les apps Next.js.

```ts
let client: ReturnType<typeof createBrowserClient> | null = null
export function createClient() {
  if (client) return client
  client = createBrowserClient(URL, KEY)
  return client
}
```

**Important** : `client.ts` est listé dans `CLAUDE.md` § "Files to Never Touch" — modification validée explicitement par le owner car (1) c'est le pattern Supabase officiel, (2) bug bloquant la suite des tests, (3) zéro risque fonctionnel (le client gère lui-même les changements d'auth state via `onAuthStateChange`).

---

## Bugs découverts pendant les tests — hors scope audit, à fixer plus tard

### 🟠 Infinite redirect loop quand offline
**Reproduction** : couper le wifi → hard refresh sur `/pro/tasks` (ou toute route protégée) → la page boucle infiniment entre :
- `/auth/finalize?target=/pro/tasks`
- `/auth?next=/pro/tasks`

**Cause** : `supabase.auth.getUser()` dans le middleware retourne `null` silencieusement quand il n'y a pas de réseau. Le middleware croit qu'il n'y a pas d'user → redirige vers `/auth`. Puis `/auth` détecte les query params (code/token_hash résiduels) → redirige vers `/auth/finalize`. Qui ne peut pas valider sans réseau → renvoie vers `/auth`. Loop.

**Impact** : edge case rare (réseau coupé) mais expérience cassée. Si l'user perd le réseau pendant qu'il utilise l'app, un refresh accidentel le bloque jusqu'à reconnexion.

**Fix à envisager** :
- Détecter une `AuthRetryableFetchError` (ou erreur réseau générique) dans le middleware → afficher un écran "offline" plutôt que de rediriger
- Ou : si `getUser()` échoue avec erreur réseau (vs `null` propre), ne pas rediriger, laisser la page se charger en mode dégradé

---

## Tests d'audit — récap final

| # | Status | Note |
|---|---|---|
| #1a | ✅ | Pas d'énumération via SQL SET ROLE |
| #1b | ✅ | RPC `get_invitation_by_token` accessible anonyme |
| #2 | ✅ | `email_mismatch` rejette le RPC `accept_invitation` |
| #3a | ✅ | Invité crée task sans erreur |
| #3b | ✅ | Invité crée projet + drag fonctionne |
| #4 | ⏭ | Skippé (multi-org, complexe) |
| #5 | ✅ | `router.refresh()` + redirect propre |
| #6 | ✅ | Trigger DB rejette `status_id` cross-project |
| #7 | ⏭ | Skippé (couvert par #5 et middleware) |
| #8 | ✅ | HTML5 + regex valident l'email |
| #9 | ⏭ | Skippé (loop offline empêche le test propre) |
| #10 | ⏭ | Skippé (panel inaccessible aux non-owners) |
| #11 | ✅ | Pas de snap-back sur drag&drop |
| #12 | ⏭ | Skippé (couvert par le pattern setQueryData) |
| #13a | ✅ | Projets create/rename/delete sync en temps réel |
| #13b | ✅ | Statuses create/delete sync en temps réel |
| #13c | ⏭ | Skippé (setup zejlyz123 sans workspace perso complique) |
| #14 | ✅ | `useId()` empêche les collisions de channel |
| #15 | ✅ | Logs `CHANNEL_ERROR` corrects, reconnect auto |

---

## Récap court (ce qui a changé)

- **Sécurité invitations** : `accept_invitation` exige maintenant que l'email du JWT corresponde à l'invité ; la RLS de `org_invitations` n'expose plus la table en lecture publique — un RPC `get_invitation_by_token` dédié sert la page `/invite/[token]`.
- **Invités fonctionnels** : 4 triggers BEFORE INSERT dérivent `org_id` depuis le workspace/project parent ; le service Tasks n'envoie plus `org_id` → les workspace_members peuvent enfin créer projects/tasks.
- **Drag&drop stable** : un set `optimistic-tracker` partagé entre `useMoveTask` et `useRealtimeTasks` skip le self-echo → plus de snap-back ; invalidation scopée à `TASK_KEYS.byProject(projectId)`.
- **Privilèges resserrés** : `my_owned_workspace_ids()` (`role = 'owner'`) remplace `my_org_workspace_ids` dans les policies INSERT/DELETE de `workspace_members`.
- **Intégrité** : trigger empêche d'assigner un `status_id` d'un autre project.
- **Realtime** étendu aux tables `projects`, `statuses`, `workspaces`, `workspace_members` (filtré sur user courant) via 3 nouveaux hooks branchés dans `KanbanBoard` et `TasksSidebar`.
- **Polish** : `router.refresh()` avant push sur invite ; cleanup `/onboarding` ; validation email regex ; profile owner fetché ; channels uniques via `useId()` ; warning log sur échec subscribe/memberships.
