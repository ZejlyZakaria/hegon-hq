# Dashboard Redesign — Brief pour Opus

> Brief à donner à Opus pour qu'il propose un redesign UI/UX exceptionnel du Dashboard HEGON.
> Créé le 2026-05-20. À utiliser quand le budget Opus le permet.
> Objectif : Dashboard "émotionnellement plat" → home premium d'un OS personnel.

---

## 📋 PROMPT À COLLER DANS OPUS

```
Tu es un designer produit senior, niveau Linear / Superhuman / Apple. 
Mission : proposer un redesign UI/UX EXCEPTIONNEL du Dashboard de HEGON.

═══ CONTEXTE PRODUIT ═══

HEGON est une app web premium personnelle (single-user, multi-tenant ready) 
qui centralise toute la vie : tasks, watching, books, habits, goals, journal, 
sports. North star : "Tu ouvres HEGON et tu oublies que c'est une app web. 
Tu as la sensation d'un espace cohérent, calme, réactif." Dark mode permanent 
(#09090b). Stack : Next.js 15, Tailwind v4, Framer Motion, TanStack Query.

Le Dashboard est l'écran d'accueil — le PREMIER truc qu'on voit en ouvrant 
HEGON. C'est la vitrine de l'OS personnel.

═══ AVANT DE COMMENCER, LIS CES FICHIERS ═══

1. hq/blueprint.md — règles fondamentales + les 10 signaux "OS feel"
2. hq/design-system.md — design system complet (tokens, motion, patterns)
3. hq/modules.md — la matrice des modules + couleurs accent par module
4. memory project_os_vision.md (si accessible) — la vision OS, layouts par module
5. Le code actuel : src/modules/dashboard/components/ (DashboardContent, 
   today/TodaySection, right-panel/, row2/LifeStatsRow, row3/MediaSportsRow, 
   header/DashboardHeader)

═══ ÉTAT ACTUEL DU DASHBOARD ═══

- DashboardHeader : greeting basé sur l'heure ("Good evening, Zakaria") + date
- TODAY section : 3 cards dynamiques 
  - PRIORITY (indigo) : tâche la plus urgente
  - UP NEXT (couleur sport) : prochain événement sport
  - IN PROGRESS (violet) : média en cours
- Upcoming in Sports : scroll horizontal d'événements
- Your Tasks : tabs Today/Upcoming/Completed + week progress bar
- Right Panel : DailyScoreWidget + WeeklyActivityWidget
- Row 2 : LifeStatsRow (stats cross-module)
- Row 3 : MediaSportsRow

═══ LE PROBLÈME (diagnostic existant) ═══

Le Dashboard est "fonctionnel mais émotionnellement plat" : 3 rectangles avec 
un titre, pas de motion, pas de profondeur, pas d'effet "aha". Il ne donne pas 
la sensation premium d'un OS. C'est le point faible UX le plus visible du produit.

═══ CE QUE JE VEUX DE TOI ═══

Un redesign qui transforme ce Dashboard en une home premium VIVANTE, sans tout 
reconstruire. Traite spécifiquement :

1. HIÉRARCHIE VISUELLE : qu'est-ce qui doit dominer ? Le regard doit aller où 
   en premier ? Actuellement tout a le même poids.

2. MOTION & VIE : 
   - Animation d'arrivée séquentielle des cards (stagger)
   - Micro-interactions au hover/press
   - Transitions premium (spring physics, < 350ms par l'anti-patterns.md)
   - L'app doit "respirer"

3. ÉTAT ÉMOTIONNEL : 
   - Un état "all done today" qui CÉLÈBRE (le user a tout fait → moment de 
     satisfaction, pas un écran vide)
   - Le greeting peut être plus vivant/contextuel

4. SENSATION OS : appliquer les 10 signaux "OS feel" du blueprint. Comment le 
   Dashboard incarne "un espace cohérent, calme, réactif" ?

5. OPPORTUNITÉS (HEGON a des features uniques à surfacer) :
   - Connexions cross-module (une task qui nourrit un goal qui dépend d'un habit)
   - Streaks globaux ("47 jours d'affilée sur HEGON")
   - Lien vers "Trophy Room / Year Wrapped" (système d'achievements + récap annuel)
   - "What should I focus on now" (intelligence simple, basée sur les données)

═══ CONTRAINTES ═══

- PAS une refonte totale from scratch — des améliorations concrètes et 
  implémentables sur la base existante
- Respecter le design system (tokens sémantiques bg-surface-*, text-text-*, 
  accents par module)
- Respecter les anti-patterns (pas d'animation > 350ms, pas de gradient sur 
  card de liste, pas de 2 boutons primary, etc.)
- Dark mode #09090b, desktop-first
- Réaliste : implémentable par un solo dev en quelques jours

═══ FORMAT DE RÉPONSE ATTENDU ═══

1. DIAGNOSTIC (court) : les 3-4 vrais problèmes du Dashboard actuel
2. VISION (1 paragraphe) : ce que devient le Dashboard, en une phrase forte
3. LAYOUT PROPOSÉ : structure en ASCII ou description précise zone par zone, 
   avec hiérarchie claire
4. MOTION SYSTEM : les animations précises (quoi, quand, durée, easing)
5. L'ÉTAT "ALL DONE" : à quoi il ressemble, comment il célèbre
6. PUNCH-LIST PRIORISÉE : tableau impact/effort des changements concrets, du 
   quick win au gros morceau
7. CE QU'ON GARDE : ce qui marche déjà et qu'il ne faut PAS toucher

Sois précis, opinionated, niveau Linear/Apple. Challenge les choix actuels. 
Je veux un Dashboard dont on dit "wow" en 2 secondes.
```

