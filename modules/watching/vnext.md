# HEGON Watching — V2 Spec

> Source de vérité pour la V2 du module Watching. Décisions prises pendant la session brainstorming du 2026-05-19 entre Zakaria et Claude.

---

## 1. Vision & positionnement

**L'âme : médiathèque profonde curée + mémoire émotionnelle des films/séries qu'on aime.**

Ce n'est PAS un diary chronologique stressant. Ce n'est PAS un tracker épisode par épisode. C'est une bibliothèque où chaque média est un objet curé avec sa propre histoire personnelle (best episodes, notes, rating fin), enrichie de listes thématiques, de recommandations, et de stats.

**Pitch LinkedIn :** *"Letterboxd te demande de noter chaque film. HEGON te laisse construire ta médiathèque comme un cinéphile construit sa bibliothèque — avec les meilleures pièces mises en valeur, et une rétrospective annuelle qui te fait wow."*

**Apps de référence :**
- **Letterboxd** — ratings, custom lists, year in review
- **Trakt** — tracking (mais on REJETTE le tracking ep par ep)
- **Spotify Wrapped** — référence absolue pour le Year Wrapped
- **Notion watch tracker** — curation, cross-module

**Décision clé : PAS de tracking épisode par épisode.** Raison : Zakaria regarde 5 épisodes à la fois (animes, sitcoms type Friends). Tracker S04E12 précisément n'a aucune valeur pour ce pattern. On garde un `current_episode` simple (où tu en es) pour les stats, et des "Best Episodes" pinned manuellement.

---

## 2. État actuel (V1 — déjà construit, design solide ~80%)

D'après le code + capture d'écran 2026-05-19 :
- 3 types : film / série / anime
- 4 tabs : Movies / TV Shows / Animes / Library
- Sections : Don't Miss (trending), My Top 10, Recently Watched, Want to Watch, In Progress
- Status flags : in_progress, recently_watched, want_to_watch, watched
- `current_episode` / `current_season` pour séries/animes
- `is_favorite` (❤️ visible sur les cards)
- Slide-over au clic actuel : cover + titre + rating + status + episode + notes
- Stats inline header : "127 films · 23 séries · 8 en cours"
- Données via TMDB (proxy `/api/tmdb` existant)

**Design V1 = très solide.** On garde 80%, on ajoute les features V2 sans refondre le layout.

---

## 3. Features V2 — lockées

| # | Feature | Effort | Signature |
|---|---|---|---|
| 1 | **Year Wrapped** (récap annuel story-format) | Élevé | 🥇🔥 PHARE |
| 2 | Detail Page `/perso/watching/[id]` | Moyen | 🥈 |
| 3 | Best Episodes (séries/animes) | Moyen | 🥉 |
| 4 | Custom Lists (5e tab) | Moyen | ✅ |
| 5 | What to Watch Next (heuristique) | Moyen | ✅ |
| 6 | Stats riches + rating /10 décimal | Bas | ✅ |
| 7 | Cross-module (Goals/Habits/Journal/Books) | Bas | ✅ |
| 8 | Slide-over quick actions (sans rating) | Bas | ✅ |
| 9 | Animations Linear-grade | Moyen | ✅ |

---

## 4. 🥇 FEATURE PHARE — Year Wrapped

> Zakaria : *"l'idée de year wrapped est la meilleure chose proposée dans watching, wooow. Il doit avoir un design 11/10 avec la meilleure animation possible."*

**Concept :** comme Spotify Wrapped. Récap visuel de l'année en Watching, format **story** (cards verticales qu'on parcourt), animations séquentielles, export shareable.

**Contenu (1 card par moment) :**
1. Intro : "Your 2026 in Watching" + nombre total films/séries
2. "You watched **127 films**" (compteur animé)
3. "Top genre : **Drama** — 45 films" (avec visuel)
4. "Best month : **October** — 18 films"
5. "Your highest rated : **Interstellar 10/10**" (poster en fond)
6. "~**280 hours** watched" (= X jours non-stop)
7. "Longest streak : **47 days**"
8. Card finale : résumé partageable + bouton "Share / Export PNG"

