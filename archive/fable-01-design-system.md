# Audit externe — Design system HEGON

> Fable · 2026-07-13 · Aucun fichier modifié hors celui-ci. `hq/` non lu (consigne), à l'exception de ce fichier.

---

## 1. Ce que j'ai réellement inspecté

**Lu en entier (code)** :
- `src/app/globals.css` (701 l.) — tokens, matériaux `@utility`, base, scrollbars, reduced-motion.
- `src/shared/utils/utils.ts` — `cn()` + config `extendTailwindMerge`.
- `scripts/ds-coverage.mjs` (134 l.) + `coverage.generated.json` (via extraction Node : totaux, adoption, worst files).
- **24 des 28 primitifs, en entier** : badge, button, carousel-nav, color-palette, dialog, ErrorFallback, filter-select, icon-picker, inline-form-actions, input, label, motion, panel, popover (headers + content), search-input, section-header, segmented-control, select (partiel : 40 premières lignes, variants), slider, sliding-panel, tab-nav, tabs, textarea, tooltip.
- **Skimmés seulement** : calendar, command (extraits via grep), dropdown-menu, form — tailles + grep de tokens (aucun token HEGON dans les quatre ; `command` contient des `zinc-*`).
- `StyleguidePage.tsx` : 110 premières lignes (mécanique token-reader, typage coverage) + rendu à l'écran.

