# HEGON Habits — V2 Spec (✅ COMPLET)

> Brainstorm terminé le 2026-05-20 entre Zakaria et Claude.
> Toutes les décisions sont lockées. Prêt pour l'implémentation.

---

## 1. Vision & positionnement

Habits V2 = tracking satisfaisant + heatmap iconique + montre signature + plante vivante + récompenses.
Références : Streaks, Habitify, Apple Fitness (rings/awards).
L'âme : un module qui vit et grandit avec toi — pas un simple tracker de cases à cocher.

---

## 2. État actuel (V1) — ce qui change

- ❌ HabitCard trop grosse, trop d'infos visibles
- ❌ Progress bar top + texte = faible
- ❌ Layout 2 colonnes (left cards, right panel)
- ✅ Right panel : bien, à garder avec hero card accent
- ✅ Heatmap GitHub-style : iconique, à mettre en valeur
- ✅ Custom frequency (daily/weekly/custom) : déjà en V1, rien à faire

---

## 3. Structure de page — 3 colonnes

| Colonne | Contenu | Largeur |
|---|---|---|
| **Left** | Montre + Progress bar + Plante | ~220-240px |
| **Centre** | HabitCards (tab Today / Calendar / All Habits) | flex-1 |
| **Right** | Stats panel (existing + hero card accent) | ~288px |

---

## 4. Colonne Left

### 4.1 🕰️ Montre signature
- Design inspiré haute horlogerie (AP Royal Oak / Patek Nautilus)
- **Time-only** — affiche uniquement l'heure, pas de complications habits
- Déblocable par streak (cf. §7 Achievements)
- Collection de montres (min 3 designs), l'user choisit laquelle afficher
- Position : tout en haut de la colonne left

### 4.2 Progress bar
- Coexiste avec la montre et la plante
- Info immédiate : "aujourd'hui, X/Y habits complétés"
- Positionnée entre la montre et la plante

### 4.3 🌱 Plante mensuelle
- Grandit selon le taux de completion du mois en cours
- Reset le 1er de chaque mois (nouveau cycle)
- 3-4 types de plantes débloquables via achievements (cf. §7)
- L'user choisit quelle plante afficher (comme les watch faces)
- Snapshot de la plante conservé dans le monthly recap (§6)
- Position : sous la progress bar

---

## 5. HabitCards (colonne centre)

### 5.1 Tab Today — HabitCard redesign

**Contenu visible sur la card :**
```
[icon]  Titre du habit    🔥 Xj    [Complete]
```

- Icon + titre + streak current + bouton Complete
- C'est tout. Rien d'autre sur la card.

**Interaction :**
- **Clic sur la card** (hors bouton Complete) → ouvre le sliding panel
- **Bouton Complete** → action directe, pas de panel
- Animation Complete : bouton bascule en "Completed" + micro-animation streak

**Sliding panel contient :**
- Titre + description + type + goal linké
- Streak current + best streak
- Stats détaillées (completion rate 30/90j, jour préféré)
- Skip day + Pause (cf. §5.4)
- Edit + Delete

### 5.2 Tab Calendar — Week Grid

Remplace le calendar V1. Format inspiré Notion habit tracker :
- Lignes = jours de la semaine
- Colonnes = habits (icône en header)
- Checkboxes par cellule (complété / non complété)
- Progress bar + % par jour
- Note optionnelle par jour
- Ligne "AVERAGE" en bas avec score par habit

### 5.3 Tab All Habits — Heatmap

- GitHub contribution graph style (déjà en V1)
- 6 mois de données
- Élément iconique du module, à mettre en valeur

### 5.4 Skip day & Pause (dans sliding panel)

**Skip day (one-off) :**
- Action "Skip today" dans le sliding panel
- Ce jour apparaît neutre (ni complété ni manqué)
- Streak préservé

**Pause (période) :**
- Date début + date fin dans le sliding panel
- Pendant la pause : habit masqué du tab Today
- Streak gelé à sa valeur actuelle, reprend normalement après

---

## 6. Monthly Recap

