# HEGON Testing

> Source de vérité pour la stratégie de test.
> Standard : pragmatique, pas académique. On teste ce qui casse silencieusement.
> Principe fondateur : **un service cassé = données perdues. Un composant cassé = UI moche.**

---

## Table des matières

### 0. Philosophie
- Ce qu'on teste et pourquoi
- Ce qu'on ne teste pas et pourquoi
- Le critère de valeur d'un test

### 1. Stack
- 1.1 Vitest — unit + integration
- 1.2 Playwright — E2E
- 1.3 MSW — mock des APIs externes
- 1.4 Supabase local — vraie DB pour les services
- 1.5 Setup et configuration
- 1.6 Variables d'environnement — `.env.test`

### 2. Pyramide de tests HEGON
- 2.1 Niveau 1 : Utilities (fonctions pures)
- 2.2 Niveau 2 : Service layer (queries Supabase)
- 2.3 Niveau 3 : API Routes (auth + validation)
- 2.4 Niveau 4 : E2E (chemins critiques)

### 3. Conventions
- 3.1 Localisation des fichiers
- 3.2 Nommage
- 3.3 Structure d'un test
- 3.4 Isolation DB — exécution parallèle

### 4. Mocking — règles strictes
- 4.1 Ne jamais mocker Supabase dans les tests service
- 4.2 Toujours mocker les APIs externes (MSW)
- 4.3 Ce qu'on peut mocker sans risque

### 5. Patterns par type
- 5.1 Test d'une fonction utilitaire
- 5.2 Test d'un service Supabase
- 5.3 Test d'une API route
- 5.4 Test E2E Playwright

### 6. Coverage — objectifs réalistes
- 6.1 Cibles par couche
- 6.2 Ce qu'on ne mesure pas
- 6.3 Quand un test a de la valeur

### 7. CI — intégration
- 7.1 Ce qui bloque un merge
- 7.2 Ce qui est informatif
- 7.3 Ordre d'exécution
- 7.4 Supabase local en CI — GitHub Actions

### 8. Nouveau module — checklist tests
- 8.1 Ce qui doit exister avant de marquer V1 done
- 8.2 Priorité dans l'ordre

### 9. FORBIDDEN

---

## 0. Philosophie

### Ce qu'on teste et pourquoi
HEGON n'a pas pour objectif 100% de coverage — l'objectif est de détecter les pannes silencieuses avant la prod.

Les pannes silencieuses sont celles qui :
- Ne génèrent pas d'erreur visible (données incorrectes, filtres qui laissent passer trop)
- Touchent la couche qui détient la logique critique (le service layer)
- Sont liées à de la logique pure qu'un dev peut casser en refactorisant

### Ce qu'on ne teste pas et pourquoi
| Ce qu'on ne teste pas | Raison |
|---|---|
| Composants UI | Trop fragiles, trop coûteux, cassent pour des raisons cosmétiques |
| Pages Next.js | Couvertes par le CI build — si ça build, la page est valide |
| Hooks TanStack Query | Testés indirectement via les services qu'ils appellent |
| Styling / CSS | Zéro valeur — les yeux suffisent |
| Zustand stores | Logique triviale, testée via E2E |

### Le critère de valeur d'un test
Un test a de la valeur si sa suppression laisserait un vrai bug passer inaperçu.
Si le test ne détecte que "le code fait ce qu'il fait", il n'a pas de valeur.

---

## 1. Stack

### 1.1 Vitest — unit + integration
Vitest pour tous les tests unitaires et d'intégration. Raisons :
- Native ESM + TypeScript sans config
- 10x plus rapide que Jest sur ce type de projet
- API identique à Jest — zéro courbe d'apprentissage
- S'intègre nativement avec le module resolver de Next.js

```bash
npm install -D vitest @vitejs/plugin-react vite-tsconfig-paths
```

### 1.2 Playwright — E2E
Playwright pour les tests end-to-end. Raisons vs Cypress :
- Supporte Next.js App Router natif (pas de hack pour SSR)
- Cross-browser (Chromium, Firefox, WebKit) dans un seul run
- API async/await native, pas de callbacks
- Meilleur support des WebSockets (pertinent pour Phase 3)