**Vu à l'écran (screenshots, session authentifiée, 1440×900)** :
- `/settings/styleguide` : haut (Foundations), milieu (Controls, Badges), fin (Coverage « DEBT, WORST FIRST », Known gaps).
- `/perso/watching` (module migré, page Movies).
- `/pro/tasks` (état « No project selected » uniquement).
- `/life/books` (la route `/perso/books` de l'énoncé **404** — Books vit sous `/life/books`).

**Mesures exécutées (grep/Node, chiffres reproductibles)** :
- Variants réellement utilisés (`variant="…"`), adoption par primitif, worst files du JSON.
- `<Button` sans `variant=` : **72 / 118** occurrences (61 %).
- `animate-pulse` par module (skeletons artisanaux) : watching **51**, goals 16, sports 16, books 13… total ≈ 104.
- `zinc-*` par module : sports **593**, tasks 27, watching 24, books/goals/habits/journal **0**.
- Hex inline par module : tasks 76, watching 68, habits 31, goals 20, books 3, journal 1.
- Importeurs réels de `tabs`, `command`, `label`, `ErrorFallback`, `panel`, `section-header`.

---

## 2. Constats — faits vérifiables

1. **Deux primitifs sont morts.** `tabs.tsx` (shadcn stock) : zéro importeur dans tout `src/`. `command.tsx` : zéro importeur — même `command-center` ne l'utilise pas (aucun import cmdk ni `ui/command` dans ce module). `tabs` collisionne de surcroît avec `tab-nav` par le nom.
2. **Le défaut des primitifs de champ est le legacy.** `input`, `textarea`, `select` ont deux variants : `default` (shadcn : `bg-input/30`, ring 3px) et le variant HEGON… nommé **`tasks`** (du module où il est né). Le bon chemin coûte une prop à chaque call site ; 54 occurrences de `variant="tasks"` la paient aujourd'hui.
3. **Même inversion sur Button** : 12 variants (6 shadcn + 6 HEGON), le commentaire du fichier dit « prefer [HEGON] in new code », mais `default` reste `bg-primary` shadcn — et **61 % des `<Button>` du code utilisent le défaut**.
4. **Le module migré est 3ᵉ pire au compteur.** Watching : 71 « off-system hits » (41 = `raw-scrim`, 19 = `handrolled-glass`, dont les scrims/pills de hero légitimes). La page Coverage l'affiche telle quelle. Le « Known gaps » du styleguide déclare lui-même que le compteur sur-compte.
5. **La métrique est aveugle au plus gros signal.** Les regex comptent `text-[13px]` mais pas `text-sm`/`text-xs` (l'échelle Tailwind stock, hors scale HEGON) ni les couleurs hors tokens. Sports = **593 `zinc-*`** invisibles au compteur ; Books = **0 zinc, 3 hex** — la matrice affiche pourtant sports 154 vs books 39, un écart de 4× là où l'écart réel de langage est de deux ordres de grandeur.
6. **Les primitifs ne parlent pas leur propre langue typographique.** La scale (`caption/micro/label/body/title`) existe, `cn()` la protège désormais — mais `button` est en `text-sm`, `filter-select` en `text-xs`, `DialogTitle` en `text-lg` (taille qui n'existe pas dans la scale), `tooltip` en `text-[11px]`. Seuls `badge`, `panel`, `section-header` consomment la scale.
7. **Deux systèmes de radius coexistent dans les primitifs.** Les rôles (`rounded-control`…) et des bruts : `tab-nav` `rounded-md`, `icon-picker` `rounded-lg`/`rounded-md`, `ErrorFallback` `rounded-lg`, `popover` défaut `rounded-md`.
8. **`ErrorFallback` est hors-système** : `zinc-100/500/700/800`, `bg-red-500/10`, lien `/dashboard` codé en dur. C'est l'écran des mauvais jours, et c'est le seul écrit dans l'ancien langage. Utilisé par 5 `error.tsx`.
9. **`slider.tsx` est du shadcn nu** (`bg-muted`, `bg-primary`) — 1 usage (goals). Le slider signature du produit (rating Watching) ne l'utilise pas.
10. **`icon-picker` réinvente Popover** : listener `mousedown` maison, fermeture manuelle, et ouverture **vers le haut codée en dur** (`bottom-[calc(100%+4px)]`) — il suppose son unique call site actuel (bas de panel Habits).
11. **`panel` et `section-header` dupliquent le même header** (title/subtitle/actions, mêmes classes) au lieu de se composer — deux sources de vérité pour la même chose, nées en même temps.
12. **`popover` et `dropdown-menu` gardent des défauts stock** (`w-72 p-4 bg-popover` / etc.) : chaque call site HEGON les réécrit (`bg-surface-3 border-border-strong…`) — la maladie que `filter-select` a précisément été créé pour soigner sur Select.
13. **Aucun primitif pour : Skeleton, EmptyState, Toast, confirmation destructive.** ~104 `animate-pulse` artisanaux (51 dans le module migré) ; `DeleteConfirmModal` vit dans `watching/components/modals/` ; sonner est configuré dans le layout avec des classNames `!important` non tokenisés. Le styleguide n'a pas de section Forms ni Feedback (nav : Foundations…Coverage/Known gaps).
14. **Trois vérités couleur cohabitent** : les tokens `:root`, la « AGREED MODULE PALETTE (NOT yet applied) » en commentaire de 20 lignes dans globals.css, et `HEGON_COLORS` (17 hex hardcodés dans `color-palette.tsx`). Plus les 11 tokens legacy `--section-*` + leurs 11 classes helpers.
15. **Les tokens z-index existent mais ne sont pas consommés** : tooltip `z-200`, dialog/popover `z-50`, sliding-panel `z-40/50` — tous en dur.
16. **Le reduced-motion global** (`* { animation-duration: 0.01ms }`) gèle aussi `animate-spin` — les spinners de chargement deviennent des icônes statiques sous `prefers-reduced-motion`.
17. **Points forts factuels** (pour que la liste ne mente pas par omission) : `cn()` corrigé et documenté ; `focus-visible` global ; `cursor:pointer` restauré une fois pour toutes ; l'échelle posters + `--rail-peek` (le tile mobile dimensionné par le « peek », pas en px) ; la doc glass-thin dans le CSS (le pourquoi, pas le quoi) ; le styleguide lit les tokens au runtime et **déclare ses trous** ; `segmented-control`/`search-input`/`carousel-nav`/`filter-select` règlent chacun un vrai problème d'alignement documenté dans leur commentaire.

---

## 3. Jugements — ce que je ferais différemment

> Tout ce qui suit est de l'ordre de l'opinion argumentée. Les hypothèses porteuses sont listées en §4.

### Q1 — Ce qui manque, et ce qui fera mal en premier

Par impact décroissant, pendant les 7 migrations à venir :

1. **Le flip des défauts** (ce n'est pas un « manque » de couverture, c'est LE manque) : tant que `default` = shadcn sur button/input/textarea/select/popover/dropdown, chaque écran migré peut régresser silencieusement à la première retouche. 61 % des Buttons sont déjà sur le mauvais défaut. À faire **avant** toute migration de module : renommer `tasks` → `default` (l'ancien default devient `legacy`), re-baser Button sur `quiet`, tokeniser les contents Popover/Dropdown. Un commit transversal, mécanique, qui rend toutes les migrations suivantes moins chères et surtout *stables*.
2. **Skeleton + EmptyState + confirmation destructive dans `ui/`** : ce sont les trois choses que chaque module réinvente pendant une migration (104 pulse-divs le prouvent, et `DeleteConfirmModal` sera copié-collé au premier module qui supprime). Pas besoin de plus : un `<Skeleton className>` qui fixe la couleur/radius, un `<EmptyState icon title action>`, et promouvoir le modal de confirmation existant.
3. **Le feedback** : sonner tokenisé et documenté (une section « Feedback » au styleguide). Une app de capture perso vit de ses toasts.
4. **Loading/erreur inline** : `ErrorFallback` réécrit dans le système (10 lignes), et une convention d'erreur de champ (le `aria-invalid` existe déjà dans les variants — il n'est juste montré nulle part).
5. Ce que je n'ajouterais **pas** : i18n (mono-utilisateur, EN assumé — non-sujet), light theme (déclaré non-planifié, cohérent), densité configurable (l'échelle posters + rail-peek couvrent déjà le vrai problème de densité), tokens de spacing supplémentaires.

a11y, en deux lignes : au-dessus de la moyenne des projets perso (focus-visible global, Radix partout où ça compte, `useReducedMotion` dans les primitifs motion). Deux vrais trous : `text-tertiary` sur `surface-1` est vraisemblablement sous AA pour du texte 10–12px (assumé, non mesuré précisément — à décider consciemment, pas par défaut), et le nuke reduced-motion qui gèle les spinners (constat 16).

### Q2 — Niveau d'abstraction des 28, un par un

**À supprimer (3)** : `tabs` (mort + collision de nom), `command` (mort + zinc), `slider` (stock non adapté, 1 usage — soit l'adapter en 10 lignes soit le descendre dans goals ; un primitif à usage unique qui n'a même pas été habillé n'est pas un primitif).

**À réécrire/re-baser (6)** : `input`/`textarea`/`select` (flip du défaut + renommage `tasks`), `button` (flip + purger les variants legacy inutilisés — `link`, `secondary` à 1 usage), `ErrorFallback` (réécrire dans le système), `icon-picker` (re-baser sur Popover ; l'ouverture vers le haut hardcodée cassera à la 3ᵉ utilisation ailleurs — sinon le descendre dans habits, c'est du single-usage).

**À fusionner (2)** : `panel` + `section-header` — un seul header de section, `Panel` le compose. Deux fichiers nés le même jour qui dupliquent les mêmes 10 lignes, c'est le début du drift interne.

**Sur-conçus mais tolérables (2)** : `badge` (5 variants × 3 tailles × dot × uppercase ≈ trop de surface pour 16 usages réels — mais la grammaire flag/metadata/number est la meilleure idée du système, je n'y toucherais pas ; la surface excédentaire se taillera d'elle-même à l'usage) ; `motion` (`Spring.Launchpad/Widget` nommés d'après dashboard-os, non consommés par le module vitrine — le motion-DS vit à deux endroits, cf. Q5).

**Justes (le reste, ~15)** : `carousel-nav`, `search-input`, `segmented-control`, `filter-select`, `inline-form-actions`, `tab-nav`, `sliding-panel`, `dialog`, `tooltip`/`Hint`, `section-header`, `panel` (modulo fusion), `form`, `calendar`, `label` (plumbing, pas produit — le sortir du compteur d'adoption), `popover`/`dropdown-menu` (modulo défauts). La signature commune des réussites : petits, un seul problème réel, et le **pourquoi** écrit dans le fichier. C'est le bon niveau d'abstraction ; le système n'a pas de problème de sur-ingénierie généralisée, il a un problème de **défauts et de doublons**.

Détail transversal : trois primitifs ont un accent par défaut `--color-accent-goals` (`tab-nav`, `inline-form-actions`) — le folklore du module d'origine, comme `variant="tasks"`. Les défauts devraient être neutres ou obligatoires.

### Q3 — La métrique (`ds-coverage.mjs`)

Ce n'est **pas du théâtre** : c'est généré, honnête, auto-critique (« a number that flatters you is useless »), et le « worst file » par smell est réellement actionnable. Mais elle compte la mauvaise unité, et ça se voit à deux endroits :

- **Elle ne peut pas guider la priorité** : sports 154 vs books 39 (4×) alors que le vrai écart de langage est ~200× (593 zinc vs 0). Les deux plus gros signaux de non-migration — couleurs hors tokens et échelle typo stock — sont invisibles par construction.
- **Elle punit le module vitrine** : watching 3ᵉ pire, dont 41 scrims majoritairement légitimes. Un compteur que le module de référence ne peut pas faire baisser cessera d'être lu.

Ce que je compterais à la place (toujours généré, mêmes ~130 lignes) :
1. **Couleur hors token** : hex inline + classes de palette Tailwind (`zinc-*`, `slate-*`, `red-400`…) hors liste blanche (`white/black` sur artwork, brand colors déclarées). C'est LE proxy de migration.
2. **Typo stock** : `text-xs|sm|base|lg|xl` comptés comme off-scale au même titre que `text-[13px]`.
3. **Ré-implémentations** : fichiers de module dont le nom matche `(Modal|Panel|Badge|Skeleton|Tabs|Select)` sans import du primitif correspondant — le coût réel n'est pas la classe arbitraire, c'est le composant parallèle.
4. **Densité, pas volume** : hits **par fichier**, pas par module — 66 fichiers ne doivent pas coûter 66× plus.
5. Retirer `raw-scrim` tel quel (sur-compteur assumé) ou le restreindre aux éléments à texte.
6. Scanner aussi `src/app/(main)` et `src/shared/components` hors `ui/` — les pages routées échappent au compteur aujourd'hui.

### Q4 — L'ordre de propagation

La matrice (dette brute, pire d'abord) donnerait sports → dashboard-os → tasks. Je ferais presque l'inverse. Le critère juste pour un système à 1/8 modules migrés n'est pas « où est la dette » mais « **qu'est-ce qui prouve que le système généralise, au moindre coût, sur les surfaces que tu vois tous les jours** ».

0. **D'abord le flip transversal des défauts** (Q1.1) — sinon chaque migration re-paie la taxe.
1. **Books** — 0 zinc, 3 hex, 8 primitives déjà, l'écran parle déjà le langage (vu) : une session, victoire complète, et la première **deuxième** preuve du système.
2. **Goals puis Habits** — 14 primitives utilisées chacun, surfaces cartes classiques ; c'est là que Skeleton/EmptyState partagés se rentabilisent immédiatement.
3. **Journal** — petit (16 smells), vite clos.
4. **Tasks** — avant-dernier, exprès : surface la plus atypique (kanban, drag), 76 hex, le plus grand risque de régression d'interaction. On y arrive avec un système éprouvé par 5 modules.
5. **Sports** — **pas une migration, une reconstruction**. 593 zinc = c'est l'app d'avant, intacte. Migrer classe par classe est le pire ROI du projet ; je le traiterais comme un redesign qui *consomme* le système, planifié comme tel.
6. **dashboard-os** — à sortir de la liste : c'est la surface de référence du matériau glass, avec son propre moteur. Son « chantier système » n'est pas d'adopter des primitives mais de **verser son motion dans `motion.tsx`** (cf. Q5).

### Q5 — Ce qui sera regretté dans six mois

1. **Avoir ajouté les variants HEGON à côté des défauts shadcn au lieu de re-baser les défauts.** C'est la décision structurelle. Le styleguide enseigne `quiet/glass/contrast` ; la mémoire musculaire (et tout codegen — les agents qui écrivent ce code ont le réflexe shadcn) tape `<Button>` et `<Input>` nus. Le système perd par défaut, littéralement. Chaque semaine qui passe ajoute des call sites au mauvais défaut (118 Buttons aujourd'hui) et renchérit le flip.
2. **`variant="tasks"` et `accent = goals` par défaut** — le folklore des modules d'origine fossilisé dans l'API. Dans six mois, plus personne ne saura pourquoi le champ canonique s'appelle « tasks ».
3. **Trois vérités couleur** (tokens appliqués / palette « agreed, not applied » en commentaire / `HEGON_COLORS`). Quelqu'un appliquera la moitié de la palette commentée, et le styleguide — qui lit les tokens *live* — dira la vérité d'un système que plus personne ne saura nommer. Trancher : appliquer ou supprimer le commentaire.
4. **Le motion-DS à deux endroits** : `Spring.{Launchpad,Widget}` dans `motion.tsx`, et des springs locaux dans le module vitrine (`SPRING_SMOOTH/SNAPPY` dans `StatusCard.tsx`). Le styleguide l'avoue (« undocumented »). Six mois = six familles de springs, et l'uniformité de mouvement — l'argument même du système — sera morte. C'est le moment de verser : trois tokens (`entrance`, `interactive`, `hero`) et on gèle.
5. **La métrique actuelle comme instrument de pilotage** : elle contredira l'expérience perçue (module vitrine 3ᵉ pire, sports « seulement » 4× books) jusqu'à ce qu'elle ne soit plus consultée. Une métrique qu'on n'ouvre plus est pire que pas de métrique : elle donne l'illusion d'un contrôle.

### Ce que je retirerais (net)

`tabs.tsx` · `command.tsx` · `slider.tsx` (ou 10 lignes d'adaptation) · les variants Button legacy non utilisés après flip · `HEGON_COLORS` (brancher sur les tokens) · le bloc « AGREED PALETTE » commenté (appliquer ou supprimer) · les 11 `--section-*` + helpers dès que sports est reconstruit · le smell `raw-scrim` de la métrique · `label` et le plumbing du compteur d'adoption (bruit « nobody » injustifié).

---

## 4. Ce que je suppose (hypothèses porteuses, non prouvées)

- **L'usage quotidien réel** : je suppose Watching/Habits/Tasks quotidiens et Sports plus consultatif — l'ordre de propagation (Q4) en dépend en partie. Si Sports est en réalité l'écran le plus ouvert, son traitement « reconstruction plus tard » se discute.
- **Sports sera de toute façon redessiné** — déduit des 593 zinc et de son absence totale d'adoption (2 primitives), pas d'une roadmap que je n'ai pas lue (consigne hq/).
- **Le contraste `text-tertiary`/`surface-1` ≈ 3.5:1** : estimé, pas mesuré à l'instrument.
- **`variant="default"` de Button = involontaire dans la majorité des 72 cas** : certains sont peut-être des CTA blancs délibérés (le default EST un bouton clair). Le chiffre exact d'« erreurs » est entre 0 et 72 ; la thèse du défaut inversé tient même à 30.
- **Les 41 `raw-scrim` de watching sont majoritairement légitimes** : vérifié par sondage (worst file `ListDetail.tsx` : 10) et par connaissance des écrans, pas par revue exhaustive des 41.
- Le jugement « les primitifs récents sont au bon niveau » repose sur leur lecture + un seul module consommateur : la 3ᵉ utilisation réelle (Books) reste le vrai test.

## 5. Ce que je n'ai pas pu vérifier

- **Les écrans non visités** : `/perso/watching/library`, `/lists`, une fiche détail, `/perso/watching/person/287`, les trois écrans Sports, Goals/Habits/Journal, le dashboard-os, et Tasks au-delà de son empty state. Les jugements visuels reposent sur 6 captures.
- `calendar.tsx`, `dropdown-menu.tsx`, `form.tsx` : non lus en entier (tailles + grep de tokens seulement). S'il s'y cache une adaptation HEGON partielle, je l'ai manquée.
- Le comportement runtime des primitifs (focus clavier réel, lecteurs d'écran, overflow de textes longs) : rien testé au clavier ni au screen reader.
- Les consommateurs réels des tokens `--section-*` legacy (supposés : sports seul) et l'usage effectif de `Spring.Launchpad/Widget` dans dashboard-os (supposé, non grep-é).
- La représentativité du snapshot `coverage.generated.json` (daté du 2026-07-12) vis-à-vis du code du jour — je n'ai pas relancé `npm run ds:coverage`.
- Mobile : aucune capture `--w=390` dans cet audit.
