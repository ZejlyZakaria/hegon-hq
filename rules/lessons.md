# HEGON — Leçons chèrement apprises (les pièges concrets, avec la preuve et le remède)

> Une ligne par piège. Chaque entrée a coûté une vraie panne, un faux diagnostic ou une soirée.
> Extrait le 2026-09-17 de la mémoire cross-session de Claude Code, pour que tout agent (Codex
> compris) les ait sous les yeux. Une règle ne pourrit pas ; ajoute, ne réécris pas.

## Watching — les invariants qui ne se rediscutent pas

- **UN SEUL écrivain de statut** : tout statut (`watched`, `in_progress`, `want_to_watch`, `paused`, `dropped`) passe par `lib/watch-status.ts` + `useWatchActions`. Le cron `series-sync` n'écrit la posture que sur preuve de croissance. Zod dans le service, pas seulement dans le hook.
- **Diffusé ≠ annoncé, l'état est un CALCUL** : `season_aired` (réel) décide, `season_episodes` (annonce) n'affiche que. `lib/series-state.ts`. « Caught up » = vu tout ce qui est diffusé, la série continue — un calcul, jamais un flag. `recently_watched` est dérivé de `watched_at`.
- **Un seul écrivain d'année de visionnage** : `season_years` / `cour_years`, édités par Watch History (multi) ou le picker StatusCard (single), bornés au dernier épisode diffusé. Toute surface qui date ou compte passe par `buildMediaView` (la « lentille »), sinon un anime à cours voit ses années écrasées. Test d'une correction : le total all-time ne bouge pas.
- **TMDB `episode_run_time` est mort** (vide pour presque toutes les séries) : la durée honnête = médiane des épisodes réels (`append_to_response=season/N`) ; `last_episode_to_air` = un final double (−920 h de surcompte, vécu).
- **Grammaire des marques sur une affiche** : couleur = SOURCE (teal = moi, or = le monde), cluster gauche = ce que le titre EST (ruban TOP 10, bookmark watchlist, cercle de statut, cœur), cluster droit = ACTIONS (menu, « + »). Un ruban pend d'un bord ; un statut est un cercle plein à glyphe noir. Bookmark : violet = pas encore sorti, blanc = sans priorité, rouge/ambre/gris = priorité.
- **Le verre seulement là où ça bouge derrière** : une affiche figée → plat (`Badge variant="flag"`). Glass = physique (blur + saturate + clamp), jamais de texte coloré dessus. Dashboard seulement.
- **Awards** : Oscars ← Wikidata (labels en `en,mul` — Nolan/RDJ sortaient en QID brut), Emmys ← emmys.com JSON-LD par catégorie (résolution TMDB : plus proche de la cérémonie avant la popularité, page Wikipedia exacte sans qualificatif). L'année inférée perd contre l'année déclarée. emmys.com groupe Writing/Directing par série → nos comptes ≤ réalité.
- **PostgREST plafonne à 1 000 lignes** : toujours paginer par `Range` (`fetchAllAwardRows`) — « remaining 0 » a menti une soirée entière.
- **Follow people** : un follow = la personne entière, liste plate (pas de « principal »). `person_upcoming` = cron mardi + sync à chaud après un follow ; les nouvelles saisons viennent de `tv/{id}.seasons` datées (pas de `next_episode_to_air`, exposé quelques jours avant seulement).

## UI / design — ce qui casse en silence

