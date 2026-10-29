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
| f | **La possession utilise la namespace TMDB : `movie:id` ou `tv:id`.** `serie` et `anime` partagent TV ; `film` reste movie. Un stub `is_reference` n?est pas poss?d?. | `findOwnedMediaId(tmdbId, type)` filtre la namespace ; `findOwnedMediaIds` / `lib/possession.ts` utilisent `mediaIdentity` et `ownedRowFor`. Tests de collision movie/TV. | Un ID num?rique commun qui r?sout le mauvais m?dia ; un anime class? s?rie consid?r? neuf. |
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

## Wave 2A ? prospective historical foundation (2026-10-12)

`media_items` is current state; `viewing_records`/`viewing_ranges` retain personal assertions. The transactional `personal_history_command` is the history-aware write boundary; the revision trigger guards direct personal mutations. Provider/posture changes cannot retract history; explicit correction retains originals. Date components carry actual YEAR/MONTH/DAY/UNKNOWN precision; legacy timestamps are projections only. `history_version = 1` means partial prospective history, never complete reconstruction. Stats remains on legacy reads. The implementation-grade schema, date-path audit, security, transition and rollout contract is [wave2a-history.md](wave2a-history.md).

## Wave 2B: provider and scoped-rating boundaries (2026-10-13)

Provider workers now call the service-only transactional `observe_provider` boundary. It alone advances accepted evidence/cache and can reopen posture on identity/date-proven growth; counts, labels, regressions and restorations cannot. Scoped opinions append via owner-authorized `scope_rating_command`, retaining stable identities and episode membership; ordinal maps are compatibility projections, global rating independent. Page/panel guard the displayed cour ID/range against accepted rating context; RPC additionally binds snapshots/revision under the shared work lock. Migration remains local, rollout unperformed. Schema/manifest/evidence: [wave2b-provider-ratings.md](wave2b-provider-ratings.md).

## Wave 2C: historical consumption (2026-10-18)

Stats selects active FIRST_WATCH/REWATCH evidence and subtracts exact first-watch intervals from legacy coverage, including tombstones only over their removed overlap. COMPLETION adds only uncovered frozen consumption not represented by first-watch, usable legacy or another completion; milestone duration stays unknown. Cour writes require validated snapshot membership rather than aired-count offsets. Dates retain YEAR/MONTH/DAY/UNKNOWN precision. Scoped ratings use stable identity, lineage and evidenced viewing context, never recorded_at as watch time or repeated analytical votes. New rewatches require explicit scope and frozen membership/duration; correction/retraction retains originals. The function-only migration extends personal_history_command for UNKNOWN-date rewatches and optional provider-context locking; applied and verified remotely2026-10-20, with no object-count/RLS or existing-data changes. Full manifest/source-selection/validation: [wave2c-history-consumption.md](wave2c-history-consumption.md).

## Wave2D: prospective permanent milestone observations (2026-10-21)

A server-authenticated fresh canonical snapshot feeds receipt-qualified lower-bound metrics, separate from live legacy-compatible progress. Watching owns immutable achievement_unlocks; Goals owns public.goal_completion_receipts and sticky completion guard, consuming a measured-value contract without querying Watching internals in its writer. No client-supplied eligibility or rounded-percent completion. Trusted service-only RPCs, owner/org RLS, fingerprint/definition checks and unique identities enforce the boundary. First observed satisfied is not an original historical unlock date.10 milestones permanent,2 current-rating achievements live. Local-only migration/schema deltas and exact semantics: [wave2d-milestone-receipts.md](wave2d-milestone-receipts.md).

### Targeted calendar/hours amendment (2026-10-21)

Stored legacy full watch dates are accepted DAY calendar data at every year; no2024 cutoff. Explicit partial dates stay partial; scope year stamps are not moved by a conflicting title date. Shared deduplication avoids repeated FIRST duration across Stats/receipts. Marathoner shares Stats' complete mixed hours ledger from a fresh server snapshot (including fingerprinted legacy rewatch inputs); no client metric authorizes persistence. Other milestone/Goal proof policies remain unchanged. This supersedes prior blanket legacy-date suppression and Marathoner's frozen-duration-only policy. No stored-row rewrite or receipt backfill.

Calendar follow-up2026-10-21: remaining legacy coverage retains its own stored calendarDate even when annual season-year attribution differs, and newer partial history cannot overwrite an older unrelated watched_at. Monthly/daily Activity follows that calendar fact; annual scope-year totals stay unchanged. No conflicting-year hours/episodes are guessed into calendar detail. UNKNOWN scope/year coincidence does not mask legacy coverage. The Activity card fills12 calendar positions only when month-level activity exists. Rating Distribution retains exact score frequencies, shared numeric point/tick coordinates and bounded cubic controls; rounding a maximum upward is forbidden.

Final2D blocker amendment2026-10-21: a scoped assertion's date cannot redate an unrelated legacy remainder even within the same year; standalone partial projections remain partial. workKeyOf is shared by analytics/service/receipt metrics and unwraps retained history identities, including local titles withoutTMDB. Stats/history/ranges/ratings/rewatches take explicit user+org scope; Stats cache keys and receipt/Stats query closures include that captured org and wait for it to exist. SQL receipt scope unchanged. No data or schema changes.


### Nested parts and viewing periods amendment (2026-10-28)

For multi-season anime the lens may expose nested accepted AniList parts without replacing its TMDB seasons. Complete contiguous boundaries must fit those storage seasons and catalogue total; trailing empty announcements add no membership. Unknown/incompatible mapping falls back conservatively. Airing gaps never establish a part. Scoped part/season ratings share the same transactional command, stable keys, catalogue snapshot tokens and personal revision; saved legacy opinions remain readable without claiming verified coverage; proven growth keeps the previous opinion in history, while writes still require current catalogue evidence. History retains old opinions, and annual aggregation excludes overlapping season/part votes. Global title opinion stays independent. First-view season labels derive from personal records plus retained legacy years; a full-title completion extends only its terminal season. Hours still come from the existing deduplicated consumption ledger, never from these labels or release years. Detail period and audit UI share one history query; no per-part fetch. See season-periods-and-parts-2026-10-28.md.


Annual Top Picks eligibility is first-view consumption from the accepted ledger in the selected year; rewatch-only activity cannot qualify even a favorite. All-time keeps its existing eligibility. Both the main Top Picks card and See all use the same computed list; hours/calendar/counts keep the broader activity set. See top-picks-rewatch-fix-2026-10-28.md.


Top Picks annual presentation consumes historicalRatingResult.scopes from the SAME lineage/period/overlap vote selection as its score; historicalRating remains the numeric wrapper. Compact rated subject identifiers and poster selection are independent of activity/hour presentation. No unrated current-season progress is appended. Stable nested/overlay cour identities win over ordinals. See rated-scope-and-bleach-part4-2026-10-28.md.


### Audited backwards correction (2026-10-29)

personal_history_command is the sole series/anime rollback authority. history_intent=correction plus an actual storage-position decrease triggers frozen-membership-aware invalidation, even with coarse known scope completions. It retains supported prefix coverage/date precision, independent rewatches and unknown identities; appends score clears; retires proven-false calendar/membership without changing runtime or legitimate dates. Before/after projections live in read-only owner/org progress_corrections audit, never Stats. TypeScript capture no longer clips backwards series history; UI uses the returned row and invalidates affected caches. Migration20261029000000 is pending owner application before this client path is deployed. See wave2e-backward-correction-2026-10-29.md.
