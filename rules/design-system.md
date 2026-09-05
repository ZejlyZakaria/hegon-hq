# HEGON Design System

> Source de vérité pour tous les décisions visuelles et d'interaction.
> Standard : Linear / GitHub / Vercel Dashboard.
> Principe fondateur : **high density + high clarity**.

---

## Table des matières

### 0. Design Principles
### Decision Rules
### Motion Restrictions
### Cross-Module Hints
### Information Priority by Module

### 1. Fondations
- 1.1 Surfaces (bg levels)
- 1.2 Texte (text levels)
- 1.3 Borders
- 1.4 Accent colors (règles strictes par module)
- 1.5 Spacing scale
- 1.6 Typography scale
- 1.7 Border radius tokens
- 1.8 Z-index scale
- 1.9 Motion tokens

### 2. Système d'états
- 2.1 Active
- 2.2 Completed
- 2.3 At-risk
- 2.4 Overdue
- 2.5 Archived
- 2.6 Disabled

### 3. Système d'interaction
- 3.1 Hover
- 3.2 Press
- 3.3 Focus
- 3.4 Disabled

### 4. Composants
- 4.1 Boutons (primary, secondary, ghost)
- 4.2 Inputs & Textarea
- 4.3 Cards
- 4.4 Badges & Tags
- 4.5 Modals
- 4.6 Tabs
- 4.7 Tooltips
- 4.8 Toasts
- 4.9 Skeletons
- 4.10 Empty states
- 4.11 Entity links

### 5. Layout
- 5.1 Topbar
- 5.2 Module header
- 5.3 Left / Right panel split
- 5.4 CTA placement

### 6. Navigation
- 6.1 Tabs (underline style)
- 6.2 Sidebar

### 7. Patterns spécifiques
- 7.1 Système de warning
- 7.2 Milestones / étapes
- 7.3 Dates relatives
- 7.4 Icon sizes
- 7.5 Scrollbars
- 7.6 Formulaires (select, checkbox, toggle)

### 8. Règles globales
- 8.1 Information density (max par card)
- 8.2 Hiérarchie visuelle (1 dominant / 2 secondary)
- 8.3 Motion (functional only)
- 8.4 Ce qui est identique partout
- 8.5 Ce qui peut varier par module
- 8.6 Exceptions intentionnelles
- 8.7 FORBIDDEN

---

## 0. Design Principles

Ces principes sont la loi philosophique derrière chaque règle du système.
Quand une décision est ambiguë, revenir ici pour trancher.

1. **Clarity over decoration** — si un élément n'aide pas l'utilisateur à comprendre ou agir, il n'a pas sa place
2. **Density over whitespace** — beaucoup d'informations dans peu d'espace, mais chaque élément parfaitement lisible
3. **One system, not multiple tools** — HEGON doit se sentir comme un seul produit cohérent, pas une collection de modules
4. **Predictability over surprise** — l'utilisateur ne doit jamais se demander comment quelque chose va se comporter
5. **Accent is a signal, not decoration** — la couleur accent communique une action ou un état, elle ne décore pas
6. **Every element earns its place** — si tu ne peux pas justifier pourquoi un élément existe, il n'existe pas
7. **Motion must clarify, never distract** — une animation qui n'aide pas à comprendre est une animation à supprimer
8. **Stillness is a choice, not an absence** — ne pas animer est une décision active, pas un oubli

---

## Decision Rules

Ces règles définissent quand choisir un pattern plutôt qu'un autre.
Sans elles, chaque décision redevient subjective.

### Card vs List
- **Card** → contenu mixte, scanning visuel, entités avec plusieurs attributs visibles
- **List** → flux opérationnel dense, entités répétées et homogènes

### Modal vs Panel
- **Modal** → création, confirmation, action destructive — focus complet requis
- **Panel** → exploration contextuelle, inspection sans quitter l'écran principal

### Badge vs Metadata
- **Badge** → état ou type uniquement (completed, at-risk, overdue, archived)
- **Metadata** → information descriptive supportante (date, lien, compteur)

### Primary CTA vs Secondary
- **Primary** → uniquement si l'écran a une action dominante évidente
- **Secondary ou Ghost** → tout le reste — par défaut, douter = ghost

### Accent Conflict Rule
Quand plusieurs modules sont visibles sur le même écran (ex: Dashboard) :
- L'accent du module actif/hôte gagne
- Les modules embarqués ou liés utilisent leur accent uniquement sur les éléments interactifs directs
- Deux accents ne peuvent jamais apparaître à égale intensité visuelle sur le même écran

---

## Motion Restrictions

Zones où l'animation est **interdite** même si techniquement possible :

- Listes de plus de 10 items — pas de stagger, apparition directe
- Rafraîchissement automatique de données (polling, real-time)
- Containers scrollables — pas d'animation sur le scroll
- Mises à jour de métriques passives (compteurs qui changent en arrière-plan)
- Éléments décoratifs de fond

**Pourquoi :** un produit premium ne se distingue pas parce qu'il anime plus — il se distingue parce qu'il sait quand ne pas animer. La densité d'information d'HEGON rend cette règle critique.

---

## Cross-Module Hints

HEGON est un OS personnel — les modules ne sont pas des apps séparées, ils partagent des entités et des états. Ces règles définissent comment cette connexion se reflète visuellement.

- Les entity links ont toujours le même style peu importe le module source (`Link2 10px text-tertiary`)
- Le sens d'un état est identique partout — `at-risk` est toujours amber, `completed` est toujours opacity-50, dans tous les modules sans exception
- Si un Goal devient at-risk, ses Habits liés affichent un signal amber subtil sur leur entity link — pas de changement de la card elle-même
- Si un Goal est completed, ses Habits liés affichent leur entity link en `text-tertiary` strié — indication passive que le contexte est résolu
- L'accent du module hôte gagne toujours quand deux modules coexistent visuellement (règle Accent Conflict, voir Decision Rules)

---

## Information Priority by Module

Chaque module définit une hiérarchie fixe. Cette hiérarchie ne change pas selon la vue ou le contexte — elle est stable et prévisible.

| Module  | Primary                        | Secondary                    | Tertiary                  |
|---------|--------------------------------|------------------------------|---------------------------|
| Goals   | Progress vers l'objectif       | Deadline / état              | Milestones count, linked habits |
| Habits  | Statut complétion du jour      | Streak actuel                | Goal lié, fréquence       |
| Journal | Contenu + date                 | Mood / tags                  | Streak, word count        |
| Tasks   | Priorité + deadline            | Statut                       | Projet lié, assignee      |
| Watching | Ton état & ton verdict (statut, position S·E, rating) | Ta mémoire (take, années/History, rewatches, best eps) | Catalogue (cast, providers, notes externes, details, recos) |

**Règle :** si un élément n'est pas dans la colonne Primary d'un module, il ne peut pas être mis en `text-primary` ou occuper la position visuelle dominante de la card.

