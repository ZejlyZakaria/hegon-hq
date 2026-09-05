# HEGON — Digger Audit 2026-05-18

> Diagnostic brutal sans filtre, standard Linear / Vercel / Apple.
> Périmètre : middleware, infra Supabase, modules tasks (heavy), goals/dashboard/watching (échantillon), shared (state/realtime/query/auth).
> Méthode : lecture complète de `hq/` + memory index + ~25 fichiers source critiques.

---

## Changelog

| Date | Changement |
|---|---|
| 2026-05-18 | Audit initial — 6 axes, ~40 findings, score 5.9/10 |
| 2026-05-18 | **Patch review #1** — corrections §1.5 (priority_rank GENERATED, le `.order("priority")` alphabétique était cassé), §3.6 (`staleTime` à 10 min plutôt qu'`Infinity` pour garder le filet `refetchOnReconnect`), §3.2 (warning explicite : trigger DB = décision structurante, exception à la règle "service layer is the law" du blueprint, à acter dans `blueprint.md §12` avant d'implémenter) |
| 2026-05-18 | **Patch review #2 (Sprint 4 décisions)** — §3.1 cookie pattern validé GO avec 5 garde-fous (3 Sonnet + 2 ajoutés : set après `acceptInvitation`, clear sur dernier workspace supprimé) + warning mesure avant commit. §3.2 pivot trigger → RPC `recalc_goal_progress(p_goal_id)` : reste explicite dans service layer, blueprint §9 respecté (pas d'exception nécessaire), testable end-to-end. Spec §A.5 réécrite : FUNCTION PLPGSQL au lieu de TRIGGER. |

---

## Table des matières

