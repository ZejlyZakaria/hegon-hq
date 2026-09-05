HEGON — Roadmap Tasks
Plan 6 semaines | Début : Samedi 18 avril | Pas de travail le dimanche
================================================================


================================================================
PHASE 2 — Préparation avant Journal
================================================================

Titre       : Migration CSR — QueryProvider global + suppression TasksProvider
Description : Supprimer TasksProvider séparé. Mettre à jour QueryProvider global avec refetchOnWindowFocus: false, staleTime 5min, gcTime 10min. Convertir Sports (Football, Tennis, F1) et Dashboard de SSR vers client-side TanStack Query.
Tag         : Performance
Priorité    : Critical
Due date    : 18 avril

---

Titre       : Sécurité — CSP + remotePatterns
Description : Ajouter Content-Security-Policy header dans next.config.ts. Remplacer hostname "**" dans remotePatterns par la liste explicite des domaines connus (image.tmdb.org, *.supabase.co, crests.football-data.org, etc.).
Tag         : Security
Priorité    : Critical
Due date    : 20 avril

---

Titre       : Design system retrofit — Tasks
Description : Appliquer le design system complet sur le module Tasks. Vérifier cards, badges, modals, empty states, skeletons, couleur accent zinc. Corriger tout écart avec hq/design-system/guidelines.md. Écrire les tests service Tasks dans la même session. IMPORTANT : implémenter avec le design visuel des maquettes OS (hq/OS-vision/new-designs/os-tasks.png) — goal link sur les cards, list view Today Focus, module header. Le design OS s'applique au contenu des modules dès maintenant — seule la navigation (dock, sidebar) change en Phase 3.
Tag         : Frontend
Priorité    : High
Due date    : 21 avril

---

Titre       : Design system retrofit — Goals
Description : Appliquer le design system complet sur le module Goals. Vérifier Life Compass, GoalCard, GoalDetailPage, MilestoneList, modals. Corriger tout écart avec hq/design-system/guidelines.md. Écrire les tests service Goals dans la même session. IMPORTANT : implémenter avec le design visuel des maquettes OS (hq/OS-vision/new-designs/os-goals.png) — grille → liste dense comme Habits, accent #22c55e, GoalDetailPage "The Path". Voir hq/modules/goals/v1.md section "Session 22 avril" pour le détail complet.
Tag         : Frontend
Priorité    : High
Due date    : 22 avril

---

Titre       : Design system retrofit — Habits
Description : Appliquer le design system complet sur le module Habits. Vérifier HabitsPage, HabitCard, HabitModal, right panel, heatmap, calendar. Corriger tout écart avec hq/design-system/guidelines.md. Écrire les tests service Habits dans la même session. IMPORTANT : implémenter avec le design visuel des maquettes OS (hq/OS-vision/new-designs/os-habits.png) — perfectionner l'animation de completion (ring fill + opacity + streak pop), progression par unité (20m/20m, 2.5L/2.5L).
Tag         : Frontend
Priorité    : High
Due date    : 23 avril

---

Titre       : Design system retrofit — Dashboard
Description : Appliquer le design system complet sur le Dashboard. Vérifier Today section, cards sport, tasks section, week progress. Corriger tout écart avec hq/design-system/guidelines.md.
Tag         : Frontend
Priorité    : High
Due date    : 24 avril

---

Titre       : Auth — Email confirmation + SMTP Resend
Description : Activer "Confirm email" dans Supabase (Authentication → Providers → Email). Configurer Resend comme SMTP custom pour délivrabilité fiable. Tester réception email de confirmation sur un nouveau compte.
Tag         : Security
Priorité    : Critical
Due date    : 19 avril

---

Titre       : Auth — Email notification nouveau signup
Description : Edge Function Supabase (ou Resend) : à chaque nouveau signup, envoyer un email à zejly12@gmail.com avec nom + email du nouvel utilisateur. Déclenché via trigger DB sur auth.users ou via webhook Supabase Auth.
Tag         : Security
Priorité    : High
Due date    : 19 avril

---

Titre       : Setup infrastructure tests (Vitest + Playwright + MSW)
Description : Installer et configurer Vitest avec workspace (pool unit + pool service), Playwright, MSW. Créer src/tests/setup.ts, src/tests/fixtures/, src/tests/mocks/handlers.ts, e2e/auth.setup.ts. Configurer .env.test. Vérifier que npm run test passe.
Tag         : Testing
Priorité    : High
Due date    : 25 avril


================================================================
PHASE 2 — Journal V1
================================================================

Titre       : Journal — Migration SQL + types + service
Description : Créer supabase/migrations/YYYYMMDD_phase2_journal.sql — tables journal_entries avec org_id, mood, tags, RLS 4 policies, indexes. Créer src/modules/journal/types.ts et service.ts avec CRUD complet.
Tag         : Database
Priorité    : Critical
Due date    : 27 avril

