# HEGON Performance & Scalability

> Source de vérité pour toutes les décisions de performance, architecture de données, et scalabilité.
> Standard : Vercel / Linear / Notion — apps qui se sentent instantanées.
> Principe fondateur : **perceived speed > actual speed**.

---

## Table des matières

### 0. Philosophie
- Le contrat utilisateur
- Pourquoi perceived speed prime
- Standard de référence

### 1. Architecture de données — la règle unique
- 1.1 Le pattern standard : Client-side TanStack Query
- 1.2 Pourquoi pas SSR pour une app authentifiée
- 1.3 L'exception : pattern Hydration (Phase 3+)
- 1.4 Streaming — Suspense + loading.tsx
- 1.5 Checklist nouveau module

### 2. TanStack Query — conventions
- 2.1 QueryProvider global (settings non-négociables)
- 2.2 staleTime par type de donnée
- 2.3 gcTime
- 2.4 refetchOnWindowFocus policy
- 2.5 Invalidation après mutation
- 2.6 Règle du QueryClient unique

### 3. Service layer & Supabase
- 3.1 Promise.all obligatoire pour les fetches parallèles
- 3.2 Select uniquement les colonnes nécessaires
- 3.3 Pagination — jamais fetcher toutes les lignes
- 3.4 cache() React — usage et limites
- 3.5 Connection pooling — Supabase + Vercel serverless

### 4. Base de données — indexes
- 4.1 Règle des indexes obligatoires
- 4.2 Colonnes qui nécessitent un index
- 4.3 Convention de nommage
- 4.4 Index dans la migration, pas après

### 5. Bundle & Assets
- 5.1 next/image obligatoire — jamais de balise img native
- 5.2 Named imports uniquement
- 5.3 Server Components par défaut — "use client" seulement si nécessaire
- 5.4 Librairies lourdes — règles d'import
- 5.5 next/font — fonts optimisées, zéro CLS
- 5.6 Prefetch — quand l'activer, quand le couper

### 6. Temps réel — Phase 3
- 6.1 Pourquoi le cache TQ ne suffit pas seul (the cron problem)
- 6.2 Pattern useRealtimeSync
- 6.3 Quelles données nécessitent du real-time
- 6.4 Ce que Phase 3 résout définitivement

### 7. Monitoring
- 7.1 Sentry Performance
- 7.2 Vercel Analytics — Core Web Vitals
- 7.3 Supabase Query Performance dashboard
- 7.4 Ce qu'on surveille, ce qu'on ignore

### 8. Scalabilité — règles pour chaque nouveau module
- 8.1 Checklist avant de coder
- 8.2 Ce qui scale automatiquement
- 8.3 Ce qui ne scale pas et comment le gérer
- 8.4 Landing page — projet séparé, règles SSG

### 9. Roadmap performance
- 9.1 État actuel post-migration CSR
- 9.2 Phase 3 : Real-time → cron problem résolu
- 9.3 Phase 3+ : Hydration → premier chargement résolu
- 9.4 Ce qui reste hors scope

### 10. FORBIDDEN
- Ce qui est interdit sans exception

---

## 0. Philosophie

### Le contrat utilisateur
Un utilisateur de HEGON ouvre l'app plusieurs fois par jour. La navigation entre modules doit se sentir instantanée après la première visite. Le premier chargement peut avoir un skeleton — l'utilisateur l'accepte une fois. Il ne l'accepte pas à chaque retour.

### Pourquoi perceived speed prime
2.5s réelles avec données déjà présentes = meilleure expérience que 1.5s réelles avec skeleton. La rapidité perçue est ce que l'utilisateur ressent, pas ce que Lighthouse mesure.

### Standard de référence
Vercel Dashboard, Linear, Notion. La question à poser : "Est-ce que ça se sent aussi rapide que Linear ?"

---

## 1. Architecture de données — la règle unique

### 1.1 Le pattern standard : Client-side TanStack Query
**Tout HEGON utilise TanStack Query côté client.** Sans exception pour les modules authentifiés.

```
Page monte → TQ vérifie le cache → cache frais : données immédiates
                                 → cache périmé : données immédiates + refetch silencieux
                                 → pas de cache : skeleton → fetch → données
```

### 1.2 Pourquoi pas SSR pour une app authentifiée
HEGON est une app 100% authentifiée. Zéro besoin de SEO sur les pages internes. Le SSR sur une app auth apporte :
- Des données fraîches au premier chargement (avantage mineur)
- Un refetch complet à chaque navigation (inconvénient majeur)
- Une incohérence avec les modules client-side (inconvénient architectural)

