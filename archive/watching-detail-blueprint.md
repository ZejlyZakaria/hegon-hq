# Watching Detail — Blueprint de redesign (2026-07-08)

> **Statut** : validé par l'owner le 07-08. Le « design figé » de `hq/watching_v3.md` est **levé** (décision owner explicite).
> **Exécution** : répartie — la **zone gold-standard** (StatusBar + My Take fusionnée) est implémentée en premier et sert de **référence visuelle à matcher** ; les P0 et P2 sont exécutés par un autre agent. Cocher les cases au fur et à mesure.
> **Preuves visuelles** : captures dans `scripts/.shots/` (`d-*.png` desktop, `m-*.png` mobile, 5 états). Harness : `node scripts/shoot.mjs <route> out.png` + variante scroll-aware `scripts/.shots/shoot2.mjs <route> out.png --scrollTo=px|end` (le `--full` standard ne capture pas le conteneur de scroll interne).
> **Benchmark owner** : harmonie/cohérence macOS-iOS. Pas Letterboxd. Interdits absolus : hairlines d'accent décoratifs, murs de formulaire, gradients tape-à-l'œil. Profondeur = surface + scale.

---

## 0. Le principe directeur

La cohérence cassait parce que **l'état était traité par ajout de surfaces** (CurrentlyWatching dans le rail, CTAs dans My Record, notices paused/dropped ailleurs) : 4 domiciles pour les actions, 3 matériaux sur le premier rang, ordre mobile absurde (Currently Watching rendu en 8ᵉ section — vérifié à l'écran).

Le renversement :

**SQUELETTE DE PAGE FIXE + UNE SEULE SURFACE ÉTAT-AWARE : la « StatusBar ».**

