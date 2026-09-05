# Watching V3 — Roadmap & décisions

> **ÉTAT (2026-07-02) : Couche 1 IMPLÉMENTÉE** (trailer, providers, IMDb, age, OMDb, Episodes+Best Ep fusionnés, player AnimeThemes v1). Heatmap **masquée** (`HEATMAP_ENABLED=false`) en attendant l'Option B (datasets IMDb). Player AnimeThemes **pas encore commité** au 07-02. NEXT = passe UX + favoris player, puis Couche 2. Détail live → `hq/now.md`.


> Spec vivante de la V3 de Watching. Brainstorm structuré session 2026-07-02 (nuit).
> Principe directeur (validé) : **la V3 ne doit pas être "plus de data TMDB", mais "Watching qui te comprend".** Watching = **le module mémoire de tout ce que tu regardes**. Toute feature qui ne sert pas cette mission = écartée.
> ⛔ Le **design** de la fiche detail est figé — on ajoute des features fonctionnelles, on ne refait pas le design existant. Hero features actuelles à préserver : **Stills, Watch History, Best Ep**.

Découpage en 3 couches : 1 = fiche vivante (data, effort faible, polish) · 2 = capture perso (le moat) · 3 = insights/connexions (le payoff). Dépendances : Couche 3 a besoin des données de la Couche 2 ; "pourquoi arrêté" dépend du statut Dropped ; Person pages dépendent du cast stocké en DB.

---

## ✅ COUCHE 1 — FIGÉE (validée intégralement)