### 1.3 L'exception : pattern Hydration (Phase 3+)
SSR pour le rendu initial + TQ hydraté avec les données serveur = données immédiates au premier chargement + cache TQ pour les visites suivantes. À implémenter en Phase 3 sur Dashboard et Sports.

### 1.4 Streaming — Suspense + loading.tsx
Next.js App Router permet d'afficher le shell de la page immédiatement pendant que les sections lourdes chargent. Chaque route doit avoir son `loading.tsx` et les sections indépendantes doivent être wrappées dans `<Suspense>`.

```
Requête → shell HTML immédiat (layout, sidebar, header)
        → sections streamées progressivement au fur et à mesure
```

**Règles :**
- Chaque route dans `app/(main)/` a un `loading.tsx` avec le skeleton approprié
- Les sections d'une même page qui fetchent indépendamment sont dans des `<Suspense>` séparés — elles n'attendent pas les unes les autres
- Le skeleton doit reproduire fidèlement la forme du vrai contenu (pas un spinner générique)

Avec TanStack Query (CSR), Suspense s'active via `suspense: true` dans le hook. Le composant doit être wrappé dans `<Suspense>` côté parent.

```tsx
// Hook — activer le mode suspense
const { data } = useGoals(orgId, { suspense: true })

// Page — sections indépendantes dans des Suspense séparés
// ✅ correct — chaque section streame indépendamment
<Suspense fallback={<GoalsSkeleton />}>
  <GoalsGrid orgId={orgId} />
</Suspense>
<Suspense fallback={<StatsSkeleton />}>
  <GoalsStats orgId={orgId} />
</Suspense>

// ❌ interdit — tout attend tout
<Suspense fallback={<PageSkeleton />}>
  <GoalsGrid orgId={orgId} />
  <GoalsStats orgId={orgId} />
</Suspense>
```

**Note :** ce pattern est pour les composants client-side TQ. Ne pas utiliser `async/await` dans les composants pour fetcher — tout passe par les hooks TQ.

### 1.5 Checklist nouveau module
- [ ] service.ts avec toutes les queries Supabase
- [ ] query-keys.ts avec les key factories
- [ ] Hooks TQ avec staleTime choisi depuis la convention §2.2
- [ ] Indexes DB créés dans la migration (§4.1)
- [ ] Pas de QueryClient séparé — utiliser le global
- [ ] `loading.tsx` créé pour la route
- [ ] Sections indépendantes dans des `<Suspense>` séparés

---

## 2. TanStack Query — conventions

### 2.1 QueryProvider global (settings non-négociables)
```ts
new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,   // 5min par défaut
      gcTime: 10 * 60 * 1000,      // 10min
      refetchOnWindowFocus: false,  // app perso, usage multi-onglets intensif
      refetchOnReconnect: true,
      retry: 2,
    },
    mutations: {
      retry: 1,
    },
  },
})
```

### 2.2 staleTime par type de donnée
| Type | staleTime | Exemples |
|---|---|---|
| Données structurelles | 10min | workspaces, projets, paramètres, profil |
| Données actives | 5min | tâches, goals, habits, media |
| Données temps-réel | 2min | dashboard today, habits today |
| Données référentielles | 10min | matchs sport, classements, F1 |

**staleTime: 0 est interdit sans raison documentée dans le code.**

### 2.3 gcTime
- Toujours >= staleTime
- Défaut global : 10min
- Ne jamais descendre en dessous de 5min

### 2.4 refetchOnWindowFocus policy
`false` globalement. Raison : l'utilisateur bascule constamment entre HEGON et d'autres outils. Chaque focus déclencherait un refetch inutile. Phase 3 Real-time remplace ce besoin proprement.

### 2.5 Invalidation après mutation
```ts
onSuccess: () => {
  queryClient.invalidateQueries({ queryKey: KEYS.all })
}
```
Toujours invalider après une mutation. Ne jamais modifier le cache manuellement sauf pour de l'optimistic update justifié.

### 2.6 Règle du QueryClient unique
**Un seul QueryClient dans toute l'application.** Défini dans `src/shared/providers/QueryProvider.tsx`. Tout provider séparé par module est interdit — il rompt la cohérence du cache entre modules.

### 2.7 Query keys — convention obligatoire