Chaque mois a un résumé accessible dans l'historique :
- Stats du mois (completion rate, meilleur streak du mois, total completions)
- Snapshot de l'état de la plante à la fin du mois
- Design soigné, format narrative (pas juste des chiffres)

---

## 7. Right Panel

- **Même contenu qu'en V1** : streak global, completion rate du mois (donut), heatmap compacte
- **Une seule hero card** avec gradient rose `from-rose-950/60 via-zinc-950` — couleur accent habits
- Les autres cards restent neutres
- Pattern à appliquer à tous les Life modules (Goals=vert, Journal=amber, Books=sky)

---

## 8. Reminders

**Supprimés du scope V2.** L'app est web (pas installée), pas mobile-friendly. Zakaria utilise l'app en batch en fin de journée — les reminders n'ont pas de valeur dans ce contexte. Ni push navigateur ni email.

---

## 9. Wrapped cards (récap annuel)

| Card | Contenu |
|---|---|
| Meilleur streak de l'année | X jours consécutifs |
| Habit le plus consistant | Nom + completion rate |
| Jour de la semaine le plus productif | Lundi / Mardi / … |
| Total completions sur l'année | X completions |
| Mois le plus fort | Mois + score |

---

## 10. Achievements

| Achievement | Déclencheur | Récompense |
|---|---|---|
| Streak 7j 🥉 | 7 jours consécutifs (≥1 habit) | Watch face Silver débloquée |
| Streak 30j 🥈 | 30 jours consécutifs | Watch face Gold débloquée |
| Streak 100j 🥇 | 100 jours consécutifs | Watch face Platinum débloquée |
| Streak 365j 💎 | 365 jours consécutifs | Watch face Diamond débloquée |
| Perfect Week | 7 jours 100% tous les habits | — |
| Perfect Month | 1 mois entier 100% | Nouvelle espèce de plante débloquée |
| Habit Collector | 5 habits actifs simultanément | — |
| Comeback | Reprendre un habit après 14j d'absence | — |

**Note :** les paliers de streak débloquent les watch faces. Perfect Month débloque une nouvelle espèce de plante. L'user choisit quelle watch face / quelle plante afficher (collection).

---

## 11. Architecture technique (à détailler à l'implémentation)

**Nouvelles tables nécessaires :**
- `habit_skips` — (habit_id, skip_date, reason?) pour les skip days
- `habit_pauses` — (habit_id, pause_start, pause_end) pour les pauses
- `habit_monthly_snapshots` — (habit_id, month, plant_state, stats_json) pour le monthly recap

**Colonnes à ajouter sur `habits` :**
- `watch_face` — watch face sélectionnée (défaut : 'default')
- `plant_type` — espèce de plante sélectionnée (défaut : 'default')

**Colonnes à ajouter sur `users` / préférences :**
- Unlocked watch faces + plant types → table `user_unlocks` (cross-module avec achievements)

---

## 12. Notes d'implémentation

- **Sliding panel** : même fondation que TaskDetailPanel (à construire en Phase 3)
- **3 colonnes** : flex layout, left `w-56 shrink-0`, centre `flex-1 min-w-0`, right `w-72 shrink-0`
- **Montre** : image IA haute qualité (Midjourney/DALL-E) pour le cadran seul (dial only, sans boîtier ni bracelet), inspiré Patek Philippe. Aiguilles heures/minutes en SVG superposées, animées via CSS `transform: rotate(Xdeg)` calculé depuis `new Date()`, mis à jour chaque seconde. 3-4 designs = 3-4 images différentes. Zéro library supplémentaire.
- **Plante** : **Lottie** (`lottie-react` ~30KB gzippé), animation JSON pré-rendue, `goTo(completionPct)` pour contrôler la frame. 5-6 paliers : 0-20% graine → 20-40% pousse → 40-60% petite → 60-80% plante → 80-99% grande → 100% en fleur. Fichiers disponibles sur LottieFiles.com.

---

## Status

✅ **COMPLET — brainstorm terminé le 2026-05-20**
Prêt pour l'implémentation. Dépend de la fondation Sliding Panels (Phase 3).
