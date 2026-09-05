# Books — V1 Build Spec

> Accent : Sky `#0ea5e9`
> Sous-titre header : "Read deeply. Think clearly."
> Rôle : pendant de Watching pour la lecture. Même logique, même structure, données différentes.

---

## Modèle de données

```sql
books (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id uuid REFERENCES organizations NOT NULL,
  user_id uuid REFERENCES auth.users NOT NULL,

  -- Métadonnées
  title text NOT NULL,
  author text,
  cover_url text,
  external_id text,              -- Google Books volume ID
  year int,
  genre text[],
  total_pages int,
  description text,

  -- Tracking personnel
  status text DEFAULT 'want_to_read'
    CHECK (status IN ('want_to_read', 'reading', 'read', 'abandoned', 'paused')),
  current_page int DEFAULT 0,
  rating int CHECK (rating BETWEEN 1 AND 5),
  notes text,                    -- notes libres
  highlights text[],             -- extraits marquants (ajoutés manuellement)
  favorite boolean DEFAULT false,
  started_at timestamptz,
  finished_at timestamptz,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
)
```

---

## Layout de la page

```
┌─────────────────────────────────────────────────────────┐
│  HEADER                                                  │
│  "Books"  (text-2xl font-bold text-text-primary)        │
│  "Read deeply. Think clearly."  (text-sm text-tertiary) │
├─────────────────────────────────────────────────────────┤
│  STATS ZONE  (4 métriques inline)                       │
│  Reading · Completed this year · Want to read · Avg ★   │
├─────────────────────────────────────────────────────────┤
│  TABS                                                    │
│  Reading · Want to Read · Completed                     │
├──────────────────────────────────┬──────────────────────┤
│  CONTENT  (flex-1)               │  RIGHT PANEL  (w-72) │
│                                  │                      │
│  Liste de livres selon tab actif │  Reading Streak      │
│  (BookCard items)                │  Pages This Month    │
│                                  │  Recently Finished   │
└──────────────────────────────────┴──────────────────────┘
```

**Règles layout (famille Tracker — identique à Goals et Habits) :**
- Fond de page : `#09090b` (zinc-950)
- Pas de stats dans le header — zone stats séparée entre header et tabs
- Pas de tabs dans le header
- Right panel : `w-72 shrink-0`, séparé par `border-l border-white/5`
- Accent couleur : Sky `#0ea5e9` — progress bars, dots streak, highlights actifs
- `"use client"` sur tous les composants interactifs

---

## Features V1

### 1. Ajout d'un livre
Deux modes :
- **Recherche Google Books API** (via proxy `/api/books/search`) : chercher par titre/auteur → sélectionner → données pré-remplies (titre, auteur, cover, pages, description, year, genre)
- **Ajout manuel** : formulaire simple si le livre n'est pas dans l'API

### 2. Tabs principales
- **Reading** : livres `reading`, avec progress bar pages
- **Want to Read** : liste `want_to_read`
- **Completed** : livres `read`, triés par finished_at DESC — vue complète avec search + filtres

### 3. Stats zone (header)
- **Reading** : nombre de livres `reading`
- **Completed** : livres `read` cette année (filtré par finished_at)
- **Want to Read** : nombre `want_to_read`
- **Avg Rating** : moyenne des ratings (livres `read` avec rating)

### 4. Progress tracking
- Champ `current_page` éditable inline sur la card
- Progress = `(current_page / total_pages) * 100`
- Barre de progression accent sky sur chaque card "Reading"

### 5. Detail page `/books/[id]`
- Cover + infos (titre, auteur, date début/fin, pages)
- Changer le statut
- Mettre à jour la page courante
- Rating (1-5 étoiles)
- Notes libres (textarea auto-save)
- Highlights : extraits marquants ajoutés manuellement (liste + input)
- Toggle favorite

### 6. Right panel — BooksRightPanel
- **Reading Streak** : jours consécutifs avec au moins une mise à jour de progression — même pattern que JournalRightPanel (semaine Mo→Su avec dots accent)
- **Pages This Month** : total pages lues ce mois-ci avec progress ring
- **Recently Finished** : 3 derniers livres `read` avec cover + rating

---

## API — Google Books

- **Endpoint** : `https://www.googleapis.com/books/v1/volumes?q={query}&key={API_KEY}`
- **Proxy** : `/api/books/search` (clé serveur uniquement, jamais exposée côté client)
- **Quota** : 1 000 req/jour — suffisant pour usage solo
- **Données récupérées** : volumeInfo.title, authors[0], imageLinks.thumbnail, pageCount, description, publishedDate, categories

---

## Architecture

```
src/modules/books/
├── service.ts              ← getBooks, getBook, createBook, updateBook, deleteBook, updateProgress
├── types.ts
├── hooks/
│   ├── query-keys.ts       ← BOOK_KEYS.all, byStatus, byId, search
│   ├── useBooks.ts         ← useBooks(status?), useBook(id)
│   └── useBookSearch.ts    ← debounced search via /api/books/search
└── components/
    ├── BookCard.tsx
    ├── BookDetailPage.tsx      ← page /books/[id], layout 65/35
    ├── AddBookModal.tsx        ← search Google Books + ajout manuel
    ├── BooksSection.tsx        ← section réutilisable (Reading, WantToRead, etc.)
    ├── BooksRightPanel.tsx     ← streak + pages ring + recently finished
    └── BooksEmptyState.tsx

src/app/(main)/life/books/
├── page.tsx
└── [id]/page.tsx

src/app/api/books/
└── search/route.ts         ← proxy Google Books API
```

---

## Critère de passage

Un utilisateur peut chercher un livre via Google Books, l'ajouter à sa liste, mettre à jour sa progression page par page, consulter la detail page avec notes et highlights, et voir sa bibliothèque organisée par statut avec le right panel de stats.