```ts
// ✅ correct — key factory dans hooks/query-keys.ts
export const GOAL_KEYS = {
  all: ["goals"] as const,
  list: (userId: string) => [...GOAL_KEYS.all, "list", userId] as const,
  detail: (id: string) => [...GOAL_KEYS.all, "detail", id] as const,
} as const;

// ❌ interdit — array inline dans useQuery
useQuery({ queryKey: ["goals", "list", userId], ... })
```

Un fichier `query-keys.ts` par module dans `hooks/`. Jamais d'array inline dans `useQuery`. Les factories garantissent l'invalidation correcte cross-module.

---

## 3. Service layer & Supabase

### 3.1 Promise.all obligatoire pour les fetches parallèles
```ts
// ✅ correct
const [tasks, projects] = await Promise.all([
  getTasks(orgId),
  getProjects(orgId),
])

// ❌ interdit — waterfall
const tasks = await getTasks(orgId)
const projects = await getProjects(orgId)
```

### 3.2 Select uniquement les colonnes nécessaires
```ts
// ✅ correct
.select('id, title, status, due_date')

// ❌ à éviter sauf justification
.select('*')
```

### 3.3 Pagination — jamais fetcher toutes les lignes
Toute table susceptible de croître (tasks, habits completions, journal entries) doit être paginée ou limitée. Pas de `.select()` sans `.limit()` sur ces tables.

### 3.4 cache() React — usage et limites
`cache()` de React déduplique les appels **dans un seul render serveur**. Il ne persiste pas entre les navigations. Ne pas confondre avec le cache TQ qui persiste dans le navigateur.

### 3.5 useCurrentUserId — hook partagé (pattern obligatoire)

```ts
// src/shared/hooks/useCurrentUserId.ts
// Lit getSession() depuis le localStorage Supabase — zéro réseau, synchrone en pratique
```

**Règle :** jamais passer `userId` en prop depuis un Server Component pour éviter un hook. Utiliser `useCurrentUserId()` dans le wrapper client. Zéro `supabase.auth.getUser()` dans les modules (coût +150ms/render). Server Components de modules = `"use client"` pur.

**Pattern module :**
```ts
// page.tsx "use client" pur
export default function GoalsPage() {
  return <GoalsContent />;
}

// GoalsContent.tsx — hook récupère userId en interne
function GoalsContent() {
  const userId = useCurrentUserId();
  const { data } = useGoals(); // le hook appelle useCurrentUserId() lui-même
  ...
}
```

### 3.6 Connection pooling — Supabase + Vercel serverless
Vercel déploie des fonctions serverless. Chaque invocation crée une nouvelle connexion à Supabase. Sans pooler, à l'échelle les connexions simultanées peuvent dépasser les limites de Supabase.

**Règle :** utiliser le port **6543** (pooler Supabase) dans la `DATABASE_URL` pour les environnements serverless, pas le port 5432 (connexion directe).

```
# ✅ correct — connexion via pooler (port 6543)
DATABASE_URL=postgresql://postgres.[ref]:[password]@aws-0-[region].pooler.supabase.com:6543/postgres

# ❌ connexion directe — limite les connexions simultanées
DATABASE_URL=postgresql://postgres:[password]@db.[ref].supabase.co:5432/postgres
```

Le client Supabase JS (`createClient`) gère ses propres connexions — cette règle s'applique principalement si on ajoute un ORM (Prisma, Drizzle) en Phase 5+. À configurer dès que c'est le cas.

---

## 4. Base de données — indexes

### 4.1 Règle des indexes obligatoires
Toute colonne utilisée dans `.eq()`, `.in()`, `.order()`, `.gte()/.lte()` sur une table fréquemment lue **doit** avoir un index. L'index est créé dans la migration de création de la table, pas ajouté plus tard.

### 4.2 Colonnes qui nécessitent un index
- `org_id` — sur toutes les tables (RLS + filtres)
- `user_id` — sur toutes les tables avec données utilisateur
- `status` — sur tasks, goals, habits, media_items
- `due_date`, `target_date`, `completed_date` — sur toutes les tables avec dates filtrées
- Clés étrangères (`goal_id`, `project_id`, `habit_id`, etc.)

### 4.3 Convention de nommage
```sql
idx_{table}_{colonne}
-- exemples :
idx_tasks_org_id
idx_goals_status
idx_habit_completions_completed_date
```

### 4.4 Index dans la migration, pas après
L'index fait partie de la migration de création. Ajouter un index "plus tard" sur une table existante en prod = opération lourde. Le créer dès le départ coûte rien.

