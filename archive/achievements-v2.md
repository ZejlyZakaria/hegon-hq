# HEGON — Achievements & Wrapped (Méta-module)

> Méta-module cross-cutting. Décisions prises pendant la session brainstorming du 2026-05-19/20 entre Zakaria et Claude.
> N'est rattaché à aucun module — vit au niveau shell. Agrège les données de tous les modules.

---

## 1. Vision & positionnement

**Une "Trophy Room" : le hall of fame personnel de ta vie.**

Après 3 ans d'usage, cette destination = ta **capsule temporelle**. C'est l'âme émotionnelle du *"HEGON se souvient de ta vie"*. Probablement la chose la plus différenciante de tout le produit, parce qu'aucun concurrent ne peut l'avoir (aucun n'a tous les modules de ta vie connectés).

**Deux systèmes distincts mais complémentaires :**
- **Achievements** = le JEU. Défis permanents tiered, débloquables n'importe quand. "Cinéphile", "Bookworm".
- **Wrapped Year** = le BULLETIN. Snapshot annuel partageable (Spotify-style), montre ce que tu as débloqué dans l'année.

**Pourquoi c'est une pierre angulaire produit :**
- Rétention : tu reviens pour remplir tes achievements et voir ton wrapped grandir
- Viralité : le Wrapped est conçu pour être partagé (Insta/TikTok/LinkedIn)
- Émotionnel : ta vie résumée à travers tous tes domaines
- Moat : impossible à copier sans avoir tous les modules HEGON

---

## 2. 🎮 Système 1 — Achievements

### 2.1 La ligne critique : Achievements vs Goals

**LE point qui évite la confusion produit :**

| | Goals (module) | Achievements (méta) |
|---|---|---|
| Qui définit | **TOI** | **HEGON** (système, hardcodé) |
| Nature | Personnels, spécifiques | Universels, pour tous les users |
| Exemples | "Regarder Dune", "Lancer HEGON", "50 films en 2026" | "Cinéphile (100 films)", "Marathon (5 films/jour)" |
| Intention | Ce que JE veux accomplir | Récompenses émergentes pour avoir utilisé HEGON |
| Deadline | Toi tu la fixes | Aucune — débloquable à vie |

**Règle d'or : Goals = intentionnel et personnel. Achievements = émergent et systémique.** Ne jamais mélanger les deux.

### 2.2 À-vie + unlocked_at

- Un achievement débloqué reste **à vie** dans la Trophy Room.
- Chaque achievement débloqué porte une date `unlocked_at`.
- Pas de reset annuel frustrant.
- Le **Wrapped** filtre par année : "achievements débloqués EN 2026".

### 2.3 Tiers — quantitatif vs événementiel

Tous les achievements ne sont PAS tierables :

| Type | Tiers ? | Exemple |
|---|---|---|
| **Quantitatif** (compteur) | ✅ Oui | Cinéphile : 100🥉 / 250🥈 / 500🥇 / 1000💎 |
| **Streak** | ✅ Oui | 7j🥉 / 30j🥈 / 100j🥇 / 365j💎 |
| **Événementiel** (one-shot) | ❌ Non | Marathon (5 films en 1 jour) — binaire |

**Tiers (working names) :** Bronze 🥉 / Silver 🥈 / Gold 🥇 / Platinum 💎

### 2.4 Progress visible

Chaque achievement non-débloqué affiche sa progression vers le prochain palier :
```
🥉 Cinéphile · 137/250 vers 🥈
🥈 Bookworm · 18/50 vers 🥇
🔒 Marathon · 0/1 (5 films en 1 jour)
✅ First Watch · débloqué
```
Transforme "surprise aléatoire" en "objectif motivant".

### 2.5 Achievements cross-module (signature HEGON)

La majorité sont par module. Mais les plus puissants sont **cross-module** — la vraie signature HEGON :
- **Renaissance** : débloquer un achievement dans 5 modules différents
- **Story Lover** : 50 films + 20 livres combinés
- **Dedicated** : utiliser HEGON 30 jours d'affilée (n'importe quel module)
- **Life Logger** : au moins 1 item dans chaque module