```bash
npm install -D @playwright/test
npx playwright install
```

### 1.3 MSW — mock des APIs externes
Mock Service Worker pour intercepter les appels aux APIs externes (OpenWeather, TMDB, TheSportsDB) dans les tests. MSW intercepte au niveau réseau — le code ne sait pas qu'il est en test.

```bash
npm install -D msw
```

Handlers dans `src/tests/mocks/handlers.ts` — un handler par API externe.

### 1.4 Supabase local — vraie DB pour les services
Les tests de service layer tournent contre une instance Supabase locale (`supabase start`). **Jamais de mock Supabase pour les services** — voir §4.1.

Chaque test qui touche la DB :
- Commence par insérer ses propres données de test (fixtures)
- Se nettoie après lui-même (DELETE ou transaction rollback)
- Est isolé — l'ordre d'exécution ne compte pas

### 1.5 Setup et configuration

**`vitest.config.ts`** à la racine :
```ts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  test: {
    globals: true,
    setupFiles: ['./src/tests/setup.ts'],
    exclude: ['e2e/**', 'node_modules/**'],
    // Pas de `include` ni `environment` ici — définis par workspace ci-dessous
    workspace: [
      {
        // Pool 1 : tests sans DB — parallèle (rapide)
        extends: true,
        test: {
          name: 'unit',
          include: ['src/shared/**/*.test.ts', 'src/app/api/**/*.test.ts'],
          environment: 'node',
          pool: 'threads',
        },
      },
      {
        // Pool 2 : tests service avec DB — séquentiel (sûr)
        extends: true,
        test: {
          name: 'service',
          include: ['src/modules/**/service.test.ts'],
          environment: 'node',
          pool: 'forks',
          poolOptions: {
            forks: { singleFork: true },
          },
        },
      },
    ],
  },
})
```

**`playwright.config.ts`** à la racine :
```ts
import { defineConfig } from '@playwright/test'

export default defineConfig({
  testDir: './e2e',
  use: {
    baseURL: 'http://localhost:3000',
    storageState: 'e2e/.auth/user.json', // session pré-authentifiée
  },
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
})
```

**Scripts `package.json`** :
```json
"test": "vitest run",
"test:watch": "vitest",
"test:e2e": "playwright test",
"test:e2e:ui": "playwright test --ui"
```

### 1.6 Variables d'environnement — `.env.test`
Créer `.env.test` à la racine (gitignored). Ce fichier est requis pour faire tourner les tests service et E2E.

```bash
# .env.test — NE PAS COMMITTER

# Supabase local (supabase start → affiche ces valeurs)
NEXT_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...  # affiché par supabase start
SUPABASE_SERVICE_ROLE_KEY=eyJ...      # affiché par supabase start

# APIs externes — factices, interceptées par MSW
OPENWEATHER_API_KEY=test_key
TMDB_API_KEY=test_key

# Upstash Redis — mock en test, pas besoin de vraies valeurs
UPSTASH_REDIS_REST_URL=http://localhost
UPSTASH_REDIS_REST_TOKEN=test_token

# Compte E2E Playwright (créé manuellement dans Supabase local)
TEST_USER_EMAIL=test@hegon.local
TEST_USER_PASSWORD=test_password_local
```

Ajouter `.env.test` et `e2e/.auth/` dans `.gitignore` :
```
.env.test
e2e/.auth/
```

Pour récupérer les valeurs Supabase locales : `supabase status` après `supabase start`.

---

## 2. Pyramide de tests HEGON

```
         /‾‾‾‾‾‾‾‾‾‾‾‾\
        /   E2E (peu)   \        ← chemins critiques seulement
       /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
      /   API Routes (tout) \    ← auth + validation 100%
     /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
    /   Service layer (core)  \  ← queries critiques + mutations
   /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
  /   Utilities (tout, rapide)  \ ← fonctions pures = 100%
 /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
```

