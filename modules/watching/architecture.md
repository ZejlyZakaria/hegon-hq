# Watching — Architecture

> **Ce fichier ne décrit pas le code. Il énonce ce que le code doit garantir.**
> Réécrit le 2026-09-13 au format du template, chaque point d'étranglement **vérifié dans le code**
> (fichier + ligne). L'état d'adoption, les régressions, les mesures : jamais ici — dans
> [`audit-2026-09.md`](audit-2026-09.md). Le *pourquoi* daté : [`decisions.md`](decisions.md).
> La version de juillet, avec son raisonnement et ses sections d'état périmées :
> [`hq/archive/watching-architecture-2026-07.md`](../../archive/watching-architecture-2026-07.md).
>
> **La loi qui a coûté le plus cher à apprendre :** un invariant vit à un point d'étranglement (un
> seul endroit qui le *force*), jamais dans la mémoire d'un call site. Toute la saga du module est
> une seule maladie rejouée — une règle écrite en commentaire, puis violée vingt lignes plus bas.

## Les invariants

| # | L'invariant | Son point d'étranglement (vérifié 2026-09-13) | La faute qu'il évite |
|---|---|---|---|
| a | **Un statut est une dérivation pure**, jamais un objet assemblé à la main. `watched` / `in_progress` / `paused` / `dropped` / `want_to_watch` / `reference` se dérivent de la position et des faits du monde. | `lib/watch-status.ts` (`claimedStatus`, `addStatusPatch`, `markWatchedPatch`, `markCaughtUpPatch`, `positionPatch`…) + `lib/series-state.ts` (`seriesState`, `nextStep`, `caughtUpAt`…) — **functional core, zéro I/O**. Les hooks transportent le patch, `useWatchActions` est l'unique écrivain côté app ([[reference_watching_one_writer]]). ⚠️ **Le cron `watching-series-sync` est HORS de ce point d'étranglement** — il écrit `watched`/`in_progress` en direct (`index.ts:322-338`) : écart traité par l'audit, axe 3. | La ligne limbo (aucun statut vrai) ; « Watched » sur un écran, « Dropped » sur le suivant ; une série terminée dé-vue par un hoquet TMDB (Haikyu!!, 12/09). |
| b | **Une écriture est un parse qui REFUSE l'illégal** — pas « valide », refuse. Exactement un statut vrai à l'insert ; une position porte son `caught_up_at` ; une série qu'on regarde a une position. | `schemas/media.schema.ts` (`insertMediaSchema`, `updateMediaSchema`, `superRefine`) parsé **dans le tuyau** : `service.ts:283` (`updateMediaItem`) et `:331` (`insertMediaItem`) — pas seulement dans les hooks. Parse de validation seule : on écrit `data` brut, le parse ne fait que jeter. | Une porte future (ou pas encore écrite) qui contourne le hook ; une position écrite sans recalcul de `caught_up_at` (l'Undo du « +1 »). ⚠️ Piège : `updateMediaSchema` **strippe** les clés non déclarées — déclarer = préserver. |
| c | **Les coordonnées sont une lentille unique.** Aucune surface ne lit `current_season` / `current_episode` / `season_*` / `cour_*` brut pour afficher ou stamper ; elle consomme un `MediaView`, seul traducteur entre l'espace de stockage (TMDB à plat) et l'espace d'affichage (cours), en lecture **et** en écriture (`writeYear`, `writeRating`, `toStorage`, `fromStorage`). **Traduire tard** : les fonctions pures reçoivent des unités VUE + la vue, et convertissent au bord (`decisions.md` 2026-07-20). | `lib/media-view.ts` (`buildMediaView`) + `hooks/useMediaView.ts` ; **garde ESLint `NO_RAW_COORDINATE_READ` au niveau `error`** sur `components/**` et `app/(main)/perso/watching/**` (`eslint.config.mjs:109-154`), `components/stats/**` exempté par décision (`decisions.md` 2026-07-XX). `lib/media-view.test.ts`. | L'overlay « oublié » sur une surface (StatusCard à plat sur Blue Lock) ; les années d'un anime écrites dans `season_years` et lues depuis `cour_years` — deux colonnes qui ne se parlent pas (bug du 20/07). |
| d | **Un fait a un seul propriétaire.** Faits du monde (`status`, `season_aired`, `season_end_dates`…) → le cron de sync. Ta posture (`current_*`, stance, `*_years`, `*_ratings`) → toi. Colonnes disjointes. Tout ce qui est dérivable (`seriesState`, caught-up, last watched) est **calculé à la lecture**, jamais stocké ([[reference_series_aired_vs_announced]]). | La séparation des colonnes dans `types.ts` (commentaires « world facts » vs posture) + `lib/series-state.ts` pour les dérivations. **FRAGILE** au sens du chantier : rien n'empêche mécaniquement le cron d'écrire une colonne de posture — c'est précisément la faute (a) du 12/09. Cliquet à poser : voir l'audit, axe 3. | Deux copies d'une série qui divergent ; un booléen figé qu'on oublie de rebasculer. |
| e | **La sécurité vit dans la base.** L'isolation est garantie par la RLS `org_isolation` (`my_orgs()`), pas par du code applicatif. Les tables de référence partagée (`trending_cache`, `for_you_cache`, `anime_cours`) = lecture globale, écriture service-role, sans `org_id`. Garde démo read-only sur les tables mutables. | Les policies RLS Postgres (baseline `supabase/migrations/`), le split de clés (CLAUDE.md §6 bis). Un check applicatif oublié n'ouvre aucune brèche. | Une fuite cross-org par un `select` sans filtre. |
| f | **La possession est décidée par `tmdb_id`, pas par `type`.** Le type (`film`/`serie`/`anime`) est un indice de routage deviné (`tmdbResultType`), jamais une clé de possession. Un stub `is_reference` n'est pas possédé. | `service.ts:645` `findOwnedMediaId` et `:678` `findOwnedMediaIds` — aucun `.eq("type")`, filtre `tmdb_id` + `is_reference` seulement. `lib/possession.ts` porte la règle pour les listes mixtes (⚠️ TMDB réutilise ses numéros entre films et séries). `lib/possession.test.ts`. | Un anime rangé en `serie` échappe à la recherche → discover le croit neuf → **doublon** (l'index unique `(user_id, tmdb_id, type)` le permet). |
| g | **Un module ne touche jamais les tables d'un autre.** Watching n'écrit pas dans Goals ou Habits ; il appelle une fonction qu'ils exposent, au bord de l'écriture. Pas de bus d'événements (`decisions.md` 2026-07). | `lib/sync-goals.ts`, `lib/sync-habits.ts`, appelés dans l'`onSuccess` de `useAddMedia`, `useUpdateMedia`, `useDeleteMedia`. **FRAGILE** : trois call sites, pas un point de fan-out unique — un 4ᵉ hook d'écriture pourrait oublier l'appel. | Le monolithe modulaire qui pourrit en boule de boue. |