- [Synthèse globale](#synthèse-globale)
- [Top 10 prioritaire](#top-10-à-corriger-en-premier)
- [Axe 1 — Crashes & erreurs silencieuses](#1-crashes--erreurs-silencieuses)
- [Axe 2 — Flash & flickering](#2-flash--flickering)
- [Axe 3 — Performance perçue](#3-performance-perçue)
- [Axe 4 — Qualité du code](#4-qualité-du-code)
- [Axe 5 — Sécurité & données](#5-sécurité--données)
- [Axe 6 — UX & cohérence](#6-ux--cohérence)
- [Annexe — Patterns de fix prêts à coller](#annexe--patterns-de-fix-prêts-à-coller)

---

## Synthèse globale

| Axe | Score | Note |
|---|---|---|
| Crashes & erreurs silencieuses | **5/10** | Invariants critiques à trou (priority, multi-org, position uniqueness) |
| Flash & flickering | **6/10** | Realtime invalide trop large, optimistic update détruit les jointures |
| Performance perçue | **6.5/10** | Middleware double-query, cross-module mutations dupliquées 4× |
| Qualité du code | **6/10** | `eslint-disable` file-wide, code mort commenté, DRY violations |
| Sécurité & données | **6/10** | Profiles RLS ouvert, defense-in-depth manquante côté middleware |
| UX & cohérence | **6/10** | Tasks pas retrofité, états d'erreur absents, autosaves bavards |
| **Moyenne** | **5.9/10** | |

**Verdict :** HEGON est un produit fonctionnel mais à **mi-distance de Linear**. Fondations CSR + real-time + multi-tenant solides, mais application régulièrement en deçà des règles écrites dans `hq/`. Le module Tasks (le plus visible pour la vidéo LinkedIn) est paradoxalement le moins poli. La Phase 2.5 collab a été shippée vite — il y a des trous d'invariants qui ne tiennent pas devant une démo publique.

---

## Top 10 à corriger en premier

1. **[CRITIQUE]** Dashboard invités CodySoft cassé — `dashboard/service.ts` filtre par `created_by` (§1.1)
2. **[CRITIQUE]** Table `profiles` lisible par tous les users authentifiés — privacy GDPR (§5.1)
3. **[CRITIQUE]** `getCurrentOrgId` non-déterministe pour multi-org — invité atterrit dans la mauvaise org (§1.2)
4. **[ÉLEVÉE]** `useUpdateTask` optimistic écrase les jointures `assignee/status/tags` → flash 200-500 ms (§1.4)
5. **[ÉLEVÉE]** `useRealtimeTags` invalide `TASK_KEYS.all` à chaque ajout de tag → repaint global (§2.2)
6. **[ÉLEVÉE]** `useRealtimeTasks` invalide la liste entière → flash de re-render des TaskCards (§2.3)
7. **[ÉLEVÉE]** Middleware double query Supabase sur chaque navigation → +300 ms par clic (§3.1)
8. **[ÉLEVÉE]** Cross-module recalcul goal dupliqué 4× dans `useTasks` → 5 round trips Supabase par drag (§3.2)
9. **[ÉLEVÉE]** Tasks pas retrofité design system — inline styles, native input, `onMouseEnter` (§6.1)
10. **[ÉLEVÉE]** `createWorkspaceInvitation` pas owner-only au niveau service + zero rate limit (§5.3)

**Bonus à 11-12 (vite faits, gros impact) :**
- Toast spam sur autosave Tasks → retirer `toast.success` sur `useUpdateTask`
- 76 lignes commentées dans `middleware.ts` → delete
- `next.config.ts` `hostname: "**"` + CSP manquant → spec déjà dans `security.md §4.2/4.3`

---

## 1. Crashes & erreurs silencieuses

### 1.1 `[CRITIQUE]` Dashboard invisible pour les invités CodySoft

**Fichier :** [src/modules/dashboard/service.ts:85-112](../src/modules/dashboard/service.ts#L85) (`getTodayTasks`)
**Aussi :** [src/modules/dashboard/service.ts:377-403](../src/modules/dashboard/service.ts#L377) (`getTasksActivityByDay`)

**Problème :** filtre `eq("created_by", userId)`. Un membre invité voit **zéro tâche** sur son dashboard même s'il en a 30 assignées dans le workspace partagé. Toute la Phase 2.5 Collab vient d'être livrée — le dashboard la trahit.

**Root cause :** dashboard codé avant la Phase 2.5, jamais ré-audité après l'ajout des assignees / workspace_members.

**Fix recommandé :**
```ts
// Avant
.eq("created_by", userId)

// Après — tasks où je suis créateur OU assigné
.or(`created_by.eq.${userId},assignee_id.eq.${userId}`)
```
Et pour `getTasksActivityByDay`, même chose. Reflechir aussi : peut-être que le dashboard d'un invité doit *uniquement* montrer ses assignements (pas ses créations dans son org perso) — décision UX à prendre.

---

### 1.2 `[CRITIQUE]` `getCurrentOrgId` non-déterministe pour multi-org

**Fichier :** [src/shared/utils/getOrgId.ts:11-15](../src/shared/utils/getOrgId.ts#L11)

**Problème :** `.from("memberships").select("org_id").limit(1).single()` sans `.order(...)`. Pour un invité ayant 2 orgs (perso + Zak), Postgres renvoie l'org dans l'ordre arbitraire → il atterrit dans son org vide. Mémoire `project_tasks_roadmap` le liste "TODO plus tard" — mais c'est un crash UX silencieux **maintenant**, pas plus tard.

**Fix recommandé :**
```ts
// Avant
const { data } = await supabase
  .from("memberships")
  .select("org_id")
  .limit(1)
  .single();

// Après
const { data } = await supabase
  .from("memberships")
  .select("org_id, created_at")
  .order("created_at", { ascending: true })  // org perso d'abord (déterministe)
  .limit(1)
  .single();
```
Et **plus important** : dans `OrgService.acceptInvitation`, après succès, appeler `useOrgStore.getState().setOrg(result.org_id, ...)` automatiquement → l'invité ne dépend plus de l'ordre.

---

### 1.3 `[ÉLEVÉE]` Race condition sur `position` à la création

**Fichier :** [src/modules/tasks/service.ts:74-94](../src/modules/tasks/service.ts#L74) (`createTask`)
**Aussi :** [src/modules/tasks/service.ts:300-313](../src/modules/tasks/service.ts#L300) (`createProject`)

**Problème :** `SELECT MAX(position)` puis `INSERT position = max+1` en deux requêtes séparées. Deux users qui créent simultanément → même `position`. Tasks visuellement à la même hauteur dans le kanban.

**Fix recommandé :** trigger Postgres BEFORE INSERT qui calcule `max+1` atomiquement :
```sql
CREATE OR REPLACE FUNCTION set_task_position()
RETURNS TRIGGER AS $$
BEGIN
  IF NEW.position IS NULL THEN
    SELECT COALESCE(MAX(position), 0) + 1
    INTO NEW.position
    FROM tasks
    WHERE project_id = NEW.project_id AND status_id = NEW.status_id;
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER tasks_set_position
  BEFORE INSERT ON tasks
  FOR EACH ROW EXECUTE FUNCTION set_task_position();
```
Côté service, supprimer le SELECT MAX et juste insert sans `position` → trigger gère.

---

### 1.4 `[ÉLEVÉE]` Optimistic update détruit les jointures

**Fichier :** [src/modules/tasks/hooks/useTasks.ts:53-69](../src/modules/tasks/hooks/useTasks.ts#L53) (`useUpdateTask.onMutate`)

**Problème :** `{ ...task, ...updatedTask }` écrase `task.assignee`, `task.status`, `task.project`, `task.tags` quand `updatedTask` ne fournit que les IDs. `TaskCard` lit `task.assignee.full_name` → undefined pendant 200-500 ms, avatar disparaît puis revient. Flicker permanent à chaque edit.

**Fix recommandé :** résoudre les jointures depuis le cache TQ avant de patcher :
```ts
onMutate: async (updatedTask) => {
  const queryClient = useQueryClient();
  await queryClient.cancelQueries({ queryKey: TASK_KEYS.byProject(updatedTask.project_id) });

  // Récupérer status et assignee depuis caches existants
  const statuses = queryClient.getQueryData<Status[]>(STATUS_KEYS.byProject(updatedTask.project_id));
  const members = queryClient.getQueryData<Assignee[]>(PROJECT_ASSIGNEE_KEYS.byProject(updatedTask.project_id));
  const newStatus = statuses?.find((s) => s.id === updatedTask.status_id);
  const newAssignee = members?.find((m) => m.id === updatedTask.assignee_id) ?? null;

  const previousTasks = queryClient.getQueryData<Task[]>(TASK_KEYS.byProject(updatedTask.project_id));

  queryClient.setQueryData<Task[]>(TASK_KEYS.byProject(updatedTask.project_id), (old) => {
    if (!old) return old;
    return old.map((task) =>
      task.id === updatedTask.id
        ? {
            ...task,
            ...updatedTask,
            status: newStatus ?? task.status,
            assignee: newAssignee,
            // tags ne change pas via updateTask
          }
        : task
    );
  });

  return { previousTasks };
},
```

---

### 1.5 `[ÉLEVÉE]` `pickPriorityTask` perd les CRITIQUES sans due_date

**Fichier :** [src/modules/dashboard/service.ts:85-112](../src/modules/dashboard/service.ts#L85)

**Problème :** `.order("due_date", { ascending: true, nullsFirst: false }).limit(50)`. Une tâche `priority: critical` sans `due_date` se retrouve au-delà du LIMIT 50 et n'arrive jamais à `pickPriorityTask`. Le slot PRIORITY du dashboard ment.

⚠️ **Piège évité (patch 2026-05-18) :** un `.order("priority", { ascending: true })` direct sur la colonne text ne marche **pas** — Postgres trie alphabétiquement → ordre obtenu `critical < high < low < medium`. `low` se retrouve avant `medium`. La fix réelle nécessite une colonne `priority_rank` calculée.

**Fix recommandé :**

```sql
-- 20260518_tasks_priority_rank.sql
ALTER TABLE tasks ADD COLUMN priority_rank smallint
  GENERATED ALWAYS AS (
    CASE priority
      WHEN 'critical' THEN 0
      WHEN 'high'     THEN 1
      WHEN 'medium'   THEN 2
      WHEN 'low'      THEN 3
    END
  ) STORED;

CREATE INDEX idx_tasks_priority_rank ON tasks(priority_rank);
```

Puis côté service :
```ts
.order("priority_rank", { ascending: true })
.order("due_date", { ascending: true, nullsFirst: false })
.limit(50)
```

Avantage du `GENERATED STORED` : recalculé automatiquement par Postgres à chaque UPDATE, indexable, zéro maintenance côté app.

---

### 1.6 `[MOYENNE]` `getInvitationByToken` swallow + reset à `invalid`

**Fichier :** [src/app/invite/[token]/page.tsx:38-40](../src/app/invite/[token]/page.tsx#L38)

**Problème :** `catch` sans logging affiche "Invalid invitation" sur n'importe quelle erreur réseau. Un invité avec coupure Wi-Fi voit "invitation invalide" → contacte le owner. Faux signal.

**Fix recommandé :** distinguer erreur réseau (retry button) vs RPC rejection (state invalid) :
```ts
try {
  const inv = await OrgService.getInvitationByToken(token);
  // ...
} catch (err) {
  const isNetwork = err instanceof TypeError || (err as { message?: string }).message?.includes("fetch");
  setState(isNetwork ? "network_error" : "invalid");
}
```

---

### 1.7 `[MOYENNE]` `formatEventDate` casse au passage DST

**Fichier :** [src/modules/dashboard/service.ts:42-44](../src/modules/dashboard/service.ts#L42)

**Problème :** `Math.ceil(diff / 86400000)`. Le passage à l'heure d'été génère un jour de 23h → "In 0 days" pour un événement le lendemain. Arrive 2× par an Europe/Paris.

**Fix recommandé :**
```ts
import { differenceInCalendarDays } from "date-fns";

const days = differenceInCalendarDays(date, now);
```

---

### 1.8 `[MOYENNE]` `useInitOrg` fail silencieux

**Fichier :** [src/shared/hooks/useInitOrg.ts:14-19](../src/shared/hooks/useInitOrg.ts#L14)

**Problème :** erreur memberships → `console.warn` puis `return`. Si la query échoue (réseau, RLS broken), l'orgStore reste sur la valeur persistée — qui peut être un orgId révoqué. Aucune UI ne signale.

**Fix recommandé :** propager via toast OU forcer un re-auth si la liste arrive vide alors que `getUser()` existe.

---

**Score axe 1 : 5/10** — Tu as les outils (RLS, real-time, optimistic), mais les invariants critiques (priority, multi-org, position uniqueness) ont des trous. Du Linear ne ferait pas ces trous.

---

## 2. Flash & flickering

### 2.1 `[ÉLEVÉE]` KanbanBoard skeleton sur 5 loadings en OR

**Fichier :** [src/modules/tasks/components/kanban/KanbanBoard.tsx:153-161](../src/modules/tasks/components/kanban/KanbanBoard.tsx#L153)

**Problème :** `workspacesLoading || projectsLoading || statusesLoading || tasksLoading || (hasProjects && !selectedProjectId)` → skeleton plein écran. Même cache chaud, si une seule query est `pending` (par ex. realtime invalidation en cours), tu vois le skeleton pendant 100-500 ms. Sur chaque retour à `/pro/tasks`.

**Fix recommandé :** afficher `statuses + tasks` dès qu'ils sont prêts, lazy-attendre les autres en arrière-plan. Le skeleton local sur la sidebar suffit. Le kanban ne devrait dépendre que de `tasksLoading && !tasks` :
```tsx
if (!hasProjects && !projectsLoading) return <TasksEmptyState />;
if (!tasks && tasksLoading) return <TasksSkeleton />;
// reste du render — workspaces/projects load en arrière-plan, sidebar a son propre skeleton
```

---

### 2.2 `[ÉLEVÉE]` `useRealtimeTags` invalide `TASK_KEYS.all` à chaque ajout de tag

**Fichier :** [src/modules/tasks/hooks/useRealtimeTags.ts:18, 29](../src/modules/tasks/hooks/useRealtimeTags.ts#L18)

**Problème :** `queryClient.invalidateQueries({ queryKey: TASK_KEYS.all })`. Un collègue ajoute un tag à *une* tâche → toute la liste de tasks de tous les projets de tous les workspaces refetch. Sur 5 collab actifs ça repaint le kanban toutes les 2 secondes.

**Fix recommandé :** `setQueryData` ciblé :
```ts
useRealtimeSync({
  channelName: `task-tags-workspace-${workspaceId}`,
  table: "task_tags",
  enabled: !!workspaceId,
  onchange: (payload) => {
    const taskId = (payload.new as { task_id?: string })?.task_id
                 ?? (payload.old as { task_id?: string })?.task_id;
    if (!taskId) return;

    // Trouver le projectId de la tâche depuis les caches existants
    const allTaskCaches = queryClient.getQueriesData<Task[]>({ queryKey: TASK_KEYS.all });
    for (const [key, tasks] of allTaskCaches) {
      if (tasks?.some((t) => t.id === taskId)) {
        // Invalide UNIQUEMENT le cache qui contient cette tâche
        queryClient.invalidateQueries({ queryKey: key });
        return;
      }
    }
  },
});
```

---

### 2.3 `[ÉLEVÉE]` `useRealtimeTasks` invalide la liste entière sur chaque change

**Fichier :** [src/modules/tasks/hooks/useRealtimeTasks.ts:26](../src/modules/tasks/hooks/useRealtimeTasks.ts#L26)

**Problème :** `useRealtimeStatuses` fait du `setQueryData` chirurgical (mémoire validée), `useRealtimeTasks` reste sur `invalidateQueries`. Pour un kanban multi-user, chaque drag d'un collègue déclenche un refetch complet → flash de re-render de toutes les TaskCard. Le `optimistic-tracker` ne couvre que l'utilisateur local.

**Fix recommandé :** appliquer le pattern `setQueryData` de `useRealtimeStatuses` :
```ts
onchange: (payload) => {
  if (!projectId) return;
  const key = TASK_KEYS.byProject(projectId);
  const newRow = payload.new as Task | null;
  const oldRow = payload.old as { id?: string } | null;
  const changedId = newRow?.id ?? oldRow?.id;
  if (isOptimistic(changedId)) return;

  if (payload.eventType === "DELETE") {
    queryClient.setQueryData<Task[]>(key, (old) => old?.filter((t) => t.id !== changedId));
    return;
  }
  if (payload.eventType === "UPDATE" && newRow) {
    queryClient.setQueryData<Task[]>(key, (old) =>
      old?.map((t) => (t.id === newRow.id ? { ...t, ...newRow } : t))
    );
    return;
  }
  // INSERT → invalider car on n'a pas les jointures
  queryClient.invalidateQueries({ queryKey: key });
},
```

---

### 2.4 `[ÉLEVÉE]` TasksTopbar search en `<input>` natif + inline styles

**Fichier :** [src/modules/tasks/components/layout/TasksTopbar.tsx:137-151](../src/modules/tasks/components/layout/TasksTopbar.tsx#L137)

**Problème :** pas de `Input variant="tasks"`, hover dégradé par rapport au reste de l'app. Violation directe de `project_card_pattern` ("toujours Input shadcn variant tasks — jamais native input"). Le focus visuel diffère de Goals/Habits/Books.

**Fix recommandé :**
```tsx
<div className="relative flex-1">
  <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-text-tertiary pointer-events-none" />
  <Input
    variant="tasks"
    type="text"
    value={filters.search}
    onChange={(e) => setFilters({ search: e.target.value })}
    placeholder="Search tasks..."
    className="pl-9 bg-surface-1 hover:bg-surface-2 border-border-subtle focus:border-border-focus"
  />
</div>
```

---

### 2.5 `[MOYENNE]` `onMouseEnter`/`onMouseLeave` inline dans TasksTopbar + TasksSidebar

**Fichiers :**
- [src/modules/tasks/components/layout/TasksTopbar.tsx:118-125](../src/modules/tasks/components/layout/TasksTopbar.tsx#L118)
- [src/modules/tasks/components/layout/TasksSidebar.tsx:81-86, 114-119](../src/modules/tasks/components/layout/TasksSidebar.tsx#L81)

**Problème :** re-render à chaque hover, JS thread blocked sur grandes listes. Violation directe de `feedback_card_pattern` ("jamais onMouseEnter/onMouseLeave inline style").

**Fix recommandé :** `hover:bg-surface-2` Tailwind. C'est mécanique.

---

### 2.6 `[MOYENNE]` TopBar avatar flash sur chaque nav

**Fichier :** [src/shared/components/layout/TopBar.tsx:230-245](../src/shared/components/layout/TopBar.tsx#L230)

**Problème :** `auth.getUser()` (network roundtrip ~150 ms) dans `useEffect` pour récupérer `name + avatar`. Tu avais documenté (`project_performance_serverauth`) que `getSession()` lit le localStorage en 0 ms. À chaque mount du layout, le nom et l'avatar arrivent avec 150 ms de délai → flash.

**Fix recommandé :**
```ts
useEffect(() => {
  const supabase = createClient();
  supabase.auth.getSession().then(({ data: { session } }) => {
    const user = session?.user;
    if (!user) return;
    // ...même parsing
  });
}, []);
```

---

### 2.7 `[MOYENNE]` `useCurrentUserId` initial state `null` → double pass

**Fichier :** [src/shared/hooks/useCurrentUserId.ts:5-9](../src/shared/hooks/useCurrentUserId.ts#L5)

**Problème :** render initial avec `userId = null`, puis `useEffect` async setUserId. KanbanBoard fait `useCurrentUserId()` → `useWorkspaces(userId || "")` → query disabled, skeleton, puis re-render avec userId, query active, skeleton encore. Double pass.

**Fix recommandé :** lire `localStorage` synchrone dans `useState` initializer :
```ts
function readUserIdFromStorage(): string | null {
  if (typeof window === "undefined") return null;
  // Supabase stocke sous sb-<projectRef>-auth-token
  const key = Object.keys(localStorage).find((k) => k.startsWith("sb-") && k.endsWith("-auth-token"));
  if (!key) return null;
  try {
    const raw = JSON.parse(localStorage.getItem(key) ?? "null");
    return raw?.user?.id ?? raw?.currentSession?.user?.id ?? null;
  } catch { return null; }
}

export function useCurrentUserId(): string | null {
  const [userId, setUserId] = useState<string | null>(readUserIdFromStorage);
  useEffect(() => {
    const supabase = createClient();
    const { data: { subscription } } = supabase.auth.onAuthStateChange((_, session) => {
      setUserId(session?.user?.id ?? null);
    });
    return () => subscription.unsubscribe();
  }, []);
  return userId;
}
```

---

### 2.8 `[MOYENNE]` `useRealtimeWorkspaces` channel sans filtre

**Fichier :** [src/modules/tasks/hooks/useRealtimeWorkspaces.ts:14-21](../src/modules/tasks/hooks/useRealtimeWorkspaces.ts#L14)

**Problème :** channel "workspaces-all" sans filter. RLS limite ce que tu reçois côté WAL, mais chaque event `INSERT/UPDATE/DELETE` global → `invalidateQueries({ queryKey: WORKSPACE_KEYS.all })`.

**Fix recommandé :** filter par `org_id` ou écouter uniquement les changements de `workspace_members` pour l'utilisateur (déjà fait pour ça précisément avec le second sub).

---

**Score axe 2 : 6/10** — La fondation realtime est bonne, mais les invalidations sont systématiquement trop larges. Linear a fait l'effort de patcher la cache à la ligne près ; HEGON invalide par projet/workspace. Sur 5 collab actifs, ça se voit.

---

## 3. Performance perçue

### 3.1 `[CRITIQUE]` Middleware double query Supabase sur chaque navigation

**Fichier :** [src/middleware.ts:138-156, 179-204](../src/middleware.ts#L138)

**Problème :** `auth.getUser()` (network) + `workspaces` query + parfois `org_invitations` query. 200-400 ms ajoutés à **chaque** navigation cross-route. Tu as documenté CSR-first précisément pour éviter ça (perf.md §1.2). Le middleware ré-introduit ce que tu as banni des modules.

⚠️ **Réalité (patch 2026-05-18) :** le `getUser()` reste **mandatory** pour la sécurité (validation JWT). Le cookie ne skip *que* la query `workspaces`. Gain réel attendu : ~30-50 ms par nav, **pas 200 ms**. **Mesure avant de commit définitivement.** Si après mesure le gain est <30 ms, drop le cookie et garde le middleware simple — la complexité ne vaut le coup que si le gain est tangible.

**Fix recommandé :** le check workspace est *uniquement* nécessaire au premier login post-auth. Mémoriser dans un cookie `hegon-onboarded=1` après la première résolution → middleware skip si présent.
```ts
// Après getUser
const onboardedCookie = request.cookies.get("hegon-onboarded")?.value === "1";
if (onboardedCookie && pathname !== "/onboarding") return response;

// Sinon faire le check workspace + set cookie
const { data: workspace } = await supabase.from("workspaces").select("id").limit(1).maybeSingle();
if (workspace) {
  response.cookies.set("hegon-onboarded", "1", {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "lax",
    maxAge: 60 * 60 * 24 * 7,  // 7 jours (pas 30 — limite la fenêtre stale)
  });
}
```

**Les 5 garde-fous obligatoires :**

1. **Clear dans `signOut` server action** — sinon prochain user sur la même machine hérite du cookie.
   ```ts
   // src/infrastructure/auth/actions.ts
   cookies().delete("hegon-onboarded");
   ```

2. **Clear dans `/onboarding/complete`** — si user revient sur onboarding (cas edge : "Settings → Recreate workspace"), reset l'état.

3. **Durée 7 jours** (pas 30) — limite la fenêtre où un cookie stale peut masquer un workspace réellement absent.

4. **Set après `acceptInvitation` réussie** — un invité doit avoir le fast path dès sa première nav vers `/pro/tasks`, pas attendre le prochain workspace check.
   ```ts
   // src/app/invite/[token]/page.tsx, après le success
   document.cookie = "hegon-onboarded=1; path=/; max-age=604800; SameSite=Lax";
   ```

5. **Clear sur dernier workspace supprimé** — sinon middleware skip + RLS renvoie zéro → écran d'erreur silencieux.
   ```ts
   // useDeleteWorkspace.onSuccess
   const remaining = queryClient.getQueryData<Workspace[]>(WORKSPACE_KEYS.all);
   if (!remaining || remaining.length === 0) {
     document.cookie = "hegon-onboarded=; path=/; max-age=0";
   }
   ```

---

### 3.2 `[ÉLEVÉE]` Cross-module side-effects dupliqués 4× dans useTasks

**Fichier :** [src/modules/tasks/hooks/useTasks.ts:77-92, 141-156, 170-187, 203-220](../src/modules/tasks/hooks/useTasks.ts#L77)

**Problème :** la logique "si task a un goal_id en mode auto, recalculer le progress + invalider goal cache" est copy-pasted dans `useUpdateTask`, `useMoveTask`, `useDeleteTask`, `useArchiveTask`. À chaque drag : `GoalService.getGoal()` (network), `GoalService.recalculateProgress()` (2 requêtes), 3 invalidations. Drag une task liée à un goal → **5 round trips** Supabase.

✅ **Décision actée (patch 2026-05-18) — RPC Postgres, pas trigger.**
> Après review : pivot du trigger initialement proposé vers une **RPC explicite**. Raison : "explicite > magique". L'RPC reste appelée depuis `service.ts` (blueprint §9 respecté, pas d'exception nécessaire), discoverable dans le code TS, testable end-to-end via integration test. Le trigger aurait été invisible — un nouveau dev touchant `tasks` dans 6 mois ne verrait aucune référence à `goals` dans le TS et serait perdu pour debug.
>
> **Approche en 2 phases :**
> 1. **Phase 1 — RPC unique partagée** : `recalc_goal_progress(p_goal_id uuid)` appelée explicitement depuis chaque mutation (`useUpdateTask/useMoveTask/useDeleteTask/useArchiveTask.onSuccess`). Chaque mutation passe de **4 round trips à 2** (mutation + recalc atomique). Simple, suffisant pour 95 % des cas.
> 2. **Phase 2 (optionnelle, si perf insuffisante)** : RPC dédiée `move_task_and_recalc(p_task_id, p_new_status_id, p_new_position) RETURNS json` qui fait tout en **1 round trip** pour le drag (cas le plus critique perf). Mesurer Phase 1 d'abord, optimiser ensuite.

**Fix recommandé :** RPC `recalc_goal_progress(p_goal_id uuid)` (Phase 1) — voir Annexe §A.5 pour la spec complète SQL + appel TS. Snapshot des deux phases ci-dessous :
```sql
CREATE OR REPLACE FUNCTION recalc_goal_progress()
RETURNS TRIGGER AS $$
DECLARE
  goal_id_val UUID;
  total INT;
  completed INT;
BEGIN
  goal_id_val := COALESCE(NEW.goal_id, OLD.goal_id);
  IF goal_id_val IS NULL THEN RETURN NEW; END IF;

  -- Skip si goal est en manual
  IF (SELECT progress_mode FROM goals WHERE id = goal_id_val) != 'auto' THEN
    RETURN NEW;
  END IF;

  SELECT COUNT(*), COUNT(*) FILTER (WHERE completed_at IS NOT NULL)
  INTO total, completed
  FROM tasks WHERE goal_id = goal_id_val;

  IF total > 0 THEN
    UPDATE goals SET progress = ROUND((completed::numeric / total) * 100)
    WHERE id = goal_id_val;
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER tasks_recalc_goal_progress
  AFTER INSERT OR UPDATE OR DELETE ON tasks
  FOR EACH ROW EXECUTE FUNCTION recalc_goal_progress();
```
Puis côté client, juste `useRealtimeGoals` propagera le nouveau `progress` (déjà branché).

---

### 3.3 `[ÉLEVÉE]` `getCurrentOrgId` re-query à chaque appel service

**Fichier :** [src/shared/utils/getOrgId.ts](../src/shared/utils/getOrgId.ts)

**Problème :** Zustand cache l'orgId, mais au cold start l'orgStore est vide. `getCurrentOrgId()` est appelé dans **chaque** fonction service (`createGoal`, `updateGoal`, `getGoals`, `linkTaskToGoal`...). Sur le mount initial de `/life/goals` : 4-6 appels services en parallèle → 4-6 fois la même query `memberships`. C'est ce que React `cache()` résout côté serveur — côté client, rien.

**Fix recommandé :** singleton/memo de la première promesse :
```ts
let cachedOrgId: Promise<string> | null = null;

export async function getCurrentOrgId(): Promise<string> {
  const { orgId } = useOrgStore.getState();
  if (orgId) return orgId;

  if (!cachedOrgId) {
    cachedOrgId = (async () => {
      const supabase = createClient();
      const { data, error } = await supabase
        .from("memberships")
        .select("org_id")
        .order("created_at", { ascending: true })
        .limit(1)
        .single();
      if (error || !data) {
        cachedOrgId = null;  // permet retry
        throw new Error("No organization found");
      }
      return data.org_id;
    })();
  }
  return cachedOrgId;
}

// Reset au logout
export function resetOrgIdCache() { cachedOrgId = null; }
```

---

### 3.4 `[ÉLEVÉE]` `dashboard.getTodayTasks` ne filtre pas `due_date` ni `is_completed` côté serveur

**Fichier :** [src/modules/dashboard/service.ts:88-98](../src/modules/dashboard/service.ts#L88)

**Problème :** fetch jusqu'à 50 tasks **toutes statuses confondus**, filtre `is_completed` côté JS. Si tu as 200 tasks actives, tu en perds 150. Pagination absente.

**Fix recommandé :** inner join sur statuses + filter direct :
```ts
const { data } = await supabase
  .from("tasks")
  .select(`
    id, title, priority, due_date,
    status:statuses!inner(name, color, is_completed),
    project:projects(name)
  `)
  .or(`created_by.eq.${userId},assignee_id.eq.${userId}`)
  .eq("is_archived", false)
  .eq("statuses.is_completed", false)
  .order("priority", { ascending: true })
  .order("due_date", { ascending: true, nullsFirst: false })
  .limit(50);
```

---

### 3.5 `[ÉLEVÉE]` `getMediaItems` sans `.limit()` par défaut

**Fichier :** [src/modules/watching/service.ts:21-53](../src/modules/watching/service.ts#L21)

**Problème :** `options.limit` est optionnel. La Library complète d'un user avec 500 films fait un fetch full. Violation explicite de `performance.md §3.3`.

**Fix recommandé :** limit par défaut à 100 dans la signature de la fonction, ajouter pagination cursor pour la Library complète.

---

### 3.6 `[MOYENNE]` `useTasks` staleTime 3 min, convention 5 min

**Fichier :** [src/modules/tasks/hooks/useTasks.ts:21](../src/modules/tasks/hooks/useTasks.ts#L21)

**Problème :** `staleTime: 1000 * 60 * 3`. Avec real-time actif, `staleTime` court n'a pas de sens (le WS invalide déjà). Refetch silencieux en arrière-plan toutes les 3 min pour rien.

⚠️ **Nuance (patch 2026-05-18) :** ne **pas** mettre `Infinity`. Si le WS se déconnecte (réseau, channel timeout, Vercel hiccup), tu perds le filet de sécurité — la cache reste figée éternellement. À 10 min, le `refetchOnReconnect: true` du QueryProvider rattrape le décrochage.

**Fix recommandé :**
```ts
return useQuery({
  queryKey: TASK_KEYS.byProject(projectId || ""),
  queryFn: () => TaskService.getTasks(projectId!),
  enabled: !!projectId,
  // Real-time est le mécanisme principal de fraîcheur ; staleTime sert de filet
  // si la sub WS décroche (perf.md §2.2 — convention 10min pour data structurelle).
  staleTime: 10 * 60 * 1000,
});
```

---

### 3.7 `[MOYENNE]` Pas de bundle analyzer + Framer Motion eagerly loaded

**Problème :** Pas de `@next/bundle-analyzer` configuré (à vérifier dans `next.config.ts`). Framer Motion est importé même dans TopBar pour une animation de profile menu. dnd-kit, recharts, lucide-react, radix-ui — tout en client eagerly.

**Fix recommandé :** activer `@next/bundle-analyzer` :
```bash
npm install -D @next/bundle-analyzer
```
Et dynamic import pour les motion non-critiques au first paint :
```tsx
const ProfileMenu = dynamic(() => import("./ProfileMenu"), { ssr: false });
```

---

**Score axe 3 : 6.5/10** — Le pattern CSR est bon, les staleTime corrects, mais les fuites s'accumulent (middleware, cross-module mutations, orgId non-mémoizé). Au-delà de 3-5 collab actifs, la latence perçue va se dégrader.

---

## 4. Qualité du code

### 4.1 `[ÉLEVÉE]` `any` global eslint-disable sur les services

**Fichiers :**
- [src/modules/tasks/service.ts:1](../src/modules/tasks/service.ts#L1)
- [src/modules/tasks/types.ts:1](../src/modules/tasks/types.ts#L1)
- [src/modules/dashboard/service.ts:1](../src/modules/dashboard/service.ts#L1)

**Problème :** `/* eslint-disable @typescript-eslint/no-explicit-any */` au top du fichier. Cache des `any` non justifiés. `anti-patterns.md` interdit explicitement `any sans commentaire de justification`. Le disable file-wide est la version paresseuse — il cache des bugs.

**Fix recommandé :** retirer les disable file-wide, inline `// eslint-disable-next-line` UNIQUEMENT sur les lignes Supabase join inference + commentaire :
```ts
// eslint-disable-next-line @typescript-eslint/no-explicit-any -- Supabase nested join inference fails
const tasks = (data || []).map((task: any) => ({ ... }));
```

---

### 4.2 `[ÉLEVÉE]` Middleware contient 76 lignes de code mort commenté

**Fichier :** [src/middleware.ts:1-82](../src/middleware.ts#L1)

**Problème :** anciennement le middleware était différent, conservé en commentaire. `anti-patterns.md` interdit `TODO sans ticket` — un blob commenté c'est pire. Le diff de prochain commit va polluer la review.

**Fix recommandé :** delete. Git garde l'historique.

---

### 4.3 `[ÉLEVÉE]` `getOrgMembers` exposé renvoie TOUTE la table profiles

**Fichier :** [src/modules/tasks/service.ts:214-223](../src/modules/tasks/service.ts#L214)

**Problème :** `select(...).from("profiles").order(...)` sans filtre. Le mémoire `project_tasks_roadmap` dit "RLS Profiles are viewable by everyone". Donc cette fonction renvoie *tous les users* HEGON, pas juste l'org. `useOrgMembers()` existe encore et est exporté.

**Fix recommandé :**
1. Supprimer `getOrgMembers` (deprecated par `getProjectAssignees`).
2. Restreindre la RLS `profiles` (voir §5.1).
3. Si `useOrgMembers` est encore utilisé quelque part (à grep), migrer vers `useProjectAssignees(projectId)`.

---

### 4.4 `[MOYENNE]` Cross-module mutations 4× dupliquées

Cf. §3.2. Solution : RPC Postgres OU helper extrait :
```ts
async function recalcGoalIfAuto(queryClient: QueryClient, goalId: string) {
  let goalMode = queryClient.getQueryData<{ progress_mode: string }>(GOAL_KEYS.detail(goalId))?.progress_mode;
  if (!goalMode) {
    try { goalMode = (await GoalService.getGoal(goalId)).progress_mode; } catch { return; }
  }
  if (goalMode !== "auto") return;
  await GoalService.recalculateProgress(goalId);
  queryClient.invalidateQueries({ queryKey: GOAL_KEYS.detail(goalId) });
  queryClient.invalidateQueries({ queryKey: GOAL_KEYS.lists() });
  queryClient.invalidateQueries({ queryKey: LINKED_TASK_KEYS.byGoal(goalId) });
}
```

---

### 4.5 `[MOYENNE]` `TaskCard` mélange Tailwind tokens + inline `style`

**Fichier :** [src/modules/tasks/components/kanban/TaskCard.tsx:55-58, 62-71](../src/modules/tasks/components/kanban/TaskCard.tsx#L55)

**Problème :** `bg-surface-1 hover:bg-surface-2 border-border-subtle` (Tailwind, OK) puis `style={{ backgroundColor: var(--color-surface-2) }}` re-met du inline. Incohérent.

**Fix recommandé :** tout passer en Tailwind classes. Pas d'inline `style={{}}` sauf pour les couleurs dynamiques (tag.color).

---

### 4.6 `[MOYENNE]` `WorkspaceInvitePanel` bypasse `useRealtimeSync`

**Fichier :** [src/modules/tasks/components/panels/WorkspaceInvitePanel.tsx:73-92](../src/modules/tasks/components/panels/WorkspaceInvitePanel.tsx#L73)

**Problème :** raw `supabase.channel()` + 2 `.on(postgres_changes)` au lieu de 2 `useRealtimeSync()`. L'abstraction `useRealtimeSync` existe précisément pour ça. C'est un précédent dangereux.

**Fix recommandé :** 2 appels `useRealtimeSync` consécutifs. L'avantage de partager 1 channel pour 2 tables est marginal vs la dette de pattern divergent.

---

### 4.7 `[MOYENNE]` `useTasks.ts` types `any` dans setQueryData

**Fichier :** [src/modules/tasks/hooks/useTasks.ts:61, 120-127](../src/modules/tasks/hooks/useTasks.ts#L61)

**Problème :** `(old: any) => ...` pour des cache entries `Task[]`. Types existent, utilise-les.

**Fix recommandé :** `queryClient.setQueryData<Task[]>(key, (old) => ...)`.

---

### 4.8 `[MOYENNE]` `dashboard.service` `Promise.all` sans tolérance d'erreur

**Fichier :** [src/modules/dashboard/service.ts:407-470](../src/modules/dashboard/service.ts#L407) (`getDashboardData`)

**Problème :** `Promise.all` interne — si `getTodayFootballEvents` throw, les 6 autres résultats sont perdus. Le dashboard entier crash pour un seul service défaillant.

**Fix recommandé :** `Promise.allSettled` + dashboard partiel + log Sentry des sections cassées :
```ts
const results = await Promise.allSettled([
  getTodayTasks(userId),
  getInProgressMedia(userId),
  // ...
]);
const tasks = results[0].status === "fulfilled" ? results[0].value : [];
// ... etc
```

---

**Score axe 4 : 6/10** — Beaucoup de patterns sont là, mais l'écart entre les règles HQ et l'application est visible. Les `eslint-disable` file-wide sont les marqueurs : c'est du code "qui marche" pas du code "propre".

---

## 5. Sécurité & données

### 5.1 `[CRITIQUE]` Table `profiles` lisible par tous les users authentifiés

**Référence :** memory `project_tasks_roadmap` confirme `Profiles are viewable by everyone`.

**Problème :** N'importe quel user authentifié peut lister tous les emails + noms + avatars de la base. Privacy GDPR-risque. En audience LinkedIn, c'est un point d'arrêt.

**Fix recommandé :** RLS policy stricte :
```sql
DROP POLICY "Profiles are viewable by everyone" ON profiles;

CREATE POLICY "profiles_visible_to_org_mates" ON profiles
  FOR SELECT USING (
    id IN (
      SELECT DISTINCT m1.user_id
      FROM memberships m1
      JOIN memberships m2 ON m1.org_id = m2.org_id
      WHERE m2.user_id = auth.uid()
    )
    OR id IN (
      SELECT DISTINCT user_id
      FROM workspace_members wm
      WHERE wm.workspace_id IN (SELECT * FROM my_workspace_ids())
    )
    OR id = auth.uid()
  );
```

---

### 5.2 `[ÉLEVÉE]` Middleware = source de vérité unique

**Fichier :** [src/middleware.ts:179-211](../src/middleware.ts#L179)

**Problème :** le check workspace n'a aucun `.eq("user_id", user.id)`. Compte 100% sur RLS pour filtrer. Defense-in-depth violée (`security.md §0`). Si demain quelqu'un change la RLS workspaces en `USING (true)` pour debug, le middleware passe tout le monde sur `/dashboard`.

**Fix recommandé :** `.eq("user_id", user.id)` OR équivalent shared workspace check explicite. La RLS reste, mais le middleware vérifie aussi :
```ts
const { data: workspace } = await supabase
  .from("workspaces")
  .select("id")
  .or(`user_id.eq.${user.id},id.in.(${myWorkspaceIds.join(",")})`)
  .limit(1)
  .maybeSingle();
```

---

### 5.3 `[ÉLEVÉE]` `createWorkspaceInvitation` pas owner-only au niveau service

**Fichier :** [src/modules/tasks/service.ts:394-421](../src/modules/tasks/service.ts#L394)

**Problème :** récupère le `workspace.org_id` puis INSERT. Aucun check `workspace.user_id === auth.uid()`. Si la RLS INSERT sur `org_invitations` autorise tous les membres d'org → tout `member` peut inviter.

**Fix recommandé :**
```ts
export async function createWorkspaceInvitation(workspaceId: string, email: string) {
  const supabase = createClient();
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) throw new Error("Not authenticated");

  const { data: workspace } = await supabase
    .from("workspaces")
    .select("org_id, user_id")
    .eq("id", workspaceId)
    .single();

  if (!workspace) throw new Error("Workspace not found");
  if (workspace.user_id !== user.id) throw new Error("Only the workspace owner can invite");

  // ... INSERT
}
```
Et RLS côté DB :
```sql
CREATE POLICY "org_invitations_insert_owner_only" ON org_invitations
  FOR INSERT WITH CHECK (
    workspace_id IS NULL  -- org invite (legacy)
    OR workspace_id IN (SELECT * FROM my_owned_workspace_ids())
  );
```

---

### 5.4 `[ÉLEVÉE]` Pas de rate limit sur invitations

**Problème :** [src/modules/tasks/service.ts:407-417](../src/modules/tasks/service.ts#L407) — direct INSERT, aucune protection. Un user peut générer 10 000 invitations en boucle.

**Fix recommandé :** route API `/api/invitations/create` proxy avec Upstash ratelimiter (10/min/user, 50/jour/user) :
```ts
// src/shared/lib/ratelimit.ts
export const invitationRatelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, "1 m"),
  prefix: "ratelimit:invitations",
});

// src/app/api/invitations/route.ts
export async function POST(request: Request) {
  const supabase = await createServerClient();
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) return NextResponse.json({ error: "Unauthorized" }, { status: 401 });

  const { success } = await invitationRatelimit.limit(user.id);
  if (!success) return NextResponse.json({ error: "Too many requests" }, { status: 429 });

  // ... rest
}
```

---

### 5.5 `[MOYENNE]` Erreurs Postgres exposées dans les toasts

**Fichier :** [src/modules/tasks/hooks/useTasks.ts:38, 75, 188](../src/modules/tasks/hooks/useTasks.ts#L38)

**Problème :** `toast.error("Failed to create task: ${error.message}")`. Les `error.message` Supabase peuvent contenir des noms de tables, colonnes, contraintes → info leak.

**Fix recommandé :** message générique + log détail :
```ts
onError: (error: Error) => {
  console.error("[useCreateTask]", error);  // Sentry pickup
  toast.error("Couldn't create task. Try again.");
}
```

---

### 5.6 `[MOYENNE]` `acceptInvitation` côté client direct RPC sans validation token

**Fichier :** [src/modules/orgs/service.ts:99-106](../src/modules/orgs/service.ts#L99)

**Problème :** `supabase.rpc("accept_invitation", { p_token: token })`. La RPC fait email mismatch check (validé par mémoire). Mais aucune validation côté client : `token` n'est pas sanitized.

**Fix recommandé :**
```ts
const TOKEN_REGEX = /^[A-Za-z0-9_-]{20,128}$/;
export async function acceptInvitation(token: string) {
  if (!TOKEN_REGEX.test(token)) throw new Error("Invalid token format");
  // ... rest
}
```

---

### 5.7 `[MOYENNE]` `next.config.ts` `hostname: "**"` + pas de CSP

Déjà documenté dans `security.md §7.2` comme TODO. À fixer **avant la vidéo LinkedIn** sinon attacker peut proxifier n'importe quelle URL via next/image.

**Fix recommandé :** liste explicite + CSP via spec déjà écrite (security.md §4.2/4.3).

---

**Score axe 5 : 6/10** — Le multi-tenant est solide (RLS, my_orgs, trigger org_id). Mais la table `profiles` ouverte, le middleware sans defense-in-depth, et le rate-limit manquant sur les invitations baissent le score. Linear/Vercel ont une privacy doctrine plus stricte.

---

## 6. UX & cohérence

### 6.1 `[ÉLEVÉE]` Patterns design system non appliqués sur Tasks

**Fichiers :**
- `TasksTopbar` : native input + inline `style` (§2.4, §2.5)
- `TasksSidebar` : `onMouseEnter` inline hover (§2.5)
- `TaskCard` : mix Tailwind classes / `style={{ var(...) }}` (§4.5)

**Problème :** Goals, Habits, Books, Journal ont été retrofités (Phase 2 design pass). Tasks attend toujours. Résultat : `/pro/tasks` se sent comme un autre produit que `/life/goals`.

**Fix recommandé :** design pass complet sur Tasks (déjà dans le plan d'action `project_design_system_pass`). Standard = GoalCard/HabitsPage. Checklist :
- [ ] TasksTopbar — remplacer native input par `Input variant="tasks"`
- [ ] TasksTopbar — virer tous les `onMouseEnter`/`onMouseLeave`, passer en classes Tailwind
- [ ] TasksSidebar — idem
- [ ] TaskCard — tout en Tailwind, retirer `style={{ var(...) }}`
- [ ] KanbanColumn — vérifier mêmes patterns
- [ ] ListView, CalendarView — audit complet

---

### 6.2 `[ÉLEVÉE]` `useUpdateTask` toast "Task updated" pour des autosaves silencieux

**Fichier :** [src/modules/tasks/hooks/useTasks.ts:79](../src/modules/tasks/hooks/useTasks.ts#L79)

**Problème :** chaque autosave de `EditTaskModal` triggers `toast.success("Task updated")`. Si tu changes 5 champs avant de fermer le modal → 5 toasts. Spam. Linear ne fait jamais ça.

**Fix recommandé :** retirer le toast sur `useUpdateTask.onSuccess`. Garder uniquement create + delete + archive.

---

### 6.3 `[ÉLEVÉE]` Pas d'état d'erreur sur les listes

**Fichier :** [src/modules/tasks/components/kanban/KanbanBoard.tsx:153-165](../src/modules/tasks/components/kanban/KanbanBoard.tsx#L153)

**Problème :** pas de check `isError`. Si Supabase répond 500, on voit l'empty state "No tasks" → l'utilisateur pense que ses tâches ont été supprimées. UX traumatique.

**Fix recommandé :** ajouter un composant `<ListError onRetry={...} />` partout où il y a un skeleton state. Pattern unifié à ajouter à `hq/design-system.md §4.10` (à côté d'Empty states).

---

### 6.4 `[MOYENNE]` `WorkspaceInvitePanel` largeur `w-110` non-spec

**Fichier :** [src/modules/tasks/components/panels/WorkspaceInvitePanel.tsx](../src/modules/tasks/components/panels/WorkspaceInvitePanel.tsx)

**Problème :** `w-110` (440px). `navigation.md §4` dit Sliding Panel = 75-80% viewport, `§6` dit Slide-Over = 480px. Le panel actuel est ni l'un ni l'autre. Nomenclature ambiguë + dimension non-conforme.

**Fix recommandé :** clarifier dans `navigation.md` ce qu'est ce panel (probablement un slide-over selon §6) et utiliser `w-120` (480px).

---

### 6.5 `[MOYENNE]` TaskCard drag handle invisible au touch

**Fichier :** [src/modules/tasks/components/kanban/TaskCard.tsx:60-74](../src/modules/tasks/components/kanban/TaskCard.tsx#L60)

**Problème :** `opacity-0 group-hover:opacity-100` → invisible sur tablette/touch. Le `TouchSensor` est dans `KanbanBoard` mais sans handle visible, l'utilisateur ne sait pas comment drag.

**Fix recommandé :** garder visible sur touch :
```tsx
className="... opacity-0 transition-opacity group-hover:opacity-100 [@media(hover:none)]:opacity-50"
```

---

### 6.6 `[MOYENNE]` `TaskCard` due_date sans année à 6 mois

**Fichier :** [src/modules/tasks/components/kanban/TaskCard.tsx:184](../src/modules/tasks/components/kanban/TaskCard.tsx#L184)

**Problème :** `format(dueDate, "MMM d")`. "Dec 12" sans année. Pour une planning long terme, ambigu.

**Fix recommandé :**
```tsx
const isThisYear = dueDate.getFullYear() === new Date().getFullYear();
const dateStr = format(dueDate, isThisYear ? "MMM d" : "MMM d, yyyy");
```

---

### 6.7 `[MOYENNE]` Empty/loading state du dashboard non géré pour zéro orgs

**Problème :** Si le user n'a aucune membership active (cas edge mais possible : org deleted) → `getCurrentOrgId` throw, dashboard plante.

**Fix recommandé :** error boundary autour du dashboard + fallback gracieux + redirect onboarding.

---

### 6.8 `[FAIBLE]` z-index `z-9999` non standard

**Fichier :** [src/shared/components/layout/TopBar.tsx:117](../src/shared/components/layout/TopBar.tsx#L117)

**Problème :** `z-9999` arbitraire. `design-system.md §1.8` scale max = `z-toast: 50`.

**Fix recommandé :** `z-dropdown` (20) suffit pour un profile menu.

---

**Score axe 6 : 6/10** — Tasks fait baisser le score (pas encore retrofité), les états d'erreur manquent partout, les autosaves sont bavards. Habits/Goals/Books/Journal sont plus proches d'un 8.

---

## Annexe — Patterns de fix prêts à coller

### A.1 — Helper `resolveJoins` pour optimistic updates

```ts
// src/modules/tasks/hooks/useTasks.ts
import type { Task, Status, Assignee, Tag } from "../types";

function resolveTaskJoins(
  queryClient: QueryClient,
  projectId: string,
  partial: Partial<Task>,
): { status?: Status; assignee?: Assignee | null; tags?: Tag[] } {
  const statuses = queryClient.getQueryData<Status[]>(STATUS_KEYS.byProject(projectId));
  const members  = queryClient.getQueryData<Assignee[]>(PROJECT_ASSIGNEE_KEYS.byProject(projectId));
  return {
    status: partial.status_id ? statuses?.find((s) => s.id === partial.status_id) : undefined,
    assignee: partial.assignee_id !== undefined
      ? (members?.find((m) => m.id === partial.assignee_id) ?? null)
      : undefined,
  };
}
```

### A.2 — Helper `recalcGoalIfAuto`

```ts
// src/modules/goals/hooks/recalc.ts
import type { QueryClient } from "@tanstack/react-query";
import * as GoalService from "@/modules/goals/service";
import { GOAL_KEYS, LINKED_TASK_KEYS } from "./query-keys";

export async function recalcGoalIfAuto(queryClient: QueryClient, goalId: string) {
  let goalMode = queryClient.getQueryData<{ progress_mode: string }>(GOAL_KEYS.detail(goalId))?.progress_mode;
  if (!goalMode) {
    try { goalMode = (await GoalService.getGoal(goalId)).progress_mode; } catch { return; }
  }
  if (goalMode !== "auto") return;
  await GoalService.recalculateProgress(goalId);
  queryClient.invalidateQueries({ queryKey: GOAL_KEYS.detail(goalId) });
  queryClient.invalidateQueries({ queryKey: GOAL_KEYS.lists() });
  queryClient.invalidateQueries({ queryKey: LINKED_TASK_KEYS.byGoal(goalId) });
}
```

### A.3 — Migration RLS `profiles`

```sql
-- 20260518_restrict_profiles_visibility.sql
DROP POLICY IF EXISTS "Profiles are viewable by everyone" ON profiles;

CREATE POLICY "profiles_visible_to_org_mates" ON profiles
  FOR SELECT USING (
    -- Self
    id = auth.uid()
    -- Org-mate (membre de la même org)
    OR id IN (
      SELECT DISTINCT m1.user_id
      FROM memberships m1
      JOIN memberships m2 ON m1.org_id = m2.org_id
      WHERE m2.user_id = auth.uid()
    )
    -- Workspace-mate (membre du même workspace partagé)
    OR id IN (
      SELECT DISTINCT wm.user_id
      FROM workspace_members wm
      WHERE wm.workspace_id IN (SELECT * FROM my_workspace_ids())
    )
  );

CREATE POLICY "profiles_update_self_only" ON profiles
  FOR UPDATE USING (id = auth.uid()) WITH CHECK (id = auth.uid());
```

### A.4 — Migration trigger `position` atomique

```sql
-- 20260518_tasks_position_trigger.sql
CREATE OR REPLACE FUNCTION set_task_position()
RETURNS TRIGGER AS $$
BEGIN
  IF NEW.position IS NULL THEN
    SELECT COALESCE(MAX(position), 0) + 1
    INTO NEW.position
    FROM tasks
    WHERE project_id = NEW.project_id AND status_id = NEW.status_id;
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

DROP TRIGGER IF EXISTS tasks_set_position ON tasks;
CREATE TRIGGER tasks_set_position
  BEFORE INSERT ON tasks
  FOR EACH ROW EXECUTE FUNCTION set_task_position();
```

### A.5 — RPC goal progress recalc (Phase 1)

**Migration SQL :**

```sql
-- 20260518_recalc_goal_progress_rpc.sql
CREATE OR REPLACE FUNCTION recalc_goal_progress(p_goal_id UUID)
RETURNS INT
LANGUAGE plpgsql
SECURITY INVOKER  -- respecte RLS de l'appelant
AS $$
DECLARE
  total      INT;
  completed  INT;
  new_progress INT;
  goal_mode  TEXT;
BEGIN
  -- Bail out si goal pas en mode auto
  SELECT progress_mode INTO goal_mode FROM goals WHERE id = p_goal_id;
  IF goal_mode IS NULL OR goal_mode != 'auto' THEN
    RETURN NULL;
  END IF;

  SELECT
    COUNT(*),
    COUNT(*) FILTER (WHERE completed_at IS NOT NULL)
  INTO total, completed
  FROM tasks
  WHERE goal_id = p_goal_id AND is_archived = false;

  IF total = 0 THEN
    new_progress := 0;
  ELSE
    new_progress := ROUND((completed::numeric / total) * 100);
  END IF;

  UPDATE goals SET progress = new_progress WHERE id = p_goal_id;
  RETURN new_progress;
END;
$$;

-- Permissions
GRANT EXECUTE ON FUNCTION recalc_goal_progress(UUID) TO authenticated;
```

**Appel côté service :**

```ts
// src/modules/goals/service.ts
export async function recalculateProgress(goalId: string): Promise<number | null> {
  const supabase = createClient();
  const { data, error } = await supabase.rpc("recalc_goal_progress", { p_goal_id: goalId });
  if (error) throw error;
  return data;  // null si goal pas en mode auto, sinon le nouveau progress 0-100
}
```

**Appel côté hook (helper extrait — cf. §A.2) :**

```ts
// src/modules/goals/hooks/recalc.ts
export async function recalcGoalIfAuto(queryClient: QueryClient, goalId: string) {
  const newProgress = await GoalService.recalculateProgress(goalId);  // 1 round trip, gère bail-out auto
  if (newProgress === null) return;  // pas en mode auto, rien à invalider

  queryClient.invalidateQueries({ queryKey: GOAL_KEYS.detail(goalId) });
  queryClient.invalidateQueries({ queryKey: GOAL_KEYS.lists() });
  queryClient.invalidateQueries({ queryKey: LINKED_TASK_KEYS.byGoal(goalId) });
}
```

**Gains mesurés attendus :**
- Avant : drag d'une task liée → `moveTask` + `getGoal` + `recalculateProgress fetch` + `recalculateProgress update` = **4 round trips**
- Après Phase 1 : `moveTask` + `recalc_goal_progress` RPC = **2 round trips** (-50%)
- Bail-out instantané si le goal n'est pas en mode auto (1 SELECT inline dans la function, pas de fetch tasks).

### A.5b — RPC fusionnée pour le drag (Phase 2, si nécessaire)

À implémenter **uniquement** si après mesure Phase 1 le drag reste perçu lent en multi-user :

```sql
-- 20260518_move_task_and_recalc_rpc.sql
CREATE OR REPLACE FUNCTION move_task_and_recalc(
  p_task_id        UUID,
  p_new_status_id  UUID,
  p_new_position   NUMERIC
)
RETURNS JSON
LANGUAGE plpgsql
SECURITY INVOKER
AS $$
DECLARE
  v_goal_id   UUID;
  v_completed BOOL;
  v_progress  INT;
BEGIN
  -- Move
  SELECT is_completed INTO v_completed FROM statuses WHERE id = p_new_status_id;

  UPDATE tasks SET
    status_id    = p_new_status_id,
    position     = p_new_position,
    completed_at = CASE WHEN v_completed THEN NOW() ELSE NULL END,
    updated_at   = NOW()
  WHERE id = p_task_id
  RETURNING goal_id INTO v_goal_id;

  -- Recalc inline si goal lié
  IF v_goal_id IS NOT NULL THEN
    v_progress := recalc_goal_progress(v_goal_id);
  END IF;

  RETURN json_build_object('goal_id', v_goal_id, 'new_progress', v_progress);
END;
$$;

GRANT EXECUTE ON FUNCTION move_task_and_recalc(UUID, UUID, NUMERIC) TO authenticated;
```

Gain : **1 round trip** pour le drag (au lieu de 2 en Phase 1). N'implémenter que si Sentry montre que le drag médiane > 200 ms en prod.

### A.6 — Cookie pattern pour middleware fast-path

```ts
// src/middleware.ts (extrait)
// Après getUser :
const onboardedCookie = request.cookies.get("hegon-onboarded")?.value === "1";

if (onboardedCookie && pathname !== "/onboarding") {
  return response;  // skip workspace check
}

// Sinon check workspace + set cookie
const { data: workspace } = await supabase.from("workspaces").select("id").limit(1).maybeSingle();
if (workspace) {
  response.cookies.set("hegon-onboarded", "1", {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "lax",
    maxAge: 60 * 60 * 24 * 30,  // 30 jours
  });
  if (pathname === "/onboarding") {
    return NextResponse.redirect(new URL("/dashboard", request.url));
  }
}
```
Et clear le cookie au logout dans `signOut` action.

---

## Liens internes

- Memory : `project_tasks_phase1_done`, `project_phase3_plan`, `project_card_pattern`, `project_modal_pattern`, `project_performance_serverauth`
- HQ : [hq/anti-patterns.md](anti-patterns.md), [hq/performance.md](performance.md), [hq/security.md](security.md), [hq/design-system.md](design-system.md), [hq/navigation.md](navigation.md)

---

*Audit produit par `/digger` le 2026-05-18.*
*À ré-auditer après application du Top 10.*