### 2.1 Niveau 1 : Utilities
Fonctions pures dans `utils.ts`, `*-utils.ts`. Tests rapides, 100% coverage visé. Zéro setup nécessaire.

Exemples : calcul priorité tâche, formatage date, tri par status, calcul streak habits.

### 2.2 Niveau 2 : Service layer
Les fonctions dans chaque `service.ts`. C'est la couche la plus critique — elle détient toute la logique de lecture/écriture.

Tester :
- Les queries de lecture avec filtres (`getGoals`, `getTasks`, `getHabits`)
- Les mutations critiques (create, update, delete, complete)
- Les cas limites (org_id vide, user sans données, pagination)

Ne pas tester :
- Les fonctions triviales (simple `select *` sans filtre)
- Les fonctions appelées uniquement par d'autres fonctions du même service

### 2.3 Niveau 3 : API Routes
Toutes les routes dans `src/app/api/`. Tester systématiquement :
- Requête sans token → 401
- Requête avec input invalide → 400
- Rate limit dépassé → 429 (mock Redis)
- Requête valide → 200 + structure de réponse correcte

### 2.4 Niveau 4 : E2E
Chemins critiques uniquement. Pas la peine de tout couvrir — couvrir ce qui, si cassé, rend l'app inutilisable.

Chemins prioritaires :
1. Login → dashboard visible
2. Créer une tâche → apparaît dans le kanban
3. Compléter un habit → streak incrémente
4. Créer un goal → apparaît dans la liste

---

## 3. Conventions

### 3.1 Localisation des fichiers
```
src/
├── modules/
│   └── goals/
│       ├── service.ts
│       ├── service.test.ts        ← tests du service, co-localisés
│       └── hooks/
├── shared/
│   └── utils/
│       ├── utils.ts
│       └── utils.test.ts          ← tests utilitaires, co-localisés
├── app/
│   └── api/
│       └── weather/
│           ├── route.ts
│           └── route.test.ts      ← tests API routes, co-localisés
└── tests/
    ├── setup.ts                   ← setup global Vitest
    ├── fixtures/                  ← données de test réutilisables
    │   ├── goals.ts
    │   └── tasks.ts
    └── mocks/
        └── handlers.ts            ← MSW handlers APIs externes
e2e/
├── auth.setup.ts                  ← setup session Playwright
├── tasks.spec.ts
├── habits.spec.ts
└── .auth/
    └── user.json                  ← session stockée (gitignored)
```

### 3.2 Nommage
```ts
// Fichiers : {nom-du-fichier-testé}.test.ts
service.test.ts
utils.test.ts
route.test.ts

// Describes : nom du module + fonction
describe('Goals service', () => {
  describe('getGoals', () => { ... })
  describe('createGoal', () => { ... })
})

// Tests : "should {comportement attendu} when {condition}"
it('should return only goals for the current org', ...)
it('should throw when org_id is missing', ...)
it('should return 401 when user is not authenticated', ...)
```

### 3.4 Isolation DB — exécution parallèle
Vitest exécute les fichiers de test en parallèle par défaut. Si deux fichiers de test service tournent simultanément sur la même DB locale, ils peuvent interférer — un test compte des lignes pendant qu'un autre en insère.

**Solution : deux pools séparés dans `vitest.config.ts`**

```ts
export default defineConfig({
  test: {
    // Tests utilitaires et API routes — parallèle, pas de DB
    include: ['src/shared/**/*.test.ts', 'src/app/api/**/*.test.ts'],

    workspace: [
      {
        // Pool 1 : tests sans DB — parallèle (rapide)
        extends: true,
        test: {
          name: 'unit',
          include: ['src/shared/**/*.test.ts', 'src/app/api/**/*.test.ts'],
          pool: 'threads',
        },
      },
      {
        // Pool 2 : tests service avec DB — séquentiel (sûr)
        extends: true,
        test: {
          name: 'service',
          include: ['src/modules/**/service.test.ts'],
          pool: 'forks',
          poolOptions: {
            forks: { singleFork: true }, // un seul worker = séquentiel
          },
        },
      },
    ],
  },
})
```