---

## 📎 Notes pour Zakaria

**Joindre à Opus :** une capture d'écran du Dashboard actuel (très important — Opus juge mieux avec le visuel).

**Pourquoi ce module pour le reliquat de tokens :**
- Le Dashboard est l'écran d'accueil = le plus vu
- Opus l'a flaggé "émotionnellement plat" dans sa re-review
- Pas de V2 redesign Dashboard prévu → le travail ne sera pas jeté (lasting)
- Brief contenu → rentre dans un petit budget

**Après réception de la proposition Opus :**
- Sauvegarder sa réponse dans ce même fichier (section "## Proposition Opus" ci-dessous)
- Décider quoi implémenter
- Si on valide une direction → créer `hq/modules_version_2/dashboard-v2.md`

---

## Proposition Opus

> Rédigé le 2026-05-20 par Opus 4.7, après lecture du **code réel** (`src/modules/dashboard/components/`), du blueprint, du design-system et des anti-patterns.
> ⚠️ Note : le « ÉTAT ACTUEL » du brief était périmé. État réel vérifié dans le code (voir Diagnostic §0).

---

### 0. Ce que le Dashboard est VRAIMENT aujourd'hui (relevé code, pas brief)

Le brief décrivait 3 cards séparées + des sections obsolètes. La réalité :

- **Header** (`header/DashboardHeader.tsx`) : date `text-2xl font-light` + greeting `text-sm text-tertiary`.
- **TODAY** (`today/TodaySection.tsx`) : **UNE seule card** `bg-surface-1` découpée en **3 colonnes** `divide-x` :
  - `PRIORITY` → tasks today+overdue (dot orange `#f97316`)
  - `IN PROGRESS` → **les Goals** (pas les médias !), top 2 (dot vert `#22c55e`)
  - `UP NEXT` → events sport empilés (dot violet `#8b5cf6`)
- **Right panel** (`w-72`) : Weather + DailyScore (donut + 2 mini-cards) + WeeklyActivity (barres).
- **Row 2** (`row2/LifeStatsRow.tsx`) : Goals · Habits · Reading (3 cards égales).
- **Row 3** (`row3/MediaSportsRow.tsx`) : Watching · Football · Tennis · F1 (4 cards à bg image dégradé).

Total : **~11 rectangles de poids visuel quasi identique** + panneau droit. C'est la racine du problème.

