# Culture — V1 Build Spec (FUTUR — pas encore construit)

> Accent : Teal `#14b8a6`
> Header phrase : "Every event, era, and idea you've explored."
> Rôle : encyclopédie personnelle des événements culturels, historiques, artistiques que l'utilisateur explore (événements, ères, personnages, idées).
> Famille de layout : **Timeline** (vue chronologique comme vue principale).
> Source : extrait de `memory/project_os_vision.md` (2026-04-22)

---

## Modèle de données (à finaliser)

```sql
culture_events (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id uuid REFERENCES organizations NOT NULL,
  user_id uuid REFERENCES auth.users NOT NULL,

  -- Métadonnées
  title text NOT NULL,
  era text CHECK (era IN ('ancient', 'medieval', 'modern', 'contemporary')),
  event_date date,          -- date approximative ou exacte
  event_year int,           -- pour événements anciens, plus simple
  topic text,               -- thème : "guerres", "philosophie", "art"
  cover_url text,

  -- Tracking personnel
  content text,             -- markdown — récit personnel / notes
  rating int CHECK (rating BETWEEN 1 AND 5),
  is_favorite boolean DEFAULT false,

  -- Relations
  goal_id uuid REFERENCES goals ON DELETE SET NULL,

  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
)

culture_persons (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id uuid REFERENCES organizations NOT NULL,
  user_id uuid REFERENCES auth.users NOT NULL,

  name text NOT NULL,
  birth_year int,
  death_year int,
  era text CHECK (era IN ('ancient', 'medieval', 'modern', 'contemporary')),
  description text,
  cover_url text,

  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
)

-- Many-to-many : events ↔ persons
culture_event_persons (
  event_id uuid REFERENCES culture_events ON DELETE CASCADE,
  person_id uuid REFERENCES culture_persons ON DELETE CASCADE,
  PRIMARY KEY (event_id, person_id)
)

-- Many-to-many : events ↔ events (connexions explicites comme backlinks Tech)
culture_event_connections (
  source_id uuid REFERENCES culture_events ON DELETE CASCADE,
  target_id uuid REFERENCES culture_events ON DELETE CASCADE,
  PRIMARY KEY (source_id, target_id)
)
```

---

## Layout de la page

```
┌─────────────────────────────────────────────────────────┐
│  HEADER                                                  │
│  "Culture" (text-2xl font-bold)                         │
│  "Every event, era, and idea you've explored."          │
├─────────────────────────────────────────────────────────┤
│  STATS ZONE                                              │
│  Events explored · Favorites · Avg rating                │
├─────────────────────────────────────────────────────────┤
│  TABS                                                    │
│  Timeline · Collections · Persons                       │
├──────────────────────────────────┬──────────────────────┤
│  CONTENT (selon tab actif)       │  RIGHT PANEL  (w-72) │
│                                  │                      │
│  Timeline : sections par ère     │  Events this year    │
│  Collections : par thème         │  (progression si     │
│  Persons : grid figures hist.    │  goal lié)           │
│                                  │  Recently added (3)  │
└──────────────────────────────────┴──────────────────────┘
```

---

## Features V1

### 1. Tab Timeline (vue principale)
Sections par ère :
- **Ancient** (-3000 à 500)
- **Medieval** (500 à 1500)
- **Modern** (1500 à 1900)
- **Contemporary** (1900 à aujourd'hui)

Chaque ère = section expandable avec liste d'événements triés par date.

### 2. Tab Collections
Regroupement par `topic` (thème personnel défini par l'utilisateur). Ex: "Guerres mondiales", "Philosophie grecque", "Art moderne".

### 3. Tab Persons
Grid de cards de figures historiques avec photo + nom + dates (ex: "1769-1821").
Click → page détail de la person avec liste des events liés.

### 4. Stats zone
- **Events explored** : total
- **Favorites** : `is_favorite = true`
- **Avg rating** : moyenne sur events avec rating

### 5. Right panel
- **Events this year** : count + progression si lié à un goal (ex: "Explorer 20 événements en 2026 — 7/20")
- **Recently added** : 3 derniers events (image + titre + note rating)

### 6. Detail page `/perso/culture/[id]`
- Cover + titre + ère + date
- Contenu complet écrit par l'utilisateur (markdown)
- **Figures historiques liées** (many-to-many via `culture_event_persons`)
- **Connexions vers autres événements** (backlinks bidirectionnels manuel)
- **Médias liés** : Books lus sur ce sujet + Watching vus sur ce sujet (cross-module via topic match ou liens explicites)
- Toggle favorite
- Rating

### 7. Création
- Modal court : titre, ère, date approximative, topic, cover (URL ou upload)
- Picker persons liées (multi-select sur `culture_persons`)
- Goal lié (optionnel)

---

## Cross-module

- **Goal lié** : `goal_id` FK
- **Books** : auto-suggest books dont les tags overlap avec le topic du event
- **Watching** : auto-suggest films/séries dont les tags overlap avec le topic
- **Journal** : option de référencer un event dans une entry (à voir avec Journal V2)

---

## Architecture (à confirmer au moment du build)

```
src/modules/culture/
├── service.ts
├── types.ts
├── hooks/
│   ├── query-keys.ts
│   ├── useEvents.ts
│   ├── usePersons.ts
│   └── useEventConnections.ts
└── components/
    ├── CultureTimelineView.tsx
    ├── CultureCollectionsView.tsx
    ├── CulturePersonsView.tsx
    ├── EventCard.tsx
    ├── PersonCard.tsx
    ├── EventDetailPage.tsx
    ├── PersonDetailPage.tsx
    ├── EraSection.tsx
    ├── AddEventModal.tsx
    ├── AddPersonModal.tsx
    └── CultureEmptyState.tsx
```

---

## Status

- 🔲 Pas construit
- 📝 Spec extraite de `project_os_vision.md` mémoire
- ⏭ V2 brainstorm à faire avant build (dans `hq/modules_version_2/culture-v2.md`)
