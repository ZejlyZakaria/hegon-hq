# Diagnostic DB — `watching.media_items` (et tables liées)

> Opus · 2026-07-15 · **Read-only.** 831 lignes, 3 comptes (owner + démo + 1), 427 `tmdb_id` distincts.
> Méthode : remplissage réel par colonne (toutes les lignes) × croisement avec la lecture dans le code.

## Verdict

**La structure est saine.** Après tous les changements récents, il y a **UNE colonne morte**, **UNE grosse redondance structurelle** (déjà au backlog), et **deux dénormalisations mineures** (justifiées comme fallbacks). Le reste des 49 colonnes est utilisé et légitime.

---

## 🔴 1. Colonne MORTE : `recently_watched`

- **Remplie à 100 %** (831/831), **écrite dans ~5 endroits** (`useAddMedia` ×4, `withRecency` dans `useUpdateMedia`).
- **Mais AUCUNE lecture ne décide plus rien.** Depuis que « Last Watched » dérive de `watched_at` et que `resolveTransition` a été purgé de ce drapeau, il n'est plus lu que dans les listes `STATUS_FIELDS`/`STATS_FIELDS` de `useUpdateMedia` — c.-à-d. juste pour déclencher une invalidation, jamais pour une vérité.
- **Action** : migration `DROP COLUMN recently_watched` + retirer les écritures (`useAddMedia:196,277,302,320`, `withRecency`) + le retirer des field-lists de `useUpdateMedia`. Déjà marqué `@deprecated` dans `MediaStateFlags`.

## 🟠 2. Redondance structurelle : les FAITS-DU-MONDE stockés par utilisateur (facteur **1.95×**)

- 831 lignes pour **427 `tmdb_id` distincts** → chaque fait du monde est stocké **~2 fois** (une par user qui possède le titre).
- Colonnes concernées (≈9) : `status`, `season_aired`, `season_episodes`, `season_end_dates`, `season_air_dates`, `season_posters`, `seasons`, `episodes`, `last_synced_at`.
- **C'est exactement `series_facts(tmdb_id)`** (déjà au backlog). Ces colonnes appartiennent à une table de référence **partagée**, `media_items` ne gardant que ce qui est **à toi** (position, stance, années, notes, ratings, watched_at, favorite, priority).
- Bénéfice : ~2× moins d'appels TMDB (déjà entamé côté job via le groupement par tmdb_id), **zéro divergence possible** entre deux copies, et le futur overlay AniList (anime v2) s'y accroche naturellement.
- = migration + refactor service, **à froid**.

## 🟡 3. Dénormalisations mineures (garder pour l'instant)

- **`seasons`** = `season_episodes.length` · **`episodes`** = `sum(season_episodes)`. Toutes deux **dérivables**, lues seulement comme **fallback** (`season_episodes?.length ?? media.seasons`) pour les vieilles lignes sans `season_episodes`. Coût quasi nul, utilité de secours. Candidates à supprimer **quand** `series_facts` normalisera tout ça — pas avant.
- **`season_air_dates`** (début de saison) vs **`season_end_dates`** (fin) : se recoupent mais usages distincts (air[0] = date de sortie S1 pour le plancher de rewatch ; end = fin pour le plancher d'année). **Garder les deux.**

## ✅ Colonnes vérifiées comme légitimes (non exhaustif)

- `rating` (note TMDB, le monde) ≠ `user_rating` (la tienne) — deux sens, garder.
- `priority` (rang Top-10) ≠ `priority_level` (urgence want-to-watch high/med/low) — deux sens, garder.
- `org_id` (tenant, clé RLS `org_isolation`) ≠ `user_id` (qui a ajouté) — 3 distincts chacun, garder.
- `caught_up_at` 3 %, `last_watched_at` 1 %, `drop_reason` 0.5 %, `note_updated_at` 5 % — **conditionnelles**, pas mortes (lues par `seriesState`, `lastWatched`, StatusCard, MyTake).
- `is_reference`, `status`, `studio`, `directors`, `cast_members`, `tags`, `notes` — tous lus.

## Tables liées (santé OK)

`media_lists` 17 · `media_list_items` 166 · `episode_highlights` 663 (marque par user×épisode) · `rewatches` 3 · `trending_cache` 3. Rien d'anormal.

---

## Ordre d'action recommandé

1. **Droper `recently_watched`** (petit, net, sans risque — plus rien ne le lit). Migration + retrait des écritures.
2. **`series_facts(tmdb_id)`** (le gros) — absorbe la redondance 1.95×, `seasons`/`episodes` s'y dissolvent, base de l'overlay AniList. À froid.