| Feature | Décision / Placement |
|---|---|
| **Trailer** | TMDB `append_to_response=videos`, embed YouTube → haut de fiche |
| **Watch Providers** | `/watch/providers` (où streamer, par région) → sous le titre |
| **IMDb link** | via `external_ids` → petit bouton dans la section **Details** (trivial) |
| **Age badge** (TV-MA/R/16+) | `content_ratings`(TV)/`release_dates`(film) → **info dans Details uniquement, JAMAIS sur le poster** (owner refuse les badges sur la library) |
| **OMDb — notes externes** | IMDb / RT / Metacritic, affichées **discrètement** (≠ la note perso = l'étoile). Mappé via l'id IMDb |
| **Episodes → Stills** | Sélecteur de saison (tabs S1/S2…) + grille **2 colonnes / scroll horizontal** (4 ép. visibles, 2×2). Carte = still + `E7 · Titre` + ★note. Tap → résumé. Data TMDB `/tv/{id}/season/{n}` **à la volée, rien en DB**. ⚠️ nouveau pattern 2-lignes (assumé, justifié par la densité) |
| **Episodes → Heatmap** | Grille façon **SeriesGraph** (saisons × épisodes, colorées par note IMDb). **PAS en plein milieu de la fiche** → bouton **📊 "Ratings" → MODAL plein écran** (la grille a besoin d'espace, vue "power" à la demande) |
| ~~Gallery~~ | ❌ rejeté ("null") |

**Source heatmap = OMDb** (vraies notes IMDb par épisode : `?i={seriesImdbId}&Season=N`), pas les `vote_average` TMDB (souvent vides). Donc OMDb sert 2 fois : notes externes (titre) + heatmap (épisode).

**Dépendances techniques Couche 1** : clé **OMDb** (gratuite, ~1000 req/j, fetch à la demande + cache session) · le reste via le proxy `/api/tmdb` existant.

### ⭐ AnimeThemes OP/ED — traité comme SA PROPRE PHASE (pas un quick win)
Ce n'est pas "une section", c'est **un mini-Spotify d'anime themes dans HEGON**. Vision validée par l'owner : **player audio ultra-premium façon Spotify, la vidéo en bonus, avec favoris**.

- **AnimeThemes.moe fournit vidéo (webm CDN) ET audio** des OP/ED → vrai player, pas des images. ✅
- **Insight clé : séparer la LISTE (par fiche) du PLAYER (global persistant)** — sinon la musique s'arrête en quittant la fiche.
- **3 surfaces** :
  1. **Fiche anime → section "Openings & Endings"** = tracklist (OP1/OP2/ED1… · titre · artiste · ▶ · ♡). ▶ lance dans le player global (sans quitter la fiche), pochette = poster anime.
  2. **Player global persistant** = mini-player docké (pochette/titre/artiste/play/progress/next-prev) → tap → **Now Playing** plein écran avec **toggle Audio ⇄ Vidéo** (audio par défaut + pochette, vidéo webm en bonus visuel).
  3. **Favoris "My Themes"** = playlist des ♡, jouable en continu (shuffle/enchaînement).
- **Recherche** : fiche = auto (map anime→id AnimeThemes) ; global = favoris + auto "thèmes des animes vus" (source gratuite, déjà en library).
- ⚠️ Coût : mapper TMDB → id **AniList/MAL** une fois et le **stocker** sur le média (1 colonne). Player global = **état audio Zustand + composant app + gestion de queue** (le vrai boulot). Vérifier la **policy AnimeThemes** (hotlink CDN).

**Décisions ENCORE OUVERTES sur le player (à trancher demain) :**
1. Modèle 3 surfaces validé ? (reco : oui)
2. Player persistant **app-wide** ou **Watching-only** ? (reco : Watching-only en v1)
3. Audio par défaut + vidéo en toggle → confirmer
4. Entrée favoris : onglet "Themes" dans Watching, ou page dédiée ?
5. Phase à part (pas mélangé aux quick wins TMDB) → OK ?

---

## 🔜 COUCHE 2 — le moat (À DISCUTER)
La **capture personnelle**. Pour chaque item : quels choix exacts + **où/quand on capture**.
- **Statut "Dropped"** + **pourquoi arrêté** (too slow / lost interest / bad ending / no time). ⚠️ gap actuel : les séries abandonnées restent en `in_progress`. Point d'entrée proposé pour la Couche 2 (simple + débloque "pourquoi arrêté").
- **Watch context** (avec qui : seul/famille/amis/couple — où : maison/ciné/voyage)
- **Why this film** (source : reco / ami / TikTok / YouTube / Reddit / acteur / réal…)
- **Mood tags** (dark / sad / mind-blowing…) — distinct des `tags`(genres) et des Lists
- **Rewatch** (compteur + dates des re-visionnages). ⚠️ changement de modèle de données (ruisselle dans Watch History + Stats)
- **Note par épisode** (les étoiles "Votre note" par ép., vues sur le viewer IMDb) — parkée ici depuis la Couche 1
- ⚠️ **LE sujet transverse** : l'**UX de capture** — optionnel, 1-tap, chips, skippable. Si on rate ça (mur de formulaire), toute la Couche 2 meurt. Le vrai défi de la V3.

## 🔜 COUCHE 3 — insights & connexions (À DISCUTER)
Le **payoff**, alimenté par la Couche 2. Effort élevé.
- **Person pages** = Actor Journey + Director Journey + Univers partagé → **une seule feature** : page personne (acteur/réal) avec *tes* stats (films vus, heures, note moyenne, classement). ⚠️ dépend du **cast stocké en DB** pour tous les titres (aujourd'hui fetch à la volée → backfill à prévoir).
- **Time Capsule** ("tu as vu Interstellar il y a 5 ans")
- **Nouvelles Stats** dérivées de la Couche 2 (répartition mood / sources / qui-où)

Idées écartées par l'owner (pour mémoire) : Timeline (= déjà Watch History), Souvenir (= review suffit), Gallery.

---

## Reprise demain
1. Trancher les **5 décisions du player AnimeThemes**.
2. Démarrer la **Couche 2** par le **statut Dropped**, puis attaquer **l'UX de capture** (le gros sujet).
3. Décider le **séquençage global** en phases + par quoi on code en premier (reco initiale : Phase 1 = trailer/providers/IMDb/age badge + Dropped ; player anime = phase à part ; capture = phase 2 ; insights = phase 3).
