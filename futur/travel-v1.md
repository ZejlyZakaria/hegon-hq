# Travel — V1 Build Spec (FUTUR — pas encore construit)

> Accent : Indigo `#6366f1` (proposé — à valider, conflit avec Books écarté)
> Header phrase : "Collect places. Live fully."
> Rôle : capture des voyages — bucket list, voyages à venir, passés. Photo gallery + récit + lien avec Journal entries.
> Famille de layout : **Timeline** (vue chronologique comme vue principale).
> Source : extrait de `memory/project_os_vision.md` (2026-04-22)

---

## Modèle de données (à finaliser)

```sql
travel_trips (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id uuid REFERENCES organizations NOT NULL,
  user_id uuid REFERENCES auth.users NOT NULL,

  -- Métadonnées
  title text NOT NULL,
  city text,
  country text,
  cover_url text,
  start_date date,
  end_date date,
  status text DEFAULT 'bucket_list'
    CHECK (status IN ('bucket_list', 'upcoming', 'active', 'past')),

  -- Tracking
  rating int CHECK (rating BETWEEN 1 AND 5),
  notes text,
  budget numeric,
  budget_currency text DEFAULT 'EUR',

  -- Relations
  goal_id uuid REFERENCES goals ON DELETE SET NULL,

  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
)

travel_photos (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  trip_id uuid REFERENCES travel_trips ON DELETE CASCADE NOT NULL,
  org_id uuid REFERENCES organizations NOT NULL,
  url text NOT NULL,
  caption text,
  taken_at timestamptz,
  position int DEFAULT 0,
  created_at timestamptz DEFAULT now()
)

travel_highlights (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  trip_id uuid REFERENCES travel_trips ON DELETE CASCADE NOT NULL,
  org_id uuid REFERENCES organizations NOT NULL,
  day_number int,  -- jour 1, 2, 3...
  content text NOT NULL,
  created_at timestamptz DEFAULT now()
)
```

---

## Layout de la page

```
┌─────────────────────────────────────────────────────────┐
│  HEADER                                                  │
│  "Travel" (text-2xl font-bold)                          │
│  "Collect places. Live fully."  (text-sm tertiary)      │
├─────────────────────────────────────────────────────────┤
│  STATS ZONE                                              │
│  Trips · Cities · Countries · Avg rating                │
├─────────────────────────────────────────────────────────┤
│  SECTION 1 — Upcoming Trip / Active Trip                │
│  ┌──────────────────────────┬───────────────────────┐   │
│  │  2/3 infos du trip        │  1/3 map intégrée    │   │
│  │  ville, dates, "28 days" │  pin sur la ville    │   │
│  └──────────────────────────┴───────────────────────┘   │
├─────────────────────────────────────────────────────────┤
│  SECTION 2 — Bucket List                                │
│  Grid de cards (cover + ville + pays)                   │
├─────────────────────────────────────────────────────────┤
│  SECTION 3 — Past Trips                                 │
│  Timeline antéchronologique                             │
└─────────────────────────────────────────────────────────┘
```

**Pas de right panel** — la map de l'Upcoming Trip joue ce rôle.

---

## Features V1

### 1. Création de trip
- Modal : titre, ville, pays, dates start/end, cover photo (upload optionnel), status initial
- Picker goal lié (optionnel) — ex: "Visiter 10 pays cette année"

### 2. Sections
- **Upcoming Trip** : trip dont `start_date > today` le plus proche. Affiché grand avec map.
- **Active Trip** : si un trip a `start_date <= today <= end_date` — remplace l'Upcoming.
- **Bucket List** : tous les trips avec `status = 'bucket_list'`, grid de cards.
- **Past Trips** : tous les `status = 'past'`, timeline antéchronologique.

### 3. Detail page `/perso/travel/[id]`
- Cover photo (hero) + location + dates
- **Photo gallery** : grid d'images, click → lightbox
- Rating (1-5 stars, pour past trips)
- Notes libres (markdown)
- **Day-by-day highlights** : timeline jour 1 → jour N avec ce que tu as fait
- **Linked journal entries** : auto-fetched depuis Journal entries dont `entry_date` ∈ [start_date, end_date]
- **Budget** : total avec currency

### 4. Map intégrée
- Lib à décider : Leaflet (gratuit) ou Mapbox (free tier généreux)
- Pin sur la ville du trip
- Optionnel : ajouter des POI (points of interest) sur la map

### 5. "28 days" countdown
- Sur l'image de l'Upcoming Trip, badge "28 days" (calcul `start_date - today`)
- Pas d'animation countdown (l'audit interdit les loops visibles)

---

## Cross-module

- **Goal lié** : `goal_id` FK
- **Journal** : auto-link des entries faites pendant le voyage
- **Books** : option future — books lus pendant le voyage (lecture loisir)
- **Watching** : option future — films vus dans l'avion

---

## API externes possibles

- **Map** : Leaflet (OpenStreetMap, gratuit) ou Mapbox (50k tiles/mois gratuit)
- **Photos** : upload Supabase storage ou Cloudinary
- **Currency conversion** : open.er-api.com (gratuit) si tu veux comparer budgets

---

## Architecture (à confirmer au moment du build)

```
src/modules/travel/
├── service.ts
├── types.ts
├── hooks/
│   ├── query-keys.ts
│   ├── useTrips.ts
│   ├── useTrip.ts
│   └── useTripPhotos.ts
└── components/
    ├── TripCard.tsx
    ├── UpcomingTripHero.tsx       ← layout 2/3 + 1/3 map
    ├── BucketList.tsx
    ├── PastTripsTimeline.tsx
    ├── TripDetailPage.tsx
    ├── TripPhotoGallery.tsx
    ├── TripDayHighlights.tsx
    ├── TripMap.tsx                 ← composant map réutilisable
    ├── AddTripModal.tsx
    └── TravelEmptyState.tsx
```

---

## Status

- 🔲 Pas construit
- 📝 Spec extraite de `project_os_vision.md` mémoire
- ❓ Couleur accent à valider (Indigo `#6366f1` proposé)
- ⏭ V2 brainstorm à faire avant build (dans `hq/modules_version_2/travel-v2.md`)