**Règle complémentaire :** chaque test service crée son propre `orgId` via `createTestOrg()` et le supprime dans `afterEach`. Les données sont isolées par org — deux tests qui tournent en parallèle ne lisent pas les mêmes lignes.

### 3.3 Structure d'un test
```ts
// Arrange — Préparer les données
const { orgId, userId } = await createTestOrg()
await insertFixture('goal', { orgId, title: 'Test goal' })

// Act — Exécuter
const goals = await getGoals(orgId)

// Assert — Vérifier
expect(goals).toHaveLength(1)
expect(goals[0].title).toBe('Test goal')

// Cleanup — Nettoyer
await cleanupTestOrg(orgId)
```

---

## 4. Mocking — règles strictes

### 4.1 Ne jamais mocker Supabase dans les tests service
**Règle absolue pour les tests service.** Mocker Supabase dans les tests service = tester que le mock fonctionne, pas que le service fonctionne.

Raison concrète : un mock Supabase ne détecte pas une query mal écrite, une colonne manquante, un RLS qui bloque ou un index absent. La vraie DB locale le détecte.

```ts
// ❌ interdit dans les tests service
vi.mock('@/infrastructure/supabase/server', () => ({
  createServerClient: vi.fn(() => ({ from: vi.fn() ... }))
}))

// ✅ correct — vraie DB locale
const supabase = createLocalTestClient()
```

**Exception — tests API routes :** mocker `createServerClient` est autorisé dans `route.test.ts` pour isoler la logique auth (tester le 401, le 429, etc.) sans avoir besoin d'une vraie session. On teste le comportement de la route, pas la DB. Voir §5.3 pour le pattern complet.

### 4.2 Toujours mocker les APIs externes (MSW)
Les appels vers OpenWeather, TMDB, TheSportsDB dans les tests = non-déterministes, lents, payants.

```ts
// src/tests/mocks/handlers.ts
import { http, HttpResponse } from 'msw'

export const handlers = [
  http.get('https://api.openweathermap.org/*', () =>
    HttpResponse.json({ main: { temp: 20 }, weather: [{ main: 'Clear' }] })
  ),
  http.get('https://api.themoviedb.org/*', () =>
    HttpResponse.json({ results: [] })
  ),
]
```

### 4.3 Ce qu'on peut mocker sans risque
- APIs externes (MSW) ✅
- `Date.now()` pour tester la logique de dates ✅
- Redis/Upstash dans les tests API routes ✅
- Variables d'environnement (`vi.stubEnv`) ✅

---

## 5. Patterns par type

### 5.1 Test d'une fonction utilitaire
```ts
// src/shared/utils/utils.test.ts
import { describe, it, expect } from 'vitest'
import { getTaskPriority, formatRelativeDate } from './utils'

describe('getTaskPriority', () => {
  it('should return critical before high', () => {
    const tasks = [
      { priority: 'high', due_date: '2026-01-01' },
      { priority: 'critical', due_date: '2026-01-02' },
    ]
    expect(getTaskPriority(tasks)[0].priority).toBe('critical')
  })

  it('should sort by due_date when priority is equal', () => {
    const tasks = [
      { priority: 'high', due_date: '2026-02-01' },
      { priority: 'high', due_date: '2026-01-01' },
    ]
    expect(getTaskPriority(tasks)[0].due_date).toBe('2026-01-01')
  })
})
```