- **Lightning CSS supprime `backdrop-filter`** si tu écris `-webkit-backdrop-filter` à la main : jamais de préfixe manuel.
- **tailwind-merge détruit les tokens custom** (`text-caption` à côté d'une couleur dans `cn()` = supprimé) : redéclarer les échelles dans `extendTailwindMerge`.
- **Le Data Cache Next sert une entrée périmée UNE fois (SWR)** : donnée tierce vivante (épisodes TMDB) → `cache: "no-store"` serveur, cache client pour le quota. Header `x-tmdb-cache` pour le prouver.
- **Responsive = 4 devices** : mobile · iPad portrait (`md`) · iPad paysage (`lg`, tactile → `can-hover`) · desktop (`xl`). Le trou classique = iPad portrait traité comme mobile. Le skeleton d'une route doit refléter la surface **au pixel** (mêmes colonnes, mêmes breakpoints).
- **Pas de top accent lines décoratives** : la profondeur vient de la surface (`surface-card`) et de l'échelle. Palette Graphite (Apple systemGray), matériau sans gradient.
- **Radix** : jamais de Popover/Portal scrollable dans un Dialog (→ dropdown inline) ; jamais un Select comme picker inline (bug aria-hidden → Popover).
- **Les ancres ne s'imbriquent pas** : une rangée à plusieurs portes = ancre du titre étirée par pseudo-élément, liens du meta en `relative z-10` (`MediaRow titleHref`). Une tuile qui contient un contrôle est un `div role="button"`, pas un `<button>`.
- **Tooltips** : `Hint` (shadcn, `shared/ui/tooltip`) partout où on écrirait `title=`.
- **Le chrome ne clignote jamais** : ce que le serveur sait (modules cachés, statut démo), le layout serveur le passe en props — le Dock attendait deux requêtes client et apparaissait nu à chaque reload.
- **Une section n'attend pas sa donnée la plus lente** : Openings & Endings attendait la plus lente de N pochettes iTunes ; la liste d'abord, l'art en seconde requête, un glyphe neutre en attendant (jamais l'affiche → rien ne saute).

## CI, robots, infra — les cliquets et leurs pièges

