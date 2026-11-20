# Watching — Backlog

> **LE fichier vivant du module.** Réécrit le 2026-09-17 (fin de Watching v4) : les douze
> chantiers construits sont sortis, les arbitrages datés sont dans `decisions.md`, les invariants
> dans `architecture.md`, l'état d'audit dans `audit-2026-09.md`. Chaque ligne ici porte son
> **déclencheur** — sans lui, un backlog devient un cimetière. Le statut d'une feature se prouve
> (on l'ouvre), il ne se lit pas.

## ▶️ Ce qui reste de Watching v4 (§1→§8, §11, §12 construits et commités les 14-17/09)

1. **§9 For You v2** — profil de goût structuré (LLM depuis takes / rewatches / highlights), moteur
   déterministe sur `/discover` ; « le modèle est la bouche, pas le cerveau ». **À cadrer avant de
   coder** (une conversation de cadrage à part). Déclencheur : l'owner rouvre For You.
2. **§10 Thèmes films/séries** — les DEUX (miroir du modèle anime) : grand player → « Watch video »
   = iframe YouTube ; scrubber → mp3 self-hosté (Supabase Storage), réutilise le `ThemePlayer`.
   Image = TMDB. Data/thème : `{ label, audioUrl, youtubeId, tmdbImage }`. Gate owner sur la démo
   (audio sous copyright). Déclencheur : l'owner a téléchargé les mp3 d'un thème qu'il aime.
3. **Cartes « Universes »** sur la main page → page univers (vu / reste à voir) — idée owner du
   04/09, **non tranchée**. TMDB `discover` + `company` (logos fournis), le vu/pas-vu = notre DB.
   Déclencheur : l'owner la redemande.
4. **Achievements créables** = gabarits paramétrés (décision 15/09 : ni créateur libre, ni 20
   templates figés) — table par-dessus `stats/achievements.ts` (12 règles forme + paramètres).
   Déclencheur : envie de tweaker un seuil sans redéploiement, ou ouverture des inscriptions.

## 🧾 Dettes mesurées, pas urgentes (déclencheur = la page paraît lente)

- **Page Awards** : **980 KB** mesurés le 17/09 après l'index lean (`getOwnedTitles` 19 colonnes
  + l'étagère par ids : 377 → 230 KB). Ce qui reste, c'est le CANON : `award_entries` lauréats =
  **730 KB en 2 pages** (1 728 entrées) pour 20 cartes et des compteurs. Levier suivant = une
  requête dédiée à ce que la page affiche (3 derniers lauréats par catégorie + couverture seen/total
  calculée en SQL contre la bibliothèque), pas le repli. Déclencheur : la page paraît lente.
- **`getPersonUpcoming`** passe les ids suivis en `in(...)` dans l'URL : casse au-delà de ~1 000
  personnes suivies. Remède = une vue jointe côté SQL.
- **Pochettes iTunes** (Openings & Endings) : un appel par morceau, amorti par le cache local, ne
  bloque plus la liste depuis le 17/09. Remède doctrinal = *cron → table* `theme_covers`.
- **Limites de source connues** : emmys.com groupe par série dans Writing/Directing (nos comptes ≤
  réalité) ; 73 titres Emmy non résolus ; 43 lauréats sans affiche TMDB (téléfilms 1950-90).

## 🔗 Inter-modules — voulus, BLOQUÉS par la condition 3 (déclencheur = audit du module partenaire)

| Lien | Ce qu'il fait | Déclencheur |
|---|---|---|
| **Books ↔ Watching** | `media_items.based_on_book_id` + badge « Adapted into… » sur la fiche livre (spec V2, jamais construit, 0 occurrence) | **Audit Books** fait |
| **Journal ↔ Watching** | mentionner un film/série dans une entrée de journal (spec V2, jamais construit) | **Audit Journal** fait — et ce que Journal sait référencer |
| **Watching → Dashboard** | tuile « nouvelle saison / nouvel épisode disponible » (la moitié dashboard de la notification ; la moitié Watching est dans v4 §1) · **+ « Oscars dans 23 jours, 38 nommés à voir »** (owner 16/09 ; la moitié Watching = la bannière « This year » de la page Awards, `award_ceremonies` + nommés) | **Audit dashboard-os** fait |

> Suggestion pour le 2ᵉ audit (« petit et jugé raté ») : **dashboard-os** débloque la tuile, **Journal**
> débloque le lien journal — dans cet ordre, les trois liens s'ouvrent dans l'ordre où ils comptent.
> Books (4 385 l.) est le plus gros : en dernier.

> **Chaque ligne porte son DÉCLENCHEUR.** Sans lui, un backlog devient un cimetière : on relit une
> ligne, on ne sait plus si elle compte encore, et on la relit à la session suivante.

| Ce qui est différé | Pourquoi pas maintenant | Ce qui le réveillerait |
|---|---|---|
| ⭐ **PRIORITÉ — Attribution d'année d'une saison RÉ-OUVERTE / multi-années.** Une saison finie (stampée son millésime dans `season_years`/`cour_years`) qui **grandit** (nouveaux ép une autre année) ne peut porter qu'**UNE** année. Le `+1` mid-saison date le viewing (`last_watched_at`) mais ne re-stampe rien (par design : on stampe l'année d'une saison **quand on la finit**, `watch-status.ts` « THE TWO LAWS »). Ex. **Bleach** : saison finie en 2023 (40 ép), +10 ép en 2026 → les ép vus en 2026 restent attribués à 2023 → **absent de la vue Stats 2026** (présent en all-time). Le stamp 2023 « fait de l'ombre » à l'activité 2026. | Le fix honnête = attribution **par cour / par date de visionnage** (sous-saison), sur le système d'année qui est **« un seul écrivain », délicat**. Un `+1` qui écraserait 2023→2026 ré-attribuerait les 40 ép de 2023 à 2026 = **bug inverse**. Vrai chantier, pas un patch de fin de session. | Un **2ᵉ titre** concerné (saison finie qui rouvre sur 2+ ans), OU le chantier **anthologie / position non-linéaire** déjà listé ci-dessous (même racine : le modèle stampe au niveau saison, pas épisode) — faire les deux ensemble. |
| Table de référence partagée `series_facts(tmdb_id)` — sortir les 9 colonnes volatiles/synchronisées (`status`, `season_aired`, `season_end_dates`…) de `media_items` vers une table `tmdb_id`-only, pour que 2 comptes qui possèdent la même série ne stockent plus 2 copies qui peuvent diverger. Design fait + revu par Fable (~85 %→solide après 6 corrections), doc archivé `hq/archive/series-facts-design.md`. | Différer ne coûte rien à 1-3 utilisateurs (backfill trivial) ; c'est un vrai chantier (migration + refactor service). | **AniList v2** (le prochain chantier anime touchera de toute façon `tmdb_id`-scoping) **ou** l'ouverture des inscriptions (le vrai coût de la duplication apparaît avec plusieurs comptes qui partagent des séries). Le premier des deux. |
| `theme_favorites` pèse ~8 KB sur **chaque** page Watching, pas seulement `/themes` — le `ThemePlayer` (et sa playlist complète) vit dans le layout, donc la liste part même si l'utilisateur n'ouvre jamais le lecteur. Seul l'état des ♥ est réellement nécessaire avant l'ouverture. | Poids mesuré mais pas prioritaire — pas dans le chemin critique des pages grille. | Une passe perf qui rouvre le layout Watching, ou si le poids grossit avec le nombre de favoris. |
| **iTunes : un appel PAR MORCEAU** pour les pochettes des OP/ED (`service.ts` ~l. 1270, N+1 annoté, R3/R6 — le navigateur parle à un fournisseur pendant un rendu), amorti par un cache localStorage. Le remède doctrinal = *cron → table → app* : une table `watching.theme_covers(tmdb_id, track, cover_url)` remplie une fois par thème. | L'API iTunes n'a pas de lot ; c'est derrière la fiche anime seulement ; le cache rend la 2ᵉ visite gratuite. Une table + migration pour ça n'est pas un patch d'audit. | iTunes qui rate-limite (429 vu dans Sentry), OU une passe qui rouvre le `ThemePlayer`/les thèmes de toute façon (la ligne « thèmes films/séries » ci-dessous). |
| **Vague 2 de `/movies` : `useOwnedMediaIds` attend les résultats trending** (`DontMissSectionClient.tsx:318`) alors que `useOwnedTmdbIds` a déjà tous les `tmdb_id` possédés du type en vague 1. Fusionnable en une seule lecture qui renvoie `{tmdb_id, id, type, is_reference}`. | Les deux lookups n'ont **pas la même sémantique de possession** (par type vs `tmdb_id` seul, stubs exclus — invariant f, `lib/possession.ts`). Fusionner = toucher la règle qui a déjà produit un doublon. 1 requête, ~230 ms de queue réseau, pas dans le premier contenu. | Une passe qui rouvre `possession.ts` (ex. le bug TMDB « id 1399 = film ET série » qui remonte), OU le prochain audit rapidité qui met la vague 2 en tête. |
| **Le cron `watching-series-sync` écrit encore la posture** (`watched`/`in_progress`, `index.ts:322`) — désormais **seulement sur preuve de croissance** (correctif Haikyu!! du 13/09), mais il reste le seul écrivain hors `watch-status.ts` (invariant d, FRAGILE). La forme juste = l'idée owner du 12/09 : **détecter contre la référence, MONTRER, ne jamais réécrire le statut en silence.** ✅ *La surface Watching existe depuis le 14/09 (bandeau Next episode du panneau).* Reste : retirer l'écriture de posture du cron une fois la tuile dashboard construite. | Le gate par preuve ferme le bug connu en attendant ; retirer l'écriture sans la tuile = perdre le seul signal « nouvelle saison » que l'app a aujourd'hui. | La tuile dashboard (audit dashboard-os) — ou une 2ᵉ série dé-vue à tort malgré le gate. |
| **Panneau saison — 3 restes du contre-examen du 14/09** : (1) ouvrir le panneau **S1** refait `tv/<id>/season/1` alors que la requête de détail du montage porte déjà `append_to_response=season/1` (médiane runtime, `service.ts` ~l. 1164) — un doublon pour la saison la plus ouverte ; (2) cour terminal avec `end_episode = null` **et** `episodes = null` : `buildOverlay` pose `full = aired` → `isSeasonComplete` vrai sur un cour en diffusion → pas de bloc *Next episode*, « Watched through » offert à tort (aucune donnée en base ne le déclenche aujourd'hui) ; (3) le fallback vignette d'un épisode sans still = `backdrop_url` en `original` (déjà en cache via le hero, non mesuré). | Aucun des trois n'a de cas réel aujourd'hui ; (1) est une requête de 570 Ko max, en cache 30 min. | (1) : la prochaine passe rapidité de la fiche (amorcer le cache TanStack depuis la réponse de détail) ; (2) : un anime overlay dont AniList ne connaît pas le nombre d'épisodes du dernier cour ; (3) : une mesure qui montre le backdrop `original` téléchargé deux fois. |
| **Don't Miss anime `flex-basis`** — l'accordéon anime une propriété de layout (mise en page à chaque frame sur le thread principal). Mesuré le 15/09 : 60-62 fps, max 18-30 ms, 0 long task → fluide aujourd'hui, **sans marge**. Un 9-10 = reconstruire en `transform` avec largeurs pré-calculées (1-2 h, risque sur le layout). | La mesure ne montre rien à corriger ; ce serait du travail pour un chiffre. | Un saccadement **visible** sur un appareil réel (vieux portable, rail plus lourd). |
| Contrainte type-conditionnelle manquante en base : rien n'empêche aujourd'hui d'écrire `cour_years`/`cour_ratings` sur un film ou une série non-overlay. | Petit trou, pas encore exploité (aucune écriture erronée observée) ; la lentille `MediaView` route déjà correctement en écriture normale. | Une régression observée qui écrit `cour_years` hors contexte anime-overlay, ou l'ajout d'une contrainte DB dans une prochaine migration de toute façon prévue sur `media_items`. |
| Deux irrégularités de chargement repérées le 22/07, **cause non établie** (pas d'A/B avant/après fait) : `library` — son skeleton **disparaît** alors que les données chargent encore (93→0 pulsations avant les données) ; `movies` — **~400 ms de vide** avant que son skeleton n'apparaisse. | Pas encore diagnostiqué — `library` est une route qui n'a pas été touchée récemment, donc probablement pas une régression introduite. | La prochaine fois qu'on touche au chargement de ces deux routes, ou si l'audit rapidité (Audit 3) les fait remonter avec des chiffres. |
| Anthologies (ex. True Detective) : le modèle de position est **linéaire** (un curseur saison/épisode), il ne peut pas exprimer « j'ai vu la S1 et la S4, pas la S2/S3 ». | Trou de modèle identifié mais pas urgent — une seule série connue concernée dans la library actuelle. | Une 2ᵉ anthologie ajoutée à la library qui rend le curseur linéaire visiblement faux à l'usage. |
| Position à corriger manuellement pour 2 titres hérités d'une affirmation invérifiable du script de repair (`One-Punch Man`, `Silo` — `True Detective` déjà classé comme cas d'anthologie ci-dessus). | Correction manuelle ponctuelle, pas un chantier. | La prochaine fois que l'un de ces titres est ouvert/édité. |
| Remède « titre du vrai cour dans le popover » pour Re:Zero et les 3 autres titres cour≠saison (voir `decisions.md`, entrée 2026-07-20) — accepté mais pas codé. | Décision figée, implémentation non prioritaire tant que ça ne « regratte » pas. | Une remarque owner sur la confusion des numéros de saison d'un de ces 4 titres. |

---

## Wave 2 triggers from the 2026-10-11 audit follow-up

| Deferred work | Trigger / required evidence |
|---|---|
| Legacy date reconciliation (prospective YEAR/MONTH/DAY capture is implemented in Wave 2A) | Explicit Wave 2E approval; no precision inference from timestamp shapes. |
| Permanent achievement unlock evidence/date | Historical model approved; Wave 1 corrects calculations only. |
| Kuroko catalogue/personal restoration COMPLETED2026-10-29 | Live TMDB25/25/25, prior personal audit and explicit owner confirmation reconcile75episodes/30h in2018, completedS3E25,8.5scores. See kuroko-reconciliation-2026-10-29.md. Reopen only on new contradictory evidence; no weakening of generic regression guards. |
| Reopening Watching-linked completed Goals after edits | Owner defines completion/reversal policy in Goals audit or Wave 2; active goal exact-count completion fixed now. |
| Legacy TV crew role enrichment | Provider refresh with explicit jobs/created_by; do not infer director from legacy roleless JSON. |

Wave 2C implements prospective scoped rewatches and history-aware analytics; locked and function migration remotely applied/verified2026-10-20. Wave2D is deferred to the next session. Evidence: [wave2c-history-consumption.md](wave2c-history-consumption.md). Remaining legacy reconstruction is explicitly Wave 2E; no automatic backfill.

Wave2D permanent consumption milestones and Goals completion receipts pass final local validation2026-10-28; remote migrations and watchdog reference are verified. Wave2D is now closed for the validated scope: deployed watching-anime-cours-sync v11 source verified against commit d2a041a. Its next cron execution is routine monitoring; no post-deploy runtime proof is asserted. See final-wave2d-review-2026-10-28.md and the proposed wave2e-reconciliation-scope.md. Older dated rollout notes below are historical. [Manifest and evidence](wave2d-milestone-receipts.md). Remaining triggers: explicit2E approval for legacy reconciliation/backfill; proven invalid original receipt plus owner-reviewed audit policy for controlled correction/invalidation tooling. No ordinary live recomputation revokes a receipt.

2026-10-21 owner amendment: accepting existing legacy dates in calendar analytics is implemented now, including pre-2024. It requires no reconciliation/backfill and supersedes the earlier blanket no-legacy-calendar-inference policy. Two targeted pre-review2D issues corrected locally; final review not started. Remote2D migration remains pending owner approval after review.

RESOLVED2026-10-21, final2D review blockers (regressions/re-review documented below): preserve legacy month/day for same-year partial history; retain stable work identity for deleted local titles; align Stats/history/rewatch/cache organization scope with receipts. All three have deterministic local counterexamples in wave2d-milestone-receipts.md. No2E work required.

The three final2D findings are corrected: same-year scoped calendar, stable deleted-local identity, owner/org-scoped Stats/history/rewatch/cache and captured receipt organization.235 targeted tests and re-review pass. Remaining rollout trigger: explicit owner authorization for controlled remote migration20261021000000 and read-only post-checks. No2E started.

Owner-requested date diagnostic2026-10-21 completed read-only. Trigger for controlled legacy date corrections: personal evidence source plus reviewed before/after mapping and explicit write authorization; current signatures alone cannot recover real months. Existing accepted-legacy-date policy remains until owner chooses otherwise. Screenshot layout issue: unknown-duration paragraph is a direct chart-grid child; group with Hours Watched on the next authorized UI correction. Evidence: legacy-date-diagnostic-2026-10-21.md. No2E started or mutation script executed.


Owner-deferred2026-10-28: reconcile Bleach S2 part-by-part viewing years only after a dedicated owner discussion and reviewed mapping. Keep current production history unchanged; abandoned Part4-only migration is outside supabase/migrations, archived in scripts/.shots/deferred-bleach-part4-viewing-year.sql. Do not revive it automatically during2E. Generic Top Picks rated-scope labels remain validated.


Wave2E initial read-only audit completed2026-10-28: remote-only20261028010000 already applied (fresh registry/body/current-row evidence supersedes older never-applied notes), while local folder/reference stop at20261028000000. Trigger for repository alignment: owner review and approval of audit finding, preserving the applied migration without remote replay/repair. Three anime cursor/year conflicts require personal clarification before changing data; current Stats already ignores the four future stamps. Bleach part-year reconciliation remains excluded. Report: wave2e-reconciliation-audit-2026-10-28.md. No automatic cleanup/backfill.


2026-10-29 migration registry alignment RESOLVED: exact archived20261028010000 restored and verified against remote;31/31versions and watchdog watermark match. New proven replay limitation: native local baseline lacks pg_cron, and existing targeted data repairs20261021010000/20261028000000/20261028010000 require historical personal prerequisites. Trigger for fixing full fresh replay: owner-reviewed bootstrap/baseline or data-repair strategy, preserving applied SQL identity and production data. No automatic skip or fake seed. Three anime and Bleach reconciliation remain unchanged. See migration-alignment-2026-10-29.md.


## Separate technical debt - empty-database reconstruction (owner decision2026-10-29)

Not a Wave2E completion gate. Native PostgreSQL lacks Supabase extensions, and three historical data-repair migrations require original personal prerequisites. Preserve applied migration files unchanged; no empty replacement, fake personal seed or artificial registry repair. Trigger: owner explicitly opens a dedicated bootstrap/reconstruction or clean-environment CI workstream. That work must define an auditable schema/data repair replay policy and prove it against an appropriate local Supabase environment. Evidence: migration-alignment-2026-10-29.md. Current2E continues with owner clarification of Tokyo Ghoul, The Seven Deadly Sins and Fire Force; no data correction authorized.


2026-10-29 generic backwards-cleanup correction is locally implemented and validated; new20261029000000 is pending owner application. Trigger for the three residual anime repairs: explicit owner approval of the exact mapping in wave2e-backward-correction-2026-10-29.md after review of the generic mechanism. Existing true positions are confirmed; no need to ask which cursor is right again. Do not fabricate a backwards transition or assume this no-backfill migration cleans already-correct cursors. Bleach excluded; empty-DB reconstruction remains separate nonblocking debt.


2026-10-29 targeted pre-migration review blockerR1: invalid/retained cour identities sharing ordinal and numeric score cause retained active compatibility score loss. Trigger: immediate local fix + regression + re-review before any20261029000000 application. Pending set/dry-run are clean but do not waive business readiness. No production change; three anime untouched. See wave2e-backward-review-2026-10-29.md.


R1 follow-up2026-10-29: local stable-identity fix and exact PostgreSQL regression PASS, including both identity timestamp orders and different-score controls; mutation control proves the old faulty rule fails. READY FOR RE-REVIEW, not remote application approval. Next trigger: fresh targeted re-review/preflight, then owner-run rollout if safe. No three-anime correction. Report: wave2e-r1-fix-2026-10-29.md.


2026-10-29 Wave2E LOCKED / COMPLETED by explicit owner approval: generic migration and separately authorized three-anime cleanup applied via normal versioned CLI push;33/33aligned, no pending. R1, stale maps and Tokyo false calendar resolved with full audit; real-code replay retains all hours and legitimate personal data. See wave2e-final-reconciliation-2026-10-29.md. No further correction queue for these three titles; UNKNOWN and excluded Bleach remain unchanged. Full empty-DB bootstrap debt stays separately deferred until its owner-approved workstream.


## Anime Themes follow-up (2026-11-11)

- Exact provider-ID binding / renamed-song identity migration: trigger is a demonstrated ambiguous title match or favorite split after an upstream rename. Current title/synonym/year selection is tested; do not rewrite existing favorite natural keys without a migration plan.
- More precise iTunes cover verification: trigger is a demonstrated wrong song/artist cover. Current first-result art remains optional and cannot block tracks.
- Durable full theme catalogue mirror: trigger is an explicit requirement to discover never-cached themes during upstream outage. Current24h successful Next cache and process-local failure cooldown do not guarantee cold-instance offline availability.