**Ambition design (11/10 demandé) :**
- Format story vertical (tap/click pour avancer, comme Instagram/Spotify)
- 1 stat = 1 plein écran avec typographie bold massive
- Gradient background unique par card (cohérent avec accent violet Watching)
- Animations à la révélation : compteurs qui montent, posters qui slide, texte qui apparaît en cascade
- Spring physics partout
- Card finale exportable en PNG (html-to-image ou canvas) pour partage social
- Accessible toute l'année comme "2026 so far" (pas seulement en décembre)

**Pourquoi c'est la feature phare :**
- Contenu LinkedIn/Insta viral organique (tout le monde partage ses chiffres)
- Spotify a prouvé que ce pattern crée de l'engagement émotionnel massif
- Réutilisable plus tard sur TOUS les modules (Books wrapped, Goals wrapped, Habits wrapped) → série de contenu
- Différenciateur fort vs Letterboxd (leur "year in review" est statique et moche)

**Note :** livrable V2.5 (pas critique pour la TOUTE première vidéo, mais prioritaire juste après — c'est l'arme LinkedIn).

---

## 5. Detail Page `/perso/watching/[id]`

**Décision : Detail Page (pas sliding panel)** car interaction longue + sections multiples + URL shareable. Cf. `navigation.md §6`.

Structure (brouillon — détails à finaliser au build) :
```
HERO        : Cover backdrop large + Titre · Année · Genre · Runtime · Rating
              Boutons : [Status ▾] [Rating slider] [♥ Favorite]
DESCRIPTION : Synopsis TMDB + Réalisateur / Cast
MY TAKE     : Notes libres (markdown, auto-save)
BEST EPISODES : (séries/animes) cards pinned + bouton Add
IN LISTS    : lists où ce média apparaît
SIMILAR     : 3-5 médias de ta library avec genres match
CONNECTIONS : Goal lié · Book lié (based_on) · Journal mentions
```

Refinement de l'ordre/contenu **au moment de l'implémentation** (Zakaria veut voir le design avant de finaliser).

---

## 6. Slide-over (quick actions depuis card)

Au clic sur une card (pas le titre) → slide-over léger pour actions rapides :
- Mark as Watched / In Progress / Want to Watch
- Add to Favorite (♥)
- Add to a list (dropdown)
- Open full detail page
- Delete (avec modal de confirmation)

**PAS de quick rating** dans le slide-over. Raison : le rating se fait sur progress bar /10 (permet 7.5), impossible avec des étoiles rapides. Rating uniquement dans la Detail Page.

---

## 7. Rating system — /10 décimal

**Schema change :**
```sql
-- Avant : rating int CHECK (1-5)
-- Après : rating numeric(3,1) CHECK (rating >= 0 AND rating <= 10)
```
- Échelle : **0 à 10, step 0.5** (19 valeurs : 0.5, 1.0, ..., 10.0)
- UI : slider/progress bar horizontal avec graduations aux .5, valeur live "7.5/10", accent violet, snap animé

---

## 8. Best Episodes (séries/animes uniquement)

**Concept :** au lieu de tracker tous les épisodes, on pin manuellement les MEILLEURS épisodes avec rating + commentaire. 100% dépendant de TMDB (qui fournit les thumbnails via `still_path`).

**Schema :**
```sql
-- RÉFÉRENCE partagée (pas d'org_id, données publiques TMDB)
tmdb_episode_cache (
  tmdb_id int, season int, episode int,
  episode_title text, still_path text, air_date date, overview text,
  cached_at timestamptz,
  PRIMARY KEY (tmdb_id, season, episode)
)

-- PRÉFÉRENCE (org_id, créée par user)
episode_highlights (
  id uuid, media_id uuid REFERENCES media_items, org_id uuid,
  season int, episode int,
  rating int CHECK (1-5),   -- ou numeric pour /10 cohérent ? à décider au build
  comment text,
  created_at timestamptz
)
```

**Fetch TMDB :** lazy (seulement quand user ouvre le modal pour une série), cache 6 mois, via `/api/tmdb` (auth + rate limit existants).

**Fallback animes manquants :** TMDB ne couvre pas tous les animes. Si pas de données → highlight sans thumbnail (juste S/E + comment + rating).