---

## 5. Bundle & Assets

### 5.1 next/image obligatoire
```tsx
// ✅ correct
import Image from 'next/image'
<Image src={url} alt={alt} width={48} height={48} />

// ❌ interdit
<img src={url} alt={alt} />
```

### 5.2 Named imports uniquement
```ts
// ✅ correct
import { format } from 'date-fns'

// ❌ interdit
import dateFns from 'date-fns'
```

### 5.3 Server Components par défaut
`"use client"` uniquement si le composant utilise : `useState`, `useEffect`, un hook TQ, un événement DOM, ou Framer Motion. Tout le reste reste Server Component.

### 5.4 Librairies lourdes
Framer Motion, dnd-kit, et autres librairies lourdes : importer uniquement les sous-modules nécessaires. Ne jamais importer le package entier.

### 5.5 next/font — fonts optimisées, zéro CLS
Sans `next/font`, les fonts custom sont chargées après le rendu initial → layout shift visible (CLS). `next/font` injecte les fonts en CSS inline au moment du build — zéro requête réseau, zéro layout shift.

```ts
// ✅ correct — src/app/layout.tsx
// Fonts actives (migration Geist → Inter, 2026-04-19) :
import { Inter } from 'next/font/google'
import { GeistMono } from 'geist/font/mono'  // Mono conservé pour le code

// Les fonts sont déjà inline dans le HTML — CLS = 0
```

**Règle :** toute font utilisée dans HEGON passe par `next/font`. Jamais de `@import` CSS vers Google Fonts ou autre CDN externe.

### 5.6 Prefetch — quand l'activer, quand le couper
Next.js prefetch automatiquement tous les `<Link>` visibles dans le viewport. Sur une page avec beaucoup de liens (sidebar, listes), ça peut générer des requêtes inutiles.

**Règle par défaut :** laisser le prefetch activé sur les liens de navigation primaire (sidebar, tabs principaux) — l'utilisateur va probablement cliquer dessus.

**Désactiver sur :**
```tsx
// Liens vers des pages lourdes rarement visitées
<Link href="/settings" prefetch={false}>Settings</Link>

// Liens dans des listes longues (chaque item = une page)
<Link href={`/goals/${goal.id}`} prefetch={false}>...</Link>
```

---

## 6. Temps réel — Phase 3

### 6.1 The cron problem
Avec CSR + TQ, si un cron Supabase update la DB pendant la fenêtre staleTime, la deuxième visite est instantanée mais avec les anciennes données. Les nouvelles données apparaissent seulement après expiration du staleTime (background refetch). Ce comportement est acceptable en Phase 2 — il est résolu définitivement en Phase 3.

### 6.2 Pattern useRealtimeSync
```ts
// src/shared/hooks/useRealtimeSync.ts
export function useRealtimeSync(channel, queryKey, filter) {
  useEffect(() => {
    const sub = supabase.channel(channel)
      .on('postgres_changes', { event: '*', schema: 'public', filter },
        () => queryClient.invalidateQueries({ queryKey }))
      .subscribe()
    return () => supabase.removeChannel(sub)
  }, [])
}
```
Un seul hook partagé. Chaque module l'appelle avec son channel et ses query keys.

### 6.3 Quelles données nécessitent du real-time
- Tasks (kanban multi-user)
- Dashboard Today (mise à jour sans refresh)
- Habits Today (complétion visible immédiatement)
- Goals progress (mise à jour depuis les tâches liées)

### 6.4 Ce que Phase 3 résout définitivement
Après Phase 3 : toute modification en DB (cron ou action utilisateur) se propage automatiquement à l'UI via WebSocket. Le staleTime devient une sécurité de fallback, pas le mécanisme principal de fraîcheur.

---

## 7. Monitoring

### 7.1 Sentry Performance
`tracesSampleRate` configuré. Toute transaction serveur > 2s génère une alerte. Les transactions client sont tracées pour les mutations critiques.

### 7.2 Vercel Analytics — Core Web Vitals
Activé en production. Objectifs : LCP < 2.5s, FID < 100ms, CLS < 0.1.

### 7.3 Supabase Query Performance dashboard
Consulter avant chaque release. Les slow queries (> 100ms) sont à optimiser avec des indexes ou une réécriture de query.

### 7.4 Ce qu'on surveille, ce qu'on ignore
On surveille : TTFB, LCP, queries Supabase lentes, erreurs Sentry.
On ignore : bundle size < 500KB par route (acceptable), TTI sur mobile (app desktop-first).