---

Titre       : Journal — Hooks TanStack Query
Description : Créer query-keys.ts, useJournalEntries, useJournalEntry, useCreateEntry, useUpdateEntry, useDeleteEntry. staleTime 5min. onError toast sur chaque mutation.
Tag         : Backend
Priorité    : High
Due date    : 28 avril

---

Titre       : Journal — UI layout + liste + éditeur
Description : Layout 2 colonnes 35/65. Liste entrées à gauche (date, mood, aperçu). Éditeur à droite : fond crème #f5f0e8, police Caveat/Kalam, lignes discrètes. Sidebar/header restent #09090b strict. Accent orange #f97316.
Tag         : Frontend
Priorité    : High
Due date    : 29 avril

---

Titre       : Journal — Mood selector + tags + empty state
Description : Mood selector 5 cercles colorés (pas d'emojis). Système de tags sur les entrées. Empty state "Write your first entry". loading.tsx + Suspense sections indépendantes.
Tag         : Frontend
Priorité    : High
Due date    : 30 avril

---

Titre       : Journal — Tests + polish + route complète
Description : Tests service (isolation org, CRUD). Test E2E happy path (créer entrée, vérifier dans liste). Vérifier module-checklist.md entièrement coché avant de valider V1.
Tag         : Testing
Priorité    : Medium
Due date    : 1 mai


================================================================
PHASE 2 — Books V1
================================================================

Titre       : Books — Migration SQL + service + hooks
Description : Tables books avec org_id, status (want_to_read, reading, read), rating, notes, cover_url. RLS, indexes, types, service.ts, hooks TQ. staleTime 10min.
Tag         : Database
Priorité    : Critical
Due date    : 2 mai

---

Titre       : Books — UI liste + cards + statuts
Description : Layout liste avec filtres par statut. BookCard avec cover, titre, auteur, progress bar si en cours. Accent sky #0ea5e9. Empty state par statut.
Tag         : Frontend
Priorité    : High
Due date    : 4 mai

---

Titre       : Books — Detail page + notes + progress
Description : Page détail /books/[id] — cover large, notes éditables, progress manuel (pages lues / total), date début et fin. Rating avec étoiles.
Tag         : Frontend
Priorité    : High
Due date    : 5 mai

---

Titre       : Books — Tests + polish
Description : Tests service Books. Test E2E ajout livre + changement statut. Vérifier module-checklist.md entièrement coché.
Tag         : Testing
Priorité    : Medium
Due date    : 6 mai


================================================================
PHASE 2 — Tech V1
================================================================

Titre       : Tech — Migration SQL + service + hooks
Description : Tables tech_entries avec org_id, category, tags, content (markdown), source_url, is_verified. RLS, indexes, types, service.ts, hooks TQ. staleTime 10min.
Tag         : Database
Priorité    : Critical
Due date    : 7 mai

---

Titre       : Tech — UI knowledge vault + entrées + catégories
Description : Layout liste/grid d'entrées. Filtres par catégorie (Languages, Frameworks, Concepts, Tools, Architecture). Accent purple #a855f7. Search local.
Tag         : Frontend
Priorité    : High
Due date    : 8 mai

---

Titre       : Tech — Detail page + markdown + tags
Description : Page détail /tech/[id] — rendu markdown, tags, source URL, date. Badge "verified" si is_verified. Bouton edit inline.
Tag         : Frontend
Priorité    : High
Due date    : 9 mai

---

Titre       : Tech — Tests + polish
Description : Tests service Tech (isolation org, CRUD, search). Test E2E ajout entrée + vérification affichage. Vérifier module-checklist.md entièrement coché.
Tag         : Testing
Priorité    : Medium
Due date    : 11 mai


================================================================
PHASE 2 — Command Center ⌘K
================================================================

Titre       : ⌘K — Architecture + modal global + raccourci
Description : Créer src/modules/command-center/. Modal global accessible via Ctrl+K depuis n'importe quelle page. Zustand pour l'état ouvert/fermé. Animation Framer Motion. Fermeture Escape + clic extérieur.
Tag         : Frontend
Priorité    : Critical
Due date    : 12 mai

---

Titre       : ⌘K — Recherche tous modules
Description : Recherche unifiée dans Tasks, Goals, Habits, Journal, Books, Tech. Résultats groupés par module avec icône accent couleur. Debounce 200ms. Afficher top 3 par module.
Tag         : Backend
Priorité    : High
Due date    : 13 mai

---

Titre       : ⌘K — Actions rapides
Description : Actions rapides : "New Task", "New Goal", "New Habit", "New Journal Entry", naviguer vers un module. Raccourcis claviers affichés dans les items. Catégorie "Actions" séparée des résultats de recherche.
Tag         : Frontend
Priorité    : High
Due date    : 14 mai

---

Titre       : ⌘K — Tests + polish + intégration complète
Description : Tests E2E ouverture ⌘K + recherche + navigation. Vérifier accessibilité (focus trap, aria). Vérifier module-checklist.md. Phase 2 officiellement terminée.
Tag         : Testing
Priorité    : Medium
Due date    : 16 mai


================================================================
PHASE 3 — Real-time
================================================================

Titre       : Real-time — useRealtimeSync hook partagé
Description : Créer src/shared/hooks/useRealtimeSync.ts. Pattern : supabase.channel + on('postgres_changes') + invalidateQueries. Typage complet. Un seul hook réutilisable par tous les modules.
Tag         : Backend
Priorité    : Critical
Due date    : 18 mai

---

Titre       : Real-time — Kanban Tasks
Description : Brancher useRealtimeSync sur le kanban Tasks. Déplacement de carte visible en < 500ms dans un second onglet. Tester avec 2 onglets ouverts simultanément.
Tag         : Backend
Priorité    : High
Due date    : 19 mai

---

Titre       : Real-time — Dashboard Today
Description : Brancher useRealtimeSync sur Dashboard Today section. Mise à jour automatique si tâche complétée ou événement sport ajouté. Zéro refresh manuel requis.
Tag         : Backend
Priorité    : High
Due date    : 20 mai

---

Titre       : Real-time — Habits Today
Description : Brancher useRealtimeSync sur Habits Today. Complétion d'une habitude visible immédiatement sans refresh. Streak mis à jour en temps réel.
Tag         : Backend
Priorité    : High
Due date    : 21 mai

---

Titre       : Real-time — Presence avatars
Description : Avatars des utilisateurs connectés sur le même board Tasks. Supabase Presence channel. Affichage dans le topbar du kanban. Max 5 avatars visibles.
Tag         : Frontend
Priorité    : Medium
Due date    : 22 mai

---

Titre       : Real-time — Tests + validation critère Phase 3
Description : Critère de passage : 2 onglets ouverts, déplacer une carte dans l'un, voir le changement dans l'autre en < 500ms. Documenter le résultat dans progress.md.
Tag         : Testing
Priorité    : High
Due date    : 23 mai


================================================================
PHASE 4 — Automation Engine
================================================================

Titre       : Automation — Migration SQL (tables + RLS)
Description : Créer tables automation_rules (org_id, trigger_module, trigger_event, trigger_conditions jsonb, action_type, action_config jsonb, enabled) et automation_logs. RLS 4 policies. Indexes sur org_id, enabled, trigger_module.
Tag         : Database
Priorité    : Critical
Due date    : 25 mai

---

Titre       : Automation — Edge Function evaluate-automations
Description : Edge Function qui reçoit un webhook DB change, charge les règles actives pour l'org, évalue les conditions jsonb, exécute les actions (notify, create_task, update_field), insère dans automation_logs.
Tag         : Edge-functions
Priorité    : Critical
Due date    : 26 mai

---

Titre       : Automation — Webhooks + triggers Supabase
Description : Configurer les webhooks Supabase pour déclencher evaluate-automations sur les events tasks, goals, habits. Tester le flow complet : changement DB → webhook → Edge Function → action exécutée.
Tag         : Backend
Priorité    : High
Due date    : 27 mai

---

Titre       : Automation — UI /settings/automations — liste règles
Description : Page /settings/automations. Liste des règles avec nom, module trigger, action, statut enabled/disabled. Toggle on/off par règle. Empty state "No automations yet".
Tag         : Frontend
Priorité    : High
Due date    : 28 mai

---

Titre       : Automation — UI builder de règles
Description : Modal de création de règle. Sélecteur trigger_module + trigger_event. Conditions dynamiques selon le module. Sélecteur action_type + config. Prévisualisation de la règle en langage naturel.
Tag         : Frontend
Priorité    : High
Due date    : 29 mai

---

Titre       : Automation — Tests + première règle end-to-end
Description : Critère de passage : créer une règle via l'UI, déclencher la condition, vérifier que l'action s'exécute et est loggée dans automation_logs. Phase 4 officiellement terminée. Mettre à jour progress.md.
Tag         : Testing
Priorité    : Critical
Due date    : 30 mai


================================================================
FIN DU PLAN — 6 semaines | 18 avril → 30 mai
Total : 32 tâches
Phase 2 restante : 20 tâches
Phase 3 : 6 tâches
Phase 4 : 6 tâches
================================================================