**UX d'ajout :**
```
Section "Best Episodes" → bouton [+ Add a highlight]
→ Modal :
   Étape 1 : Chips de saisons (Season 1, 2, 3...)
   Étape 2 : Grid d'épisodes de la saison (thumbnail + S01E05 + titre + air date)
   Étape 3 : Click épisode → mini-form inline (rating + comment) → [Pin]
→ Animation : la card "vole" du modal vers la section (layoutId)
```

---

## 9. Custom Lists (5e tab à côté de Library)

**Schema :**
```sql
media_lists (
  id uuid, user_id, org_id,
  name text, description text,
  cover_url text,          -- 3 options ci-dessous
  cover_media_id uuid,     -- si cover = un média de la liste
  position int, is_pinned boolean,
  created_at, updated_at
)
media_list_items (
  list_id uuid, media_id uuid,
  position int,            -- drag-drop reorder
  note text,               -- commentaire par item
  added_at timestamptz
)
```

**Décisions lockées :**
- ✅ Commentaire par item
- ✅ Drag-drop pour réordonner
- ❌ Pas de rating de liste
- ✅ **Cover 3 options** : (1) auto = 1er film, (2) custom upload, (3) choisir un autre film de la liste

---

## 10. What to Watch Next (heuristique, no AI)

Nouvelle section entre Trending et In Progress. 4 sous-sections (SQL pur, zéro coût) :
- **From your Want to Watch** (priorité haute, sortis récemment)
- **More like [favorite]** (genres de tes films 5⭐/10)
- **Quick wins** (films <90min de ta liste)
- **Forgotten gems** (Want to watch non regardés depuis 6 mois)

**V3 (quand budget ou Groq free) :** une section "AI recommendation" qui prend library + ratings → LLM → reco en 1 paragraphe.

---

## 11. Stats

Zone stats en top (enrichie). Calculs :
| Stat | Pertinence |
|---|---|
| Films watched this year | 🔥 |
| Series finished this year | 🔥 |
| Episodes watched this year (= SUM current_episode) | 🟡 estimation |
| Hours watched (= runtime × completion) | 🟢 fun |
| Avg rating per category | 🔥 |
| Most-watched genre | 🟡 |

Toutes ces stats alimentent aussi le **Year Wrapped**.

---

## 12. Cross-module connections

| Module | Connexion | Implémentation |
|---|---|---|
| **Goals** | "Regarder 50 films en 2026" | `goal_id` FK + compteur progress |
| **Journal** | Mentionner un film dans une entry | référence depuis Journal (à voir Journal V2) |
| **Habits** | "Regarder un film chaque 15j" | habit avec lien optionnel |
| **Books** | Film based on a book lu | `media_items.based_on_book_id` FK → books |

**Books connection (HEGON-only différenciateur) :**
- `media_items.based_on_book_id uuid REFERENCES books ON DELETE SET NULL`
- Detail page film : picker "Based on a book ?" → cherche dans library
- Detail page livre : badge "Adapted into [film]"
- Goal "Explore 30 stories" peut compter films + livres

---

## 13. Sections order (page principale)

```
1. Trending (ex-"Don't Miss" — RENOMMÉ)   ← découverte externe, design featured
2. What to Watch Next (NEW)                ← reco perso heuristique
3. In Progress                             ← action immédiate
4. Recently Watched                        ← mémoire récente
5. Want to Watch                           ← watchlist (priority badges)
6. My Top 10                               ← showcase permanent (rarement changé → fin)
```

---

## 14. `watched_at`

- Field optionnel sur `media_items` (`watched_at date`)
- Auto-rempli quand status → "watched" pour les NOUVEAUX ajouts
- Médias existants (127 films déjà là) : reste NULL (pas de retroactif)
- Sert aux stats, pas de diary view dédiée (coupée V2)

---

## 15. Animations & UX (Linear-grade)

- Pin best episode → card vole du modal vers la section (`layoutId`)
- Mark as watched → card fade + slide vers "Recently Watched"
- Hover card → lift 4px + scale 1.02 + shadow douce
- Rating slider → snap aux .5 avec spring
- Trending card expand → contenu fade (texte/boutons), pas la taille
- Drag dans une list → ghost preview
- Switch tabs → underline qui glisse (déjà fait V1)
- **Year Wrapped → animations séquentielles à la révélation (priorité 11/10)**

---

## 16. Décisions tranchées

