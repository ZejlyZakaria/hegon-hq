# HEGON — Anti-patterns interdits

> Liste définitive. Consulter avant code review et avant de coder un nouveau module.
> Si tu t'apprêtes à faire l'une de ces choses, stop.

---

## Code

- `any` sans commentaire de justification explicite
- Query Supabase hors `service.ts` (inline dans composant ou hook)
- `useState` dans un Server Component
- `console.log` commité
- `TODO` sans ticket ou référence
- Composant > 300 lignes
- Fichier > 500 lignes
- Hook avec plus de 5 `useEffect`

---

## Architecture CSR-first

- `supabase.auth.getUser()` dans un Server Component de module — coût +150ms, hors shell uniquement
- Passer `userId` en prop depuis un Server Component pour éviter un hook — utiliser `useCurrentUserId()` dans le wrapper client
- `page.tsx` async pour un module de contenu — par défaut `"use client"` pure
- `createServerClient()` en dehors de : shell layout, pages à redirect racine, API routes
- Import cross-module d'un composant (seuls `service.ts` et `types.ts` peuvent être importés cross-module)
- Appel direct `fetch()` dans un composant — passer par hook ou service
- Zustand pour state serveur — c'est le rôle de TanStack Query
- TanStack Query pour UI state — c'est le rôle de Zustand
- Array inline dans `useQuery` — toujours utiliser les key factories de `query-keys.ts`
- QueryClient créé dans un module — un seul global dans `QueryProvider.tsx`
- **`invalidateQueries({ queryKey: WATCHING_KEYS.all })`** dans une mutation — invalide TOUT le cache watching (15+ queries). Toujours invalider précisément : `detail(id)` + `[...WATCHING_KEYS.all, 'list-items']` au minimum ; ajouter movies/series/animes + filtered queries SEULEMENT si la mutation change un status field (watched, in_progress, want_to_watch, is_reference, recently_watched). Voir `useUpdateMedia.ts` comme référence.

---

## Design

- `bg-zinc-*` hardcodé dans un composant — utiliser les tokens sémantiques (`bg-surface-1`, etc.)
- `text-zinc-*` hardcodé — utiliser `text-text-primary`, `text-text-secondary`, `text-text-tertiary`
- `#ffffff` pur pour le texte — trop de contraste, fatigue visuelle
- Border colorée (accent) sur une card standard
- Gradient coloré sur une card de liste
- `rounded-2xl` sur une card de liste — max `rounded-lg`
- 2 boutons primary côte à côte sur le même écran
- Ring coloré par module au focus input — toujours neutre `border-focus`
- Tabs > 4 par module
- > 3 informations primaires par card
- **Modal :** `bg-[#141416]` sur DialogContent — utiliser `bg-surface-3` (`#1a1a1d`)
- **Modal :** input sans override `bg-surface-overlay` (`#1f1f22`) — les inputs disparaissent visuellement
- **Modal :** bouton primaire `bg-blue-600` hardcodé — toujours `style={{ backgroundColor: ACCENT }}`
- Accent dans le texte secondaire ou les labels — accent = signal d'action/état uniquement
- Shadow externe (`shadow-lg`, `drop-shadow`) sur les cards

---

## Radix — contrainte critique

- **Popover/Portal scrollable dans un Dialog** — jamais.
  - Raison : `react-remove-scroll` intercepte les wheel events hors du DOM Dialog. Portal rend vers `document.body` → vu comme "extérieur" → scroll bloqué.
  - Solution : dropdown inline (div absolute positioned, pas de Portal)

- **Deux containers `overflow-y-auto` empilés** — scroll capturé par le parent, enfant inaccessible.

**Pattern alternatif validé (IconPicker) :**
```tsx
// div relative = container
// button trigger
// div absolute top-[calc(100%+4px)] left-0 right-0 z-50 — pas de Portal
// div natif overflow-y-auto pour la liste scrollable
// useEffect close au click extérieur (mousedown sur document)
// onMouseDown (pas onClick) sur items pour éviter conflit close handler
```

---

## Produit

- Ajouter un module sans valider sa position dans la matrice connexions (⚠️ pas de fichier courant —
  dernière version `hq/archive/modules.md §3`, probablement périmée : juger au cas par cas en
  attendant)
- Créer une colonne DB sans décider si elle a un index
- Route API sans auth check
- Route API sans rate limiting (utiliser le pattern Upstash existant)
- Migration DB qui mélange opérations safe et destructive dans le même fichier
- Merge PR sans test de mutation (Phase 2+ après setup Vitest 25 avril)
- Écrire un spec `v1.md` après avoir commencé à coder — toujours avant

---

## Motion

- Animation > 350ms sur une action utilisateur directe
- Animation sur le scroll (parallax, révélation au scroll)
- Bounce avec damping < 15
- Loop visible (rotation, pulse) — sauf skeleton
- Particules / confetti — sauf célébration streak record annuel
- Animer 2 propriétés simultanément sans raison (ex: scale + color en même temps)
