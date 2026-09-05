# Tech — V1 Build Spec

> Accent : Purple `#a855f7`
> Rôle : knowledge vault de développeur. Documenter son parcours technique — technologies apprises, problèmes résolus, décisions d'architecture, découvertes.
> Objectif : avoir une référence personnelle vérifiée, consultable, qui grandit avec le temps.

---

## Concept

Ce n'est pas un outil de notes génériques. Chaque entrée a un **type** qui structure l'information différemment. La timeline de toutes les entrées raconte l'évolution de Zakaria en tant que développeur.

---

## Modèle de données

```sql
tech_entries (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id uuid REFERENCES organizations NOT NULL,
  user_id uuid REFERENCES auth.users NOT NULL,

  title text NOT NULL,
  type text NOT NULL CHECK (type IN (
    'til',           -- Today I Learned : découverte courte et factuelle
    'problem_solved',-- Problème + solution + contexte
    'architecture',  -- Décision d'architecture : pourquoi X plutôt que Y
    'deep_dive',     -- Exploration en profondeur d'une techno/concept
    'resource'       -- Lien/livre/vidéo avec notes personnelles
  )),
  content text NOT NULL,        -- Markdown

  -- Métadonnées
  tags text[] DEFAULT '{}',     -- technos : ['nextjs', 'supabase', 'typescript']
  source_url text,              -- pour type=resource
  difficulty text CHECK (difficulty IN ('beginner', 'intermediate', 'advanced')),
  is_verified boolean DEFAULT false,  -- passé par le fact-checker IA (Phase 5)

  -- Relations
  goal_id uuid REFERENCES goals ON DELETE SET NULL,  -- lié à un goal d'apprentissage

  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
)
```

---

## Types d'entrées — structure du contenu

### TIL (Today I Learned)
Court, factuel, une seule découverte.
```
Titre : "useEffect ne se déclenche pas sur le premier render en mode strict"
Contenu : explication + exemple de code
Tags : ['react', 'hooks']
```

### Problem Solved
Le format le plus précieux à long terme.
```
Titre : "Fix hydration error avec Zustand et Next.js App Router"
Contenu structuré :
  - Contexte : description du problème
  - Cause : pourquoi ça arrive
  - Solution : ce qui a résolu le problème
  - Code : snippet si pertinent
Tags : ['nextjs', 'zustand', 'ssr']
```

### Architecture Decision
```
Titre : "Pourquoi modular monolith plutôt que microservices pour HEGON"
Contenu :
  - Contexte : situation initiale
  - Options considérées
  - Décision prise
  - Trade-offs acceptés
  - Quand revisiter
Tags : ['architecture', 'nextjs']
```

### Deep Dive
Article personnel sur une techno ou un concept.
Structure libre mais complète.

### Resource
```
Titre : "The Pragmatic Programmer"
URL : lien
Notes personnelles : ce que j'en ai retenu
Tags : ['books', 'career', 'best-practices']
```

---

## Features V1

### 1. CRUD Entrées
- Créer : choisir le type en premier → formulaire adapté au type
- Éditeur Markdown avec preview (split view ou toggle)
- Tags : saisie libre avec autocomplete sur les tags existants
- Goal lié : picker optionnel
- Éditer, supprimer (avec confirmation)

### 2. Vue principale — Feed chronologique
Liste antéchronologique de toutes les entrées.
Chaque card affiche :
- Type badge (coloré par type)
- Titre
- Tags chips
- Date
- Preview du contenu (2 lignes)
- Indicateur `is_verified` si fact-checked (Phase 5)

### 3. Vue Timeline
Même feed mais visuellement en timeline (ligne verticale, entrées sur les côtés).
Groupé par mois.
Permet de voir la progression : "En janvier je découvrais les bases de Supabase, en mars je configurais les RLS policies."

### 4. Filtres
- Par type (TIL / Problem Solved / Architecture / Deep Dive / Resource)
- Par tag (multi-select)
- Par période
- Recherche textuelle sur titre + contenu (ILIKE)

### 5. Page détail d'une entrée
- Rendu Markdown complet
- Metadata : type, tags, date, goal lié, difficulté
- Actions : éditer, supprimer, copier le lien

### 6. Stats rapides
- Nombre d'entrées par type
- Tags les plus utilisés
- Entrées ce mois / cette année
- Streak d'écriture (jours consécutifs avec au moins une entrée)

---

## UI/UX Requirements

- Page `/tech` dans la sidebar
- Badge de type : couleur unique par type (TIL=blue, Problem=red, Architecture=yellow, DeepDive=purple, Resource=green)
- Éditeur Markdown : côté gauche édition, côté droit preview (desktop), toggle sur mobile
- Timeline view : ligne verticale accent purple, entrées alternées gauche/droite
- Tags : autocomplete sur les tags existants pour éviter les doublons (typos)
- Empty state : "Start documenting your tech journey" + bouton "Add first entry"

---

## Architecture

```
src/modules/tech/
├── service.ts          ← getTechEntries, getTechEntry, createTechEntry, updateTechEntry,
│                          deleteTechEntry, getTechStats, searchTechEntries, getTags
├── types.ts            ← TechEntry, TechEntryType, TechEntryDifficulty
├── hooks/
│   ├── query-keys.ts   ← TECH_KEYS.all, TECH_KEYS.byId(id), TECH_KEYS.byType(type), TECH_KEYS.byTag(tag)
│   ├── useTechEntries.ts
│   ├── useTechEntry.ts
│   └── useTechStats.ts
└── components/
    ├── TechEntryCard.tsx       ← card dans le feed
    ├── TechEntryDetail.tsx     ← page détail avec rendu Markdown
    ├── TechEntryModal.tsx      ← create + edit avec type-aware form
    ├── TechFeedView.tsx        ← vue liste
    ├── TechTimelineView.tsx    ← vue timeline chronologique
    ├── TechFilters.tsx         ← type + tags + search
    └── TechEmptyState.tsx
```

---

## Critère de passage

Zakaria peut documenter un problème qu'il vient de résoudre en moins de 2 minutes, le retrouver 3 mois plus tard via les filtres ou la recherche, et voir sur la timeline que ses entrées reflètent une montée en compétence réelle.