**Règle d'ordre (2026-07-08) :** dans une page de détail, un contenu Tertiary ne passe **jamais** au-dessus d'un Secondary visible. (Faute type corrigée sur Watching : Cast & Crew au-dessus de Watch History.)

---

## 1. Fondations

### 1.1 Surfaces

**Principe :** la border définit la carte. Le background ajoute la profondeur. Jamais de saut brutal entre niveaux.

```css
--surface-0:       #09090b;   /* App background (zinc-950) */
--surface-1:       #0e0e10;   /* Cards — lift subtil */
--surface-2:       #141416;   /* Hover / état actif */
--surface-3:       #1a1a1d;   /* Modals, dropdowns */
--surface-overlay: #1f1f22;   /* Éléments imbriqués dans modals */
```

**Borders associées :**
```css
--border-subtle:  rgba(255, 255, 255, 0.04);  /* Cards sur surface-0 */
--border-default: rgba(255, 255, 255, 0.07);  /* Séparateurs standard */
--border-strong:  rgba(255, 255, 255, 0.11);  /* Emphasis, focus visible */
```

**Règles d'usage :**
- `surface-0` → background de l'app uniquement, jamais sur un composant
- `surface-1` → toutes les cards de contenu (habits, goals, tasks...)
- `surface-2` → état hover des cards, éléments actifs/sélectionnés
- `surface-3` → modals, dropdowns, popovers
- `surface-overlay` → inputs dans modals, nested cards dans modals

**Ce qui est INTERDIT :**
- `bg-zinc-900/80` — le saut est trop grand, donne un effet "lumineux" sur fond très sombre
- `bg-zinc-950` sur une card — invisible, aucune séparation
- Backgrounds avec opacity variable (`/60`, `/40`) — imprévisibles selon le contexte
- Gradients colorés sur les cards — réservé aux éléments hero uniquement

---

### 1.2 Texte

```css
--text-primary:   #e2e2e6;   /* Titres, contenu principal */
--text-secondary: #a0a0a8;   /* Descriptions, info supporting */
--text-tertiary:  #71717a;   /* Labels, timestamps, metadata — zinc-500, ratio ~3.8:1 */
--text-disabled:  #3d3d44;   /* Disabled, archivé */
```

**Règles d'usage :**
- `text-primary` → titres de cards, noms d'éléments, valeurs importantes
- `text-secondary` → descriptions, sous-titres, texte de body
- `text-tertiary` → labels de section, timestamps, compteurs, metadata
- `text-disabled` → éléments désactivés, placeholders, archivés