---

## 8. Scalabilité — règles pour chaque nouveau module

### 8.1 Checklist avant de coder
- [ ] staleTime choisi depuis la convention §2.2 et documenté
- [ ] Index DB listés et inclus dans la migration
- [ ] Pas de QueryClient séparé
- [ ] Promise.all pour tous les fetches parallèles dans service.ts
- [ ] Select columns explicites (pas de `*` sur tables larges)
- [ ] `loading.tsx` créé pour la route
- [ ] Sections indépendantes dans des `<Suspense>` séparés
- [ ] Liens dans les listes longues avec `prefetch={false}`

### 8.2 Ce qui scale automatiquement
- Le cache TQ : plus l'utilisateur utilise l'app, moins il y a de requêtes
- Les indexes DB : performants qu'il y ait 100 ou 100 000 lignes
- Le pattern module (service.ts + hooks) : identique peu importe la complexité

### 8.3 Ce qui ne scale pas et comment le gérer
- **Fetches non paginés** : tables qui grandissent (completions, journal entries) → toujours `.limit()` + pagination cursor
- **Queries sans index sur colonnes filtrées** → régler dès la migration
- **Composants "use client" qui wrappent tout** → découper en Server + Client selon le besoin réel

### 8.4 Landing page — projet séparé
La landing page `hegon.fr` est un projet Next.js distinct de l'app `app.hegon.fr`. Raisons :
- L'app est 100% authentifiée, CSR → zéro SEO possible
- La landing nécessite SSG, méta-tags, Open Graph, vitesse pour non-authentifiés
- Séparation propre : deux domaines, deux projets Vercel, deux pipelines CI

---

## 9. Roadmap performance

### 9.1 État actuel post-migration CSR
- Tous les modules : instantané après première visite (TQ cache)
- Premier chargement : 2-3s avec skeletons
- Cron problem : données fraîches après expiration staleTime seulement

### 9.2 Phase 3 : Real-time → cron problem résolu
Supabase WebSocket events → invalidateQueries automatique → UI toujours à jour sans action utilisateur.

### 9.3 Phase 3+ : Hydration → premier chargement résolu
SSR pour les données initiales + TQ hydraté = premier chargement avec données immédiates. Priorité : Dashboard et Sports (pages les plus visitées en premier).

### 9.4 Ce qui reste hors scope
- Optimisation mobile (HEGON est desktop-first)
- Edge caching (Vercel Edge Network — pertinent si multi-région, pas maintenant)
- Service Worker / offline mode (hors roadmap)

---

## 10. FORBIDDEN

- `staleTime: 0` sans commentaire explicite justifiant pourquoi
- Fetches en cascade (waterfall) là où `Promise.all` est possible
- `import lib from 'heavy-lib'` — toujours les named imports
- `<img>` au lieu de `next/image`
- `@import` CSS vers une font externe — toujours `next/font`
- `select('*')` sur une table avec plus de 10 colonnes
- QueryClient créé dans un module — un seul global
- Index DB ajouté après la mise en prod d'une migration (les créer dans la migration initiale)
- `"use client"` sur un composant sans interaction ni hook client
- Route sans `loading.tsx` (chaque route en a un)
- `<Suspense>` unique qui wrappe toute une page (sections indépendantes = Suspense séparés)

---

## 11. Préférences vs Référence partagée

Règle établie sur Sports. À appliquer à tous les futurs modules avec données externes.

| Type | Règle | Exemples |
|------|-------|---------|
| **Préférence** (créée par user) | A `org_id` + `user_id` | `football_user_settings`, `user_favorites`, `books` |
| **Référence** (globale, edge function) | **Pas** d'`org_id` | `football_teams`, `tennis_players`, `f1_races` |

**Application aux futurs modules :**
- **Books** : `books` avec org_id (status, rating, notes perso) + `books_cache` sans org_id (isbn, cover, pages depuis Open Library)
- **Tech** : `tech_entries` a org_id (contenu 100% personnel, pas de cache partagé V1)
- **Culture/Travel** : TBD à la spec v1.md

**Pattern service :**
```ts
// Fetch préférences (org_id scopé via RLS)
const prefs = await supabase.from("books").select("*").eq("org_id", orgId);
// Fetch référence (pas de org_id — accessible à tous)
const cache = await supabase.from("books_cache").select("*").in("isbn", isbns);
// Merge côté service avant de retourner
```
