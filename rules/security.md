# HEGON Security

> Source de vérité pour toutes les décisions de sécurité.
> Standard : défense en profondeur — RLS protège la DB, l'API route protège le réseau, le composant ne fait que de l'UI.
> Principe fondateur : **zéro confiance côté client**.

---

## Table des matières

### 0. Philosophie
- Défense en profondeur
- Principe du moindre privilège
- Ce qui est suspect par définition

### 1. API Routes — pattern obligatoire
- 1.1 Auth check — non-négociable
- 1.2 Rate limiting — Upstash Redis
- 1.3 Validation des inputs
- 1.4 État actuel des routes

### 2. Middleware
- 2.1 Protection globale des pages
- 2.2 Routes publiques vs protégées
- 2.3 Ce que le middleware ne fait pas

### 3. RLS — Row Level Security
- 3.1 Principe multi-tenant (my_orgs())
- 3.2 Pattern obligatoire par table
- 3.3 Ce qui a été audité et corrigé
- 3.4 Règles pour les nouvelles tables

### 4. Headers HTTP
- 4.1 Headers en place
- 4.2 Content Security Policy — manquant (à faire)
- 4.3 remotePatterns — trop permissif (à corriger)

### 5. Secrets & clés
- 5.1 Variables d'environnement — règles
- 5.2 Ce qui ne doit jamais être côté client
- 5.3 Service role key — usage strict

### 6. Nouveau module — checklist sécurité
- 6.1 Checklist avant de coder
- 6.2 Checklist avant de merger

### 7. Score actuel et roadmap
- 7.1 État actuel : 8/10
- 7.2 Ce qui manque pour 10/10
- 7.3 FORBIDDEN

---

## 0. Philosophie

### Défense en profondeur
Trois couches indépendantes. Si l'une échoue, les deux autres tiennent.

```
Couche 1 — DB       : RLS bloque toute query hors périmètre org/user
Couche 2 — API      : auth check + rate limit + validation input
Couche 3 — Middleware : redirection si session absente
```

### Principe du moindre privilège
Chaque rôle Supabase n'a accès qu'à ce dont il a besoin.
- `anon` : uniquement les routes publiques (auth)
- `authenticated` : ses données uniquement, via RLS
- `service_role` : uniquement dans les Edge Functions et les routes API qui en ont besoin, jamais exposé côté client

### Ce qui est suspect par définition
Tout ce qui vient du browser est suspect : paramètres URL, body de requête, headers. Valider avant d'utiliser. Ne jamais faire confiance à un `userId` ou `orgId` venant du client — toujours les extraire depuis la session serveur.

---

## 1. API Routes — pattern obligatoire

### 1.1 Auth check — non-négociable
Toute route API qui lit ou écrit des données commence par :

```ts
const supabase = await createServerClient();
const { data: { user } } = await supabase.auth.getUser();
if (!user) {
  return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
}
```

Ne jamais utiliser `getSession()` pour vérifier l'auth côté serveur — `getUser()` valide le token JWT avec Supabase, `getSession()` lit uniquement le cookie local (non sécurisé).

### 1.2 Rate limiting — Upstash Redis
Rate limiting distribué via Upstash Redis + `@upstash/ratelimit`. Fonctionne sur Vercel serverless (cross-instance) contrairement à l'in-memory.

```ts
import { weatherRatelimit } from "@/shared/lib/ratelimit";

const ip = request.headers.get("x-forwarded-for")?.split(",")[0]?.trim() ?? "anonymous";
const { success } = await weatherRatelimit.limit(ip);
if (!success) {
  return NextResponse.json({ error: "Too many requests" }, { status: 429 });
}
```

Limites actuelles (`src/shared/lib/ratelimit.ts`) :
| Route | Algo | Limite |
|---|---|---|
| `/api/weather` | Sliding window | 20 req/min par IP |
| `/api/tmdb` | Sliding window | 30 req/min par IP |
| `/api/tennis/*` | Sliding window | 20 req/min par IP |

**Toute nouvelle route API sensible doit avoir son limiteur dans `ratelimit.ts`.**