Letterboxd ne peut pas faire "Story Lover" (pas de livres). HEGON oui.

### 2.6 Garde-fou design — ne PAS tomber dans le cheap

- **Peu d'achievements, signifiants** (20-30 max, pas 200 micro-badges)
- **Zéro garbage** : jamais "Tu as ouvert l'app 7 jours !" — célébrer du VRAI accomplissement
- **Design certificat/médaille élégant** : pense Apple Fitness awards / Coursera certificates, pas badges Candy Crush
- **Calibrer pour le long terme** : les tiers Platinum span plusieurs années (voulu — engagement durable)

### 2.7 Seed list par module (à finaliser au brainstorm de chaque module)

> Liste de départ. Chaque brainstorm de module affine SES achievements.

**Watching**
- First Watch (1er film loggé) — événementiel
- Cinéphile (100/250/500/1000 films) — tiered
- Binge Master (finir une série) — événementiel
- Critic (rate 25/100/250 médias) — tiered
- Marathon (5 films en 1 jour) — événementiel (besoin watched_at)
- Genre Explorer (films dans 10 genres) — événementiel

**Books**
- First Page (1er livre) — événementiel
- Bookworm (20/50/100 livres) — tiered
- Speed Reader (livre en <3 jours) — événementiel
- Annotator (50/200/500 highlights) — tiered

**Habits**
- Streak Starter (7j) / Unstoppable (30j) / Centurion (100j) / Immortal (365j) — tiered streak

**Goals**
- Achiever (1er goal complété) — événementiel
- Overachiever (10/25/50 goals) — tiered

**Cross-module (signature)**
- Renaissance / Story Lover / Dedicated / Life Logger

**Sport, Journal, Travel, Culture** : à définir lors de leurs brainstorms.

---

## 3. 🎁 Système 2 — Wrapped Year

### 3.1 Concept

Comme **Spotify Wrapped**. Récap visuel de l'année, format **story** (cards verticales qu'on parcourt), animations séquentielles, export shareable. Accessible toute l'année comme "2026 so far".

### 3.2 Format & ambition design (11/10 — demande explicite Zakaria)

- Format story vertical (tap/click pour avancer, comme Instagram/Spotify Wrapped)
- 1 stat = 1 plein écran, typographie bold massive
- Gradient background unique par card
- Animations à la révélation : compteurs qui montent, posters/covers qui slide, texte en cascade
- Spring physics partout
- Card finale exportable en **PNG** (html-to-image ou canvas) pour partage social
- Une des cards finales = "Achievements unlocked this year" (les badges 2026)

### 3.3 Contenu par module (chaque module fournit ses cards)

- **Watching** : films, séries, top genre, best month, highest rated, hours, streak
- **Books** : livres lus, pages, top genre, top rated, hours reading
- **Goals** : goals atteints, progress moyen, meilleur mois
- **Habits** : meilleur streak, completion rate, habit le plus consistant
- **Sport** : matchs suivis, win rate quand tu regardes
- **Journal** : entries écrites, mood dominant, mots écrits
- **Card finale** : achievements débloqués cette année + résumé global cross-module

### 3.4 Wrapped global vs par module

- Chaque module a SON wrapped (Watching Wrapped, Books Wrapped…)
- Plus un **Wrapped global HEGON** qui synthétise toute l'année cross-module
- Tous archivés dans la Trophy Room par année

---

## 4. 🏆 La destination — "Trophy Room" (working name)

### 4.1 Layout

```
┌─────────────────────────────────────────┐
│  TROPHY ROOM (working name)             │
├─────────────────────────────────────────┤
│  ACHIEVEMENTS                            │
│  Earned (12) · In progress (8)          │
│  [grille de badges tiered + barres]     │
├─────────────────────────────────────────┤
│  WRAPPED ARCHIVE                         │
│  [2026 ▸] [2025 ▸] [2024 ▸]            │
│  click → ouvre le wrapped story-format  │
└─────────────────────────────────────────┘
```

### 4.2 Emplacement

