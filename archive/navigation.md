# HEGON — Navigation System

> Spec complète Phase 2 (actuelle) → Phase 3 (cible OS).
> Validée avec Zakaria. Ne pas modifier sans justification écrite.

---

## 1. Phase 2 — Sidebar (jusqu'au 16 mai)

**Sidebar classique à gauche** (`src/shared/components/layout/Sidebar.tsx`) :
- Logo + modules groupés par section Life/Perso/Pro
- État actif par couleur d'accent du module
- Collapsible (largeur, pas totalement cachée)
- Tooltips sur items collapsed

**Pas de dock en Phase 2.** Raison : spec validée = sidebar Phase 2 → dock Phase 3. Changer 2 fois = churn UX + dette.

**Topbar :**
```
Height       : h-12 (48px)
Bg           : surface-0/80 + backdrop-blur-sm
Border-b     : border-subtle
Padding      : px-5
Position     : sticky top-0 z-sticky
```
- Gauche : date (text-xs text-tertiary) + greeting basé sur l'heure (morning/afternoon/evening)
- Droite : hint ⌘K pill (optionnel) + notifications (caché Phase 2) + avatar menu
- PAS de titre module, PAS de searchbar (⌘K remplace), PAS de stats contextuelles

---

## 2. Phase 3 — Transformation vers Dock

**Transformation progressive :**
1. Introduction des Sliding Panels (spec §4)
2. Dock gauche remplace la sidebar
3. Sidebar disparaît complètement
4. Transitions cross-route via View Transitions + Framer Motion

**Préparation Phase 2 (sans activer) :**
- Écrire `shared/components/navigation/Dock.tsx` en feature flag off
- URLs propres pour deep linking (déjà en place grâce aux sections)

---

## 3. Dock — spec (Phase 3)

```
Position       : gauche, vertical, fixe
Largeur        : 56px (collapsed only — pas d'état expanded)
Hauteur        : 100vh
Background     : surface-1 (#0e0e10)
Z-index        : z-sticky (10)
Item size      : w-10 h-10, radius rounded-lg, icon 20px
Active         : bg surface-2, icon couleur accent
Tooltip        : droite, delay 500ms
```

Max 12 modules visibles. Au-delà → séparateur + scroll.

**Pas d'état expanded** — tooltips suffisent. Sections Life/Perso/Pro invisibles visuellement, URLs conservées.

---

## 4. Sliding Panels — spec (Phase 3 cible)

```
Position       : right slide-in
Width          : 75-80% viewport
Animation      : < 280ms, spring.gentle
Behavior       : contexte précédent reste visible à gauche (réduit + blur optionnel)
Close          : ESC ou bouton "Back to [Contexte]"
Stack rule     : jamais 2 panels ouverts simultanément — nouveau ⌘K remplace l'actuel
```

**Niveaux par module :**
- **Niveau 1 High-level** : Life Compass, Week view, This week's matches...
- **Niveau 2 Deep-dive** : fiche détail, tâches liées...

**⌘K → Sliding Panel :**
- "Show tasks next week" → panel Tasks filtré semaine suivante
- "Progress on [goal]" → panel Goals avec ce goal en détail
- "Today's match" → panel Sports avec fiche du match

**Dock quand panel ouvert :**
- Reste visible gauche, position fixe
- Icône module actif = highlight accent + barre verticale gauche
- Clic autre icône → remplace le panel
- Clic même icône → ferme panel, retour contexte précédent

---

## 5. ⌘K Command Palette — spec (Phase 2 : 12-16 mai)

```
Raccourci        : ⌘K / Ctrl+K
Taille           : max-w-xl (576px), max-h-[500px]
Position         : fixed top-[20vh] centrée
Overlay          : bg black/60 + backdrop-blur-sm
Background       : surface-3 (#1a1a1d)
Border           : border-strong
Radius           : rounded-xl
Z-index          : z-modal (30)
```

**4 modes :** Actions | Navigation | Recherche | Connexions (contextuel)

**Ranking déterministe (pas d'IA) :**
1. Commandes directes (match exact action ou module)
2. Items récemment consultés (Zustand + localStorage)
3. ILIKE %query% sur `title`
4. Connexions si opened from an item

**Performance :** prefetch recent au mount · debounce 150ms · staleTime 5min · offline = utilise cache.

**Shortcuts :** ↑↓ nav · Enter select · ESC close · Tab change mode.

---

## 6. Slide-Over — spec (Phase 2)

```
Position       : droite plein écran vertical
Largeur        : 480px (sm:max-w-lg)
Background     : surface-2 (#141416)
Border left    : border-strong
Animation      : spring.gentle
Z-index        : z-modal (30)
Overlay        : bg black/40 + backdrop-blur-[2px]
Close          : clic overlay, ESC, bouton X
Structure      : Header (h-14 border-b) + Content scrollable (gap-4) + Footer (h-14 border-t)
```

**Distinct de Sliding Panel** (Phase 3, 75-80% width). Slide-over = édition d'un item depuis sa liste, 480px fixe.

### Slide-Over vs Detail Page — règle de décision

| Critère | Slide-Over | Detail Page |
|---------|------------|-------------|
| URL shareable | non | **oui** |
| Durée d'interaction | < 30s | > 30s |
| Navigation interne (tabs/sub-views) | non | **oui** |
| Reste dans la liste | **oui** | non |

**Application :**
- Habits, Tasks, Watching, Sports → **Slide-Over** (édition rapide, reste dans le flow)
- Goals, Journal, Books, Tech, Culture, Travel → **Detail Page** (contenu long, URL shareable)

**Taille contenu → composant :**

| Contenu | Composant |
|---------|-----------|
| 1-3 champs | Modal |
| 4-10 champs | Slide-over |
| 10+ ou URL shareable | Detail page |

---

## 7. Side Panel contextuel — spec (Phase 3)

```
Position       : droite 360px
Background     : surface-2
Z-index        : z-sticky (10) — derrière modals
Close          : clic extérieur, ESC, bouton X
```

Jamais plus d'un side panel ouvert. Sur demande uniquement via bouton "Connections" sur une detail page.

---

## 8. Modal — spec

> Source de vérité complète dans `hq/design-system.md` §4.5 (pattern HabitModal).

```
Overlay    : bg black/60, backdrop-blur-sm
Container  : bg surface-3 (#1a1a1d), border border-strong, rounded-xl
Width      : max-w-md (simple) / max-w-lg (formulaire complet)
Z-index    : z-modal (30)
```

Structure obligatoire : Header + Divider + Content + Divider + Footer (Cancel + Confirm)