**Ce qui est INTERDIT :**
- `#ffffff` — trop de contraste sur fond très sombre, fatigue visuelle
- `text-zinc-*` hardcodé dans les composants — utiliser les variables
- Mélanger les niveaux au hasard (ex: `text-zinc-300` pour une description alors que c'est du secondaire)

---

### 1.3 Borders

```css
--border-subtle:  rgba(255, 255, 255, 0.04);  /* Cards sur surface-0 */
--border-default: rgba(255, 255, 255, 0.07);  /* Séparateurs standard */
--border-strong:  rgba(255, 255, 255, 0.11);  /* Emphasis, topbar, modals */
--border-focus:   rgba(255, 255, 255, 0.20);  /* Focus ring neutre sur inputs */
```

**Règles d'usage :**
- `border-subtle` → cards de contenu, sections internes
- `border-default` → séparateurs entre éléments, dividers
- `border-strong` → topbar, header de modal, éléments importants
- `border-focus` → ring sur input/textarea au focus — neutre, jamais coloré par module

**Ce qui est INTERDIT :**
- Borders colorées (accent) sur les cards — jamais
- Border rouge pour les warnings — utiliser le système d'états (section 2)
- `border-zinc-*` hardcodé — utiliser les variables
- Border à opacity variable selon l'humeur (`/60`, `/40`, `/30`) — choisir subtle/default/strong

---

### 1.4 Accent colors

**Couleurs par module :**
```css
--accent-goals:   #22c55e;   /* Green */
--accent-habits:  #f43f5e;   /* Rose */
--accent-journal: #f97316;   /* Orange — contraste blanc suffisant, famille chaude */
--accent-tasks:   #71717a;   /* Zinc */
--accent-watching:#8b5cf6;   /* Violet */
--accent-sports:  #10b981;   /* Emerald */
```

**Zones autorisées — et uniquement celles-ci :**
1. Bouton CTA primaire (background)
2. Tab indicator actif (underline)
3. Progress bar / progress ring
4. Icône du module dans la sidebar (état actif)
5. Streak / stat valeur importante dans une card

**Ce qui est INTERDIT :**
- Background de card avec la couleur accent
- Border colorée sur une card
- Badge avec background coloré
- Gradient coloré sur une section entière
- Deux éléments accentués sur la même card
- Utiliser l'accent pour du texte secondaire ou des labels

**Règle des 5% :**
La couleur accent doit occuper moins de 5% de la surface visible d'un écran.
Si tu vois plus de rose/vert sur un écran Habits/Goals, c'est trop.

**Accents manquants — à ajouter dans `hq/tokens.css`** (modules spécifiés, pas encore dans le CSS) :
```css
--color-accent-dashboard: #60a5fa;   /* Blue */
--color-accent-books:     #0ea5e9;   /* Sky — pas Orange, pas de collision avec Journal */
--color-accent-tech:      #a855f7;   /* Purple */
--color-accent-culture:   #14b8a6;   /* Teal */
/* À valider : Travel = Indigo #6366f1 (Sky pris par Books) · Fitness = Lime #84cc16 · Job Hunt = Yellow #eab308 */
```

---

### 1.5 Spacing scale

**Scale autorisée (en px, via Tailwind) :**
```
4px  → gap-1, p-1   (micro — séparateurs internes)
8px  → gap-2, p-2   (small — entre éléments dans une card)
12px → gap-3, p-3   (base — padding card, gap liste)
16px → gap-4, p-4   (medium — padding section, topbar)
20px → gap-5, p-5   (large — padding page)
24px → gap-6, p-6   (xl — entre sections majeures)
32px → gap-8, p-8   (2xl — espacement macro, rare)
```

**Règle :** aucune valeur arbitraire (`p-[13px]`, `gap-[7px]`). Si la valeur n'est pas dans la scale, on choisit la plus proche.

**Densité HEGON — 3 niveaux structurels (consolidé 2026-06-17, audit Phase 3).**
La scale ci-dessus reste la palette ; en pratique le **gap structurel** (entre éléments répétés / panneaux) se réduit à 3 niveaux :

| Niveau | Token | px | Usage |
|--------|-------|----|-------|
| **S — dense** | `gap-3` | 12 | grilles de médias/covers (Library, Books), items denses |
| **M — confort** | `gap-4` | 16 | carousels, cartes d'une liste |
| **L — panel** | `gap-6` | 24 | gap horizontal entre left ↔ right panel, blocs majeurs côte-à-côte |

**Conventions fixes (axes séparés) :**
- **Padding carte** : `p-3` (12px)
- **Padding page (frame)** : `px-4 py-4 sm:px-6 sm:py-6`
- **Rythme vertical entre sections d'une page** : **`space-y-4` (16px)** — PAS `space-y-6` (décidé 2026-06-17 ; Watching / Library / Sports / Habits-All alignés)
- **Micro intra-composant** (icône↔texte, padding badge, chip) : `gap-1.5` / `gap-1` — libre, hors scale structurelle
- **Half-steps `1.5` / `2.5`** : réservés au micro-spacing intra-composant, **jamais** structurel

**Décisions 2026-06-17 :** covers Books `gap-x-4 gap-y-6` → `gap-3` (uniforme) · panneaux L↔R toujours `gap-6` (outliers `gap-5`/`gap-4` corrigés) · retirer `gap-5`/`space-y-5`, `gap-7`, `p-7`/`p-10`/`p-12` → valeur de scale la plus proche.

---

### 1.6 Typography scale

**Principe :** `text-sm` est la base. Pas de `text-base` ou `text-lg` dans les cards.

```css
--text-2xl:  24px / font-bold     → Titre de page (h1, une seule fois par page)
--text-xl:   20px / font-semibold → Titre de section majeure
--text-sm:   14px / font-medium   → Base — titres de card, contenu principal
--text-xs:   12px / font-medium   → Labels de section, badges, metadata
--text-2xs:  10px / font-medium   → Timestamps, compteurs, info tertiaire
```

**Tokens sémantiques (globals.css `@theme`, à privilégier — 2026-06-03) :** une classe = taille + interligne + graisse + tracking.
- `text-title` → 16px / 600 / -0.01em — **titre de section de page** (le tier dominant : "In Progress", "For You"…)
- `text-body` → 14px / 500 — lecture courante
- `text-label` → 12px / 500 — métadonnées, chips, dates
- `text-caption` → 10px / 600 / 0.08em — eyebrows + section-labels (uppercase)
- **Titre de carte / modal** = `text-sm font-semibold` (14px) — un cran sous `text-title`.

**Line height :**
- Titres : `leading-tight` (1.25)
- Body : `leading-snug` (1.375)
- Labels/metadata : `leading-none` (1)

**Font weight :**
- Valeurs importantes (stats, chiffres) : `font-bold`
- Titres de card : `font-semibold`
- Body / descriptions : `font-normal`
- Labels : `font-medium`

**Ce qui est INTERDIT :**
- `text-base` ou `text-lg` dans une card — jamais
- `font-black` — jamais
- Uppercase sur les labels — lisibilité réduite en haute densité
- Mélanger plus de 3 tailles dans un même composant

---

### 1.7 Border radius tokens

**Principe :** high density = radius contenu. Pas de `rounded-2xl` sur des éléments de liste.

```css
--radius-sm:   4px   → Tags, badges, chips (rounded)
--radius-md:   6px   → Boutons, inputs (rounded-md)
--radius-lg:   8px   → Cards standard (rounded-lg)
--radius-xl:   12px  → Modals, panels (rounded-xl)
--radius-full: 9999px → Avatars, dots uniquement
```

### Radius SÉMANTIQUE par rôle (canonique 2026-06-18, Phase 4)

La scale brute reste, mais **on style par RÔLE d'élément**, pas par taille — chaque rôle = 1 token dans `globals.css`, donc **changer le radius d'un rôle = 1 ligne** (et c'est le rendu visuel qui tranche). Principe : *le radius suit la taille de la surface* — petite tuile serrée = tight, grande carte = un cran plus rond. Proxy validé owner : **carte horizontale vs verticale**.

| Token / utilitaire | px | Rôle |
|--------------------|----|------|
| `rounded-card` (`--radius-card`)     | 12 | **cartes horizontales / contenu** — carousels, Goals, Habits, Stats, Books-reading, cartes detail |
| `rounded-tile` (`--radius-tile`)     | 8  | **tuiles verticales media** — posters, covers (Library, Books) |
| `rounded-control` (`--radius-control`) | 8 | boutons, inputs, select, segmented |
| `rounded-chip` (`--radius-chip`)     | 6  | tags, badges, petits chips |
| `rounded-modal` (`--radius-modal`)   | 16 | modals, sheets, gros panels |
| `rounded-full`                       | ∞  | avatars, dots, pills |

**Migration** : on remplace les `rounded-xl`/`rounded-lg`/`rounded-2xl` bruts par l'utilitaire de rôle. `xl→card` et `lg→tile` = no-op visuel (mêmes valeurs) ; les `rounded-2xl` orphelins sur cartes (ex Don't Miss 16→12) se consolident. FAIT : Goals, Watching (carousels/For You/Don't Miss/Library/More Like This), Books covers. RESTE : Habits/Stats/Sports/Dashboard + sweep control/chip/modal (boutons aujourd'hui en 6, à trancher 6 vs 8).

**Ce qui est INTERDIT :**
- `rounded-2xl`/`rounded-3xl` bruts (utiliser `rounded-modal` si c'est vraiment un modal)
- `rounded-full` sur un bouton ou un badge texte
- un radius brut (`rounded-xl`…) là où un token de rôle existe

---

### 1.8 Z-index scale

```css
--z-base:     0      → Contenu standard
--z-sticky:   10     → Topbar sticky, headers collants
--z-dropdown: 20     → Dropdowns, menus
--z-modal:    30     → Modals, dialogs
--z-overlay:  40     → Overlays derrière modals
--z-toast:    50     → Toasts, notifications (toujours au-dessus)
```

---

### 1.9 Motion tokens

> Voir **Section 9 — Motion System** pour la référence complète (tokens Framer Motion, spring values, stagger rules).
> Section 9 est la source de vérité unique pour tout ce qui concerne le mouvement.

---

## 2. Système d'états

**Principe fondateur :** un état ne change jamais la forme d'un composant (pas de border colorée, pas de shadow, pas de shape différente). Il modifie uniquement le contenu visuel à l'intérieur.

---

### 2.1 Active

Élément en cours, sélectionné, sur lequel l'utilisateur travaille.

- Background : `surface-2` (seul changement — lift d'un niveau)
- Border : `border-default`
- Texte titre : `text-primary`
- Aucun autre signal — le background suffit, pas de border-left

---

### 2.2 Completed

Tâche, habit, milestone terminé.

- Background : `surface-1` (inchangé)
- Opacité globale : `opacity-50`
- Texte titre : `text-tertiary` + `line-through`
- Icône : check avec `accent` du module
- Signal : pas de border colorée — l'opacité suffit

---

### 2.3 At-risk

Élément qui nécessite attention mais pas encore critique (ex: habit pas fait hier, goal deadline dans 3 jours).

- Background : `surface-1` (inchangé)
- Border : `border-default` (inchangé)
- Signal : petit dot `#f59e0b` (amber) + texte label `text-[10px] text-amber-400` dans le contenu de la card
- Aucun changement structural

---

### 2.4 Overdue

Deadline dépassée.

- Background : `surface-1` (inchangé)
- Border : `border-default` (inchangé — pas de border rouge)
- Texte du titre : passe en `#f59e0b` (amber-400)
- Signal : icône warning `12px` amber à côté du titre
- Label : `text-[10px] text-amber-400` — "Overdue · X days"

---

### 2.5 Archived

Élément désactivé, mis de côté volontairement.

- Background : `surface-1` (inchangé)
- Opacité globale : `opacity-40`
- Aucun signal coloré — l'opacité communique l'état
- Groupé en bas de liste ou dans une section séparée "Archived"

---

### 2.6 Disabled

Élément non interactif (bouton, input, action non disponible).

- Opacité : `opacity-40`
- Cursor : `cursor-not-allowed`
- Aucun hover effect
- Aucun changement de couleur

---

**Tableau récapitulatif :**

| État      | Background | Border  | Opacité | Signal                          |
|-----------|------------|---------|---------|----------------------------------|
| Active    | surface-2  | default | 100%    | bg lift uniquement              |
| Completed | surface-1  | default | 50%     | line-through + check accent     |
| At-risk   | surface-1  | default | 100%    | dot amber + label amber         |
| Overdue   | surface-1  | default | 100%    | titre amber + icon warning      |
| Archived  | surface-1  | default | 40%     | rien — opacité suffit           |
| Disabled  | inchangé   | inchangé| 40%     | cursor not-allowed              |

---

## 3. Système d'interaction

**Principe :** un seul signal par interaction. Jamais deux propriétés qui changent simultanément.

---

### 3.1 Hover

- Cards : `background surface-1 → surface-2`, `transition-colors duration-100`
- Boutons secondary/ghost : `background +1 niveau`, `transition-colors duration-100`
- Liens texte : `text-secondary → text-primary`, `transition-colors duration-100`
- Éléments sidebar : `background surface-2`, `transition-colors duration-100`

**Ce qui est INTERDIT :**
- Border colorée au hover
- Shadow au hover
- Scale au hover sur les cards
- Changement de couleur de texte + background simultanément

---

### 3.2 Press

- Boutons uniquement : `scale-[0.98]`, `transition-transform duration-100`
- Cards : aucun press effect — pas de scale sur des containers de contenu
- Durée : 100ms max, pas de bounce

---

### 3.3 Focus

- Inputs / Textarea : `border-focus` (`rgba(255,255,255,0.20)`) + `outline-none`
- Boutons : `ring-2 ring-white/20 ring-offset-0`
- Jamais de ring coloré par module
- Focus visible uniquement au clavier (`focus-visible`) — pas au clic

---

### 3.4 Disabled

- Opacité : `opacity-40`
- `cursor-not-allowed` sur le container
- Aucun hover, aucun press, aucun focus
- Pas de tooltip explicatif sauf si l'action sera disponible prochainement

---

## 4. Composants

### 4.1 Boutons

**Sizing unique pour toute l'app :**
```
height:  h-8 (32px)
padding: px-3 (12px)
text:    text-sm font-medium
radius:  rounded-md (6px)
```

**3 variantes :**

**Primary** — une seule action dominante par écran, le CTA principal
```
bg: accent du module
text: white
hover: opacity-90
press: scale-[0.98]
```

**Secondary** — actions importantes mais pas dominantes
```
bg: surface-2
border: border-default
text: text-primary
hover: bg surface-3
press: scale-[0.98]
```

**Ghost** — actions tertiaires, actions dans les cards
```
bg: transparent
border: aucune
text: text-secondary
hover: bg surface-2, text text-primary
press: scale-[0.98]
```

**Règles d'usage :**
- Maximum 1 bouton primary par écran visible
- Dans les cards : ghost uniquement
- Dans les modals : primary (confirm) + secondary (cancel) — jamais ghost dans un modal footer
- Bouton icon-only : `w-8 h-8 rounded-md` — même sizing que les boutons texte

**Ce qui est INTERDIT :**
- Deux boutons primary côte à côte
- `rounded-full` sur un bouton texte
- Padding custom qui sort de la scale
- Bouton primary avec border

---

### 4.2 Inputs & Textarea

```
height:     h-8 (32px) pour input, auto pour textarea
padding:    px-3 py-1.5
text:       text-sm
radius:     rounded-md (6px)
bg:         surface-2
border:     border-default
text color: text-primary
placeholder:text-tertiary
```

**États :**
```
default: bg surface-2, border-default
hover:   border-strong
focus:   border-focus (rgba(255,255,255,0.20)), outline-none
error:   border-default + label rouge sous l'input (pas de border rouge)
```

**Règles :**
- Focus ring : neutre uniquement — jamais la couleur du module
- Labels au-dessus de l'input : `text-xs font-medium text-secondary`, margin-bottom `mb-1.5`
- Helper text / erreur : `text-xs text-tertiary` ou `text-xs text-red-400` sous l'input
- Textarea : `resize-none` — hauteur fixe définie par le contexte

**Ce qui est INTERDIT :**
- Ring coloré par module au focus
- Input sans label dans un modal
- `resize-y` ou `resize` sur textarea

---

### 4.3 Cards

```
bg:      surface-1 (#0e0e10)
border:  border-subtle (rgba(255,255,255,0.04))
radius:  rounded-lg (8px)
padding: p-3 (12px)
hover:   bg surface-2, transition-colors duration-100
```

**Niveaux de cards :**
- **Card standard** : liste de contenu (habits, goals, tasks) — `p-3 rounded-lg`
- **Card section** : container d'un panel ou d'une sous-section — `p-4 rounded-lg`
- **Card hero** : élément mis en avant (Today card dashboard) — `p-4 rounded-xl`, peut avoir un gradient très subtil

**Contenu d'une card — règle des 3 :**
- Maximum 3 informations primaires visibles
- Tout le reste en secondaire ou tertiaire
- Jamais plus de 2 niveaux de fond à l'intérieur d'une card
- Pas de hauteur fixe imposée — la hauteur suit le contenu

**Ce qui est INTERDIT :**
- Background coloré (accent) sur une card standard
- Gradient coloré sur une card de liste
- Border colorée
- Shadow externe (`shadow-lg`, `drop-shadow`)
- `rounded-2xl` sur une card de liste

**Matériaux par RÔLE de surface (canonique 2026-07-08, pilote Watching detail) :**

Toute surface d'une page se classe dans UN des 4 rôles — zéro exception, réutilisable module par module :

| Rôle | Matériau | Quand |
|---|---|---|
| **Brandée** | fond `accent-{module}` (deep) + texte blanc | l'état/action du module — **1 seule par écran, maximum** (ex : StatusCard Watching, streak hero Habits, Focus Goals) |
| **Verre hero** | `bg-black/35 backdrop-blur-xl` + rim `white/10` | contrôles posés SUR de l'artwork (hero cinéma, wallpapers) — le langage des pills Back/Trailer |
| **Manage** | `surface-card` (rim light + lift) | toute carte où l'utilisateur **écrit** (take, éditeurs) |
| **Référence** | `surface-quiet` (posée, sans lift) | toute carte en **lecture seule** (details, ratings) |
| **Browse** | open — pas de carte | toute **rangée de tuiles média/contenu** (posters `rounded-tile`, stills 16:9 `rounded-card`) |

Règle de tri en une phrase : *tuiles média → open ; texte/contrôles → carte ; carte éditable → card, lecture → quiet ; état/action → brandée.*
Couches internes : carte = `surface-1` · champs sur carte = `surface-2` · popovers/menus = `surface-3`.

---

### 4.4 Badges & Tags

```
height:  auto
padding: px-2 py-0.5
text:    text-xs font-medium (10-11px)
radius:  rounded (4px)
```

**Variantes :**
```
neutral:  bg surface-2, text text-secondary, border border-subtle
success:  bg #14532d/40, text #4ade80, border #166534/40
warning:  bg #451a03/40, text #f59e0b, border #78350f/40
danger:   bg #450a0a/40, text #f87171, border #7f1d1d/40
accent:   bg accent/15, text accent, border accent/25
```

**Règles :**
- Jamais de background plein coloré sur un badge — toujours `/15` à `/40`
- Maximum 2 badges par card
- Pas d'emoji dans les badges — icône Lucide ou texte seulement

---

### 4.5 Modals

```
overlay:   bg black/60, backdrop-blur-sm
container: bg surface-3 (#1a1a1d), border border-strong, rounded-xl
width:     max-w-md (création simple) / max-w-lg (formulaire complet)
padding:   p-5
```

**Structure obligatoire :**
```
Header  : titre text-sm font-semibold text-primary + bouton close ghost icon-only
Divider : border-default
Content : espace principal du form, gap-4 entre les champs
Divider : border-default
Footer  : flex justify-end gap-2 — [Cancel secondary] [Confirm primary]
```

**Règles :**
- Inputs dans modal : bg `surface-overlay` (#1f1f22) — un niveau au-dessus de surface-3
- Titre du modal : pas de couleur accent — `text-primary` uniquement
- Bouton close : top-right, ghost, icon-only (`X` 16px)
- Pas de scroll interne sauf si contenu très long (liste de sélection)

**Implémentation Tailwind — pattern de référence (calqué sur HabitModal) :**

```tsx
// Constante accent en haut du fichier — source de vérité via globals.css
const ACCENT = "var(--color-accent-{module})";

// DialogContent
<DialogContent className="sm:max-w-md bg-surface-3 border-border-strong">

// Titre
<DialogTitle className="text-sm font-semibold text-text-primary">

// FormLabel
<FormLabel className="text-xs font-medium text-text-secondary">

// Input / Textarea / SelectTrigger — variant tasks + override bg
<Input variant="tasks" className="bg-surface-overlay focus:border-border-focus" />
<Textarea variant="tasks" className="bg-surface-overlay focus:border-border-focus" />
<SelectTrigger variant="tasks" className="w-full bg-surface-overlay focus:border-border-focus" />

// Date picker / tout trigger custom — reproduire le style input à la main
<button className="w-full h-9 px-3 flex items-center gap-2 rounded-lg border border-border-default
                   bg-surface-overlay hover:bg-surface-2 text-sm text-text-primary" />

// PopoverContent (calendar, dropdown)
<PopoverContent className="bg-surface-3 border-border-strong" />

// Bouton Cancel
<Button variant="outline"
  className="h-8 px-3 border-border-default text-text-secondary hover:text-text-primary hover:bg-surface-2">
  Cancel
</Button>

// Bouton primaire — accent via style prop, jamais hardcodé
<Button className="h-8 px-3 text-white hover:opacity-90 disabled:opacity-50"
  style={{ backgroundColor: ACCENT }}>
  Confirm
</Button>

// Bouton destructif — rouge fixe, indépendant de l'accent module
<Button className="h-8 px-3 bg-red-600 hover:bg-red-700 text-white disabled:opacity-50">
  Delete
</Button>
```

**Ce qui est INTERDIT :**
- `bg-zinc-900` hardcodé sur un DialogContent — utiliser `bg-surface-3`
- Input sans override `bg-surface-overlay` dans un modal — les inputs disparaissent visuellement
- Bouton primaire avec couleur hardcodée (`bg-blue-600`) — toujours `style={{ backgroundColor: ACCENT }}`
- `text-zinc-400` sur un label — utiliser `text-text-secondary`
- `variant="ghost"` sur un bouton dans un footer de modal

---

### 4.6 Tabs

```
container: border-b border-default
tab:       px-4 pb-2.5 pt-1, text-sm font-medium
active:    text-primary + indicateur bottom-0 left-0 right-0 h-0.5 accent
inactive:  text-tertiary, hover text-secondary
transition: colors duration-100
```

**Règles :**
- Indicateur full-width sous le tab (pas un underscore court centré)
- Pas de background sur le tab actif — uniquement l'indicateur
- CTA aligné à droite sur la même ligne que les tabs

---

### 4.7 Tooltips

```
bg:     surface-3
border: border-default
text:   text-xs text-primary
radius: rounded-md
padding: px-2 py-1
delay:  300ms avant apparition
```

**Règles :**
- Texte court uniquement — maximum 6 mots
- Jamais de tooltip sur un élément avec un label visible
- Toujours sur les boutons icon-only

---

### 4.8 Toasts

```
bg:      surface-3
border:  border-default
radius:  rounded-lg
padding: px-4 py-3
text:    text-sm text-primary
width:   max-w-xs
position: bottom-right
duration: 3s auto-dismiss
```

**Variantes :**
- Success : dot `#4ade80` à gauche
- Error : dot `#f87171` à gauche
- Info : dot `text-secondary` à gauche
- Pas d'icône large — un dot 6px suffit

---

### 4.9 Skeletons

```
bg:        surface-2
animation: pulse (opacity 0.5 → 1, duration-1000, repeat)
radius:    même que l'élément réel
```

**Règles :**
- Reproduire fidèlement la structure de l'élément réel (même layout, même sizing)
- Pas de shimmer animé latéralement — pulse uniquement
- Afficher minimum 3 skeletons dans une liste pour simuler du contenu

---

### 4.10 Empty states

```
container: centré verticalement et horizontalement dans la zone disponible
icon:      lucide, 32px, text-tertiary
title:     text-sm font-medium text-secondary
subtitle:  text-xs text-tertiary, max-w-xs, text-center
cta:       bouton primary ou secondary selon l'importance
```

**Règles :**
- Toujours actionnable — un empty state sans CTA est une impasse
- Pas d'illustration complexe — icône simple suffit
- Ton du message : positif et incitatif, jamais "Aucun résultat trouvé"

---

### 4.11 Entity links

Pattern unique pour tous les liens entre entités (habit → goal, task → project).

```
container: inline-flex items-center gap-1
icon:      lucide Link2, 10px, text-tertiary
text:      text-[10px] text-tertiary
hover:     text-secondary, transition-colors duration-100
```

Exemple : `🔗 Goal title` → remplacé par `<Link2 size={10} /> Goal title` en text-tertiary.

---

## 5. Layout

### 5.1 Topbar

```
height:     h-14 (56px)
bg:         surface-0/80 + backdrop-blur-sm (sticky)
border-b:   border-subtle
padding:    px-4
position:   sticky top-0 z-sticky
```

**Contenu :**
- Gauche : searchbar uniquement (`max-w-xs`, `flex-1`)
- Droite : actions contextuelles légères (notifications, user avatar si nécessaire)
- Pas de CTA primaire dans la topbar

**Searchbar dans topbar :**
```
height:  h-8
padding: pl-8 pr-3 (icône search à gauche)
bg:      surface-2
border:  border-default
radius:  rounded-md
text:    text-sm text-primary, placeholder text-tertiary
focus:   border-focus
```

---

### 5.2 Module header

Présent dans chaque module, sous la topbar, au-dessus des tabs.

```
padding:   px-4 pt-5 pb-4
structure: flex items-start justify-between
```

**Colonne gauche :**
```
h1:       text-xl font-bold text-primary, leading-tight
subtitle: text-xs text-tertiary, mt-0.5
stats:    text-xs text-tertiary, mt-1.5 — valeurs importantes en text-secondary
```

**Colonne droite :**
- Badges récapitulatifs : variante `neutral` uniquement — pas de badges colorés
- Maximum 3 badges

**Règle :** pas de duplication — si les stats sont dans les badges, elles ne sont pas aussi dans le subtitle. Choisir l'un ou l'autre.

---

### 5.3 Left / Right panel split

Applicable aux 3 modules Life (Goals, Habits, Journal).

```
left:  flex-[13] min-w-0   → ~65% — contenu principal (liste, éditeur)
right: flex-7   min-w-0    → ~35% — contexte, stats, calendar
gap:   gap-4
```

**Titres de sections dans les cards du right panel (Streak, Mood, Calendar...) :**
```
text-xs font-semibold text-text-secondary
```
`text-secondary` et non `text-tertiary` — le label doit être lisible sans concurrencer le contenu (chiffre, grille, liste).
`text-xs` et non `text-sm` — évite la collision visuelle avec les titres de card (`text-sm`).
`mb-2` uniquement si le parent n'a pas de `gap` — si `flex flex-col gap-*` est déjà présent, ne pas ajouter de mb (double spacing).

**Règle :** le right panel n'est visible que sur le tab principal (`Today` / `Write`). Sur Calendar et Archive, le contenu prend 100% de la largeur.

---

### 5.4 CTA placement

- Position : aligné à droite sur la même ligne que les tabs
- Variante : primary (accent du module)
- Sizing : `h-8 px-3 text-sm` — identique aux boutons standard
- Label : "+ New [Item]" — concis, toujours avec le "+"

---

## 6. Navigation

### 6.1 Tabs (underline style)

> Voir **4.6 Tabs** pour la définition technique complète du composant.

Règles de navigation spécifiques :
- Jamais plus de 4 tabs par module
- Le tab actif au chargement = toujours le premier tab (Today / Write / Overview)
- Sur mobile (si applicable plus tard) : tabs scrollables horizontalement

---

### 6.2 Dock (remplace la Sidebar — Phase 3 en place)

> La sidebar a été remplacée par le Dock en Phase 3. Ce qui suit documente le Dock.
> Fichier de référence : `src/shared/components/navigation/Dock.tsx`

```
width:         56px fixe (DOCK_W = 56)
bg:            #0e0e10 (surface-1)
border-r:      border-white/5
```

**Item actif :**
```
bg:   bg-white/[0.07]
icon: couleur accent du module (style prop)
barre verticale gauche: w-0.5 h-4 rounded-r-full, couleur accent
```

**Item inactif :**
```
icon: text-tertiary
hover: text-primary + bg-white/5
```

**Tooltips :** droite, delay 500ms, `bg-surface-3 border-border-default`

**Ambient glow :** radial-gradient en haut + linear-gradient en bas, change selon le module actif (transition 700ms)

**Ce qui est INTERDIT :**
- État expanded du dock — icônes uniquement, pas de labels visibles
- Background coloré (accent) sur l'item actif — barre verticale suffit
- Dock avec le même fond que le corps de l'app (surface-0)

---

## 7. Patterns spécifiques

### 7.1 Système de warning

**Principe :** un warning ne change jamais la forme ou la border d'une card. Il ajoute un signal dans le contenu uniquement.

| Niveau    | Condition                        | Signal visuel                                      |
|-----------|----------------------------------|----------------------------------------------------|
| Notice    | Due dans 3+ jours                | Aucun — pas de signal                              |
| Warning   | Due aujourd'hui ou dans 1-2 jours| Label `text-[10px] text-amber-400` dans la card    |
| Overdue   | Deadline dépassée                | Titre en `text-amber-400` + icon Warning 12px      |
| Critical  | Bloquant / priorité absolue      | Dot `#ef4444` 6px dans le coin top-right de la card|

**Ce qui est INTERDIT :**
- Border rouge ou amber sur la card
- Background coloré pour signaler une urgence
- Icône warning grande (>14px) dans une card de liste

---

### 7.2 Milestones / étapes

Pattern pour Goals "The Path" et tout système d'étapes ordonné.

**Un seul background pour tous les états — c'est le contenu qui distingue :**

```
Completed : opacity-50 + icône check accent + texte line-through text-tertiary
Active    : bg surface-2 + texte text-primary (bg lift uniquement, pas de border-left)
Upcoming  : bg surface-1 + border border-subtle dashed + texte text-tertiary
```

**Règles :**
- Jamais plus de 2 niveaux de fond dans le container "The Path"
- Le container parent : `surface-1`
- Les milestones à l'intérieur : `surface-1` ou `surface-2` selon état — jamais de border colorée

---

### 7.3 Dates relatives

Format unique pour toute l'app :

```
Dans le futur :
  Aujourd'hui       → "Due today"
  Demain            → "Due tomorrow"
  Dans 2-6 jours    → "Due in X days"
  Dans 7+ jours     → date courte "Apr 22"

Dans le passé :
  Hier              → "Overdue · 1 day"
  Avant-hier+       → "Overdue · X days"
  Plus de 30 jours  → "Overdue · Apr 2"

Complété :
  Même jour         → "Completed today"
  Autre jour        → date courte "Apr 14"
```

**Styling :**
- Futur lointain (7+ jours) : `text-tertiary`
- Warning zone (today, tomorrow, 2 jours) : `text-amber-400`
- Overdue : `text-amber-400`
- Complété : `text-tertiary`

---

### 7.4 Icon sizes

Tailles autorisées uniquement :

```
10px → metadata inline, entity links
12px → badges, labels dans les cards
14px → icônes dans les boutons, actions secondaires
16px → icônes standard dans les cards
18px → icône principale d'une card (habit icon, goal icon)
20px → icônes dans le module header
24px → icônes dans les empty states (accompagnées de 32px max)
32px → empty states uniquement
```

**Règle :** une icône dans un bouton = 14px. Une icône décorative dans une card = 16-18px. Jamais de `size={24}` dans une card de liste.

---

### 7.5 Scrollbars

```css
/* globals.css */
::-webkit-scrollbar { width: 4px; height: 4px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.08); border-radius: 4px; }
::-webkit-scrollbar-thumb:hover { background: rgba(255,255,255,0.15); }
```

Scrollbars fines, discrètes, invisibles au repos — apparaissent au hover du container scrollable.

---

### 7.6 Formulaires (select, checkbox, toggle)

**Select :**
```
Même styling que les inputs (h-8, px-3, rounded-md, surface-2)
Dropdown : surface-3, border-default, rounded-lg
Option hover : surface-2
Option active : surface-2 + texte text-primary + check accent à droite
```

**Checkbox :**
```
Taille : 16px
Unchecked : border border-default, bg surface-2, rounded
Checked : bg accent du module, border accent, check white 10px
Indeterminate : bg surface-2, border accent, dash white 10px
```

**Toggle (switch) :**
```
Width : w-8, height : h-4
Off : bg surface-3, border border-default
On : bg accent du module
Thumb : bg white, w-3 h-3, rounded-full, transition-transform duration-200
```

### 7.7 Radix — Contrainte critique

**Règle :** ne jamais utiliser un Radix `Popover` (Portal) dans un Radix `Dialog` pour du contenu scrollable.

**Pourquoi :** `react-remove-scroll` intercepte les wheel events hors du DOM Dialog. Popover rend son contenu via Portal vers `document.body` → vu comme "extérieur" → scroll bloqué même si le Popover est visuellement dans le Dialog.

**Pattern alternatif validé (IconPicker) :**
```tsx
// div relative = container
// button trigger
// div absolute top-[calc(100%+4px)] left-0 right-0 z-50 — PAS de Portal
// div natif overflow-y-auto pour la liste scrollable
// useEffect close au click extérieur (mousedown sur document)
// onMouseDown (pas onClick) sur les items — évite le conflit avec le close handler
```

**Règle scroll layout :** `main` a `overflow-y-auto`. Pages avec topbar sticky utilisent `min-h-full` + `sticky top-0 z-10` sur la topbar (pas `shrink-0`). Jamais deux containers `overflow-y-auto` empilés.

---

## 8. Règles globales

### 8.1 Information density (max par card)

- Maximum **3 informations primaires** visibles dans une card de liste
- Tout ce qui dépasse = secondaire (`text-tertiary`) ou masqué par défaut
- Titres : toujours `truncate` — jamais de wrap dans une card de liste
- La hauteur suit le contenu — pas de hauteur fixe imposée

---

### 8.2 Hiérarchie visuelle

- **1 élément dominant** par écran — le CTA primaire ou la valeur principale
- **Maximum 2 éléments secondaires** qui attirent l'attention
- Tout le reste est tertiaire ou invisible jusqu'à interaction
- Si deux éléments se battent pour l'attention, l'un des deux est de trop

---

### 8.3 Motion (functional only)

Chaque animation doit répondre à la question : "est-ce que l'utilisateur comprend mieux l'interface grâce à cette animation ?"

- Hover → oui (feedback d'interactivité)
- State change completed → oui (confirmation de l'action)
- Modal open → oui (focus sur le nouvel élément)
- Animation de bienvenue au chargement → non, interdit
- Particules, confetti → non, interdit sauf célébration exceptionnelle (streak record)

---

### 8.4 Ce qui est identique partout

Sans exception dans tous les modules :

- Surfaces et couleurs (variables sémantiques)
- Sizing des boutons (`h-8 px-3`)
- Sizing des inputs (`h-8`)
- Border radius tokens
- Spacing scale
- Typography scale
- Hover behavior (`surface-2, duration-100`)
- Focus ring (neutre)
- Toasts
- Modals (structure Header/Content/Footer)
- Skeletons
- Empty states (pattern)
- Scrollbars

---

### 8.5 Ce qui peut varier par module

- Couleur accent
- Icône du module
- Tabs labels et nombre
- Contenu du right panel
- Densité de la liste (certains modules plus aérés)
- Champs spécifiques dans les modals de création

---

### 8.6 Exceptions intentionnelles

| Module | Exception | Justification |
|--------|-----------|---------------|
| Journal — zone éditeur uniquement | Fond crème `#f5f0e8` + police handwriting (Caveat ou Kalam) + lignes discrètes | Rupture visuelle intentionnelle pour signaler le changement de mode (réflexion vs action). Le sidebar/header/tabs/liste restent `#09090b` strict. |
| Habits | Progress bar remplace la zone stats | Complétion du jour = info primaire, la barre occupe tout l'espace — streak/stats dans le right panel |
| Sports | Pas de zone stats | Contenu dynamique (matchs, classements) — pas de stats module propres |
| Watching — detail | **StatusCard** : carte brandée teal deep, rail droit #1 (desktop) / juste après le hero (mobile) | Domicile unique état-aware (chip + faits + LE geste de l'état) — squelette de page fixe, seule surface qui « sait » l'état. Pattern hero-card cross-module (1ʳᵉ carte du rail = surface `-deep`). Dot teal vivid = seul accent ; primaire = bouton blanc/texte noir. Spec : `hq/watching-detail-blueprint.md` |
| Watching / tuiles média | Scale au hover (1.03-1.04) sur les tuiles média (posters, stills) | Profondeur sans ombre sur fond noir — les ombres sont invisibles ; le scale est l'idiome de lift des rangées browse |

**Règle :** toute nouvelle exception doit être documentée dans ce tableau avec sa justification avant implémentation.

---

### 8.7 FORBIDDEN — liste définitive

Visuels :
- Gradients colorés sur les cards standard
- Borders colorées (accent ou warning)
- Multiple glows sur un même écran
- Background coloré sur une card de liste
- `rounded-2xl` ou `rounded-full` sur des éléments de contenu
- Emojis dans les badges ou labels
- Blanc pur `#ffffff` pour le texte
- `bg-zinc-*` hardcodé dans les composants (utiliser les variables)

Interaction :
- Ring coloré par module sur les inputs
- Scale au hover sur les cards
- Animations > 300ms
- Bounce ou spring excessif
- Deux boutons primary côte à côte

Layout :
- CTA primaire dans la topbar
- Plus de 4 tabs par module
- Plus de 2 niveaux de fond dans un même composant
- Duplication stats header + badges (choisir l'un)

---

---

## 9. Motion System

**Objectif :** chaque élément qui entre, sort, ou change d'état a une animation. Rien ne pop brutalement. Tout a de l'inertie.

**Principe Apple :** les animations sont physiques, pas linéaires. Un élément arrive vite puis ralentit naturellement — comme un objet dans le monde réel.

---

### 9.1 Tokens de base

```ts
// src/shared/motion/tokens.ts

export const spring = {
  snappy:  { type: "spring", stiffness: 400, damping: 30 },  // actions rapides (bouton, toggle)
  smooth:  { type: "spring", stiffness: 300, damping: 28 },  // cards, éléments de liste
  gentle:  { type: "spring", stiffness: 200, damping: 24 },  // modals, panels
}

export const ease = {
  out:    { duration: 0.15, ease: [0, 0, 0.2, 1] },          // hover, focus
  inOut:  { duration: 0.20, ease: [0.4, 0, 0.2, 1] },        // state transitions
  modal:  { duration: 0.20, ease: [0, 0, 0.2, 1] },          // modal open/close
}
```

---

### 9.2 Entrées d'éléments (mount animations)

**Card individuelle :**
```ts
initial:  { opacity: 0, y: 6 }
animate:  { opacity: 1, y: 0 }
transition: spring.smooth
```

**Liste de cards (stagger) :**
```ts
// Container
variants: {
  hidden: {},
  show: { transition: { staggerChildren: 0.04, delayChildren: 0 } }  // 40ms entre chaque card
}

// Chaque item
variants: {
  hidden: { opacity: 0, y: 8 },
  show:   { opacity: 1, y: 0, transition: spring.smooth }
}
```

**Règle du cap stagger :** 40ms × max 5 items = 200ms maximum de délai cumulé.
Au-delà de 5 items, les suivants apparaissent tous avec le même délai de 200ms.
Évite qu'une liste de 20 items prenne 800ms à se révéler.

**Section / panel :**
```ts
initial:  { opacity: 0, y: 4 }
animate:  { opacity: 1, y: 0 }
transition: spring.gentle
```

---

### 9.3 Modals

```ts
// Overlay
initial:  { opacity: 0 }
animate:  { opacity: 1 }
exit:     { opacity: 0 }
transition: ease.modal

// Container modal
initial:  { opacity: 0, scale: 0.96, y: 8 }
animate:  { opacity: 1, scale: 1,    y: 0 }
exit:     { opacity: 0, scale: 0.96, y: 8 }
transition: spring.gentle
```

---

### 9.4 State transitions

**Completed (habit, task) :**
```ts
// L'élément qui devient complété
animate: { opacity: 0.5 }
transition: ease.inOut

// Le bouton qui switch (AnimatePresence mode="wait")
initial:  { scale: 0.85, opacity: 0 }
animate:  { scale: 1,    opacity: 1 }
exit:     { scale: 0.85, opacity: 0 }
transition: spring.snappy
```

**Tab switch :**
```ts
// Indicateur underline
layoutId: "tab-indicator"  // Framer Motion layout animation — glisse naturellement entre tabs
transition: spring.snappy
```

**Card ajoutée à une liste (après création) :**
```ts
initial:  { opacity: 0, height: 0, marginBottom: 0 }
animate:  { opacity: 1, height: "auto", marginBottom: 8 }
transition: spring.smooth
```

**Card supprimée / archivée :**
```ts
exit: { opacity: 0, height: 0, marginBottom: 0 }
transition: spring.smooth
```

---

### 9.5 Micro-interactions boutons

```ts
// Press effect — sur tous les boutons
whileTap: { scale: 0.97 }
transition: spring.snappy
```

---

### 9.6 Chargement initial de page

Quand une page charge pour la première fois, le contenu n'apparaît pas en bloc — il arrive en vague de haut en bas :

```
1. Topbar      → délai 0ms
2. Header      → délai 60ms
3. Tabs        → délai 100ms
4. Premier card → délai 140ms
5. Cards suivantes → stagger 40ms chacune
```

Implémenté via `staggerChildren` + `delayChildren` sur le container principal.

---

### 9.7 Règles strictes

**Autorisé :**
- Mount animations sur toutes les listes
- Stagger sur les listes de 3+ éléments
- Spring pour les modals et cards
- `layoutId` pour les indicateurs de tabs
- Press scale sur les boutons

**Interdit :**
- Duration > 350ms sur une action utilisateur directe
- Animations en boucle (rotation, pulse visible) sauf skeleton
- `bounce` excessif (damping < 15)
- Animations sur le scroll (parallax, etc.)
- Animer la couleur de fond d'une card au mount — seulement opacity et y

---

## Changelog

| Date       | Version | Changement                        |
|------------|---------|-----------------------------------|
| 2026-04-16 | v1.0    | Création complète du design system |
| 2026-04-16 | v1.1    | Corrections : motion (1.9 → Section 9), état Active (bg lift, plus de border-left), text-tertiary (#71717a), accent journal (#f97316), text-lg supprimé de la scale, 80px supprimé, stagger cap 200ms, tabs (6.1 → pointeur vers 4.6), milestones (border-left supprimée) |
| 2026-04-16 | v1.2    | Ajouts : Section 0 Design Principles, Decision Rules, Motion Restrictions, Accent Conflict Rule, motion tokens CSS dans tokens.css |
| 2026-04-16 | v1.3    | Ajouts : Cross-Module Hints (états propagés entre modules liés), Information Priority by Module (hiérarchie fixe par module) |
| 2026-04-16 | v1.4    | Ajouts : 8.6 Exceptions intentionnelles (Journal éditeur crème documenté). Cohérence hq/ : design-system.md supprimé, master_plan.md section 8 mise à jour, progress.md Habits V1 marqué ✅ |
| 2026-06-03 | v1.5    | Audit cohésion inter-modules : tokens typo sémantiques (`text-caption/label/body/title`, titres 2-tiers 16px/14px) ; `SearchInput` partagé ; champs tokenisés + hover uniforme `surface-2` ; shadow policy (purge cards/modals, voile léger flottants) ; spacing topbar→contenu unifié 24px (`py-6`) ; tabs `SectionHeader` → `text-sm` ; 0 `transition-all` (propriétés exactes) ; couleur Habits rose→violet. Détail : `memory/project_cohesion_audit.md` |
| 2026-07-08 | v1.6    | Pilote Watching detail : **Matériaux par rôle de surface** (§4.3 — brandée/manage/référence/browse, 1 brandée par écran max) ; row **Watching** dans Information Priority + règle d'ordre Tertiary/Secondary ; exceptions §8.6 StatusBar Watching + scale hover tuiles média. Spec : `hq/watching-detail-blueprint.md` |

---

*Ce document est la loi. Toute décision qui contredit une règle ici doit d'abord modifier ce document.*
