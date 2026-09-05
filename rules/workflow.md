# HEGON — Workflow & Règles de collaboration

> Règles de processus entre Zakaria et Claude.
> À lire en début de conversation sur un nouveau module ou une nouvelle feature.
> Ces règles ne se répètent pas — elles s'appliquent silencieusement.

---

## 1. Ordre de création d'un module

Toujours dans cet ordre, sans sauter d'étape :

1. **Spec** `hq/futur/{module}-v1.md` — validée et alignée avec l'OS vision (⚠️ pas de fichier courant pour la matrice des connexions cross-module — dernière version `hq/archive/modules.md`, probablement périmée)
2. **Migration** `supabase/migrations/YYYYMMDD_phase_{module}.sql`
3. **`types.ts`** — DB types, input types, UI types
4. **`service.ts`** — toutes les queries Supabase
5. **`hooks/query-keys.ts`** + **`hooks/use{Module}.ts`** — TanStack Query
6. **Composants UI** — générés par Sonnet 4.5 sur claude.ai, vérifiés par Claude Code
7. **Pages** `src/app/(main)/.../page.tsx` + `loading.tsx`
8. **Tests** — après les composants, avant de marquer V1 done

---

## 2. Frontend — qui fait quoi

**Composants UI** (BookCard, AddBookModal, BooksRightPanel, etc.) :
- Générés par **Sonnet 4.5 sur claude.ai** avec le blueprint + design-system + spec module en contexte
- Claude Code **vérifie et corrige** : design system, types, hooks, logique

**Claude Code fait directement** : migration, types, service, hooks, API routes, tests, debugging, architecture, design system retrofit, reviews

---

## 3. Migration SQL

⛔ **`npx supabase db push` : JAMAIS sur ce projet** — l'owner applique tout par copier-coller dans
l'éditeur SQL Supabase. `schema_migrations` ne connaît aucune des migrations existantes ⇒ un push les
rejouerait toutes. Le fichier de migration reste obligatoire (archive/traçabilité), mais **il n'est
jamais exécuté par la CLI**. Voir [[reference_migrations_applied_by_hand]].

- Fichier : `supabase/migrations/YYYYMMDD_phase_{description}.sql`
- Structure obligatoire : TABLE → TRIGGER updated_at → RLS → INDEXES
- RLS : policy `"org_isolation"` avec `my_orgs()` helper — voir journal migration comme référence
- Index : `idx_{table}_{colonne}` sur `org_id`, `user_id`, `status`, dates filtrées, FKs
- Jamais de `select *` sur tables larges dans le service — colonnes explicites
- Vérifier les FKs cross-module à ajouter (pas de fichier courant de matrice — juger au cas par cas)

---

## 4. Validation avant de coder

Avant tout nouveau module, lire dans cet ordre :
1. `hq/rules/new-module.md` — cases à cocher avant/après
2. Spec du module :
   - Module existant → `hq/modules/{module}/architecture.md` + `decisions.md`
   - Module futur (pas encore construit) → `hq/futur/{module}-v1.md`
3. `hq/designs/` si le module a des maquettes (OS vision)

---

## 5. Tests — obligatoires avant V1 done

Référence complète : `hq/rules/testing.md`

Ce qui doit exister pour chaque module :
- Tests utils pour toutes les fonctions pures (`*-utils.ts`)
- Tests service pour mutations critiques (create, update, delete) + isolation org
- Tests API route : 401 sans auth · 400 input invalide · 429 rate limit
- 1 test E2E pour le chemin le plus utilisé

Règle absolue : **jamais mocker Supabase dans les tests service** — vraie DB locale uniquement.

---

## 6. Git

- **Jamais exécuter de commande git** — donner la commande à l'utilisateur, attendre le résultat
- Format commit : **une seule ligne** → `type(scope): description courte`
- **JAMAIS** de `Co-Authored-By`, mention de Claude, d'Anthropic, ou de tout signe d'IA
- Types : `feat` · `fix` · `refactor` · `test` · `docs` · `chore`

---

## 7. Design

- Lire `hq/rules/design-system.md` avant tout composant
- Challenger chaque décision UI contre Linear / Apple / Vercel Dashboard
- Barre section (couleur + uppercase + ligne) : **interdite à l'intérieur d'un composant**
- Titres dans les panels : `text-xs font-semibold text-text-secondary` — jamais `text-sm`, jamais `text-primary`
- `next/image` toujours — jamais `<img>` natif
- `loading.tsx` obligatoire pour chaque route de module

---

## 8. Challenger systématiquement

Aucune décision de design, d'architecture ou de produit ne passe sans être questionnée.
Question à poser avant d'implémenter : *"Est-ce que ça existe dans Linear, Apple, Vercel ?"*
L'utilisateur a le dernier mot — mais il doit d'abord entendre l'objection.

---

## 9. Suggérer claude.ai quand pertinent

| Tâche | Outil |
|-------|-------|
| Génération de composants UI | claude.ai (Sonnet 4.5) |
| Prototypage visuel | claude.ai (Claude Design) |
| Questions sans lien avec le code | claude.ai chat |
| Architecture · review · debug · migration · tests | Claude Code |
