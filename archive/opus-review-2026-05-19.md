# HEGON — Opus Re-Review 2026-05-19

> **Mission** : re-audit technique + avis UX sans filtre + roadmap features par module.
> **Méthode** : lecture complète de [audit-progress-2026-05-18.md](audit-progress-2026-05-18.md) + [digger-audit-2026-05-18.md](digger-audit-2026-05-18.md), puis vérification réelle dans le code de chaque item marqué ✅ Fait.
> **Standard** : Linear / Vercel / Apple. Brutal, sans filtre.

---

## Table des matières

1. [Synthèse — Score global](#1-synthèse--score-global)
2. [Re-vérification item par item](#2-re-vérification-item-par-item)
3. [Scores re-notés par axe](#3-scores-re-notés-par-axe)
4. [Avis UX sans filtre](#4-avis-ux-sans-filtre)
5. [Roadmap features par module](#5-roadmap-features-par-module)
6. [TL;DR & next actions](#6-tldr--next-actions)

---

## 1. Synthèse — Score global

| Métrique | Valeur |
|---|---|
| **Score réel post-application** | **7.5/10** |
| Score audit v1 (2026-05-18) | 5.9/10 |
| Cible tracker | 7.8/10 |
| Progression | **+1.6** points |
| Items vérifiés ✅ confirmés | ~36 |
| Items ⚠️ partiels / écarts | 4 |
| Items ❌ non fixés (non-claimed) | 3 (§2.1, §2.8, §3.6) |
| Items ❌ tracker dit ✅ mais code dit non | 1 (§4.1 partiel) |

**Verdict brutal :** HEGON est passé de "produit fonctionnel à mi-distance Linear" à **"produit cohérent à 3/4 de la qualité Linear"**. Le bond est réel, mais 3 zones tirent vers le bas : (a) le skeleton KanbanBoard 5-OR jamais corrigé, (b) `WorkspaceInvitePanel` qui garde l'ancien style inline+onMouseEnter, (c) les `eslint-disable` file-wide qui survivent dans 2 fichiers. Aucun de ces trous n'empêche la démo, mais ils signalent "audit appliqué à 95%" — pas à 100%.

---

## 2. Re-vérification item par item

### Sprint 1 — Critiques bloquants démo

| # | Item | Statut réel | Note |
|---|---|---|---|
| §4.2 | Middleware code mort | ✅ confirmé | 76 lignes nettoyées, code lisible |
| §6.2 | Toast spam autosave | ✅ confirmé | `useUpdateTask.onSuccess` sans toast |
| §1.1 | Dashboard invité | ✅ confirmé | `.or(created_by, assignee_id)` dans `getTodayTasks` + `getTasksActivityByDay` |
| §1.2 | `getCurrentOrgId` déterministe | ✅ confirmé | `.order("created_at")` + auto-set après `acceptInvitation` |
| §5.1 | Profiles RLS | ✅ confirmé | Policy v2 couvre self + org-mates + workspace members + owner |

### Sprint 2 — Flash & flickering

| # | Item | Statut réel | Note |
|---|---|---|---|
| §1.4 | Optimistic update préserve joins | ✅ confirmé | Résolution status + assignee depuis cache avant patch |
| §2.3 | `useRealtimeTasks` chirurgical | ✅ confirmé | `setQueryData` pour UPDATE/DELETE + INSERT invalide |
| §2.2 | `useRealtimeTags` ciblé | ✅ confirmé | Skip INSERT, scan caches pour patch ciblé |

### Sprint 3 — UX quick wins

| # | Item | Statut réel | Note |
|---|---|---|---|
| §2.6 | TopBar `getSession()` | ✅ confirmé | localStorage sync, 0ms |
| §6.6 | TaskCard due_date année | ✅ confirmé | `MMM d` vs `MMM d, yyyy` conditionnel |
| §2.4 | Input variant tasks | ✅ confirmé | `Input variant="tasks"` ligne 123 |
| §2.5 | onMouseEnter inline supprimés | ⚠️ **partiel** | Tasks Topbar/Sidebar OK, mais **WorkspaceInvitePanel** garde 5 occurrences (lignes 163-164, 185-186, 378-379, 389-390, 513-514) |
| §4.5 | TaskCard tout Tailwind | ⚠️ **partiel** | dueTone dynamique OK, mais `style={{ color: var(--color-text-secondary)}}` non-dynamique persiste (description, p.122) |
| §2.7 | `useCurrentUserId` sync | ✅ confirmé | `readUserIdFromStorage` dans useState initializer |

### Sprint 4 — Performance perçue

| # | Item | Statut réel | Note |
|---|---|---|---|
| §3.3 | `getCurrentOrgId` in-flight memo | ✅ confirmé | Promesse partagée + reset au logout |
| §3.4 | Dashboard inner join + filter | ✅ confirmé | `statuses!inner` + `eq("statuses.is_completed", false)` |
| §3.5 | `getMediaItems` limit 100 | ✅ confirmé | `options.limit ?? 100` |
| §3.1 | Cookie middleware | ✅ confirmé | 5 garde-fous présents (signOut, /onboarding, 7j, accept, deleteWorkspace) |
| §3.2 | RPC `recalc_goal_progress` | ⚠️ **écart** | RPC + helper extrait OK, MAIS migration `RETURNS void` + `SECURITY DEFINER` au lieu de `RETURNS INT` + `SECURITY INVOKER` (audit spec §A.5). Pas de garde `IF goal_mode != 'auto'` dans la fonction SQL → check mode reste côté JS (1 fetch goal en plus si cache froid). Fonctionnel mais moins optimal que la spec |
| §3.7 | Bundle analyzer | ⏭ reporté | Conforme tracker |

### Sprint 5 — Tasks design retrofit

| # | Item | Statut réel | Note |
|---|---|---|---|
| §6.1 | Design pass complet | ⚠️ **partiel** | KanbanColumn / ListRow / CalendarDayCell OK, mais TasksTopbar et TaskCard gardent beaucoup d'inline `style={{ var(--color-*) }}` au lieu de Tailwind tokens. Esprit pas pleinement respecté |

### Sprint 6 — Sécurité

| # | Item | Statut réel | Note |
|---|---|---|---|
| §5.2 | Middleware defense-in-depth | ✅ confirmé | `checkUserHasWorkspace` parallel + `.eq("user_id")` explicite |
| §5.3 | `createWorkspaceInvitation` owner-only | ✅ confirmé | Check `workspace.user_id !== user.id` |
| §5.4 | Rate limit 10/h | ✅ confirmé | Count `org_invitations` WHERE invited_by + créé < 1h |
| §5.5 | Erreurs génériques | ✅ confirmé | "Couldn't update task. Try again." + console.error |
| §5.6 | Token regex | ✅ confirmé | `/^[A-Za-z0-9_-]{20,128}$/` |
| §5.7 | next.config + CSP | ✅ confirmé | CSP complet, remotePatterns explicites, pas de `**` |

### Sprint 7 — Crashes restants

| # | Item | Statut réel | Note |
|---|---|---|---|
| §1.3 | Position trigger | ✅ confirmé | Migration `20260519_audit_fix05` + service.ts ne passe plus position |
| §1.5 | priority_rank | ✅ confirmé | Migration `20260519_audit_fix06` + dashboard service `.order("priority_rank")` |
| §1.6 | Network error invite | ✅ confirmé | State `network_error` + Retry button + retryKey |
| §1.7 | formatEventDate DST | ✅ confirmé | Date diff via strings Paris YYYY-MM-DD |
| §1.8 | useInitOrg toast | ✅ confirmé | console.error + toast.error |

### Sprint 8 — Qualité

| # | Item | Statut réel | Note |
|---|---|---|---|
| §4.3 | `getOrgMembers` supprimé | ✅ confirmé | Plus présent dans `tasks/service.ts` |
| §4.4 | Helper extrait | ✅ confirmé | `recalcGoalIfAuto` dans `useTasks.ts` |
| §4.6 | `WorkspaceInvitePanel` useRealtimeSync | ✅ confirmé | 2× `useRealtimeSync` (lignes 71-85) |
| §4.7 | useTasks typés | ✅ confirmé | `setQueryData<Task[]>` partout |
| §4.8 | `Promise.allSettled` | ✅ confirmé | Dashboard service + fallback par section + console.error |
| §4.1 | eslint-disable file-wide | ❌ **partiel — tracker ment** | Retiré de `tasks/service.ts`, mais **toujours présent dans `tasks/types.ts:1` et `dashboard/service.ts:1`**. Tracker dit "✅ Fait" — c'est faux pour 2 fichiers/3 |

### Sprint 9 — UX & cohérence

| # | Item | Statut réel | Note |
|---|---|---|---|
| §6.3 | Etat d'erreur listes | ✅ confirmé | KanbanBoard + ListView ont `isError` + retry |
| §6.4 | w-110 → w-120 | ✅ confirmé | `w-120` ligne 139 |
| §6.5 | TaskCard touch handle | ✅ confirmé | `[@media(hover:none)]:opacity-50` |
| §6.7 | Empty state zero orgs | ✅ confirmé | DashboardContent Refresh + lien `/onboarding` |
| §6.8 | z-9999 → z-20 | ✅ confirmé | TopBar, Sidebar (×2), Dock |

### Items audit non-claimed (gaps connus à traiter)

| # | Item | Statut | Note |
|---|---|---|---|
| §2.1 | KanbanBoard skeleton 5-OR | ❌ **non fixé** | Toujours 5-OR condition ligne 153-159. Pas dans le tracker — vrai gap UX |
| §2.8 | useRealtimeWorkspaces sans filter | ❌ **non fixé** | Pas dans le tracker |
| §3.6 | useTasks staleTime 10 min | ❌ **non fixé** | Toujours `1000 * 60 * 3` (3 min) dans `useTasks.ts:41`. La note Sonnet du tracker disait 10 min — code jamais touché |

---

## 3. Scores re-notés par axe

| Axe | v1 (audit) | Cible | **v2 (réel)** | Justification |
|---|---|---|---|---|
| Crashes & erreurs silencieuses | 5/10 | 8/10 | **8/10** ✅ | Tous les invariants critiques fixés (priority, multi-org, position, DST, network errors, useInitOrg). Bon |
| Flash & flickering | 6/10 | 8/10 | **7/10** | Optimistic + realtime chirurgical OK, mais §2.1 (5-OR skeleton) toujours là → flash sur chaque retour à /pro/tasks |
| Performance perçue | 6.5/10 | 8/10 | **7.5/10** | Cookie + RPC + in-flight memo + allSettled. -0.5 car RPC `RETURNS void` (devrait être INT pour skip 1 round-trip extra) et staleTime tasks toujours à 3 min |
| Qualité du code | 6/10 | 8/10 | **7/10** | Helper extrait, types corrigés, services nettoyés. -1 car 2 eslint-disable file-wide persistent + inline `style={{ var(--color-*) }}` non-dynamiques jamais purgés |
| Sécurité & données | 6/10 | 8/10 | **8.5/10** ✅ | Profiles RLS strict, defense-in-depth middleware, owner-only invites, rate limit DB, token regex, CSP complet. Excellent. -0.5 car `recalc_goal_progress` en `SECURITY DEFINER` (mineur, scope du dégât = ~zéro) |
| UX & cohérence | 6/10 | 7/10 | **7/10** ✅ | Etats d'erreur ajoutés, z-index normalisé, empty states. Inline `style` + onMouseEnter dans WorkspaceInvitePanel = dette résiduelle |
| **Moyenne** | **5.9** | **7.8** | **7.5/10** | **+1.6 vs v1, -0.3 vs cible** |

**Cibles atteintes :** Sécurité dépasse, Crashes pile, autres -0.5 à -1.0.

---

## 4. Avis UX sans filtre

### 4.1 Le module Tasks tient la route mais montre encore les coutures du "récemment retrofité"

La barre `bg-surface-0` + filtres en dropdown + 3 view modes = du Linear. Le TaskCard avec assignee top-right + tags colorés + due_date overdue en orange = solide.

**Mais l'expérience d'invitation (`WorkspaceInvitePanel`) est l'élément le plus visible pour la démo collab** — et c'est le seul qui n'a PAS été retrofité. Email input natif, hovers inline, color picker en grid de pastilles "à l'arrache" comparé au reste.

> **Conséquence démo LinkedIn :** si tu fais la démo sur la collab, ce panel sera vu par tout le monde et il sent 6 mois de moins que le reste.

### 4.2 Le dashboard est ambitieux mais émotionnellement plat

Le composant "Today" avec 3 cards dynamiques (PRIORITY/UP NEXT/IN PROGRESS) est une idée forte. Mais visuellement c'est 3 rectangles avec un titre — pas de motion, pas de profondeur, pas d'effet "aha".

Une démo LinkedIn convaincante attendrait :
- (a) une animation d'arrivée séquentielle des cards
- (b) une preuve immédiate que les données viennent de **tous** les modules (badges/icônes plus marqués)
- (c) un état "tout est fait pour aujourd'hui" qui célèbre (confettis discrets, message)

Là c'est "fonctionnel mais sec".

### 4.3 Le skeleton de KanbanBoard est ton plus gros problème UX résiduel

À chaque clic vers `/pro/tasks` ou nav depuis n'importe où, tu vois 100-500ms de skeleton plein écran parce que le check est `workspaces || projects || statuses || tasks loading`. Sur un compte chaud avec cache, c'est inutile.

> **C'est le seul truc qui rappelle "app React avec spinners".** Un retour instantané sur les colonnes (skeletons locaux par colonne seulement) le ferait passer de "produit web" à "app native".

### 4.4 Les incohérences inter-modules existent mais sont subtiles

- Books utilise un layout grille très propre, Watching utilise des sliders Swiper, Tasks utilise kanban — **chacun a son langage et c'est OK**
- Mais les **boutons primaires d'action** divergent : Tasks utilise `bg-[#71717a]` gris, Goals utilise un vert/violet, Habits utilise une CTA `bg-rose` — **pas de couleur primaire unifiée**
- Les **modals** ont été uniformisés (GoalModal référence) mais EditTaskModal n'est pas encore à 100% au standard
- Les **avatars** : `MemberAvatar` dans Tasks, gradient `bg-linear-to-br` violet/indigo dans TopBar — incohérent

### 4.5 Les 3 quick wins UX les plus impactants

| # | Quick win | Impact | Effort |
|---|---|---|---|
| 1 | Fixer le KanbanBoard skeleton (§2.1) — afficher tasks+statuses dès qu'ils sont prêts, skeleton local par colonne | 🔥 Énorme — élimine le flash sur chaque nav vers /pro/tasks | ~1h |
| 2 | Retrofit `WorkspaceInvitePanel` au design system (Input variant, hovers Tailwind, color picker shadcn) | 🔥 Démo collab × 2 en perception qualité | ~2-3h |
| 3 | Animation d'arrivée séquentielle des 3 Today cards + état "all done aujourd'hui" | 🔥 Dashboard passe de "fonctionnel" à "produit fini" | ~2-3h |

**Bonus haut ROI vidéo :** ajouter une transition Framer Motion `layoutId` partagé entre `TaskCard` (kanban) et `EditTaskModal` (édition) — comme Linear. C'est la touche qui fait dire "wow comment t'as fait ça" en démo. ~4h.

---

## 5. Roadmap features par module

> Objectif : pour chaque module, identifier les 3-5 features qui le ferait basculer de "tracker fonctionnel" à "outil comparable aux meilleures apps dédiées".

### 🎬 Watching — réf : Letterboxd, Trakt, Notion watch tracker

| # | Feature | Impact / Effort | Pourquoi ça change tout |
|---|---|---|---|
| 1 | **Diary / activity log** — chaque watch enregistré avec date, rating, courte note | 🟢 Haut / 🟡 Moy | Letterboxd 90% du temps c'est ça : "qu'est-ce que j'ai vu cette année". Sans ça, HEGON Watching reste un wishlist tracker, pas un journal cinéphile |
| 2 | **Rating + review courts (text 2-line)** sur chaque média complété | 🟢 Haut / 🟢 Bas | Convertit "j'ai regardé X" en "je sais ce que je pensais de X". Indispensable pour la rétrospective annuelle |
| 3 | **Year wrapped / stats page** — top genres, runtime total, top 10 année | 🟡 Moy / 🟡 Moy | Une feature virale annuelle (capture LinkedIn-able) qui rend visible le tracking quotidien |
| 4 | **Custom lists** ("Films à revoir", "Top 2026", "Films de Tarantino vus") | 🟡 Moy / 🟡 Moy | Letterboxd vit de ça. C'est ce qui transforme une DB en jardin personnel |
| 5 | **Suivi épisodique TV** : tracker S2E5 avec date + commentaire par épisode | 🟡 Moy / 🔴 Élevé | Trakt vit de ça. Mais grosse complexité — à ne faire que si la base "diary" cartonne |

### 🎯 Goals — réf : Notion goals, Koan, Weekdone

| # | Feature | Impact / Effort | Pourquoi |
|---|---|---|---|
| 1 | **Goal timeline / progress chart** — courbe d'évolution % over time | 🟢 Haut / 🟡 Moy | Sans visualisation temporelle, un goal "60% done" ne dit rien. La courbe montre vélocité, plateaux, regressions — le vrai signal motivationnel |
| 2 | **Weekly check-ins** — chaque semaine, popup "où en es-tu sur ce goal ? Confiance 1-5 ?" | 🟢 Haut / 🟡 Moy | Koan/Weekdone construit toute leur valeur là-dessus. Force la rétrospection régulière |
| 3 | **Sub-goals / hierarchical decomposition** — un Goal peut avoir des sous-Goals | 🟡 Moy / 🟡 Moy | Notion goals fait ça. Indispensable quand un goal est "lancer ma boîte" — il faut le décomposer en 5-7 sub-goals plus actionables |
| 4 | **Goal templates** — "Faire un marathon", "Apprendre Rust", "Lever 100k€" pré-remplis | 🟡 Moy / 🟢 Bas | Onboarding × 3. Le user voit ce que l'app peut faire au lieu de partir d'une page blanche |
| 5 | **Streak / consistency metric par goal** (basé sur activité tasks/habits liées) | 🟡 Moy / 🟡 Moy | "Tu as bossé sur ce goal 12 jours sur 14" = feedback dopaminergique |

### 🔁 Habits — réf : Streaks, Habitica, Notion habit tracker

| # | Feature | Impact / Effort | Pourquoi |
|---|---|---|---|
| 1 | **Heatmap calendar GitHub-style** par habit (jaune→vert selon densité) | 🟢 Haut / 🟢 Bas | C'est LA visualisation iconique du habit tracking. Plus puissant qu'un streak chiffré |
| 2 | **Reminders** (notifications navigateur OU email matinal récap) | 🟢 Haut / 🟡 Moy | Sans reminder, un habit tracker = un cimetière de bonnes intentions. Streaks vit là-dessus |
| 3 | **Habit pause / skip days** — vacances, maladie, sans casser le streak | 🟢 Haut / 🟢 Bas | Sans ça, l'utilisateur abandonne au 1er jour skipped. Le "graceful failure" est crucial |
| 4 | **Custom frequency** : "3x/semaine", "lun/mer/ven", "1x/mois" — pas juste daily | 🟡 Moy / 🟡 Moy | Habitica/Streaks le permettent. Ouvre l'app à "courir 3x/semaine" qui est plus réaliste que daily |
| 5 | **Habit stats** : best streak, completion rate sur 30/90/365 jours, jour préféré | 🟡 Moy / 🟢 Bas | Pour la rétrospection. Renforce "je vois mes patterns" |

### 📚 Books — réf : Goodreads, Literal, StoryGraph

| # | Feature | Impact / Effort | Pourquoi |
|---|---|---|---|
| 1 | **Reading sessions / progress** — "j'ai lu pages 45→120 aujourd'hui" + temps | 🟢 Haut / 🟡 Moy | Goodreads l'a, StoryGraph l'a poussé plus loin. Sans ça, un Books tracker reste binaire (lu/pas-lu) |
| 2 | **Quotes / highlights collection** — capture des passages marquants par livre | 🟢 Haut / 🟡 Moy | Literal/Readwise construit toute leur valeur là-dessus. C'est la différence entre "je l'ai lu" et "voici ce que ça m'a appris" |
| 3 | **Year reading challenge** ("Lire 25 livres en 2026") + tracking dashboard | 🟡 Moy / 🟢 Bas | Goodreads vit de ça. 80% des users s'engagent avec un objectif annuel |
| 4 | **Mood / vibes tagging** — StoryGraph style ("dark", "hopeful", "fast-paced") | 🟡 Moy / 🟡 Moy | Différenciateur fort vs Goodreads basique. Idée de "search by mood" futur |
| 5 | **Currently reading widget dashboard** — % page actuelle + temps moyen lu/jour | 🟡 Moy / 🟢 Bas | Reconnecte avec le module depuis le dashboard. Sinon Books est "loin" |

### 📓 Journal — réf : Day One, Reflect, Notion journal

| # | Feature | Impact / Effort | Pourquoi |
|---|---|---|---|
| 1 | **Mood tracker quotidien** (slider 1-5 ou emoji) + corrélation visible avec entries | 🟢 Haut / 🟡 Moy | Day One a sa force là. Sans données quantifiées, le journal est juste "des paragraphes" — pas de pattern visible |
| 2 | **Daily prompts rotation** — "Qu'est-ce qui t'a fait sourire ?", "Une décision difficile cette semaine ?" | 🟢 Haut / 🟢 Bas | Combat le syndrome de la page blanche. Reflect/Day One le font. Onboarding × 2 |
| 3 | **Search / full-text** par contenu, tags, date range | 🟢 Haut / 🟡 Moy | Un journal sans search = un journal mort. Devient bouleversant quand tu retrouves une entry de 2 ans avant |
| 4 | **"On this day" — retour sur entries N years ago** | 🟡 Moy / 🟢 Bas | Day One feature signature. Crée l'attachement émotionnel à long terme |
| 5 | **Markdown rich + images** (étendre attachments) | 🟡 Moy / 🟡 Moy | Élève le journal de "notes" à "diary illustré" |

### 🏠 Dashboard — réf : Notion home, Superhuman, Linear home

| # | Feature | Impact / Effort | Pourquoi |
|---|---|---|---|
| 1 | **Customizable widgets** — drag/drop layout, on/off par section, layout persisté en DB | 🟢 Haut / 🔴 Élevé | Notion home / Linear home se définissent par ça. Chaque user a SA vue. Le "Customize" button du mockup doit exister |
| 2 | **Quick capture** (⌘N depuis n'importe où → task / journal / book) | 🟢 Haut / 🟡 Moy | Superhuman / Linear : la touche d'inbox universelle. Réduit la friction de 5 clics à 1 |
| 3 | **"Plan your day" view** — drag tasks from inbox to time slots (timeboxing matinal) | 🟢 Haut / 🔴 Élevé | Sunsama / Akiflow l'ont popularisé. Transforme HEGON de tracker en système de productivité actif |
| 4 | **Weekly review automatisée** — chaque dimanche, récap auto (X habits, Y tasks, Z books pages) | 🟡 Moy / 🟡 Moy | Boucle de feedback hebdo. Fait du dashboard un partenaire de réflexion |
| 5 | **Streaks dashboard global** — meta-streaks ("tu as utilisé HEGON 47 jours d'affilée") | 🟡 Moy / 🟢 Bas | Engagement compound. Donne envie d'ouvrir l'app même quand tu n'as rien d'urgent |

### ⚽ Sports (Football / Tennis / F1) — réf : Sofascore, FlashScore, ESPN

| # | Feature | Impact / Effort | Pourquoi |
|---|---|---|---|
| 1 | **Live scores ticker** — pendant un match, score qui update en temps réel | 🟢 Haut / 🔴 Élevé (API premium) | Sofascore vit là-dessus. Sans live, HEGON Sports = un wikipedia avec calendrier. Avec, c'est la 2e tab du téléphone |
| 2 | **Match notifications** — "Real Madrid joue dans 30 min", "Goal Mbappé 67e" | 🟢 Haut / 🟡 Moy | Le seul vrai habit driver pour Sports. Sans notifs, on consulte 1x/semaine maxi |
| 3 | **Player stats comparison** (Tennis) / **constructor evolution graph** (F1) | 🟡 Moy / 🟡 Moy | Profondeur data — fait du Sports un outil d'analyse, pas un simple calendar |
| 4 | **Personal predictions / poolbets entre amis** | 🟢 Haut / 🔴 Élevé | Différenciateur massif vs Sofascore. Pré-match : tu prédis le score → leaderboard avec amis. Multi-user × engagement × viral |
| 5 | **Historique perso + record en tant que fan** — "Tu as vu 23 matches du Real cette saison, win rate quand tu regardes : 67%" | 🟡 Moy / 🟡 Moy | Personnalise les sports — différenciateur HEGON-style (perso digital brain) vs Sofascore impersonnel |

---

## 6. TL;DR & next actions

### Ce qui est bon

- **Score réel : 7.5/10** (cible 7.8). Passé de 5.9 à 7.5 — +1.6 en une session. Excellent rendement.
- Sécurité atteint 8.5/10 (dépasse cible) — Profiles RLS, defense-in-depth middleware, owner-only invites, rate limit, CSP. Solide.
- Crashes atteint 8/10 (cible) — tous les invariants critiques corrigés.

### Ce qui ment dans le tracker

| Item | Tracker dit | Réalité |
|---|---|---|
| §4.1 eslint-disable file-wide | ✅ Fait | Partiel : OK pour `tasks/service.ts`, mais `tasks/types.ts:1` et `dashboard/service.ts:1` toujours désactivés |
| §3.2 RPC `recalc_goal_progress` | ✅ Fait | Diverge spec : `RETURNS void` + `SECURITY DEFINER` au lieu de `RETURNS INT` + `SECURITY INVOKER`. Pas de garde `IF mode != auto` dans SQL |
| §3.6 staleTime tasks (note Sonnet) | ✅ Patché audit | Code jamais touché — toujours à 3 min, devrait être 10 min |

### Régressions de cohérence

- `WorkspaceInvitePanel` jamais retrofité : 5× onMouseEnter inline + native `<input>` email + native `<input>` tag name
- `TaskCard` garde `style={{ color: var(--color-text-secondary)}}` non-dynamique sur la description
- `TasksTopbar` garde beaucoup d'inline `style={{ var(--color-*) }}` (lignes 100-105, 140-144, 165-168, 211-215, 245-249, 282-285, 326-329)

### Avant vidéo LinkedIn (mandatoire)

1. **Fix KanbanBoard skeleton (§2.1)** — ~1h, impact énorme
2. **Retrofit `WorkspaceInvitePanel`** — ~2-3h, démo collab × 2 en qualité perçue
3. **Mesurer bundle size** (§3.7 reporté)

### Avant vidéo LinkedIn (recommandé)

4. **Animation d'arrivée séquentielle Today cards + état "all done"** — ~2-3h, dashboard "fonctionnel" → "produit fini"
5. **Fix les 3 gaps tracker** : eslint-disable restants, RPC signature, staleTime tasks 10 min — ~30 min cumulés

### Roadmap modules — priorisation suggérée

**Phase 1 (impact max / effort min) :**
- Habits heatmap GitHub-style (~1 jour)
- Books year reading challenge (~0.5 jour)
- Journal daily prompts (~0.5 jour)
- Watching diary + ratings (~2 jours)

**Phase 2 (différenciateurs forts) :**
- Dashboard quick capture ⌘N (~2 jours)
- Goals weekly check-ins (~2 jours)
- Sports match notifications (~1-2 jours selon stack notif)

**Phase 3 (ambitieux / futur) :**
- Dashboard customizable widgets (~1 semaine)
- Sports live scores (dépendance API premium)
- Sports predictions/poolbets entre amis (~2 semaines, multi-user complexe)

---

*Re-review produit par Opus le 2026-05-19, après application complète des 9 sprints de l'audit du 2026-05-18.*
*À ré-auditer après application des 3 mandatory pré-LinkedIn et avant Phase 1 roadmap.*