---

### 1. DIAGNOSTIC — les 4 vrais problèmes

**P1 — Zéro hiérarchie. Le regard n'a aucun point d'entrée.**
3 colonnes égales, puis 3 cards égales, puis 4 cards égales. La règle 8.2 du design-system (« 1 élément dominant par écran, max 2 secondaires ») est frontalement violée. Tout crie au même volume → rien ne parle. C'est exactement ça, « émotionnellement plat » : pas un manque d'animation, un **manque de dominante**.

**P2 — Le Dashboard est l'écran le MOINS conforme au design-system de tout HEGON.**
C'est le comble : la vitrine est la pièce la plus sale. Relevé dans le code :
- `bg-zinc-950` sur les cards (`CardShell`, `MediaSportsRow.tsx:37`) → **explicitement interdit** (« `bg-zinc-950` sur une card — invisible »). Idem `bg-zinc-900/50`, `border-zinc-800/60`, `text-zinc-200/600/700` partout → anti-pattern « `bg-zinc-*`/`text-zinc-*` hardcodé ».
- Labels **`uppercase tracking-widest`** sur TOUTES les sections (PRIORITY, IN PROGRESS, GOALS, HABITS, READING, DAILY SCORE…) → design-system §1.6 : « **Uppercase sur les labels — interdit**, lisibilité réduite ».
- Tailles **sous la scale** : `text-[8px]`, `text-[9px]` en masse (le min de la scale est `text-2xs` = 10px). + valeurs arbitraires `tracking-[0.12em]`, `h-0.75`, `mt-1.25`.
Résultat sensoriel : ça lit comme un **template dashboard Bootstrap dark**, pas comme HEGON. Le manque de premium vient d'abord de là, avant toute animation.

**P3 — L'accent est de la déco, pas un signal. 7 couleurs se battent.**
Orange (Priority), vert (In Progress), violet (Up Next), puis emerald/red/amber/sky dans les sports. Aucune ne porte de sens stable : violet = Watching mais sert ici aux **sports** ; orange = Journal mais sert aux **tasks**. Et le **bleu Dashboard (`#60a5fa`, le host) n'apparaît littéralement nulle part.** Violations directes : « Accent is a signal, not decoration » + « Accent Conflict Rule » (l'accent du module hôte gagne).

**P4 — Aucune âme, aucun moment, de la redondance.**
- Empty states = **emoji grisé** (✅ 🎯 📅 📖 ⚽) → §4.10 veut une icône Lucide 32px ; les emojis sont bannis des labels. Ça fait cheap.
- Greeting plat (« Good night, Zakaria. »), zéro contexte, zéro « sur quoi je me concentre ».
- **Redondance** : le goal « Construire HEGON » s'affiche dans la colonne IN PROGRESS **et** dans le widget Goals row2 → viole « chaque info vit à un seul endroit ».
- La superpuissance de HEGON (graphe Goals↔Tasks↔Habits) est **invisible**.

---

### 2. VISION — en une phrase

> **Le Dashboard arrête d'être un mur de tuiles et devient un _briefing_ : une grande respiration en haut qui te dit où tu en es et la seule chose qui compte maintenant — puis le reste de ta vie en sourdine en dessous, calme et hiérarchisé.** Tu ouvres HEGON, et en 2 secondes tu sais quoi faire, sans scanner 11 boîtes.

Le mouvement clé : **introduire une dominante** (un _hero_ « Focus + Score »), **dégrader tout le reste** en secondaire/tertiaire, et **appliquer enfin le design-system** que les autres modules respectent déjà.

---

### 3. LAYOUT PROPOSÉ

