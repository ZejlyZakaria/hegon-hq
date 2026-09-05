# HEGON — Modules, Connexions & DB

> Matrice complète des modules, graphe de connexions, schéma DB.
> À lire avant de coder un nouveau module ou d'ajouter une connexion cross-module.

---

## 1. Matrice complète des modules

| Section | Module | Accent | Status | Route |
|---------|--------|--------|--------|-------|
| Shell | Dashboard | Blue `#60a5fa` | V1 ✅ | `/dashboard` |
| Life | **Goals** | Green `#22c55e` | V1 ✅ 14/04 | `/life/goals` |
| Life | **Habits** | Rose `#f43f5e` | V1 ✅ 15/04 | `/life/habits` |
| Life | Journal | Orange `#f97316` | V1 ✅ 24/04 | `/life/journal` |
| Life | **Books** | Sky `#0ea5e9` | V1 ✅ 26/04 | `/life/books` |
| Life | Fitness | Lime `#84cc16` (proposé) | 🔲 post-Phase 2 | `/life/fitness` |
| Perso | Watching | Violet `#8b5cf6` | V1 ✅ | `/perso/watching/…` |
| Perso | Sports | Emerald `#10b981` | V1 ✅ | `/perso/sports/…` |
| Perso | Travel | Indigo `#6366f1` (proposé) | 🔲 post-Phase 2 | `/perso/travel` |
| Perso | Culture | Teal `#14b8a6` | 🔲 post-Phase 2 | `/perso/culture` |
| Pro | Tasks | Zinc `#71717a` | V1 ✅ + V2 spec done (`hq/modules_version_2/tasks-v2.md`) | `/pro/tasks` |
| Pro | **Tech** | Purple `#a855f7` | 🔲 PARKED — spec dans `hq/futur/tech-v1.md`, à construire post-CDI | `/pro/tech` |
| Pro | Job Hunt | Yellow `#eab308` (proposé) | 🔲 post-Phase 2 | `/pro/job-hunt` |
| Global | **⌘K** | — | V1 ✅ 27/04 | overlay |

**Notes importantes :**
- Journal : éditeur crème `#f5f0e8` + Caveat/Kalam (exception design system documentée)
- Travel : **quitte Sky** (conflit avec Books) → Indigo `#6366f1` proposé, à valider
- Fitness, Travel, Job Hunt, Culture : specs `v1.md` à écrire avant coding

**Accents à ajouter dans `hq/tokens.css`** (modules spécifiés, pas encore dans le CSS) :
```css
--color-accent-dashboard: #60a5fa;
--color-accent-books:     #0ea5e9;
--color-accent-tech:      #a855f7;
--color-accent-culture:   #14b8a6;
/* À valider : travel = #6366f1 · fitness = #84cc16 · jobhunt = #eab308 */
```

---

## 2. Sections Life/Perso/Pro — statut

**Elles existent dans les URLs actuellement** — ce n'est pas un bug.

**Phase 2** : sections conservées, sidebar groupée par section.
**Phase 3+** : dock plat (une icône par module), sections invisibles visuellement mais **URLs conservées**.

**Ne jamais supprimer les préfixes URL** — casserait le code actuel + les liens internes.

---

## 3. Graphe de connexions

```
Journal = hub central — touche presque tout

Triangle moteur :    Goals → Tasks → Habits
                     (pourquoi → quoi → comment quotidien)

Triangle mémoire :   Books → Culture → Journal
                     (lu → compris → ressenti)

Point convergence :  Travel (Life + Perso = goal + journal + culture)
```

**Matrice directionnelle :**

| Module | Connecté à |
|--------|-----------|
| **Goals** | Tasks, Habits, Books, Fitness, Travel, Job Hunt |
| **Habits** | Goals, Fitness, Journal |
| **Journal** | Habits, Travel, Books, Culture, Goals (hub) |
| **Tasks** | Goals, Tech, Job Hunt |
| **Books** | Goals, Culture, Journal, Watching |
| **Culture** | Books, Journal, Travel |
| **Travel** | Journal, Culture, Goals, Watching |
| **Watching** | Books, Travel, Culture |
| **Fitness** | Habits, Goals, Journal |
| **Tech** | Tasks, Culture, Journal |
| **Job Hunt** | Tasks, Goals, Tech, Journal |
| **Sports** | Watching, Travel, Journal |