- **La CI ne voit que le versionné** → simuler par `git clone . /c/ci-sim`. Toute sortie comparée par un gate doit être déterministe.
- **« Le job a tourné » ≠ « la donnée a changé »** : une synchro idempotente ne se surveille pas par la fraîcheur de sa table (fausse alerte anime-cours, 3 runs rouges). Section C du watchdog = seulement les jobs qui écrivent à chaque passage. Un robot qui répond 200 avec une liste d'échecs meurt en silence → **500 quand tout a échoué**.
- **`net.http_post` est asynchrone, timeout 5 s par défaut** : un cron `succeeded` ne prouve que le SQL. La vérité = `net._http_response` (gardé ~6 h) via `internal.call_log` / `job_health`.
- **Jamais deux robots la même minute** (12 crons à :00 = 504 Supabase) ; `stagger_crons`. Un cron ne contient **jamais** de clé : `internal.call_edge('<fn>', payload)`.
- **Une migration ⇒ `expected-objects.json`** (`_migrations_through` + compteurs) dans le même commit, sinon le watchdog crie DÉRIVE alors que le schéma est juste. Les colonnes d'une **vue** comptent dans `colonnes`, pas dans `tables`. `npx supabase db push --linked`, jamais le SQL Editor.
- **`ds:coverage` avant tout commit qui touche un composant** (`coverage.generated.json` est un cliquet).
- **Pre-commit : un WARNING bloque** (`eslint --max-warnings 0`). La règle N+1 exige un commentaire `eslint-disable` sur une pagination légitime.
- **Fichiers en CRLF** (`src/**`, les edge functions) : éditer par script `.js` qui détecte l'EOL, jamais par heredoc ni `node -e` (un `\b` devient un backspace, `$&` disparaît — vécu deux fois le 17/09). `sed -i` convertit un fichier entier en LF.
- **Vérifier par sha256**, jamais par « Finished » dans une UI (une clé collée devant l'ancienne, un secret dont le digest ≠ la clé — 3 mensonges en une soirée). `SUPABASE_SERVICE_ROLE_KEY` = clé vivante dans Vercel/`.env.local`, clé MORTE auto-injectée dans une edge function (lire `HEGON_SECRET_KEY` là-bas).
- **Healthchecks.io** : period/grace ne se règlent pas par l'URL de ping (auto-créé = 1 j / 1 h → faux « down » chaque lundi) ; régler à la main.
- **`/usr/bin/pg_dump` du runner route vers PG 16** même après l'install du 17 : exiger la version par `grep || exit 1`.
- **DNS hegon.fr = Cloudflare devant Vercel** (trou de routage Orange). « Inaccessible » selon le réseau = DNS/FAI (forwarder 4G iPhone), jamais le code.
- **Supabase jsonb** : `.contains()` cassé → `.filter(col, "cs", json)` ; `@>` seulement sur `jsonb`.
- **LISTER les requêtes avant de les compter** (`scripts/list-requests.mjs` puis `measure-cache.mjs`) ; 4ᵉ forme du N+1 = un hook par carte (N+1 de montage) → vue batch.
- **Perf = A/B chiffré**, avant/après, dos à dos, échauffés — jamais « c'est plus rapide ». Une estimation de gain se mesure avant d'entrer au backlog (« ÷4 » annoncé, −28 % mesuré).
- **Sources data** : football gratuit = scores seuls (compos/xG/événements = 0 source gratuite) ; `public-apis` testé = un seul gain (OpenF1). Awards personne = Wikidata.

## Collaboration avec l'owner (ne se périment pas)

- **Une question (« que dis-tu ? ») = discuter**, attendre un « go » explicite. Ne jamais proposer d'arrêter — proposer la prochaine étape de TRAVAIL (a vs b).
- **Vérifier en EXÉCUTANT** : `tsc`/`eslint` verts ne prouvent pas l'app ; captures Playwright lues, DB interrogée, robot lancé à la main. Dire ce qui n'a pas pu être vérifié.
- **C'est l'agent qui trouve les bugs** (dead code, incohérences) sans qu'on le demande ; ne demander que les vrais arbitrages produit/design.
- **Git** : donner la commande `git add … && git commit -m "…"`, ne jamais l'exécuter ; commit = une ligne, le quoi et le pourquoi, **jamais** de Co-Authored-By ni de mention d'IA. Un commit par bloc validé.
- **Migrations et déploiements = l'owner** (`db push`, `functions deploy`) ; l'agent donne la commande et vérifie après.
- **UI en anglais** dès le premier jet (dates `en-GB`) ; commentaires de code libres.
- **Challenger toute décision** contre Linear / Apple / Vercel ; jamais « Apple » en public (→ « next level »).
- **Condition 3 du gel** : ne toucher aucun module non audité, même pour une ligne.
- **Mode ami** = détendu, sans tableaux. Suggérer claude.ai / ChatGPT si la tâche n'a pas besoin d'un agent de code.


- **Une note lisible n’est pas une preuve de périmètre** : appliquer le garde d’écriture/`CURRENT` à l’affichage a masqué toutes les notes legacy (Watching,2026-10-28). Séparer identité, lecture de l’opinion sauvegardée et validation des épisodes ; UNKNOWN ne prouve pas GROWTH. Tester une vraie note legacy sans catalogue accepté, en plus des fixtures parfaitement renseignées.

- **Viewing history is not the assertion journal** (2026-10-30): exposing raw enums, film Season0, rating-triggered completion milestones and tombstones made real viewings unreadable. Keep append-only audit internal, project a tested consumption timeline, and verify real screenshots plus explicit prospective date semantics.

- Partial carousel hover QA: Playwright locator.hover() may auto-scroll the target fully into view, hiding clipping bugs. Hover physical coordinates inside the visible fragment and assert scrollLeft stays fixed; check image AND caption paint beyond the viewport boundary. Reproduced/fixed Best Episodes2026-11-02 (partial-episode-hover-2026-11-02.md).


- Watching2026-11-04: caught_up_at can be created by historical corrections and exist on ended titles; never use it as consumption recency. Silo legitimately uses last_watched_at, Hill House must retain watched_at. One latestViewingAt helper owns order and badge; release availability is not resumption, and held titles still belong to consumed-library summaries.

- Watching2026-11-05: last_watched_at can itself be a legacy claim stamp (Promised Neverland2026 vs completion2022). Excluding caught_up_at alone was insufficient: completed recency uses its completion projection; ongoing recency keeps actual forward activity. Exact old user action is not recoverable without audit evidence.
