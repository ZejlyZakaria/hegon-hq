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

- **Page Awards** : 730 KB / 3 requêtes pour 20 cartes (vue `award_entries` faite le 16/09, −28 %).
  Prochain levier = une requête dédiée à ce qu'elle affiche (dernier lauréat par catégorie +
  couverture), pas le repli.
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

