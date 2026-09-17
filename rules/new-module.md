# HEGON — Module Creation Checklist

> Checklist opérationnelle à utiliser pour chaque nouveau module.
> Ne pas dupliquer le contenu des guidelines — consulter les sections référencées pour le détail.
> Un module ne passe pas en V1 tant que toutes les cases ne sont pas cochées.
>
> **Règle tests sur les modules existants :** on teste ce qu'on touche. Si un module existant est modifié (migration CSR, design system retrofit, bugfix), ses tests s'écrivent dans la même session. On ne revient pas en arrière juste pour tester du code stable.

---

## Avant de coder

### Base de données
- [ ] Tables créées avec `org_id` dès le départ — jamais mono-user *(master_plan §1)*
- [ ] `ALTER TABLE {table} ENABLE ROW LEVEL SECURITY` dans la migration
- [ ] 4 policies RLS (SELECT / INSERT / UPDATE / DELETE) dans la migration *(security §3.2)*
- [ ] Index sur `org_id`, `user_id`, `status`, dates filtrées, clés étrangères *(performance §4.1)*
- [ ] Convention de nommage des index : `idx_{table}_{colonne}` *(performance §4.3)*

### Service layer
- [ ] `service.ts` avec toutes les queries Supabase — zéro query inline *(master_plan §3)*
- [ ] `Promise.all` pour les fetches parallèles — jamais de waterfall *(performance §3.1)*
- [ ] `select('id, col1, col2...')` explicite — pas de `select('*')` sur tables larges *(performance §3.2)*
- [ ] `.limit()` sur toutes les tables susceptibles de croître *(performance §3.3)*

### TanStack Query
- [ ] `query-keys.ts` avec les key factories *(CLAUDE.md)*
- [ ] `staleTime` choisi depuis la convention et commenté *(performance §2.2)*
- [ ] `onError: toast.error(...)` sur chaque mutation *(master_plan règle 4)*
- [ ] Pas de QueryClient séparé — utiliser le global *(performance §2.6)*

### Routes & composants
- [ ] `loading.tsx` créé pour la route *(performance §1.4)*
- [ ] Sections indépendantes dans des `<Suspense>` séparés avec `suspense: true` *(performance §1.4)*
- [ ] `"use client"` uniquement si interaction, hook TQ, ou Framer Motion *(performance §5.3)*
- [ ] `next/image` pour toutes les images — jamais de `<img>` natif *(performance §5.1)*

---

## Avant de merger

### Sécurité
- [ ] Toute nouvelle route API a auth check (`getUser()`) + rate limiter + validation input *(security §1)*
- [ ] Nouveaux domaines d'images ajoutés à `remotePatterns` — pas de `**` *(security §4.3)*
- [ ] Aucune clé secrète en `NEXT_PUBLIC_` *(security §5.1)*
- [ ] Service role key utilisé uniquement si justifié et commenté dans le code *(security §5.3)*

### Tests
- [ ] Tests utilitaires pour toutes les fonctions pures du module *(testing §2.1)*
- [ ] Tests service pour les mutations critiques (create, update, delete) *(testing §2.2)*
- [ ] Test d'isolation org — deux orgs distinctes ne voient pas les données de l'autre *(testing §2.2)*
- [ ] Tests API route : 401 sans auth, 400 input invalide, 429 rate limit *(testing §2.3)*
- [ ] Au moins 1 test E2E pour le chemin le plus utilisé *(testing §2.4)*

### Design
- [ ] Accent couleur respecté depuis la table des modules *(design-system §1.4)*
- [ ] Composants (cards, badges, modals) conformes au design system *(design-system §4)*
- [ ] Empty state explicite — pas d'écran vide sans message *(master_plan règle 3)*
- [ ] Skeleton fidèle à la forme du vrai contenu *(performance §1.4)*

### Qualité
- [ ] `npm run lint` passe sans erreur
- [ ] `tsc --noEmit` passe sans erreur
- [ ] `npm run build` passe sans erreur
- [ ] Migration SQL versionnée dans `supabase/migrations/` *(blueprint §7.7 — fichier gardé pour la
  traçabilité, jamais rejoué par `db push`, voir [[reference_migrations_applied_by_hand]])*
- [ ] `hq/now.md` mis à jour si le focus a changé (le récit vit dans `git log`)

---

## Références rapides

| Sujet | Fichier |
|---|---|
| Architecture & règles fondamentales | `hq/blueprint.md` |
| Design system complet | `hq/rules/design-system.md` |
| Performance & TQ | `hq/rules/performance.md` |
| Sécurité & RLS | `hq/rules/security.md` |
| Tests & CI | `hq/rules/testing.md` |
| Modules, connexions & DB | ⚠️ pas de fichier courant — `hq/archive/modules.md` (probablement périmé) |
| Avancement | `git log` + `hq/now.md` |
| Invariants + décisions d'un module existant | `hq/modules/{module}/architecture.md` + `decisions.md` |
| Ce qui reste sur un module existant | `hq/modules/{module}/backlog.md` (chaque ligne avec son déclencheur) |
| Specs futures (module pas encore construit) | `hq/futur/` |
| Historique audits | `hq/modules/{module}/audit-{date}.md` |