**La règle qui décide si une ligne appartient à ce tableau :** peux-tu nommer l'endroit qui
l'empêche d'être violée ? Si non, ce n'est pas un invariant, c'est un commentaire — et il est
étiqueté **FRAGILE** ici, comme dans la doctrine globale.

## Ce qui est délibérément EXCLU

- **Pas de types « branded »** pour les coordonnées — le changement de props (`view: MediaView`) + le
  garde ESLint obtiennent la même contrainte à un dixième du bruit.
- **Pas de SSR sur les pages grille** — CSR gardé, `decisions.md` 2026-07-21 (le gain ne joue qu'à la
  première visite ; un SSR exigerait une seconde implémentation des lectures).
- **Pas de bus d'événements** — orchestration explicite au bord de l'écriture.
- **Pas de `series_facts` immédiat** — différé, déclencheur dans `backlog.md`.
- **Pas de « registre d'invariants » abstrait** — sept barrières concrètes valent mieux qu'une
  abstraction.
- **Stats lit les coordonnées TMDB brutes** — exemption voulue du garde (c), c'est un calcul agrégé
  invariant par espace, pas un affichage de position.

## Performance · Tests · Sécurité

Pas de sections séparées : ce sont des invariants de ce module. Le **seul pattern perf
architectural** du module — *déplacer la latence dans un cron, lire une ligne de cache*
(`trending_cache`, `for_you_cache`, `anime_cours`) — est devenu la règle globale R3/R6 de
[`hq/rules/system-design.md`](../../rules/system-design.md). Les paliers de cache sont R4
(`STALE.*`), l'identité est R2 (`useCurrentUserId()`). La méthode de mesure :
`node scripts/measure-cache.mjs /perso/watching/movies --min-posters=40`.