### 5.2 Test d'un service Supabase
```ts
// src/modules/goals/service.test.ts
import { describe, it, expect, beforeEach, afterEach } from 'vitest'
import { getGoals, createGoal } from './service'
import { createTestOrg, cleanupTestOrg } from '@/tests/fixtures/org'

describe('Goals service', () => {
  let orgId: string

  beforeEach(async () => {
    orgId = await createTestOrg()
  })

  afterEach(async () => {
    await cleanupTestOrg(orgId)
  })

  describe('getGoals', () => {
    it('should return only goals for the current org', async () => {
      const otherOrgId = await createTestOrg()
      await createGoal(orgId, { title: 'My goal' })
      await createGoal(otherOrgId, { title: 'Other org goal' })

      const goals = await getGoals(orgId)

      expect(goals).toHaveLength(1)
      expect(goals[0].title).toBe('My goal')

      await cleanupTestOrg(otherOrgId)
    })

    it('should return empty array when org has no goals', async () => {
      const goals = await getGoals(orgId)
      expect(goals).toHaveLength(0)
    })
  })
})
```

### 5.3 Test d'une API route
```ts
// src/app/api/weather/route.test.ts
import { describe, it, expect, vi, beforeAll, afterAll } from 'vitest'
import { setupServer } from 'msw/node'
import { handlers } from '@/tests/mocks/handlers'
import { GET } from './route'
import { NextRequest } from 'next/server'

const server = setupServer(...handlers)
beforeAll(() => server.listen())
afterAll(() => server.close())

describe('GET /api/weather', () => {
  it('should return 401 when user is not authenticated', async () => {
    vi.mock('@/infrastructure/supabase/server', () => ({
      createServerClient: vi.fn(() => ({
        auth: { getUser: vi.fn().mockResolvedValue({ data: { user: null } }) }
      }))
    }))

    const req = new NextRequest('http://localhost/api/weather')
    const res = await GET(req)

    expect(res.status).toBe(401)
  })

  it('should return 429 when rate limit is exceeded', async () => {
    vi.mock('@/shared/lib/ratelimit', () => ({
      weatherRatelimit: { limit: vi.fn().mockResolvedValue({ success: false }) }
    }))

    // ... setup authenticated user mock
    const req = new NextRequest('http://localhost/api/weather')
    const res = await GET(req)

    expect(res.status).toBe(429)
  })

  it('should return weather data when request is valid', async () => {
    // ... setup authenticated user + rate limit success
    const req = new NextRequest('http://localhost/api/weather')
    const res = await GET(req)
    const data = await res.json()

    expect(res.status).toBe(200)
    expect(data).toHaveProperty('temp')
    expect(data).toHaveProperty('condition')
  })
})
```

### 5.4 Test E2E Playwright
```ts
// e2e/tasks.spec.ts
import { test, expect } from '@playwright/test'

test('should create a task and see it in the kanban', async ({ page }) => {
  await page.goto('/pro/tasks')

  // Ouvrir le modal
  await page.getByRole('button', { name: 'New task' }).click()

  // Remplir
  await page.getByLabel('Title').fill('Test E2E task')
  await page.getByRole('button', { name: 'Create' }).click()

  // Vérifier
  await expect(page.getByText('Test E2E task')).toBeVisible()
})

// e2e/auth.setup.ts — run une seule fois, session stockée
import { test as setup } from '@playwright/test'

setup('authenticate', async ({ page }) => {
  await page.goto('/auth')
  await page.getByLabel('Email').fill(process.env.TEST_USER_EMAIL!)
  await page.getByLabel('Password').fill(process.env.TEST_USER_PASSWORD!)
  await page.getByRole('button', { name: 'Sign in' }).click()
  await page.waitForURL('/dashboard')
  await page.context().storageState({ path: 'e2e/.auth/user.json' })
})
```

---

## 6. Coverage — objectifs réalistes

### 6.1 Cibles par couche
| Couche | Cible | Raison |
|---|---|---|
| Utilities | 100% | Fonctions pures, coût zéro |
| Service layer | 80% | Les mutations critiques + queries filtrées |
| API Routes | 100% des paths auth/validation | Sécurité non-négociable |
| E2E | 4-6 chemins critiques | Pas d'exhaustivité, juste les blockers |
| Composants UI | 0% | Hors scope |

### 6.2 Ce qu'on ne mesure pas
Ne pas chasser un chiffre global de coverage. Un projet avec 40% de coverage bien ciblé est plus solide qu'un projet avec 80% de coverage sur du code trivial.