### 1.3 Validation des inputs
Valider tout input avant utilisation. Pattern selon le type :

```ts
// Regex pour les endpoints string
const ALLOWED_ENDPOINT = /^[a-z\d_/\-]+$/i;
if (!ALLOWED_ENDPOINT.test(endpoint)) {
  return NextResponse.json({ error: "Invalid endpoint" }, { status: 400 });
}

// UUID pour les identifiants
const UUID_REGEX = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
const validIds = playerIds.filter(id => UUID_REGEX.test(id));

// Zod pour les bodies complexes
const schema = z.object({ title: z.string().min(1).max(255) });
const result = schema.safeParse(body);
if (!result.success) return NextResponse.json({ error: "Invalid input" }, { status: 400 });
```

### 1.4 État actuel des routes

⚠️ Ce tableau est un **inventaire**, pas une règle — il pourrit à chaque route ajoutée sans qu'on
revienne le mettre à jour ici (vérifié 2026-07-23 : 3 routes manquaient déjà — `books/search`,
`omdb`, `proxy-image`). La vraie règle est la checklist §6. Ne pas relire ce tableau pour savoir si
une route respecte la règle — relire le fichier de la route.

| Route | Auth | Rate limit | Validation input |
|---|---|---|---|
| `GET /api/weather` | ✅ | ✅ Redis 20/min | ✅ (aucun input user) |
| `GET /api/tmdb` | ✅ | ✅ Redis 30/min | ✅ regex endpoint |
| `POST /api/tennis/get-api-ids` | ✅ | ✅ Redis 20/min | ✅ UUID filter |
| `POST /api/tennis/add-player` | ✅ | ✅ Redis 20/min | à vérifier |
| `GET /api/books/search` | ✅ `getClaims()` | ✅ Redis (`booksRatelimit`) | à vérifier |
| `GET /api/omdb` | ✅ `getClaims()` | ✅ Redis (`tmdbRatelimit`, partagé) | à vérifier |
| `GET /api/proxy-image` | ❌ aucun | ❌ aucun | ✅ allowlist d'hosts (`image.tmdb.org`, le bucket Supabase du projet) |

**🔴 `proxy-image` viole la checklist §6.2** (aucun auth check, aucun rate limiter) — la seule
mitigation est l'allowlist d'hosts, qui protège contre le SSRF mais pas contre l'abus/le coût. À
corriger ou à documenter comme exception assumée (image publique déjà servie par un CDN public,
pas de donnée privée) — pas tranché ici, à décider par l'owner.

---

## 2. Middleware

### 2.1 Protection globale des pages
`src/middleware.ts` intercepte toutes les requêtes. Si l'utilisateur n'est pas authentifié et tente d'accéder à une route protégée → redirect vers `/auth?next={pathname}`.

### 2.2 Routes publiques vs protégées
```ts
const PUBLIC_ROUTES = ["/auth", "/auth/callback"];
```
Tout ce qui n'est pas dans `PUBLIC_ROUTES` est protégé. Ajouter explicitement les nouvelles routes publiques si nécessaire — par défaut tout est privé.

### 2.3 Ce que le middleware ne fait pas
Le middleware vérifie la session mais **ne vérifie pas l'appartenance à une org**. L'isolation des données reste gérée par RLS. Ne pas ajouter de logique métier dans le middleware.

---

## 3. RLS — Row Level Security

### 3.1 Principe multi-tenant (my_orgs())
Fonction helper réutilisable dans toutes les policies :
```sql
CREATE OR REPLACE FUNCTION my_orgs()
RETURNS SETOF uuid AS $$
  SELECT org_id FROM memberships WHERE user_id = auth.uid()
$$ LANGUAGE sql SECURITY DEFINER STABLE;
```