### ❌ Coupé / refusé
| Feature | Raison |
|---|---|
| Tracking épisode par épisode | Pattern d'usage de Zakaria (5 ep à la fois) → inutile |
| Diary view chronologique | Médias existants sans watched_at → vue vide. Recently Watched suffit |
| Rating par liste | Overkill |
| Quick rate dans slide-over | Étoiles incompatibles avec /10 décimal |

---

## 17. Cleanup pendant l'implémentation V2

- [ ] Virer les cards "+ Add" (prennent trop d'espace) → bouton "+" discret en header de section
- [ ] Renommer "Don't Miss" → "Trending"
- [ ] Réordonner les sections (Top 10 à la fin)
- [ ] Migration rating int(1-5) → numeric(3,1) 0-10
- [ ] Ajouter `is_favorite` si pas déjà en DB (semble présent vu les ❤️)
- [ ] Ajouter `watched_at` + `based_on_book_id` sur media_items

---

## 18. Top 3 signature pour vidéo LinkedIn "HEGON Watching"

🥇 **Year Wrapped** — l'arme virale, contenu shareable, émotionnel
🥈 **Detail Page + Best Episodes** — profondeur cinéphile unique
🥉 **What to Watch Next + Cross-module** — "HEGON sait ce que tu veux voir ET pourquoi"

### Angle vidéo
> "Letterboxd lets you log films. Trakt tracks episodes. But neither remembers your year like this."
> → démo Year Wrapped en story format → "And it knows this feeds your goal of watching 50 films in 2026."

---

## 19. Achievements du module (cf. `achievements-v2.md`)

> Le Year Wrapped Watching (§4) alimente le Wrapped global. Voici les achievements propres au module.

| Achievement | Type | Paliers |
|---|---|---|
| First Watch (1er média loggé) | événementiel | — |
| Cinéphile (films watched) | tiered | 100🥉 / 250🥈 / 500🥇 / 1000💎 |
| Binge Master (finir une série) | événementiel | — |
| Critic (médias notés) | tiered | 25🥉 / 100🥈 / 250🥇 |
| Marathon (5 films en 1 jour) | événementiel | — (besoin watched_at) |
| Genre Explorer (films dans 10 genres) | événementiel | — |

---

## 20. `reference` watch_status — ajout TMDB dans les listes (2026-05-23)

**Contexte :** les listes peuvent contenir des films **mixtes** — des films déjà regardés (dans Library) ET des films jamais vus, ni dans Library ni dans Want to Watch (ex : liste "Mind Blowing Films" où tu veux référencer des films cultes pas encore vus).

**Décision :** ajouter un status `reference` dans `watch_status`.

| Status actuel | Affiché dans |
|---|---|
| `watching` / `completed` / `dropped` | Library |
| `plan_to_watch` | Want to Watch |
| `reference` *(nouveau)* | Listes uniquement — nulle part ailleurs |

**Comportement :**
- Le popover "Add title" dans les listes recherche d'abord dans la library, puis dans TMDB
- Un titre TMDB inexistant en DB → créé avec `watch_status = 'reference'`
- N'apparaît pas dans Library, pas dans Want to Watch
- Affiché dans la liste avec badge **"Unwatched"** (distinct de "Plan to Watch")
- Si le user le regarde un jour → il change le status → migre vers Library ou Want to Watch

**Pourquoi pas `plan_to_watch` :** `plan_to_watch` = intention active (je veux voir ça bientôt). `reference` = présence dans une liste thématique sans urgence (mood-based).

**Implémentation requise :**
- [ ] Migration DB : ajouter `reference` à l'enum `watch_status`
- [ ] Types TypeScript : étendre `WatchStatus`
- [ ] Filtres Library/Want to Watch : exclure `reference`
- [ ] Popover "Add title" : ajouter fallback TMDB search après library search
- [ ] Badge UI dans ListDetail : afficher "Unwatched" pour `reference`

**Priorité :** V2.5 — après polish List Detail + What to Watch Next.

---

## Status & next

- ✅ Brainstorm session : 2026-05-19
- ✅ Décisions lockées
- ✅ Section Achievements ajoutée (2026-05-20) — Year Wrapped déjà en §4
- ✅ `reference` status noté (2026-05-23) — ajout TMDB dans listes
- ⏭ Year Wrapped = priorité design 11/10
- ⏭ Implémentation V2 : pas commencée
- ⏭ Prochain brainstorm : Habits / Goals / Books / Journal / Dashboard / Sports

---

# 🧠 BRAINSTORM V3 — 2026-09-04 (idées owner, PAS validées, rien à implémenter)

> Sorti de l'analyse du repo `public-apis` → [[reference_public_apis_repo_verdict]]. **Faisabilité testée
> en exécution**, scope NON tranché. Ne rien construire sans un « go » explicite.

## Idée A — Page Oscars dédiée (filtre par année, lauréats par catégorie)

**Source : Wikidata** (déjà branché côté foot, `football_sync_wikidata`). **Aucune API de récompenses
n'existe** dans public-apis — c'est Wikidata ou rien. Gratuit.

✅ **Testé et ça marche** : `?cat wdt:P31/wdt:P279* wd:Q19020` + qualificatif `P585` (année) sort les
Oscars d'une année, par catégorie, avec le lauréat et `P1686` (l'œuvre récompensée).
⭐ **Le vrai déblocage : les IDs TMDB sont dans Wikidata** — `P4947` (film) et `P4985` (personne).
Donc la page peut se brancher sur `watching.media_items` (vu / pas vu) ET sur les pages personne
existantes `/perso/watching/person/[tmdbId]`. C'est ça qui rend la feature *nôtre* et pas encyclopédique.