On **ne reconstruit pas** : on réorganise le poids visuel des blocs existants + on promeut le Daily Score.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  HEADER ENRICHI                                                                │
│  Wednesday, May 20                                                             │
│  Good evening, Zakaria — 2 tasks and 1 match before tonight.   · 12-day streak │  ← ligne d'intelligence
├───────────────────────────────────────────────────────┬────────────────────────┤
│                                                         │                        │
│   ███████  HERO — FOCUS NOW  (DOMINANTE)  ███████████   │   DAILY SCORE          │
│                                                         │   (ring promu, gros)   │
│   "La seule chose"                                      │        50%             │
│   ▸ tâche la + urgente  (ou célébration si all-done)    │   ┌────────┬────────┐  │
│     titre text-xl · échéance · ⎯ feeds Goal: …          │   │Habits  │Tasks   │  │
│     [✓ quick-complete inline]                           │   │ 0/4    │all done│  │
│                                                         │   └────────┴────────┘  │
├──────────────┬──────────────┬─────────────────────────┤   WEATHER · Paris 14°  │
│  Priority    │ In progress  │  Up next                  │   THIS WEEK (barres)   │
│  (tasks)     │  (goals)     │  (sports)                 │                        │
│  ← la card 3-col ACTUELLE, conservée mais SECONDAIRE    │                        │
├──────────────┴──────────────┴─────────────────────────┴────────────────────────┤
│  ─ Life ─────────────────────────────────────────────────────  (TERTIAIRE)     │
│  Goals · Habits · Reading        (row2, plus compact, labels sentence-case)     │
├──────────────────────────────────────────────────────────────────────────────┤
│  ─ Watching & Sports ─────────────────────────────────────────  (TERTIAIRE)    │
│  Watching · Football · Tennis · F1   (row3 — exception "hero card" assumée)     │
└──────────────────────────────────────────────────────────────────────────────┘
```

**Les 3 niveaux de hiérarchie (la clé du redesign) :**

| Niveau | Zone | Traitement visuel |
|---|---|---|
| **Dominant** | Hero Focus + Daily Score ring | `text-xl` titre, ring large accent **bleu Dashboard**, le seul gros chiffre de l'écran |
| **Secondaire** | Card TODAY 3-col (Priority/InProgress/UpNext) | inchangée dans son contenu, mais labels discrets, pas de gros chiffre, pas d'accents qui crient |
| **Tertiaire** | Row Life + Row Media/Sports | plus compact, `text-text-tertiary` dominant, n'attire l'œil qu'au hover |

**Décisions opinionated à acter :**

1. **Promouvoir le Daily Score hors du right panel → dans le hero.** C'est l'ancre émotionnelle (« comment va ma journée »). Aujourd'hui il est planqué en bas à droite. C'est l'erreur la plus coûteuse : la seule métrique qui résume tout est invisible.
2. **Le « Focus now » = intelligence simple, zéro IA.** Règle déterministe : `tâche overdue` > `tâche due today (priorité max)` > `prochain event sport <2h` > `habit pas encore fait` > `all-done`. Un seul item. C'est le « What should I focus on now » du brief, livrable en une fonction pure.
3. **Dé-dupliquer les Goals.** IN PROGRESS (colonne TODAY) garde les goals ; le widget Goals de row2 devient « Habits + Reading » resserré, OU le widget Goals row2 montre des goals _différents_ (les non-prioritaires). Choisir : je recommande **retirer Goals de row2** (il vit déjà dans TODAY) et élargir Habits/Reading.
4. **Accent hôte = bleu Dashboard.** Le ring Daily Score, les marqueurs de section et les hovers passent en `--color-accent-dashboard` (#60a5fa, déjà dans `globals.css:144`). Les couleurs module ne survivent **que sur leur propre valeur** : barre de progression Goal = vert, ring Habits = rose, barre Book = sky, cards sport row3 = leur accent (exception Explorer assumée). On **tue les dots décoratifs** orange/violet des ColLabel.

---

### 4. MOTION SYSTEM — précis (tokens existants, `src/shared/motion/tokens.ts`)

Aujourd'hui : `duration: 0.3 easeOut`, `delay: i*0.07` ad hoc, chaque section anime dans son coin. À remplacer par le système §9 du design-system, déjà défini mais **pas branché** sur le Dashboard.

**a) Vague de chargement (§9.6) — c'est CE qui crée la sensation « ça respire ».**
Un seul `motion` parent sur `DashboardContent` avec `staggerChildren` :
```
Header  →   0ms
Hero    →  60ms
Today   → 100ms
Life    → 140ms
Media   → 180ms
```
Chaque enfant : `initial {opacity:0, y:8}` → `animate {opacity:1, y:0}`, `transition: spring.smooth` (stiffness 300, damping 28). Plus aucun `duration` hardcodé.

**b) Stagger interne row3** : `staggerChildren: 0.04` (pas 0.07), cap 5 items (§9.2). 4 cards = 160ms total, propre.

**c) Daily Score ring (le moment signature)** : à l'arrivée, l'arc se **dessine** (0→valeur) + le chiffre **count-up**, `ease.inOut` 200ms. Une seule fois, pas de loop (Motion Restrictions : pas de pulse). Quand on coche le Focus, le ring **ré-anime vers le haut** en `spring.snappy` → feedback physique cross-module (§5 signal OS « feedback physique » + réactivité).

**d) Quick-complete du Focus** : check morph `AnimatePresence mode="wait"`, `spring.snappy` (§9.4). L'item passe `opacity-50` + line-through (cohérence états §2.2).

**e) Hover** : cards `surface-1 → surface-2`, `transition-colors duration-100`. **Pas de scale sur les cards** (anti-pattern). Boutons : `whileTap scale 0.97` (§9.5).

**f) Restrictions à respecter** : refresh météo / real-time = **aucune** animation (Motion Restrictions). Pas d'anim > 350ms. Stillness assumée sur le tertiaire.

---

### 5. L'ÉTAT « ALL DONE » — comment il célèbre

Condition : `Priority vide` ∧ `aucun event imminent` ∧ `habits du jour faits`. Le hero **se transforme** (il ne devient pas un vide gris) :

```
┌─────────────────────────────────────────────┬──────────────┐
│                                              │  DAILY SCORE │
│   ◦ You're clear.                            │    ◜◝ 100%   │  ← ring se remplit
│   Everything for today is done.              │   bleu→vert  │     en spring
│                                              │              │
│   12 days straight on HEGON  ›  Trophy Room  │              │
└─────────────────────────────────────────────┴──────────────┘
```

- Le **ring Daily Score se remplit à 100%** avec un dégradé bleu → vert (#60a5fa → #22c55e), `spring.gentle`. C'est ça, le moment.
- Copy **calme et fière**, pas un emoji : « You're clear. Everything for today is done. » (ton §4.10 : positif, jamais « Aucun résultat »).
- **Streak global** + lien discret vers **Trophy Room / Year Wrapped** (concept déjà planifié — `hq/modules_version_2/achievements-v2.md`). L'all-done est le point d'entrée naturel vers les achievements.
- **Pas de confetti.** Les anti-patterns réservent particules/confetti au **record de streak annuel** uniquement. Le all-done quotidien, c'est de la **stillness premium** : un ring qui se remplit + une phrase. Plus Apple, et conforme.
- Micro-détail : un **shimmer unique** (un passage, pas une loop) sur le ring au moment du remplissage = autorisé comme « célébration exceptionnelle » ponctuelle.

---

### 6. PUNCH-LIST PRIORISÉE (impact / effort)

Du quick win au gros morceau. Les 3 premiers items donnent **80% du gain « premium »** pour un effort faible.

| # | Changement | Impact | Effort | Détail |
|---|---|---|---|---|
| 1 | **Token sweep** : `bg-zinc-*`/`text-zinc-*`/`border-zinc-*` → `bg-surface-*`, `text-text-*`, `border-border-*`. Tuer `bg-zinc-950` sur cards. | 🟢🟢🟢 | XS | Find/replace guidé. Conformité DS instantanée + profondeur correcte. **À faire en premier.** |
| 2 | **Labels** : retirer `uppercase tracking-widest` → sentence-case `text-xs font-semibold text-text-secondary`. Remonter `text-[8/9px]` à `text-2xs`/`text-xs`. | 🟢🟢🟢 | XS | Change radicalement la perception « template → produit ». |
| 3 | **Empty states** : emoji → icône Lucide 32px `text-text-tertiary` + copy §4.10. | 🟢🟢 | XS | `CheckCircle2`, `Target`, `Calendar`, `BookOpen`. |
| 4 | **Motion** : brancher `spring.*` + vague `staggerChildren` sur `DashboardContent`. Supprimer les `duration` ad hoc. | 🟢🟢🟢 | S | « Ça respire » vient de là. Tokens déjà écrits. |
| 5 | **Discipline d'accent** : host = bleu Dashboard (ring, sections, hover). Tuer dots déco. Accents module **sur valeur uniquement**. | 🟢🟢 | S | Applique Accent Conflict Rule. |
| 6 | **Hero band** : extraire DailyScore du right panel → hero `Focus + Score`. Dégrader la card TODAY en secondaire. | 🟢🟢🟢 | M | Le cœur du redesign. Crée la dominante (P1). |
| 7 | **Focus intelligence** + greeting contextuel (« 2 tasks and 1 match before tonight »). | 🟢🟢 | M | Fonction pure déterministe, zéro IA. |
| 8 | **État all-done** célébration (ring 100% + copy + hook Trophy Room). | 🟢🟢 | M | Le « moment ». Dépend de #6. |
| 9 | **Dé-dup Goals** : retirer Goals de row2, élargir Habits/Reading. | 🟢 | S | Supprime la redondance (P4). |
| 10 | **Streak global « X days on HEGON »** + entrée Trophy Room. | 🟢🟢 | L | Pas de source globale aujourd'hui (seuls streaks Habits/Journal existent). Nécessite un calcul/colonne dédiée → vrai chantier, à planifier. |

**Recommandation de séquençage** : faire **#1→#5 d'un bloc** (1 journée, gain énorme, zéro risque). Puis **#6→#8** comme deuxième passe (le hero, 1-2 jours). #9 en passant. #10 = à scoper séparément (donnée manquante).

---

### 7. CE QU'ON GARDE — ne pas toucher

- **L'architecture d'info de la card TODAY** (Priority/InProgress/UpNext) : elle est juste et dense. On la **dégrade visuellement**, on ne la refait pas.
- **Row 3 Media/Sports avec bg image dégradé** : c'est déjà la partie la plus vivante, et c'est l'**exception « hero card » documentée** (famille Explorer). On aligne juste les tokens (`bg-zinc-950` → `bg-surface-1`).
- **La météo** dans le right panel : touche d'ambiance premium discrète, à garder (juste retravailler le label).
- **`DonutChart`** (`shared/DonutChart.tsx`) : composant réutilisé tel quel, juste **promu** dans le hero.
- **Toute la logique data** : `useDashboardData`, `service.ts` (Promise.all), fuseau `PARIS_TZ`, `GRACE_MS`, `episodeProgress()` — solide, **on n'y touche pas**. Le redesign est **100% présentationnel**.
- **Les hooks cross-module** (`useGoals`, `useHabitsToday`, `useBooks`) — réutilisés.

---

### Verdict

Le Dashboard n'a pas besoin d'être reconstruit. Il a besoin de **3 choses dans l'ordre** : (1) **respecter le design-system** que tout le reste de HEGON respecte déjà — c'est 80% de la sensation premium et c'est un quick win ; (2) **une dominante** (le hero Focus + Score) pour que le regard ait un point d'entrée ; (3) **brancher le motion system déjà écrit**. Le « wow en 2 secondes » ne viendra pas de plus d'effets — il viendra de **moins de bruit, une vraie hiérarchie, et un seul moment fort** (le ring du score).

> **Next step si direction validée** → créer `hq/modules_version_2/dashboard-v2.md` avec la spec d'implémentation (composants `DashboardHero`, refactor `TodaySection` en secondaire, fonction `getFocusItem()`, état all-done), + section Wrapped + Achievements (obligatoire par `session-protocol.md`).