### 3.2 Pattern obligatoire par table
```sql
-- SELECT
CREATE POLICY "org_read" ON {table}
  FOR SELECT USING (org_id IN (SELECT * FROM my_orgs()));

-- INSERT
CREATE POLICY "org_insert" ON {table}
  FOR INSERT WITH CHECK (org_id IN (SELECT * FROM my_orgs()));

-- UPDATE
CREATE POLICY "org_update" ON {table}
  FOR UPDATE USING (org_id IN (SELECT * FROM my_orgs()));

-- DELETE
CREATE POLICY "org_delete" ON {table}
  FOR DELETE USING (org_id IN (SELECT * FROM my_orgs()));
```

### 3.3 Ce qui a été audité et corrigé (Phase 0.75)
- Policies cassées `football_standings` et `football_team_competitions` — corrigées
- `REVOKE TRUNCATE` sur le rôle `anon` — fait
- `media_items` : 3 policies redondantes → 1 propre
- Ajout policy explicite `football_teams`
- Anti-patterns `service_role` supprimés

### 3.4 Règles pour les nouvelles tables
- Toute nouvelle table a RLS activé dans la migration (`ALTER TABLE {table} ENABLE ROW LEVEL SECURITY`)
- Les 4 policies (SELECT/INSERT/UPDATE/DELETE) sont créées dans la migration, pas après
- Les tables de référence partagée (données sport non-user) n'ont pas de RLS — accès lecture publique explicitement voulu

---

## 4. Headers HTTP

### 4.1 Headers en place (`next.config.ts`)
```ts
{ key: "X-Frame-Options", value: "DENY" }                          // ✅ clickjacking
{ key: "X-Content-Type-Options", value: "nosniff" }               // ✅ MIME sniffing
{ key: "Referrer-Policy", value: "strict-origin-when-cross-origin" } // ✅ leakage URL
{ key: "Permissions-Policy", value: "camera=(), microphone=(), geolocation=()" } // ✅ APIs navigateur
```

### 4.2 Content Security Policy — ✅ EN PLACE (audit 2026-05-19)
CSP complet dans `next.config.ts` :
- `default-src 'self'`
- `script-src` : self + unsafe-inline (requis Next.js) + Vercel scripts
- `img-src` : self + tous les domaines connus (tmdb, supabase, football-data, lh3.googleusercontent.com…)
- `connect-src` : self + Supabase + Sentry + Vercel
- `worker-src` : self + blob
- `object-src 'none'`, `base-uri 'self'`, `frame-ancestors 'none'`
- `upgrade-insecure-requests` en prod uniquement (pas en dev pour éviter mixed content localhost)

### 4.3 remotePatterns — ✅ EN PLACE (audit 2026-05-19)
Liste explicite dans `next.config.ts`, pas de `**` :
- image.tmdb.org, crests.football-data.org, cdn.thesportsdb.com, r2.thesportsdb.com
- femvhonlpafdajyamvcu.supabase.co
- books.google.com, lh3.googleusercontent.com

---

## 5. Secrets & clés

### 5.1 Variables d'environnement — règles
| Préfixe | Visible côté client | Usage |
|---|---|---|
| `NEXT_PUBLIC_` | ✅ oui — dans le bundle JS | Supabase URL + anon key uniquement |
| aucun préfixe | ❌ non — serveur uniquement | API keys, secrets, service role |

### 5.2 Ce qui ne doit jamais être côté client
- `SUPABASE_SERVICE_ROLE_KEY` — jamais en `NEXT_PUBLIC_`
- `TMDB_API_KEY` — serveur uniquement (déjà corrigé, was `NEXT_PUBLIC_TMDB_API_KEY`)
- `OPENWEATHER_API_KEY` — serveur uniquement
- `UPSTASH_REDIS_REST_TOKEN` — serveur uniquement

### 5.3 Service role key — usage strict