**Route dédiée** accessible depuis le menu profil (avatar topbar). C'est une destination qu'on visite, pas un écran quotidien. Genre "Settings" mais beaucoup plus fun.
Route proposée : `/wrapped` ou `/trophy-room` ou `/achievements` (nom final à trancher).

### 4.3 Noms à trancher (working names pour l'instant)
- Le méta-module : "Trophy Room" / "Achievements" / "Records" / "Time Capsule"
- Les tiers : Bronze/Silver/Gold/Platinum
- À décider au moment du design final

---

## 5. Schema DB (esquisse)

```sql
-- Définitions des achievements (référence — hardcodé ou table seed, pas d'org_id)
achievement_definitions (
  id text PRIMARY KEY,         -- 'cinephile', 'bookworm', 'marathon'...
  module text,                 -- 'watching', 'books', 'cross'...
  title text,
  description text,
  type text CHECK (type IN ('quantitative', 'streak', 'event')),
  tiers jsonb,                 -- [{tier:'bronze', threshold:100}, ...] ou null si event
  icon text
)

-- Achievements débloqués par user (préférence, org_id)
user_achievements (
  id uuid,
  user_id uuid, org_id uuid,
  achievement_id text REFERENCES achievement_definitions,
  current_tier text,           -- 'bronze'/'silver'/... ou null
  progress int,                -- valeur actuelle (137 films)
  unlocked_at timestamptz,     -- première fois débloqué
  tier_unlocked_at jsonb,      -- {bronze: date, silver: date, ...}
  PRIMARY KEY (user_id, achievement_id)
)

-- Wrapped archivés (snapshot annuel figé, org_id)
wrapped_snapshots (
  id uuid,
  user_id uuid, org_id uuid,
  year int,
  module text,                 -- 'watching', 'global'...
  data jsonb,                  -- toutes les stats figées de l'année
  generated_at timestamptz,
  UNIQUE (user_id, year, module)
)
```

**Note perf :** les achievements se recalculent via triggers ou jobs, pas à chaque render. Le Wrapped se fige en snapshot (jsonb) une fois généré pour éviter de recalculer l'historique.

---

## 6. ⚠️ Impact cross-cutting — RÈGLE pour tous les brainstorms

**Chaque brainstorm de module DOIT désormais inclure :**
1. Une section **"Wrapped"** : quelles cards/stats ce module fournit au Wrapped annuel
2. Une liste **"Achievements"** : les badges propres à ce module (tiered ou événementiel)

Sinon on devra retrofit. Cette règle est ajoutée au `session-protocol.md`.

Déjà fait :
- ✅ Watching : wrapped + achievements seed (dans `watching-v2.md` + ici §2.7)
- 🔲 Tasks : à compléter (le spec tasks-v2 n'a pas encore sa section Wrapped/Achievements — à ajouter)

---

## 7. Priorité & dépendances

- **Dépend de** : tous les modules ayant leurs stats (Watching ✅, autres à venir)
- **Year Wrapped Watching** = livrable V2.5, arme LinkedIn
- **Trophy Room complète** (tous modules + achievements) = Phase 6+ (après que les modules V2 existent)
- **Décision actée maintenant** pour ne pas retrofit : chaque module prévoit ses hooks wrapped + achievements dès sa V2

---

## 8. Top angle vidéo LinkedIn

> "Spotify gives you Wrapped once a year, for music. What if you had it for your entire life — films, books, goals, habits — all connected, all in one trophy room that grows for years?"
> → démo Trophy Room → Wrapped story format → achievements tiered → "This is 3 years of my life in HEGON."

C'est potentiellement **LA vidéo signature de tout HEGON** — celle qui résume pourquoi il existe.

---

## Status & next

- ✅ Brainstorm session : 2026-05-19/20
- ✅ Décisions lockées : 2 systèmes, ligne Goals/Achievements, tiers, cross-module, Trophy Room
- 🔲 Noms finaux à trancher (Trophy Room, tiers, route)
- 🔲 Liste complète des achievements par module → au fil des brainstorms
- 🔲 Section Wrapped/Achievements à rétro-ajouter dans `tasks-v2.md`
- ⏭ Implémentation : Phase 6+ (méta-module, après modules V2)
