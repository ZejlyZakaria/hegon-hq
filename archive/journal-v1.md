# Journal — V1 Build Spec

> Accent : Orange `#f97316`
> Rôle : espace de réflexion personnel quotidien. Pas de notes de projet (Tasks), pas de tracking de contenu (Watching) — juste toi qui poses quelque chose par écrit.
> Future goldmine pour la recherche sémantique IA (Phase 5).
> Mis à jour : 2026-04-24 (aligné sur OS vision review 2026-04-22)

---

## Modèle de données

```sql
journal_entries (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id uuid REFERENCES organizations NOT NULL,
  user_id uuid REFERENCES auth.users NOT NULL,
  entry_date date NOT NULL DEFAULT CURRENT_DATE,
  title text,                    -- optionnel — par défaut la date formatée
  content text NOT NULL DEFAULT '',
  mood text CHECK (mood IN ('calm', 'good', 'neutral', 'tired', 'rough')),
  tags text[] DEFAULT '{}',      -- tags libres (pas de table séparée)
  word_count int GENERATED ALWAYS AS (
    CASE
      WHEN trim(content) = '' THEN 0
      ELSE array_length(regexp_split_to_array(trim(content), '\s+'), 1)
    END
  ) STORED,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  UNIQUE(user_id, entry_date)    -- une entrée par jour par user
)
```

---

## Features V1

### 1. Écrire une entrée
- Une entrée par jour (date = clé unique)
- Si entrée du jour existe → l'ouvrir directement (pas de doublon)
- Éditeur texte simple avec support Markdown basique (gras, italique, listes)
- Mood picker : 5 états nommés — calm / good / neutral / tired / rough
  - Représentés par 5 petits cercles colorés (pas d'emojis)
  - Couleurs : calm=bleu doux · good=vert · neutral=zinc · tired=orange · rough=rouge
- Tags libres : saisie avec Enter, affichés en chips orange, supprimables
- Auto-save : sauvegarde automatique 2s après arrêt de frappe (pas de bouton Save)
- Titre optionnel : si vide, affiché comme "Mercredi 9 Avril" dans les listes

### 2. Tab Today — vue par défaut
- Stats zone : aucune — le canvas commence directement
- Centre (zone principale) :
  - Mood picker en haut (5 cercles, 1 clic)
  - Éditeur pleine hauteur (fond `surface-1`, police normale, padding `px-8 py-6`, curseur orange)
  - "Today's context" en bas de l'éditeur : pills cross-module read-only auto-générées
    - Habits done today (ex: "3 habits ✓")
    - Tasks completed today (ex: "2 tasks ✓")
    - Goal progress (ex: "Read 10 books — 40%")
    - Ces données viennent des autres modules via TanStack Query, jamais stockées dans journal
  - Tags chips en bas + word count discret
  - Auto-save indicator "Saved ✓" disparaît après 2s
- Right panel (toujours visible) :
  - Streak actuel + best streak
  - Calendar heatmap — mood dot coloré par jour du mois courant
    - calm=bleu · good=vert · neutral=zinc · tired=orange · rough=rouge · vide=zinc-800
  - Légende des couleurs
  - Navigation mois précédent/suivant
- Si pas encore d'entrée du jour → prompt "What's on your mind?" avec mood picker en premier

### 3. Tab All Entries
- Search bar + filtre mood + filtre date range en haut
- Liste complète antéchronologique, scroll infini (pas de pagination)
- Chaque ligne : date formatée + mood dot coloré + première ligne du contenu + word count
- Clic sur une ligne → l'éditeur se charge dans la zone centrale (même layout que Today)
  - "Context at that time" en bas = cross-module pills historiques du jour de l'entrée
    - Charger les données du jour J depuis les autres modules (habits, tasks, goals)
  - Back arrow pour revenir à la liste

---

## UI/UX Requirements

- Route : `/life/journal`
- Layout famille Canvas : 2 zones — Centre (éditeur, ~70%) + Right panel fixe (~30%)
- Right panel identique dans tous les états (Today et All Entries)
- Pas de stats zone au-dessus — le canvas commence dès le haut
- Éditeur : `surface-1` (`#0e0e10`), padding `px-8 py-6`, font normale, curseur couleur accent orange
- Transitions douces entre Today et All Entries
- Empty state premier accès : "Your journal starts here. Write anything." + mood picker

---

## Architecture

```
src/modules/journal/
├── service.ts          ← getTodayEntry, getEntry, getEntries, createEntry,
│                          updateEntry, getStreak, getCalendarData, searchEntries
├── types.ts
├── hooks/
│   ├── query-keys.ts   ← JOURNAL_KEYS.today(), JOURNAL_KEYS.byDate(date), JOURNAL_KEYS.list()
│   ├── useJournalToday.ts
│   ├── useJournalEntry.ts
│   └── useJournalCalendar.ts
└── components/
    ├── JournalEditor.tsx         ← éditeur principal avec auto-save
    ├── MoodPicker.tsx            ← 5 cercles colorés (calm/good/neutral/tired/rough)
    ├── TodayContext.tsx          ← pills cross-module read-only
    ├── JournalCalendar.tsx       ← heatmap mensuelle dans le right panel
    ├── JournalRightPanel.tsx     ← streak + calendar + légende
    ├── JournalEntryList.tsx      ← liste All Entries avec search/filter
    └── JournalEmptyState.tsx
```

---

## Décisions design finales

- **Layout** : Centre éditeur (~70%) + Right panel fixe (~30%) — PAS liste-gauche/éditeur-droite
- **Tabs** : Today · All Entries (le Calendar est dans le Right panel, pas un tab)
- **Right panel** : toujours visible, identique dans Today et All Entries
- **Mood** : 5 états nommés (calm/good/neutral/tired/rough), cercles colorés, pas d'emojis ni numéros
- **Today's context** : pills cross-module auto-générées (read-only, jamais stockées en DB)
- **Zone d'écriture** : `surface-1` (`#0e0e10`), padding généreux, curseur couleur accent orange
- **Auto-save** : indicateur discret "Saved ✓", disparaît après 2s
- **Tags** : chips orange discrètes en bas de l'éditeur
- **Calendar** : heatmap dans le right panel — mood dot coloré par jour, navigation mois par mois
- **Design à faire directement en code** — les générateurs d'images échouent tous sur ce module

---

## Critère de passage V1

Un utilisateur peut écrire son entrée quotidienne en 30 secondes, choisir son mood, ajouter des tags, et retrouver une entrée de la semaine passée via la liste ou le calendrier.
