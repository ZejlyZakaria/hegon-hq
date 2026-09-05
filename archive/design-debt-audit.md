# HEGON — Audit de dette design (conformité design-system)

> Scan de conformité au `hq/design-system.md` + `hq/anti-patterns.md` sur **tous les modules construits**.
> Réalisé le 2026-05-20 par Opus 4.7, à partir du code réel (`src/modules/`). Mesures par grep, contexte vérifié.
> Suite logique de l'audit Dashboard (`hq/futur/dashboard_redesign.md`) — qui avait révélé que le Dashboard était l'écran le moins conforme. Spoiler : **Sports est pire.**

---

## Résumé exécutif — classement du plus sale au plus propre

| Rang | Module | `*-zinc-*` | uppercase | `text-[8/9px]` | Verdict | Sévérité globale |
|---|---|---:|---:|---:|---|---|
| 1 | **Sports** (football/tennis/f1) | **~448** | ~72 | ~24 | Catastrophe systémique : `bg-zinc-950` sur cards, `rounded-2xl`, `shadow-2xl`, `text-white`, `#fff` inline | 🔴🔴🔴 |
| 2 | **Watching** | 9 | 13 | 3 | Modales hors-charte : `text-white`, **focus ring bleu**, `rounded-2xl`, `bg-[#121212]`, `shadow-2xl` | 🔴🔴 |
| 3 | **Tasks** | 20 | 4 | 1 | Cœur propre ; dette dans fichiers périphériques (Error/Empty/Skeleton/InvitePanel) + **`console.log` commités** | 🟠 |
| 4 | **Goals** | 2 | 8 | 0 | Tokens respectés ; tooltips `bg-zinc-900` hardcodés + emoji 🔥 + labels uppercase | 🟡 |
| 5 | **Habits** | 2 | 0 | 6 | Quasi conforme ; `text-[9px]` dans le right panel + 2 zinc dans le calendrier | 🟡 |
| 6 | **Books** | **0** | **0** | 0 | **Référence de conformité.** Quasi rien à signaler | 🟢 |
| 7 | **Journal** | **0** | **0** | 0 | **Référence de conformité** (l'éditeur crème est une exception *documentée*, pas une violation) | 🟢 |

**Lecture clé :** la dette n'est pas répartie — elle est **concentrée sur les modules anciens** (Sports, Watching), construits **avant** le retrofit design-system. Les modules Life récents (Books, Journal, Habits, Goals) prouvent que la charte est tenable. Le chantier = **ramener Sports + Watching au niveau de Books**, pas réécrire la charte.

> ⚠️ **Hors périmètre mais à noter** : `onboarding` (92 zinc, `bg-white text-zinc-950`, `drop-shadow-2xl`) et `command-center`/⌘K (`bg-[#111113]`, `rounded-2xl`, `shadow-2xl`) sont aussi très sales. Pas dans la demande (modules « construits » classiques), mais à mettre dans le même token sweep.

---

## 1. 🔴 SPORTS (football / tennis / f1) — le gros chantier

Les 3 disciplines partagent les **mêmes patterns hors-charte** (même auteur, même époque, avant retrofit). Plus le fichier mutualisé `SportSkeletons.tsx` (115 occurrences zinc à lui seul).

### Pattern systémique (présent dans ~tous les fichiers sports)

| Violation | Règle enfreinte | Sévérité |
|---|---|---|
| `bg-zinc-950` / `bg-zinc-900` sur les cards & tableaux | DS §1.1 « `bg-zinc-950` sur une card = invisible, interdit » + anti-pattern `bg-zinc-*` hardcodé | 🔴 |
| `border-zinc-800/60` partout | tokens `border-border-*` obligatoires + opacité variable interdite | 🟠 |
| `text-zinc-100/300/400/500` | tokens `text-text-*` obligatoires | 🟠 |
| `rounded-2xl` sur cards/heroes/tableaux | DS §1.7 max `rounded-lg` (cards) / `rounded-xl` (hero) ; `rounded-2xl` jamais | 🟠 |
| `shadow-2xl shadow-black/50`, `shadow-xl`, `shadow-lg` sur cards | DS §4.3 « Shadow externe interdit » | 🔴 |
| `text-white` / `style={{ color: "#fff" }}` | DS §1.2 « `#ffffff` interdit » | 🔴 |
| `uppercase tracking-widest` sur labels | DS §1.6 « uppercase sur labels interdit » | 🟡 |
| `text-[8px]` / `text-[9px]` | sous la scale (min `text-2xs` 10px) | 🟠 |

### Offenders représentatifs (fichier:ligne)

| Fichier:ligne | Violation | Sév. |
|---|---|---|
| `sports/components/SportSkeletons.tsx` (115×) | `bg-zinc-950`+`border-zinc-800/60`+`rounded-2xl` sur tous les skeletons | 🟠 |
| `football/.../squad/FootballLegends.tsx` (40 zinc) | modal `bg-zinc-950`+`rounded-2xl`+`shadow-2xl`, labels uppercase, `text-[9px]` | 🔴 |
| `football/.../standings/StandingsTableInner.tsx:208` | `bg-zinc-950 border border-zinc-800/60 rounded-2xl` | 🔴 |
| `football/.../squad/BestXI.tsx:328,523` | modal `bg-zinc-950 rounded-2xl shadow-2xl` ; `style={{ flex: "0 0 80%" }}` statique ; `text-white drop-shadow-sm` | 🔴 |
| `football/.../hero/HeroSwiper.tsx:174` | `rounded-2xl shadow-2xl shadow-black/50` + `drop-shadow-2xl` | 🔴 |
| `tennis/.../hero/TennisHeroSwiper.tsx:238,313` | `rounded-2xl shadow-2xl shadow-black/50`, `shadow-xl` | 🔴 |
| `tennis/.../matches/TennisUpcomingTournaments.tsx` (31 zinc, 🎾 l.393) | zinc massif + emoji 🎾 en fallback | 🟠 |
| `tennis/.../matches/TennisRecentResultsInner.tsx` (25 zinc, 🎾 l.377) | zinc + emoji | 🟠 |
| `f1/.../standings/F1DriversStandings.tsx:124,169` | `style={{ color: ... : "#fff" }}` + `bg-zinc-950 rounded-2xl` | 🔴 |
| `f1/.../standings/F1ConstructorsStandings.tsx:125,170` | idem `#fff` inline + `bg-zinc-950 rounded-2xl` | 🔴 |
| `f1/.../hero/F1Hero.tsx:98` | `rounded-2xl shadow-2xl shadow-black/50` | 🔴 |
| `football/.../matches/RecentResultsInner.tsx:144,173` + `UpcomingMatchesInner.tsx:120,153` | crests fallback emoji ⚽ + zinc | 🟡 |

**Note nuance :** les **heroes sport** (HeroSwiper, TennisHeroSwiper, F1Hero) sont des cards média immersives (famille « Explorer ») → un `rounded-xl` + gradient y est *légitime* (exception hero card DS §4.3). Mais `rounded-2xl` + `shadow-2xl` + `text-white` restent hors-charte même pour un hero. À aligner sans tuer l'effet immersif.

---

## 2. 🔴 WATCHING — modales hors-charte

Peu de `zinc` (9) mais les violations sont **graves et concentrées dans les 2 modales** (AddMediaModal, MediaDetailModal), qui sont parmi les plus gros fichiers UI du projet.

| Fichier:ligne | Violation | Sév. |
|---|---|---|
| `modals/MediaDetailModal.tsx:357,488,509` | **`focus:ring-blue-500/50`** (ring coloré au focus) | 🔴 |
| `modals/AddMediaModal.tsx:676,897,998,1024` | **`focus:ring-blue-500/50`** + `focus:ring-2 focus:ring-blue-500` | 🔴 |
| `modals/MediaDetailModal.tsx:209` | `rounded-2xl bg-[#121212] shadow-2xl` (devrait être `bg-surface-3 rounded-xl`) | 🔴 |
| `modals/MediaDetailModal.tsx:357,535` + `AddMediaModal` (×~10) | `text-white` sur inputs/textareas | 🔴 |
| `modals/AddMediaModal.tsx` (`rounded-2xl` ×~12) | `rounded-2xl` sur inputs, cards, dropdowns | 🟠 |
| `modals/AddMediaModal.tsx:875` | `bg-amber-600 ... shadow-lg shadow-amber-600/20` hardcodé (accent = background plein, interdit) | 🟠 |
| `components/WatchingHero.tsx:62` | `rounded-2xl bg-zinc-900` (hero — `rounded-xl` toléré, `bg-zinc-900` non) | 🟠 |
| `components/shared/MediaCarousel.tsx:241,332` | `border-2 border-white text-white shadow-lg` ; `bg-white/10 shadow-lg` | 🟠 |
| `components/library/LibraryCard.tsx` | `text-[8/9px]` + `drop-shadow` | 🟡 |

**Bon point :** `WatchingSkeletons.tsx` utilise déjà `bg-surface-1/2 border-border-subtle` (juste `rounded-2xl` à corriger). La couche skeleton de Watching est donc plus propre que celle de Sports.

---

## 3. 🟠 TASKS — cœur propre, périphérie sale + console.logs

Le cœur (Kanban, ListView, TaskCard) est **largement token-compliant** (1 zinc résiduel chacun). La dette est dans les fichiers d'état/erreur et un anti-pattern code.

| Fichier:ligne | Violation | Sév. |
|---|---|---|
| `actions/onboarding.ts:55,76,98,129` | **`console.log("✅ [AUTO-SETUP]…")` commités** (4×) — anti-pattern « console.log commité » | 🔴 |
| `components/TasksEmptyState.tsx` (8 zinc, 2 text-[px]) | `text-zinc-*`/`bg-zinc-*` au lieu de tokens | 🟠 |
| `components/TasksError.tsx` (4 zinc) + `F1Error`/`TennisError`/`FootballError` (4 chacun) | les écrans d'erreur sont tous en zinc hardcodé (pattern copié-collé) | 🟠 |
| `components/panels/WorkspaceInvitePanel.tsx:266,273,399,506,572,626` | labels de section `uppercase tracking-wider` (utilise les tokens `text-text-tertiary` mais uppercase interdit) | 🟡 |
| `WorkspaceInvitePanel.tsx:26-29` | `style={{ color: "#f59e0b" }}` statiques pour les icônes de rôle (devraient être classes/tokens) | 🟡 |
| `components/shared/MemberAvatar.tsx` | `text-[8/9px]` + `style` statique | 🟡 |
| `components/TasksSkeletons.tsx` | 1 zinc résiduel | 🟢 |

> `TAG_COLOR_PRESETS` (hex en dur, WorkspaceInvitePanel:20) = **PAS une violation** : ce sont des couleurs de tags choisies par l'user (donnée dynamique légitime).

---

## 4. 🟡 GOALS — propre sauf GoalDetailPage

Module conforme dans l'ensemble (utilise les tokens). Dette localisée :

| Fichier:ligne | Violation | Sév. |
|---|---|---|
| `components/GoalDetailPage.tsx:382,482` | tooltips `bg-zinc-900 border-zinc-700/50 text-zinc-100 shadow-lg` (hardcodé — devrait être `bg-surface-3`) | 🟠 |
| `components/GoalDetailPage.tsx:459` | `{habit.current_streak}🔥` — emoji dans une valeur (préférer icône Lucide `Flame`) | 🟡 |
| `components/GoalDetailPage.tsx:266,273,399,506,572,626` | labels de section `uppercase tracking-wider` (×7, avec tokens) | 🟡 |
| `components/MilestoneList.tsx:68` | état drag `bg-[#141416] shadow-lg` (shadow drag = feedback fonctionnel, tolérable, mais `bg-[#141416]` → `bg-surface-2`) | 🟡 |

---

## 5. 🟡 HABITS — quasi conforme

Le module le plus propre après Books/Journal. **Zéro uppercase.** Dette minime :

| Fichier:ligne | Violation | Sév. |
|---|---|---|
| `components/HabitsRightPanel.tsx` (5× `text-[9px]`) | tailles sous la scale (→ `text-2xs`) | 🟡 |
| `components/HabitsEmptyState.tsx` (1× `text-[9px]`) | sous la scale | 🟡 |
| `components/HabitsCalendarView.tsx` (2 zinc) | zinc résiduel dans la grille calendrier | 🟡 |

> `style={{}}` est massif dans Habits (heatmap, AllHabitsHeatmap 21×) mais **légitime** : couleurs d'intensité calculées dynamiquement. Pas une violation.

---

## 6. 🟢 BOOKS & JOURNAL — référence de conformité

**Books : 0 zinc, 0 uppercase, 0 below-scale.** Rien à corriger de significatif. `BooksRightPanel` a 2 `text-[px]` (off-scale 10/11px, cosmétique). À utiliser comme **gabarit** pour ramener les autres modules.

**Journal : 0 zinc, 0 uppercase.** L'éditeur crème `#f5f0e8` + police handwriting est l'**exception intentionnelle documentée** (DS §8.6) → **pas une violation**. `JournalRightPanel` a 3 `text-[px]` off-scale (cosmétique).

---

## 7. Findings transversaux (pas par module)

### 7.1 — `uppercase` sur labels : divergence spec ↔ réalité
Les labels `uppercase tracking-wider/widest` apparaissent **131× sur 31 fichiers**, y compris dans les pages les plus récentes et soignées (`GoalDetailPage`, `WorkspaceInvitePanel`) qui, elles, utilisent correctement les tokens. **Conclusion : l'équipe a de facto adopté l'uppercase pour les labels de section, ce qui contredit la charte (§1.6).** → **Décision à trancher** : soit on corrige le code (sentence-case partout), soit on **amende le design-system** pour autoriser l'uppercase sur les *labels de section* (≠ labels de champ). Ne pas traiter ça comme 131 bugs sans décider de la règle d'abord.

### 7.2 — Écrans d'erreur clonés en zinc
`TasksError`, `F1Error`, `TennisError`, `FootballError` = 4 zinc chacun, **structure identique copiée-collée**. Un seul composant `<ErrorState>` partagé tokenisé réglerait les 4 d'un coup (et tous les futurs).

### 7.3 — `focus:ring-{couleur}` = uniquement Watching
Les seuls rings de focus colorés du projet sont dans les 2 modales Watching (`blue-500`, `red-500`). Partout ailleurs le focus est neutre. → correction isolée, fort impact.

### 7.4 — `console.log` commités
`tasks/actions/onboarding.ts` (≥4). Anti-pattern explicite. À retirer ou passer derrière un logger.

### 7.5 — Emojis en UI (≠ comments)
Vrais usages UI (pas commentaires) : Dashboard (✅🎯📅⚽📖, déjà tracké), Sports (🎾⚽ en fallback crest/empty), Goals (🔥 streak), TodayWeatherCard (carte météo emoji — cas limite, on peut tolérer comme iconographie météo). À remplacer par Lucide en empty states (DS §4.10).

---

## 8. Plan d'action priorisé — du quick win au gros chantier

| # | Chantier | Périmètre | Impact | Effort |
|---|---|---|---|---|
| 1 | **Décider la règle uppercase** (§7.1) avant tout sweep | décision design | 🟢🟢🟢 | XS |
| 2 | **Fix focus rings colorés** Watching → `border-focus` neutre | 2 fichiers | 🟢🟢🟢 | XS |
| 3 | **Retirer les `console.log`** (tasks/actions/onboarding.ts) | 1 fichier | 🟢🟢 | XS |
| 4 | **Composant `<ErrorState>` partagé** tokenisé → remplace les 4 *Error clonés | 4→1 fichiers | 🟢🟢 | S |
| 5 | **Token sweep Tasks** (EmptyState, Skeleton, InvitePanel, périphérie) | ~7 fichiers | 🟢🟢 | S |
| 6 | **Token sweep Goals + Habits** (tooltips zinc, 🔥→Flame, text-[9px]→2xs) | ~5 fichiers | 🟢 | S |
| 7 | **Token sweep Watching modales** : `text-white`→tokens, `bg-[#121212]`→`surface-3`, `rounded-2xl`→`xl`, amber hardcodé→accent | 2 gros fichiers | 🟢🟢🟢 | M |
| 8 | **Token sweep `SportSkeletons.tsx`** (115 occ., 1 fichier, gain massif) | 1 fichier | 🟢🟢🟢 | M |
| 9 | **Token sweep Sports football** (164 occ. : Legends, BestXI, Standings, Hero, Matches) | ~12 fichiers | 🟢🟢🟢 | L |
| 10 | **Token sweep Sports tennis** (102 occ.) | ~10 fichiers | 🟢🟢 | L |
| 11 | **Token sweep Sports f1** (67 occ. : `#fff` inline, standings, hero) | ~7 fichiers | 🟢🟢 | M |
| 12 | **Hors périmètre mais même lot** : onboarding (92) + ⌘K | 2 zones | 🟢🟢 | M |

**Stratégie recommandée :**
- **Sprint 1 (1 journée, zéro risque)** : #1→#6. Décision uppercase + quick fixes + ramener Tasks/Goals/Habits à 100%. Gain de conformité énorme pour un effort XS-S.
- **Sprint 2 (1-2 jours)** : #7 + #8. Watching modales + SportSkeletons (le skeleton mutualisé = 115 occurrences en 1 fichier, meilleur ratio effort/gain de tout l'audit).
- **Sprint 3 (le gros morceau)** : #9→#11. Les 3 disciplines sport. À faire discipline par discipline, en s'appuyant sur le hero comme exception assumée (`rounded-xl` + gradient OK, mais tokens + pas de `shadow-2xl`/`text-white`).

**Méthode token sweep (find/replace guidé, par fichier) :**
```
bg-zinc-950 / bg-zinc-900            → bg-surface-1   (cards) / bg-surface-3 (modals)
bg-zinc-800                          → bg-surface-2
border-zinc-800/60 / border-zinc-700 → border-border-subtle / border-border-default
text-zinc-100 / text-white           → text-text-primary
text-zinc-300/400                    → text-text-secondary
text-zinc-500/600/700                → text-text-tertiary
text-[8px] / text-[9px]              → text-2xs (10px)  [vérifier que ça ne casse pas la densité]
rounded-2xl                          → rounded-xl (hero/modal) / rounded-lg (card de liste)
shadow-2xl / shadow-lg (sur cards)   → supprimer
focus:ring-blue-500/50               → focus:border-border-focus (ring neutre)
```
⚠️ **Ne pas faire en aveugle global** : chaque remplacement doit être validé visuellement (les cards sport ont un fond image — `bg-zinc-950` y joue parfois un rôle de fallback, à vérifier). Sweep **fichier par fichier**, pas un sed sur tout `src/`.

---

## 9. Ce qui n'est PAS de la dette (pour éviter les faux positifs)

- `style={{}}` **dynamique** : largeurs de progress bar (`width: ${pct}%`), couleurs d'équipe/intensité heatmap, accent via `var(--color-accent-*)` → **légitime**, c'est exactement l'usage prévu.
- `TAG_COLOR_PRESETS` (hex en dur) : données utilisateur, pas du style hardcodé.
- **Journal éditeur crème** : exception documentée (DS §8.6).
- **Cards média/hero immersives** (Sports heroes, Watching DontMiss) : le gradient + `rounded-xl` est l'exception « hero card » assumée — seuls `rounded-2xl`, `shadow-2xl`, `text-white` y sont à corriger.
- `text-[10px]/[11px]` : off-scale mais cosmétique (entre 2 crans de la scale). Basse priorité, pas un bug bloquant.

---

*Audit présentationnel uniquement — aucune logique métier concernée. Source : grep sur `src/modules/` au 2026-05-20.*