### 6.3 Quand un test a de la valeur
- Il a détecté un bug au moins une fois dans l'historique git
- Il teste un filtre de sécurité (RLS, auth, validation)
- Il teste une logique de tri/priorité qui peut silencieusement mal ordonner des données
- Il teste une mutation qui peut corrompre des données si mal écrite

---

## 7. CI — intégration

### 7.1 Ce qui bloque un merge
```yaml
# .github/workflows/ci.yml
- name: Unit & integration tests
  run: npm run test       # vitest run — BLOQUE si échec

- name: Type check
  run: tsc --noEmit       # BLOQUE si erreur TypeScript

- name: Lint
  run: npm run lint       # BLOQUE si erreur ESLint
```

### 7.2 Ce qui est informatif (ne bloque pas)
- Les tests E2E Playwright — trop lents pour tourner sur chaque PR. Tourner sur merge vers `main` uniquement.
- Le coverage report — généré et affiché, mais pas de threshold qui bloque.

### 7.3 Ordre d'exécution
```
1. lint + typecheck      (30s)  — rapide, bloque tôt
2. unit + integration    (60s)  — service + utils + API routes
3. build                 (2min) — vérifie que Next.js compile
4. E2E (sur main seulement)     — contre l'environnement preview Vercel
```

### 7.4 Supabase local en CI — GitHub Actions
Les tests service ont besoin de Supabase local. En CI, il faut démarrer l'instance avant les tests.

```yaml
# .github/workflows/ci.yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Setup Supabase CLI
        uses: supabase/setup-cli@v1
        with:
          version: latest

      - name: Start Supabase local
        run: supabase start   # démarre la DB locale + applique les migrations

      - name: Run unit + integration tests
        run: npm run test
        env:
          NEXT_PUBLIC_SUPABASE_URL: http://127.0.0.1:54321
          NEXT_PUBLIC_SUPABASE_ANON_KEY: ${{ steps.supabase.outputs.anon_key }}
          SUPABASE_SERVICE_ROLE_KEY: ${{ steps.supabase.outputs.service_role_key }}
          OPENWEATHER_API_KEY: test_key
          TMDB_API_KEY: test_key
          UPSTASH_REDIS_REST_URL: http://localhost
          UPSTASH_REDIS_REST_TOKEN: test_token

      - name: Lint + typecheck
        run: npm run lint && tsc --noEmit

      - name: Build
        run: npm run build
```

**Point critique :** `supabase start` applique automatiquement toutes les migrations dans `supabase/migrations/`. Les tests tournent toujours contre un schéma à jour.

---

## 8. Nouveau module — checklist tests

### 8.1 Ce qui doit exister avant de marquer V1 done
- [ ] Tests utilities pour toutes les fonctions pures du module
- [ ] Tests service pour les mutations critiques (create, update, delete)
- [ ] Tests service pour les queries avec filtres (org isolation vérifiée)
- [ ] Tests API route pour auth + validation (si le module a des routes)
- [ ] Au moins 1 test E2E pour le chemin le plus utilisé

### 8.2 Priorité dans l'ordre
1. **Mutations destructives** (delete, archive, mark complete) — testées en premier
2. **Isolation org** dans les queries — testée systématiquement
3. **Validation inputs** dans les API routes — testée exhaustivement
4. **Logique de tri/calcul** (streaks, progress, priorité) — testée avec cas limites
5. **Happy path E2E** — dernier, mais obligatoire

---

## 9. FORBIDDEN

- Mocker `createServerClient` ou `createClient` Supabase dans des tests de service
- Tests qui ne peuvent jamais échouer (assertions `expect(true).toBe(true)`)
- Tests qui dépendent d'un ordre d'exécution (chaque test est indépendant)
- `console.log` dans les tests — utiliser `expect` ou les reporters Vitest
- Tests E2E sans `storageState` — ne jamais re-login à chaque test (trop lent)
- Fixtures en dur dans les tests — toujours dans `src/tests/fixtures/`
- Tests sans cleanup — toute donnée insérée en DB doit être supprimée après