- L'ordre des sections est **identique dans les 5 états** (want_to_watch / in_progress / watched / paused / dropped). Les sections s'affichent ou pas ; elles **ne se déplacent jamais** (canon : *Predictability over surprise*).
- Toute la logique d'état vit dans **une bande à la couture hero/contenu** : chip d'état + contenu d'état + action primaire + menu `…` + ♥. C'est le seul endroit de la page qui « sait » l'état.
- **« Deux mondes, une couture »** : en haut le monde du film (artwork, verre sombre, blanc), en bas ton monde (graphite, tokens). La bande est la couture — **verre hero**, pas une dalle teal (v1 rejetée par l'owner). L'accent teal = signal ponctuel uniquement. My Record (l'ancienne carte teal) se dissout.
- Le feel doit rester **Watching** (chips, copy, teal deep), **pas une toolbar d'icônes**. Motion : transitions ressort quand le contenu de la bande change d'état, retenue partout ailleurs.

---

## 1. Squelette cible

**Desktop** (grid `2fr_1fr` conservé) :
- Hero (inchangé en rôle) → **StatusBar** (pleine largeur, overlap magazine `-mt` repris du grid)
- Colonne gauche : **My Take** → **History** → **Episodes** → **Cast & Crew** → **More Like This**
- Rail droit : **AnimeThemes** (anime) → **Where to watch** → **Ratings** → **Contributing to** → **In List** → **Details**

**Mobile** : hero → StatusBar → colonne gauche → rail, même ordre. Les actions étant dans la bande (en haut), le rail peut rester en bas sans dommage.

### Visibilité par état (rien ne bouge, tout s'affiche/se cache)

| Section | want_to_watch | in_progress | watched | paused | dropped |
|---|---|---|---|---|---|
| StatusBar | ✓ providers + start | ✓ stepper S/E | ✓ log rewatch | ✓ resume | ✓ resume + raison |
| My Take | — (sauf note existante) | ✓ | ✓ | ✓ | ✓ |
| History | — | ✓ strip | ✓ strip/année + rewatches | ✓ strip | ✓ strip |
| Episodes | **—** (spoilers) | ✓ | ✓ | ✓ | ✓ |
| Cast & Crew | ✓ (devient 1ᵉʳ bloc naturellement) | ✓ | ✓ | ✓ | ✓ |
| More Like This | ✓ | ✓ | ✓ | ✓ | ✓ |
| Rail Themes/Providers*/Ratings/Contributing†/In List/Details | ✓ | ✓ | ✓ | ✓ | ✓ |

\* Providers rail **masqué en want_to_watch** (les logos sont dans la bande — jamais les deux). † Contributing : watched seulement.

**L'état vide est résolu structurellement** : en want_to_watch, pas de My Take vide, pas d'History, pas d'Episodes → la page est courte et 100 % découverte (hero → bande décisionnelle → cast → recos → référence). Jamais de formulaire non rempli en zone premium.

---

## 2. La StatusCard (spec — **v3 finale, 07-08. Historique : v1 bande teal rejetée (masse vide), v2 bande verre + dock sticky rejetée (dock sans nécessité, 80 % de vide en watched)**)

**Carte** teal deep (`bg-accent-watching`, recette d'ombre ex-My Record : rim + hairline + lift), `rounded-card p-4` — le pattern **hero card cross-module** de l'owner (1ʳᵉ carte du rail = surface `-deep` brandée, cf. Goals Focus). Une carte empile verticalement → **la densité s'adapte à l'état, aucun état ne la laisse vide** (la leçon des v1/v2 : le format bande n'était rempli que par in_progress).

**Position** : desktop = **rail droit #1** · mobile = **juste après le hero, avant My Take** (non négociable — sinon retour du bug P0 « action quotidienne enterrée »). Rendue deux fois (`lg:hidden` / `hidden lg:block`), React Query dédoublonne les fetches.

**Anatomie fixe 3 zones** : header `[chip état] … [♥][menu …]` · body = les **faits** de l'état (lignes label/valeur) · footer = **LE geste** de l'état (bouton blanc/texte noir pleine largeur). Accent = dot teal vivid du chip In Progress uniquement.

| État | Body | Footer | Menu `…` |
|---|---|---|---|
| watched | `Watched — 2021 ✎` (éditable films/1-saison — **le picker de Details migré ici, ligne Details supprimée, zéro doublon**) · `Rewatches — 2 · last 12 Mar` | `+ Log rewatch` | Log a past rewatch (date inline) · **Delete** |
| in_progress | steppers verticaux `Season − 3/3 +` / `Episode − 2/8 +` | `✓ Mark as completed` (dernier ép. seulement, spring) | Pause · Drop · Mark watched · **Delete** |
| want_to_watch | `Where to watch` + logos providers (section rail masquée dans cet état) | `▶ Start watching` (série) / `✓ Mark as watched` (film) | Mark watched (série) · Add a note · **Delete** |
| paused | « It kept its position — S6 · E1 » | `↻ Resume watching` | Drop · Mark watched · **Delete** |
| dropped | `Reason — Lost interest ✎` (→ CaptureSheet) | `↻ Resume watching` | Mark watched · Change reason · **Delete** |

**Delete** (tous états) : item rouge en bas du menu → `DeleteConfirmModal` partagé (celui de la Library) + `useDeleteMedia` → toast + `router.back()`. La suppression n'était disponible que dans la Library — la fiche la gagne.

**Absorbe en plus (v3)** : la section **Rewatches** du rail (données + backdate) et la **ligne Watched de MediaDetails** (Details = catalogue pur désormais). Motion : AnimatePresence fade+y court sur chip/body/footer au changement d'état. **Pas de sticky, pas de dock.**

| État | Chip | Contenu central | Action primaire | Menu `…` |
|---|---|---|---|---|
| want_to_watch | Want to Watch | logos providers (« où le voir » = LA question de cet état) | **Start watching** (série) / **Mark as watched** (film) | Mark as watched (série) · Add a note |
| in_progress | In Progress | **stepper S/E compact** (blanc-sur-teal) ; dernier épisode atteint → le primaire « Mark as completed » apparaît (spring) | Mark as completed (contextuel uniquement) | Pause · Drop · Mark as watched |
| paused | Paused | « Kept its position — S{n} · E{n} » | **Resume watching** | Drop · Mark as watched |
| dropped | Dropped + chip raison (tap → CaptureSheet) | — | **Resume watching** | Mark as watched · Change reason |
| watched | Watched · {année} | compteur rewatches si > 0 | **+ Log rewatch** (1-tap aujourd'hui) | Backdate → (transitoire : picker de la section Rewatches) |

- ♥ Favorite : tous les états **sauf want_to_watch** (aimer un titre jamais vu n'a pas de sens ; comportement actuel conservé).
- **Pas de gros CTA pendant le visionnage** : le stepper EST l'interaction primaire d'in_progress ; « Mark as completed » n'apparaît qu'au dernier épisode (le moment de delight, springé).
- CTA hiérarchie corrigée : série → Start watching primaire (l'inversion actuelle est un P0 vérifié à l'écran).
- **Motion** : contenu central + primaire animés par `AnimatePresence` keyed sur l'état (fade+y léger, spring §9 design-system, < 300 ms de feel, zéro stagger).
- **Démo read-only** : toutes les actions restent VISIBLES et déclenchent le toast read-only (pattern existant — les handlers de page.tsx catchent `isDemoReadOnlyError`).

**Absorbe (composants dissous)** : `CurrentlyWatching.tsx`, `PausedNotice.tsx`, `DroppedNotice.tsx`, les CTAs/chips statut/favorite de `MyTakeRecord.tsx`, le bouton « Log rewatch » de `Rewatches.tsx` (la *donnée* rewatches reste dans History).

## 3. My Take fusionnée (spec — v2 : une COMPOSITION, pas une boîte)

Une carte `surface-card rounded-card p-5` = **rating + note** (ton verdict est UNE chose). Header de section « My Take » standard (`text-title`, hors carte). **Rangée verdict** : le chiffre d'abord (`8.5` 3xl tabular + « Great » ambre) et le slider **sur la même ligne** à droite (contraint `max-w-sm` — une échelle 1→10 étirée sur toute la colonne perd son sens). **Rangée note** : typographie journal `text-body leading-7`, éditeur seamless (mêmes métriques, auto-grow, aucun saut visuel). Vide : « How was it? » + slider sur une rangée + ligne fantôme « + Add a note ».

- **Avec contenu** : lecture typographique — note en `text-body leading-relaxed` (PAS un textarea visible), rating en ligne de valeur (`7.5 / 10 Good` ambre). Clic sur la note → bascule textarea. **Autosave debounced ~800 ms** (comme `updateProgress`), indicateur « Saved » discret. → **`FloatingSaveBar.tsx` supprimée** + machinerie isDirty de page.tsx (règle au passage l'overlap mini-player documenté dans now.md).
- **Vide (états vus)** : carte **compacte** = slider seul (la capture la moins chère) + ligne fantôme « Add a note… » qui s'étend au focus. ~⅓ de la hauteur actuelle. Pas de compteur de mots (chrome de formulaire).
- **want_to_watch** : carte **absente** (pas de rating pré-visionnage). « Add a note » du menu StatusBar force l'ouverture de l'éditeur si besoin ; la carte n'apparaît ensuite que si une note existe.
- Slider : le `RatingSlider` actuel (labels Masterpiece/Great/…) réutilisé tel quel. (Le fix « knob ambigu à 0 » est un P0 séparé.)

## 4. History fusionnée (spec — P1, à faire)

Une seule section « History » pour la mémoire temporelle :
- Série multi-saisons : le **SeasonHistoryStrip inchangé visuellement** (hero feature : Now badge, masques Coming soon/Lock, popover année+note) + pills rewatches dessous (watched).
- Film / série 1 saison : ligne « Watched {année} ✎ » (l'éditeur `WatchedYearRow` extrait de `MediaDetails.tsx:50-96`) + pills rewatches. `Rewatches.tsx` absorbé.
- in_progress : strip seul. want_to_watch : absente.

---

## 5. Carte de placement — les 22 composants

Verdicts : **G**arder · **D**éplacer · **F**usionner · **S**upprimer.

| # | Composant | Verdict | Notes |
|---|---|---|---|
| 1 | MediaHero | G (retouches P0/P2) | Desc clampée 3 lignes + « More » sur les DEUX breakpoints (mobile = intégrale aujourd'hui, desktop = coupure brute) ; gradients `zinc-950`→`surface-0` ; posters `rounded-tile`. |
| 2 | TrailerModal | G | RAS. |
| 3 | MyTakeRecord | **F** → StatusBar + My Take | Cf. §2-3. |
| 4 | CurrentlyWatching | **F** → StatusBar | Supprimé après absorption. |
| 5 | PausedNotice | **F** → StatusBar | Copy « kept its position » migre telle quelle. |
| 6 | DroppedNotice | **F** → StatusBar | Raison = chip éditable (CaptureSheet réutilisée). |
| 7 | Rewatches | **F** → History (données) + StatusBar (action) | Transitoire : section conservée tant que History pas fusionnée. |
| 8 | SeasonHistoryStrip | G/D | Remonte sous My Take. Hero feature préservée pixel pour pixel. |
| 9 | Episodes | G/D | **Caché en want_to_watch** (P0 — spoilers + actions sans objet, vérifié). Chips saisons → Select au-delà de ~10 (mur de 24 chips One Piece vérifié). Stills+Best Ep+note/ép préservés. |
| 10 | EpisodeHighlights.tsx | **S** | Orphelin (déjà acté now.md). |
| 11 | EpisodeHeatmapModal | G (dormant) | `HEATMAP_ENABLED=false` en attendant ingest IMDb. |
| 12 | CastCrew | G/D | Descend sous Episodes (catalogue après personnel). |
| 13 | MoreLikeThis | G | Dernier de colonne. Titre `text-base`→`text-title`. |
| 14 | ContributingToGoals | **D** → rail au-dessus d'In List | Famille « connexions ». `bg-surface-2/40` → carte quiet propre. |
| 15 | InList | G | Popover `surface-2`→`surface-3` (couches inversées) ; empty state compressé (header + `+` seulement). |
| 16 | WatchProviders | G rail #2, **masqué want_to_watch** | Logos dans la bande dans cet état. Zéro doublon. |
| 17 | ExternalRatings | G rail #3 | Le rail Parasite (3 cellules + Oscars + box office) = étalon référence. |
| 18 | AnimeThemes | G rail #1 (anime), tous états | Écouter les OP avant de regarder = vrai usage découverte. Mobile : après Episodes. |
| 19 | MediaDetails | G rail dernier, **purgé** | Retirer Type/Year/Runtime/Seasons (doublons hero vérifiés) + Watched (→ History). Reste : Age rating, Episodes, Status, Studio, IMDb. |
| 20 | FloatingSaveBar | **S** | Remplacée par autosave (cf. §3). |
| 21 | GoalRipple/HabitRippleToast | G | Le moment « ripple » est excellent. |
| 22 | CaptureSheet (partagée) | G (+1 usage) | Édition raison de drop depuis le chip de la bande. |

---

## 6. Système de cohérence (répliqué dans `hq/design-system.md` — source de vérité là-bas)

**Matériaux — 4 rôles, zéro exception** : brandée (StatusBar seule) · manage `surface-card` (on y écrit) · référence `surface-quiet` (lecture) · browse open (rangées de tuiles média ; `rounded-tile` posters / `rounded-card` stills 16:9 ; scale hover 1.03-1.04 = idiome Watching). Couches : carte `surface-1` · champs `surface-2` · popovers `surface-3`.

**Dominance** : 1 dominant/écran = StatusBar (seule couleur). 2 secondaires max = My Take + hero feature de l'état. Ambre = rating uniquement ; vert = rows Goals uniquement.

**Typo** : `text-title` headings · `text-body` lecture · `text-label` metadata · `text-caption` micro. Les px bruts (9/10/11/13) se mappent dessus ; hiérarchie au poids ; max 3 tiers/composant. Exceptions chiffrées : valeur rating `text-3xl tabular-nums`, titre hero.

**Rythme** : sections 32px desktop / 24px mobile (rail aligné — fini le 24 vs 32) · heading→contenu 12px · padding carte 16px.

**Information Priority — Watching** : Primary = ton état & verdict (statut, S·E, rating) · Secondary = ta mémoire (take, années, rewatches, best eps) · Tertiary = catalogue (cast, providers, notes externes, details, recos). Règle : le Tertiary ne passe JAMAIS au-dessus d'un Secondary visible.

---

## 7. Punch-list

### P0 — casse l'usage — ✅ TOUS FAITS (`a93d08d` / `dba92c1` / `b5b3e2b`)
- [x] 1. Ordre mobile : réglé par la StatusCard ; réordonnancement des sections fait (= P1-9).
- [x] 2. Desc hero — **troncature JS**, pas de line-clamp : un masque de couleur ne se fond jamais sur le backdrop photo. « …More » inline, `max-w-4xl`, 2 breakpoints.
- [x] 3. ~~CTA inversé want_to_watch~~ → réglé par la StatusCard (série → Start watching primaire).
- [x] 4. Slider non noté : knob masqué + piste atténuée. **+ owner : chiffres AU-DESSUS de la barre** (le doigt ne couvre plus le chiffre visé).
- [x] 5. Episodes en want_to_watch → **pas caché mais `readOnly`** (décision owner : la valeur catalogue reste ; seules les actions absurdes — noter/best-ep un épisode non vu — disparaissent).

### P1 — restructuration
- [x] 6. **StatusCard** (ex-StatusBar ; v3 = carte teal deep — 07-08 nuit).
- [x] 7. **My Take fusionnée + autosave + exit FloatingSaveBar** (07-08 nuit).
- [x] 8. **History** — résolu autrement : rewatches + année Watched **absorbés dans la StatusCard**, le SeasonHistoryStrip reste seul à gauche. Pas de section « History » séparée.
- [x] 9. Réordonnancement — gauche : Take→History→Episodes→Cast→MoreLikeThis · rail : StatusCard→Themes→Details→Contributing→InList.
- [ ] 10. Saisons scalables : > 10 → Select — `Episodes.tsx` (mur de 23 chips sur One Piece). **SEUL P1 RESTANT.**
- [x] 11. Details purgé (Year/Runtime/Seasons/Status) **+ enrichi gratuitement** via l'OMDb déjà fetché : Awards parsé, Aired/Released, Country, Language, Box office, IMDb votes. Section « Ratings » du rail dissoute (scores → hero, inline dans la meta row).
- [ ] 12. Matériaux : Contributing quiet (`ContributingToGoals.tsx:36`) ; InList popover surface-3. (`Rewatches.tsx` n'existe plus.)

### P2 — hygiène tokens *(agent Opus)*
- [ ] 13. `rounded-2xl`→`rounded-card` : `MediaDetails.tsx:137` · `ExternalRatings.tsx:36` · stills `Episodes.tsx:63` (`rounded-xl`→`rounded-card`). (MyTakeRecord/CurrentlyWatching/FloatingSaveBar : réglés par la refonte.)
- [ ] 14. Hero : gradients `zinc-950`→`surface-0` (`MediaHero.tsx:64,153-154`) ; posters `rounded-tile` (`:88,:180`).
- [ ] 15. `MoreLikeThis.tsx:31` `text-base`→`text-title`.
- [ ] 16. Sweep micro-typo par tokens (StillCard, pills SeasonHistoryStrip).
- [ ] 17. `text-white` pur → tokens hors contexte image/brandé.
- [ ] 18. Supprimer `EpisodeHighlights.tsx` (orphelin).
- [ ] 19. ~~Bannière last-episode border colorée~~ → dissoute dans la StatusBar.
- [ ] 20. IMDb chip `hover:scale-105`→`transition-colors` (`MediaDetails.tsx:130`) ; empty state In List compressé.

### Docs
- [x] `hq/design-system.md` : row Info Priority Watching + exceptions §8.6 + matériaux par rôle (07-08).
- [x] Skill `/design-check` : chemins + accent corrigés (07-08).
- [ ] `hq/watching_v3.md` : ligne « design figé » → pointer ce blueprint.

---

## 8. Les 2 réserves (à lever pendant l'exécution)

1. ✅ **LEVÉE (07-09)** — état `dropped` vérifié à l'écran sur **One Piece** : chip « Dropped », ligne Reason avec affordance « Add a reason », bouton Resume, menu (Mark as watched / Change reason / Delete). Rien à corriger.
2. ✅ **CADUQUE** — le risque « toolbar générique » disparaît avec la v3 : c'est une **carte** (teal deep, chip d'état, faits, un geste), pas une bande d'icônes.

## 9. Notes d'exécution

- **Commits** : owner valide d'abord, single-line, zéro mention IA.
- **Démo** : ne jamais cacher les actions en démo — visibles + toast read-only (`isDemoReadOnlyError`).
- **Screenshots obligatoires** après chaque itération : `node scripts/shoot.mjs /perso/watching/<id> out.png` (+ `--w=390` mobile). Ids de test récoltés le 07-08 : HotD in_progress `5590ed88-b682-451d-b325-cd4a229e0ee4` · One Piece paused `1c735879-e581-447c-8edc-19767e944a6b` · HxH watched-série `f92b6e4f-fc72-4112-9426-69416f9ca7b0` · Parasite watched-film `ed8be2bd-9c37-4bff-b070-24af146a5c89` · Mushoku Tensei want `05d0f77d-d955-46f8-9ced-1da527bf9a4e`.
- ⚠️ Le mot de passe du compte démo noté dans now.md ne fonctionne plus (vérifié 07-08).