**Exemple flow complet :**
`Goal "Lire 10 livres"` → `Habit "30min lecture/soir"` → `Book "tracker progress"` → `Culture "résumé thématique"` → `Journal "ce que j'en retiens"`

**Règle :** avant d'ajouter un module, valider sa position dans ce graphe et documenter ses connexions.

---

## 4. Implémentation cross-module

**Phase 2 (maintenant)** : FKs directes là où c'est simple.

```sql
-- Déjà en place :
ALTER TABLE tasks ADD COLUMN goal_id uuid REFERENCES goals ON DELETE SET NULL;
ALTER TABLE habits ADD COLUMN goal_id uuid REFERENCES goals ON DELETE SET NULL;

-- À ajouter quand le module existe :
ALTER TABLE tech_entries ADD COLUMN goal_id uuid REFERENCES goals ON DELETE SET NULL;
ALTER TABLE books ADD COLUMN goal_id uuid REFERENCES goals ON DELETE SET NULL;
```

**Phase 3-4** : si graphe dense (10+ connexions N↔N), table `connections` générique :

```sql
CREATE TABLE connections (
  id uuid primary key,
  org_id uuid references organizations,
  source_type text, source_id uuid,
  target_type text, target_id uuid,
  connection_type text,
  created_at timestamptz default now(),
  unique(source_id, target_id, connection_type)
);
CREATE INDEX ON connections (source_type, source_id);
CREATE INDEX ON connections (target_type, target_id);
```

**Pourquoi pas maintenant :** FKs directes fonctionnent, sont typées, performantes. Table générique = over-engineering Phase 2.

---

## 5. Cross-module propagation

**Phase 2** : invalidation TanStack Query manuelle sur mutation.

```ts
const complete = useMutation({
  mutationFn: completeHabit,
  onSuccess: (habit) => {
    queryClient.invalidateQueries({ queryKey: HABIT_KEYS.today() });
    if (habit.goal_id) {
      queryClient.invalidateQueries({ queryKey: GOAL_KEYS.detail(habit.goal_id) });
    }
  },
});
```

**Phase 3** : Supabase Realtime postgres_changes → invalidation automatique.
**Phase 4** : Automation Engine = event bus cross-module complet.

---

## 6. Tables DB — état au 2026-04-24

### Schéma `public`
```
organizations, memberships
workspaces, projects, statuses, tags
tasks, activity_log, attachments, comments, task_dependencies, task_tags
goals, goal_milestones
habits, habit_completions
```

### Schéma `watching`
```
media_items
```

### Schéma `sport`
```
PRÉFÉRENCES (ont org_id) :
  football_user_settings, football_best_xi, football_best_xi_players,
  football_legends, user_favorites

RÉFÉRENCE partagée (pas d'org_id — données globales) :
  football_teams, football_competitions, football_next_matches, football_past_matches,
  football_standings, football_team_competitions
  f1_circuits, f1_races, f1_drivers, f1_teams, f1_results,
  f1_driver_standings, f1_constructor_standings, f1_qualification_results
  tennis_matches, tennis_players, tennis_players_cache, tennis_rankings, tennis_tournaments
```

### À créer Phase 2 restante
```
journal_entries (public, org_id)              ← 27 avril
books (public, org_id) + optionnel books_cache ← 2 mai
tech_entries (public, org_id)                  ← 7 mai
```

### À créer Phase 3-4
```
automation_rules, automation_logs             ← Phase 4
connections                                   ← Phase 3-4 si graphe dense
```

### À créer Phase 5
```
colonnes vector(1536) sur tasks, goals, journal_entries, tech_entries, culture_entries
```

---

## 7. Pattern préférences vs référence partagée

Règle établie sur Sports. À appliquer à tous les futurs modules.

| Type | Règle | Exemple |
|------|-------|---------|
| **Préférence** (créée par user) | A `org_id` + `user_id` | `football_user_settings`, `user_favorites`, `books` |
| **Référence** (globale, edge function) | **Pas** d'`org_id` | `football_teams`, `tennis_players`, `f1_races` |

**Application aux futurs modules :**
- **Books** : `books` avec org_id (status, rating, notes) + `books_cache` sans org_id (données Open Library — isbn, cover, pages)
- **Tech** : `tech_entries` a org_id (contenu personnel). Source URLs = champs dans la table, pas de cache partagé V1.
- **Culture/Travel** : TBD à la spec v1.md