⚠️ **CONTRADICTION NON RÉSOLUE, VÉRIFIÉE DANS LE CODE (audit 2026-07-23) — CLAUDE.md §6 bis fait
autorité, pas ce paragraphe.** Depuis la rotation de clé de juillet (`CLAUDE.md §6 bis`), la variable
`SUPABASE_SERVICE_ROLE_KEY` **auto-injectée** porte la valeur du **legacy JWT mort** — les edge
functions ont migré vers `HEGON_SECRET_KEY` (3 endroits privés : function secrets, Vercel,
`.env.local`). Or **2 routes API Next.js lisent encore `process.env.SUPABASE_SERVICE_ROLE_KEY`**
directement : `src/app/api/tennis/get-api-ids/route.ts` et `src/app/api/tennis/add-player/route.ts` —
zéro usage de `HEGON_SECRET_KEY` trouvé dans `src/`. **Statut réel non tranché ici** : la note de
rotation (`project_supabase_cron_auth`, mémoire) dit que « ajout joueur tennis » a été smoke-testé
**avec succès après** le rotation — donc soit la valeur Vercel de `SUPABASE_SERVICE_ROLE_KEY` a été
tenue à jour en parallèle (ce qui contredirait la lecture stricte de CLAUDE.md §6 bis), soit ces 2
routes tournent aujourd'hui avec une clé morte sans que ça ait été remarqué. **À vérifier par une
exécution réelle avant de faire confiance à l'un ou l'autre paragraphe.**

Règle à suivre en attendant (celle de CLAUDE.md §6 bis, la plus récente et la plus vérifiée) :
- Le bypass RLS ne se fait **jamais** via `SUPABASE_SERVICE_ROLE_KEY` — cette variable auto-injectée
  n'est plus fiable.
- Utiliser `HEGON_SECRET_KEY`, présente dans les 3 endroits privés listés ci-dessus.
- Documenter le pourquoi dans le code à chaque usage (règle inchangée).

---

## 6. Nouveau module — checklist sécurité

### 6.1 Checklist avant de coder
- [ ] Tables avec RLS activé dans la migration
- [ ] 4 policies (SELECT/INSERT/UPDATE/DELETE) dans la migration
- [ ] `org_id` sur toutes les tables (§3.2)

### 6.2 Checklist avant de merger
- [ ] Toute nouvelle route API a auth check + rate limiter + validation
- [ ] Aucune clé secrète en `NEXT_PUBLIC_`
- [ ] Service role key utilisé uniquement si justifié et commenté
- [ ] Nouveaux domaines d'images ajoutés à `remotePatterns` (pas de `**`)
- [ ] Tests d'isolation org écrits et passants — vérifier que deux orgs distinctes ne voient pas les données de l'autre (voir testing guidelines §8.1)

---

## 7. Score actuel et roadmap

### 7.1 État actuel : 9.5/10 (post-audit 2026-05-19)
| Domaine | État |
|---|---|
| Auth check toutes les routes API | ✅ |
| Rate limiting Redis distribué + DB count pour invitations (10/h) | ✅ |
| Validation inputs (regex + UUID + token regex invitation) | ✅ |
| Middleware auth global + defense-in-depth workspace check explicite (§5.2) | ✅ |
| RLS audité et corrigé (3 schémas) + profiles policy stricte (audit §5.1) | ✅ |
| Security headers (5/5) + CSP complet (audit §5.7) | ✅ |
| `remotePatterns` explicites (audit §5.7) | ✅ |
| `createWorkspaceInvitation` owner-only + RLS `org_invitations` (audit §5.3) | ✅ |
| Erreurs génériques (pas de leak Postgres) + console.error pour Sentry (audit §5.5) | ✅ |
| Secrets serveur-side | ✅ |

### 7.2 Reste pour atteindre 10/10
- A11y audit complet (avant prospection LinkedIn)
- Pen test ou audit externe formel
- Logs d'audit DB (qui a fait quoi quand) — partiellement couvert par `task_activities` table (Tasks V2)

### 7.3 FORBIDDEN
- `getSession()` côté serveur pour vérifier l'auth — toujours `getUser()`
- Route API sans auth check
- Route API sans rate limiter (pour les routes exposées)
- `NEXT_PUBLIC_` sur une clé secrète
- `select('*')` avec service role key sur des données user
- RLS désactivé sur une table avec données user
- Nouvelle table sans les 4 policies dans la migration
- `hostname: "**"` dans remotePatterns — lister les domaines explicitement