⚠️ **Saleté observée dans la réponse réelle (à normaliser, comme pour les Honours foot) :**
- **Doublons** : la récompense est portée à la fois par le FILM et par la PERSONNE
  (« Best Actor → Oppenheimer » ET « Best Actor → Cillian Murphy »). Dédup obligatoire.
- **Labels manquants** : `Q147077` (Emma Stone), `Q165219` (RDJ) ressortent en QID brut — portée du
  `SERVICE wikibase:label` quand la variable est liée dans un OPTIONAL. Corrigeable.
- **Catégories parasites** : « Technical Achievement », honorifiques → whitelister les ~23 catégories.

📐 **Archi imposée** : edge function + cache en base + cron, **jamais de SPARQL live depuis la page**
(exactement le pattern Honours). Wikidata est lent et tombe.

❓ **L'arbitrage non tranché** : 97 ans × 23 catégories = une encyclopédie, pas un second brain — il la
consultera une fois. **L'angle qui vaut le coup = le croisement avec SA bibliothèque** (« 1996 : tu as
vu 4 des 5 nommés Best Picture »). À trancher avant de coder quoi que ce soit.

## Idée B — Cartes « studios » sur la main page → page studio (vu / reste à voir)

**Source : TMDB `discover` + `company`/`network`** — clé déjà en place, logos fournis (`logo_path`).
Le « vu / pas vu » est 100 % notre DB, zéro coût API.

✅ **Testé, volumes réels :** Pixar (`with_companies=3`) **138** · Marvel Studios (420) **103** ·
Ghibli (10342) **83** · A24 (41077) **172** · Walt Disney Pictures (2) **361**.

⛔ **NETFLIX CASSE L'IDÉE — et c'est structurel.** Netflix n'est pas un studio, c'est un
diffuseur : TMDB le modélise en **network** (séries) et non en company. Mesuré :
`with_networks=213` → **2 852 séries** (un mur, « ce qu'il te reste » ne veut plus rien dire) ;
`with_companies=178464` → **32 films seulement** (les films Netflix sont crédités aux boîtes de prod,
pas à Netflix). Les deux chemins sont mauvais. **Idem HBO (376), Disney (361) = catalogue, pas univers.**

⭐ **La ligne de partage** : la feature est excellente sur un **univers d'auteur fini et complétable**
(Pixar, Ghibli, Marvel, A24) où « ce qu'il reste » est un objectif atteignable, et elle s'effondre sur
un **catalogue de diffuseur** (Netflix, HBO, Disney+). ⇒ Si on la fait, c'est **« Universes »**, pas
« Studios », et sur une liste **curée à la main**, pas sur tout ce que TMDB expose.

⚠️ **Filtres obligatoires** : `discover` renvoie le **non-sorti** (Pixar sort « Coco 2 » daté **2029**)
et les **courts-métrages** gonflent les totaux. Borner à `primary_release_date.lte=today` + long-métrage,
sinon le compteur « il te reste X » est faux.
