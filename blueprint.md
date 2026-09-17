# HEGON — Blueprint (La Loi)

> Ce document contient les règles fondamentales et les décisions structurantes.
> Il pointe vers des fichiers spécialisés pour le détail.
> Toute décision future doit être cohérente avec ce document.
> Mis à jour : 2026-09-17 (les §1 et §4 datés sont sortis — l'état vit dans `now.md`, la stack dans `AGENTS.md`)

---

## Table des matières

| Sujet | Fichier |
|-------|---------|
| **Workflow & règles de collaboration** | **`hq/rules/workflow.md`** ← lire en début de session |
| Avancement | `git log` (un commit = une ligne, le quoi et le pourquoi) · `hq/now.md` pour l'état |
| Design system complet + Motion system | `hq/rules/design-system.md` + `hq/tokens.css` |
| Architecture CSR + patterns code + DB perf | `hq/rules/performance.md` |
| Modules + connexions + graphe + tables DB | ⚠️ pas de fichier courant — dernière version archivée `hq/archive/modules.md` (probablement périmée) |
| Navigation + ⌘K + Sliding Panels | ⚠️ pas de fichier courant — dernière version archivée `hq/archive/navigation.md` (probablement périmée) |
| Anti-patterns interdits | `hq/rules/anti-patterns.md` |
| Checklist module (à cocher avant chaque module) | `hq/rules/new-module.md` |
| Sécurité + RLS | `hq/rules/security.md` |
| Tests + CI | `hq/rules/testing.md` |
| Maquettes OS par module | `hq/designs/` |
| Invariants + décisions + backlog d'un module existant | `hq/modules/{module}/{architecture,decisions,backlog,vnext}.md` |
| Specs futures (module pas encore construit) | `hq/futur/{module}-v1.md` |
| L'ordre du travail et les déclencheurs | `hq/roadmap.md` |

**Avant de coder un module V1 :** lire `hq/rules/new-module.md` + la spec `hq/futur/{module}-v1.md` si elle existe.
**Avant une prochaine version d'un module existant :** lire `hq/modules/{module}/backlog.md` (ce qui reste, avec déclencheurs) puis `decisions.md`.

---

## 1. État actuel

> L'état courant vit dans `hq/now.md` (court, réécrit à chaque jalon) ; la stack et les règles
> opérationnelles dans `AGENTS.md` / `CLAUDE.md` à la racine. Ce fichier ne porte que la loi.

## 2. Positionnement produit

> ⚠️ **Recadré le 2026-09-07 — lire `hq/product-strategy.md`.** Ce paragraphe décrit HEGON comme de la
> **gestion de la vie** (cadrage productivité). L'origine réelle du projet est une **collection** — la
> même obsession que les cartes Yu-Gi-Oh et le maillot du joueur favori, avec pour intention d'ouvrir
> l'app dans 20 ans et de montrer à ses enfants ce qu'on a vécu. Ce n'est pas un détail de vocabulaire :
> collection et productivité sont deux marchés opposés (l'une accumule, l'autre vide), et le
> différenciateur défendable de HEGON est **l'agrégation transversale**, pas la gestion.

**Application web premium personnelle qui centralise la gestion de la vie** — objectifs, habitudes, tâches, connaissances, culture, sport, voyage, lecture, visionnage, journal.

Elle vise le **niveau de cohérence et de sensation d'un OS** (Linear, Superhuman, Things 3, Craft) mais elle tourne dans un navigateur et assume sa nature d'app web.

**Ce que HEGON n'est pas :** pas un OS système · pas un Notion (modules opinionnés) · pas un Zapier (automation dirigée Phase 4) · architecture multi-tenant prête, usage single user (Zakaria).

**North star :** *"Tu ouvres HEGON et tu oublies que c'est une app web. Tu as la sensation d'un espace cohérent, calme, réactif."*

**Benchmarks :** cohérence visuelle = Linear · densité lisible = Superhuman · fluidité = Things 3/Craft · réactivité = Linear/Raycast · command palette = Raycast/Superhuman

---

## 3. Les 10 règles qui gouvernent tout

1. **Une seule application, pas une collection de modules.** Tout doit se sentir comme une continuation du même espace.
2. **Clarity over decoration.** Un élément qui n'aide pas à comprendre ou à agir n'a pas sa place.
3. **Density over whitespace.** Beaucoup d'informations dans peu d'espace — mais chaque élément parfaitement lisible.
4. **Predictability over surprise.** L'utilisateur ne doit jamais se demander comment un élément va se comporter.
5. **Accent is a signal, not decoration.** La couleur accent communique une action ou un état.
6. **Motion clarifies, never distracts.** Animation = aide à comprendre ou supprimée.
7. **Stillness is a choice.** Ne pas animer est une décision active.
8. **Client-first avec TanStack Query.** Server Components uniquement pour shell et pages à `auth check + redirect` racine. Raison mesurée : `getUser()` serveur = +150ms/render.
9. **Service layer is the law.** Zéro query Supabase en dehors de `service.ts`.
10. **No hypothetical engineering.** Pas de construction pour cas imaginés.

### Les 4 piliers

| Pilier | Fichier de référence |
|--------|---------------------|
| Design system | `hq/rules/design-system.md` + `hq/tokens.css` |
| Performance | `hq/rules/performance.md` |
| Sécurité | `hq/rules/security.md` |
| Tests | `hq/rules/testing.md` |

---

## 4. Stack

> Voir `AGENTS.md` §Stack (la vérité au jour le jour). Explicitement exclu, et ça ne bouge pas :
> Redux/MobX · Zod runtime global · tRPC · CSS-in-JS · Service Worker custom.

## 5. Les 10 signaux "OS feel"

1. **Chrome persistant** — Sidebar/Dock + Topbar toujours visibles, layout `(main)` ne re-render pas entre pages.
2. **⌘K universel** — Raccourci partout, < 50ms avant affichage, navigation clavier complète.
3. **CSR + optimistic UI** — Aucune page utilisateur ne bloque sur `getUser()` serveur. Mutations optimistic intra-module, invalidation cross-module.
4. **Transitions sans flash** — Phase 2 : navigation layout persistent. Phase 3 : Sliding Panels avec contexte préservé.
5. **Feedback physique** — Scale 0.97 au press, lift au hover, tokens Framer Motion spring.
6. **Raccourcis clavier réels** — ⌘K · `N` create · `?` overlay shortcuts (Phase 3 : `G` + lettre Gmail-style).
7. **État persistant** — Zustand + persist : org active, filtres, sidebar collapsed. User retrouve son contexte exact.
8. **Zéro spinner** — Skeletons fidèles au contenu réel, jamais "Loading…".
9. **Consistency absolue des états** — `completed` = opacity-50 + line-through partout. `overdue` = amber partout. Sans exception.
10. **Typographie signature** — Inter UI + Geist Mono code + Caveat/Kalam Journal éditeur uniquement.

---

## 6. Décisions structurantes

| Décision | Choix |
|----------|-------|
| Mobile | Desktop-first responsive dégradé. Pas de PWA V1. Dock bottom si mobile Phase 3+. |
| Offline | Online-first. TQ cache 5min fallback. Mutations offline bloquées. |
| Multi-tenant | **DONE.** 1 user = 1 org. Partage optionnel Phase 3+ (invitation + rôle). |
| Access Control | Post-Journal V1. Approbation manuelle Zakaria. 6 étapes documentées. |
| Notifications | In-app only. Toast + badge dock Phase 3+. Pas d'email/push. |
| Search Phase 2 | ILIKE sur title via ⌘K. |
| Search Phase 3 | Postgres tsvector + GIN index sur title + content. |
| Search Phase 5 | pgvector sémantique en complément. |
| APIs externes | Cache 24h via tables `cached_external`. Rate limit 100 req/min/user. |
| Automation | DSL + UI builder Phase 4. Réévaluer si < 10 règles créées après livraison. |
| Dock overflow | Max 12 modules visibles. Au-delà → séparateur + scroll. |
| Sections URL | Life/Perso/Pro conservées dans URLs Phase 2+. Invisibles visuellement Phase 3+. |
| Table connections | FKs directes Phase 2. Table générique Phase 3-4 si graphe dense. |
| Auth | Whitelist Supabase obligatoire. Finalize flow. Ne pas toucher sans raison forte. |
| Automation niveau | DSL + UI builder. Réévaluer seulement si < 10 règles après Phase 4. |
| Phase 6 | V2 tous modules en bloc après Phase 5. Pas de scission par principe. |

---

## 7. Glossaire

| Terme | Définition |
|-------|-----------|
| Module | Feature self-contained : `service.ts` + `types.ts` + `hooks/` + `components/` |
| CSR-first | `page.tsx` = `"use client"` pure, fetch via TanStack, auth via `useCurrentUserId()` |
| Shell | `app/(main)/layout.tsx` + Sidebar (Phase 2) ou Dock (Phase 3) + Topbar — seule partie Server |
| Master hook | Hook "page" qui fetch toutes les données du module en parallèle (Promise.all interne) |
| Accent | Couleur hex unique par module — signal d'action/état, pas décoration |
| Right panel | Colonne ~35%, contenu statique stats/contexte (Tracker dense) |
| Sliding Panel | Overlay droite 75-80% viewport, contexte préservé gauche, Phase 3 |
| Side Panel | Overlay droite 360px, item connections sur demande, Phase 3 |
| Slide-Over | Overlay droite 480px, édition rapide d'un item (Phase 2) |
| Detail Page | Route `/module/[id]`, URL shareable, contenu long (Goals, Journal, Books, Tech) |
| Command palette | ⌘K — 4 modes : Actions + Navigation + Recherche + Connexions |
| Dock | Barre verticale gauche 56px, Phase 3 (remplace Sidebar) |
| Connection | FK directe (Phase 2) ou table `connections` (Phase 3-4 si besoin) |
| Préférence | Table avec `org_id` — données créées par l'user |
| Référence | Table sans `org_id` — données globales partagées (edge functions) |
| Access Control | Approbation manuelle comptes par Zakaria via email (post-Journal V1) |

---

*La loi. Toute décision qui contredit ce document doit d'abord le modifier.*
*Mis à jour : 2026-04-24 — restructuration hq/ flat + contenu distribué par fichier spécialisé*
