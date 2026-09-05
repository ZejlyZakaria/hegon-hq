# Rotation de la clé `service_role` fuitée — plan d'exécution

> ## 📍 ÉTAT AU 16/07 ~22h30 — **PHASES 0 → 3 FAITES ET VÉRIFIÉES. RESTE : SOAK + PHASE 4.**
> - **Ph.0 ✅** dump `cron.job` (bureau owner) · **19 functions commitées** (`2242c50`) · fix lint-staged
>   (`--no-warn-ignored` : ESLint hurlait sur les fichiers Deno qu'il est configuré pour ignorer).
> - **Ph.1 ✅** `internal.call_edge` (publishable, timeout 300 s) · **14 crons reconstruits**, noms
>   `domaine-sujet-cadence` · `job_run_details` purgé (**3110 lignes contenaient la clé**) · **webhook
>   signup migré** sur le helper (+ ne transmet plus `encrypted_password`, + garde anti-blocage du
>   signup). **`select count(*) from cron.job where command like '%eyJ%'` → 0.** Mail de test reçu.
> - **Ph.2 ✅** `.env.local` + **Vercel** (redéployé) + **GitHub Actions**. Publishable déjà en place
>   côté anon. **Preuve : Zverev ajouté en prod** (route tennis = seul chemin privilégié de l'app).
> - **Ph.3 ✅** 18 functions → `HEGON_SECRET_KEY` (notify-new-signup exclue, elle ne lit que Resend) ·
>   check `CRON_SECRET` mort supprimé · **18/18 déployées** · **4 invocations réelles vertes** (football
>   Bournemouth, tennis 50 joueurs, F1 22 pilotes, anime nothing stale).
> - **➡️ RESTE : soak (1 jour de `net._http_response` sans 500) puis « Disable JWT-based API keys ».**
>
> ### 🔑 LA LEÇON DE LA SOIRÉE — « posé » ≠ « fonctionne »
> **3 fois** une clé a été « enregistrée » sans l'être : `.env.local` (nouvelle clé **collée DEVANT
> l'ancienne**, 224 car. au lieu de 41), puis le **secret Supabase** (empreinte ≠ clé réelle). À chaque
> fois l'UI disait *Finished* / *Updated*, tout semblait vert, et **seule une exécution réelle a dit la
> vérité**. → Vérifier une clé par son **empreinte sha256** (`supabase secrets list` publie un sha256 du
> secret : comparable en local **sans jamais exposer la valeur** — méthode validée sur `TMDB_API_KEY`) et
> par une **invocation qui écrit**. Jamais par le message de l'interface.

> Opus · 2026-07-16 · **✅ Revu par Fable (sécurité + archi DB). Verdict : GO, avec amendements — tous intégrés.**
> Objectif owner : *« le faire une fois pour toutes et ne jamais y revenir »* — robuste, simple, ennuyeux.
> Contrainte : la **démo LinkedIn** ne doit pas tomber.

---

## 1. Le problème

La clé **legacy `service_role`** (JWT, bypass RLS total) a **fuité 2×**. **La remplacer ne suffit pas** —
tant qu'elle existe, elle marche. Il faut qu'elle **cesse de fonctionner**. C'est le seul but ici, et
c'est ce qui rend inutile la chasse aux copies introuvables : **tuer la clé à la source les rend toutes
inertes.**

## 2. L'état réel (vérifié, pas supposé)

| Fait | Vérification |
|---|---|
| **14 crons**, **20 copies de la clé EN DUR** dans `cron.job.command` | `select … from cron.job` |
| **13/14 crons sans `timeout_milliseconds`** → aveugles (pg_net coupe à 5 s) | `net._http_response` |
| Les 16 edge functions lisent `Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")` (auto-injecté) | grep |
| **~12 fonctions sport sans source locale** | seules 4 en local |
| **🔴 `src/app/api/tennis/add-player/route.ts` + `get-api-ids/route.ts`** lisent `process.env.SUPABASE_SERVICE_ROLE_KEY` → **la clé est dans Vercel** | *(trouvé par Fable — j'avais raté)* |
| **🔴 `.github/workflows/ci.yml:37`** → la clé est dans les **secrets GitHub Actions** | *(trouvé par Fable)* |
| 8 scripts `scripts/*.mjs` lisent la même variable depuis `.env.local` | grep |
| **Clé jamais passée par git** | `git log --all` → vide |
| **✅ `sb_secret_` fonctionne en Bearer sur les edge functions** | test réel → **200** |
| 🔴 Bug F1 : `sync-f1-constructor-standings` → le vrai nom est `f1_sync_constructor_standings` | confirmé owner |

## 2 bis. 🗺️ CARTE EXHAUSTIVE DES CONSOMMATEURS (vérifiée par grep repo-entier, 2026-07-16)

> **La leçon** : j'avais cherché la clé **là où je m'attendais** (`supabase/functions/`, `.env.local`,
> crons) et raté Vercel. Fable a grepé **tout le repo**. Grep exhaustif refait depuis → **la liste de
> Fable était complète, rien de nouveau.** Ne JAMAIS chercher un secret avec une hypothèse.

| Où | Combien | Action | Statut |
|---|---|---|---|
| `.env.local:3` | 1 | échanger la **valeur** | ph.2 |
| `.github/workflows/ci.yml:37` | 1 | échanger la valeur du secret GitHub | ph.2 |
| **Vercel** : `src/app/api/tennis/add-player/route.ts:40` + `get-api-ids/route.ts:40` | 2 | échanger la **valeur** de `SUPABASE_SERVICE_ROLE_KEY` dans Vercel (**garder le NOM**) | ph.2 |
| **Scripts** : `backfill-cast`, `backfill-people`, `backfill-season-episodes`, `fix-release-dates-primary`, `fix-watched-at`, `repair-watched-at`, `resolve-anime-cours`, `sync-series` | **8** | ✅ **RIEN** — lisent `.env.local`, couverts par l'échange de valeur | — |
| **Edge functions locales** : `for-you-refresh:235`, `watching-trending-refresh:42`, `watching-series-sync:226`, `watching-anime-cours-sync:86` | 4 | `SUPABASE_SERVICE_ROLE_KEY` → `HEGON_SECRET_KEY` | ph.3 |
| **Edge functions restantes** | **15** | ✅ **RAPATRIÉES en local le 16/07** (`functions download`) — reste à les grep | **ph.0** |

### ✅ Les 19 fonctions (pas 16 !) — toutes en local avec leur `index.ts`, vérifié
`football_sync_competitions` · `football_sync_next_matches` · `football_sync_past_matches` ·
`football_sync_standings` · `football_sync_teams` · `f1_enrich_last_winners` · `f1_seed_drivers` ·
`f1_seed_initial` · `f1_sync_constructor_standings` · `f1_sync_driver_standings` · `f1_sync_race_results` ·
`tennis_sync_matches` · `tennis_sync_played_matches` · `tennis_sync_ranking` · **`notify-new-signup`** ·
`watching-trending-refresh` · `for-you-refresh` · `watching-series-sync` · `watching-anime-cours-sync`

**⚠️ 5 fonctions ne sont appelées par AUCUN cron** : `f1_enrich_last_winners`, `f1_seed_drivers`,
`f1_seed_initial`, `tennis_sync_played_matches` (= des **seeds lancés à la main**, dixit l'owner ; contenu
à discuter plus tard) et surtout **`notify-new-signup`** — déclenchée par un **webhook d'auth**, donc
**invisible dans la carte des crons**. Elle serait tombée en phase 4 sans prévenir. → **la phase 3 doit
couvrir les 19, pas seulement celles des crons.**

| `20260412_phase1_rls_policies.sql` (`service_role_*`) | — | ❌ faux positifs : ce sont des **noms de policies**, pas des usages | — |

### ✅ GREP DES 19 FAIT (2026-07-16) — plus aucun inconnu côté functions
- **18/19 lisent `SUPABASE_SERVICE_ROLE_KEY`**, toutes via le même
  `createClient(SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY)` → **phase 3 mécanique, aucune exception**
  (6 F1 · 5 football · 3 tennis · 4 watching).
- **`notify-new-signup` ne lit QUE `RESEND_API_KEY`** → hors phase 3. (Son `Bearer` = celui de Resend.)
- Autres secrets rencontrés, **à ne pas toucher** : `TMDB_API_KEY`, `FOOTBALL_DATA_KEY`, `RESEND_API_KEY`.
- 🟡 **`for-you-refresh:225` porte encore le check mort `CRON_SECRET`** (`if (secret && header !== secret)
  → 401`) — le piège exact qui a donné un 401 `text/plain` en juin. Inoffensif tant que la variable n'est
  pas définie, mais **armé**. → **le supprimer en phase 3** + `supabase secrets unset CRON_SECRET`.

### 🔴 LA 21ᵉ COPIE — trouvée le 16/07, invisible partout ailleurs
`notify-new-signup` n'est pas appelée par un cron : c'est un **Database Webhook**, donc un **trigger sur
`auth.users`** dont l'en-tête `Authorization: Bearer eyJhbGciOiJ…` (**JWT legacy**) est **stocké dans la
définition du trigger** — ni dans `cron.job`, ni dans le repo, ni dans Vercel. **Elle serait morte en
silence en phase 4** (plus aucun mail de signup, zéro erreur). Elle porte aussi le **timeout `5000`** =
le même aveuglement pg_net que `watching-series-sync`.

```
CREATE TRIGGER "notify-new-signup" AFTER INSERT ON auth.users FOR EACH ROW
EXECUTE FUNCTION supabase_functions.http_request('…/notify-new-signup','POST',
  '{"Content-type":"application/json","Authorization":"Bearer eyJhbGciOiJ…"}','{}','5000')
```
*(`on_auth_user_created` → `handle_new_user()` = SQL pur, aucune clé. Rien à faire.)*

➡️ **Phase 1 bis** : remplacer ce webhook par un trigger qui passe par **`internal.call_edge`** (clé dans
le Vault, zéro copie). **Ne PAS y recoller la nouvelle clé** — ce serait refaire la faute exacte qu'on
répare, à l'endroit le moins auditable de la base.
> **⚠️ RÈGLE DURE** : un trigger sur `auth.users` qui lève **fait échouer l'INSERT = le signup**. Le
> webhook actuel ne bloque pas *par accident* (pg_net est async). Le nôtre doit ne pas bloquer **par
> écrit** : `exception when others then return new`. Un mail de notif raté ne doit JAMAIS empêcher une
> inscription.

**Leçon (la même que Vercel, 2ᵉ récidive)** : j'ai cartographié les consommateurs **par le mécanisme que
j'imaginais** (les crons) → raté le trigger. Un secret ne se cherche pas par hypothèse : il se cherche
**partout où Postgres peut stocker du texte** (`cron.job`, `pg_trigger`, `pg_proc`, Vault, env).

## 3. La question « ouverte » — RÉSOLUE par reformulation (Fable)

> *« `SUPABASE_SERVICE_ROLE_KEY` reste-t-il injecté après la désactivation du legacy ? »*

**Mauvaise question.** Peu importe que la *variable* survive : **sa valeur EST le JWT legacy**, et la
phase 4 a précisément pour but de rendre cette valeur invalide. Donc **dans tous les cas**, les 16
fonctions cassent en phase 4 si elles n'ont pas migré avant.

➡️ **La phase 3 n'est pas une couverture : elle est OBLIGATOIRE.** Et c'est déjà le minimum possible —
on ne peut pas patcher la variable en place, car **`supabase secrets set` refuse le préfixe réservé
`SUPABASE_`**. Donc : renommer la variable dans les 16 fonctions + redéployer. Point.

## 4. Le plan

### Phase 0 — Filet de sécurité (RIEN avant ça)
1. **Dumper les crons dans un fichier local** : `select jobid, jobname, schedule, command from cron.job`
   → fichier hors git. **Sans ça, le rollback de la phase 1 n'existe pas.** (Il contient la clé fuitée :
   sans importance, elle meurt en phase 4.)
2. **Rapatrier les ~12 fonctions sport** (`npx supabase functions download <nom>`) **ET LES COMMITER** :
   dé-ignorer `supabase/functions/**` dans `.gitignore`. **Télécharger sans commiter n'est pas une
   sauvegarde** — `supabase/` est gitignoré, les 16 sont à un `rm -rf` de disparaître.
   > **⚠️ STRUCTURE PLATE OBLIGATOIRE** : `supabase/functions/<nom-exact>/index.ts`. **Ne PAS imbriquer**
   > par sport (`f1/f1_sync_.../`) → Supabase lirait une fonction nommée `f1` et `functions deploy`
   > casserait ⇒ le backup ne serait **plus restaurable** (= le but de la phase 0 perdu). Les noms
   > portent déjà le préfixe (`f1_*`, `football_*`, `tennis_*`) → le tri alphabétique fait le rangement.
   > **Le préfixe range, pas le dossier** (même logique que les noms de crons).
3. **⚠️ Il existe des fonctions HORS crons** (ex. `f1_enrich_last_winners`, repérée par l'owner, absente
   des 14 crons) → probablement appelées par l'app. **Rapatrier TOUT ce que liste le dashboard**, pas
   seulement ce que les crons référencent.
3. Noter la valeur actuelle de `.env.local` (rollback).

### Phase 1 — Une seule porte d'appel (helper), zéro secret dedans
1. ~~Créer une nouvelle `sb_secret_`~~ **inutile ici** (cf. §7) — la plomberie prend la **publishable**.
2. ~~`vault.create_secret`~~ **RETIRÉ** — voir §7.
3. **Le helper** (schéma privé, `security invoker`, nom validé, `search_path` vide, timeout **300 000**) :

```sql
create schema if not exists internal;   -- non exposé à PostgREST, aucun grant par défaut

create or replace function internal.call_edge(fn text, payload jsonb default '{}'::jsonb)
returns bigint
language plpgsql
-- security INVOKER (défaut) : le cron tourne en `postgres`, qui lit déjà pg_net.
-- Pas de definer = aucun chemin d'escalade, rien à verrouiller.
set search_path = ''
as $$
declare
  -- Clé PUBLIQUE (celle du navigateur) : simple laissez-passer de gateway, cf. §7.
  -- Rien à protéger ici — c'est tout l'intérêt.
  gateway_key constant text := 'sb_publishable_…';
  req_id bigint;
begin
  if fn !~ '^[a-z0-9_-]+$' then
    raise exception 'call_edge: invalid function name %', fn;   -- tue l'injection d'URL
  end if;

  select net.http_post(
    url := 'https://femvhonlpafdajyamvcu.supabase.co/functions/v1/' || fn,
    headers := jsonb_build_object(
      'Content-Type', 'application/json',
      'Authorization', 'Bearer ' || gateway_key
    ),
    body := payload,
    timeout_milliseconds := 300000   -- au-dessus du wall-clock des functions, sinon on re-crée l'aveuglement
  ) into req_id;

  return req_id;
end;
$$;

revoke all on function internal.call_edge(text, jsonb) from public;
```

> **⚠️ Pourquoi PAS `public` + `security definer` (mon erreur initiale) :** une fonction `security definer`
> dans `public` est exposée par PostgREST et exécutable par `anon` → n'importe qui pouvait l'appeler
> avec un `fn` forgé et **exfiltrer le header Bearer** → la nouvelle clé offerte au monde.

4. **Reconstruire les 14 crons** : `cron.unschedule` + `cron.schedule`, **noms préfixés**
   (`football-*`, `tennis-*`, `f1-*`, `watching-*`), appel `internal.call_edge('…')`, **nom F1 corrigé**.
5. **Purger l'historique** : `delete from cron.job_run_details;` — pg_cron y garde le `command` **en
   clair, 20× la clé**, et Supabase ne purge pas.
6. **Supprimer les snippets SQL sauvegardés** dans l'éditeur du dashboard (ils contiennent la clé).
7. **Vérifier** : `net._http_response` → **200** partout.

> ✅ **`net._http_response` / `http_request_queue` : RIEN à faire.** Vérifié : ils stockent les
> *réponses*, pas les headers de requête → la clé n'y est pas. (Noté ici pour que personne ne « nettoie »
> ça plus tard par réflexe.)

### Phase 2 — Migrer l'app + tous les consommateurs hors legacy
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` → **`sb_publishable_`** (`.env.local` **et Vercel**).
- **🔴 Vercel : `SUPABASE_SERVICE_ROLE_KEY` → la nouvelle `sb_secret_`.** *(Garder le NOM de la variable,
  changer la VALEUR — supabase-js accepte la clé secrète en remplacement direct. Ne PAS renommer la
  variable dans les 11 fichiers.)* Sinon **les 2 routes tennis cassent en prod.**
- **🔴 GitHub Actions** : même échange de valeur sur le secret du repo.
- `.env.local` : même échange de valeur (couvre les 8 scripts, zéro changement de code).
- **Smoke test** : login owner + **login démo** + une page démo + **les 2 routes tennis**.

### Phase 3 — Migrer les edge functions (OBLIGATOIRE, cf. §3)
- `supabase secrets set HEGON_SECRET_KEY=<sb_secret_…>`
- `Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")` → `Deno.env.get("HEGON_SECRET_KEY")` dans **les 16**, redéployer.
- **Vérifier chacune** (invoke + `net._http_response`).

### Phase 4 — Tuer la fuite
- **Soak d'abord** : chaque cron des 14 doit avoir tiré **au moins un cycle avec un 200 enregistré**
  (les quotidiens : attendre ou déclencher à la main), + les 2 routes tennis, + un run CI.
- Puis **« Disable JWT-based API keys »** → la clé fuitée devient **inerte**. But atteint.
- **⚠️ NE PAS SUPPRIMER les clés legacy** — *disable* seulement, et pendant **une semaine de vert**.
  Disable est réversible ; delete ne l'est pas.
- **Smoke test** identique à la phase 2.

### Phase 5 — Nettoyage
- `.env.local` propre · mémoire + CLAUDE.md (traçabilité).

## 5. Rollback (réel à chaque étape)
- **Ph.1** : recréer les crons **depuis le dump de la phase 0**.
- **Ph.2-3** : re-basculer les valeurs d'env sur les clés legacy — **elles marchent encore**, c'est
  exactement pourquoi Disable est en phase 4 et pas avant.
- **Ph.4** : ré-activable tant qu'on n'a pas supprimé.
- **Règle d'or** : ne jamais désactiver le legacy avant que 1→3 soient vertes et vérifiées.

## 6. Rayon d'explosion de la démo (précisé par Fable)
La démo ne touche que le chemin **anon/publishable + RLS**. Donc elle ne peut casser qu'en **phase 2**
(mauvaise clé publishable → login démo mort ; rollback = 1 variable, immédiat) ou en **phase 4** (si la
phase 2 était incomplète). **Un cron cassé dégrade la fraîcheur des données, jamais la démo.**

## 7. ✅ LE TEST A RÉPONDU (2026-07-16) — `sb_publishable_` → **200**

`curl` sur `watching-anime-cours-sync` avec la **clé publishable** → **la fonction s'exécute et répond**
(`x-deno-execution-id` + `{"ok":true,…}`). Donc, confirmé par l'exécution et non par la doc :

> **Le Bearer d'un cron n'est qu'un laissez-passer de gateway.** La fonction ne s'en sert pas — elle
> fabrique son propre client privilégié depuis son env. **Les 20 copies de la clé fuitée servaient à
> ouvrir une porte qui accepte déjà n'importe quelle clé valide, publique comprise.** (C'est le chemin
> documenté : le navigateur invoque les functions avec la clé publique.)

### Ce que ça change — la plomberie ne transporte PLUS AUCUN secret
- Le helper porte la **publishable**. → plus de secret dans `cron.job`, ni dans `job_run_details`, ni
  dans le trigger du webhook. **Pas « mieux caché » : ABSENT.** On ne fuite pas ce qui n'est pas là.
- **⛔ VAULT RETIRÉ DU PLAN** (contre le plan initial + Fable). Le Vault existait pour protéger un
  secret ; il n'y a plus de secret à cet endroit. **Un coffre-fort pour une clé publique = théâtre**, et
  une dépendance de panne en plus (`vault.decrypted_secrets`, `select … into strict` qui peut lever
  dans un trigger d'`auth.users`). Le helper EST déjà le point unique de changement.
- La clé privilégiée survit **uniquement là où le travail privilégié se fait** : env des 18 functions
  (ph.3), Vercel, `.env.local`. **Jamais dans ce qui les déclenche.**

### 🟡 Le revers — à traiter APRÈS, chantier séparé
Si une clé publique ouvre la porte, **les fonctions sont déjà invocables par n'importe qui**. Aucune
donnée ne fuit (elles ne lisent aucun paramètre entrant), mais c'est un **robinet à coût** (quota TMDB,
temps d'exécution). **Ce n'est pas une régression de cette rotation** — c'est vrai depuis toujours, la
service_role dans les crons n'y changeait rien. Ne pas mélanger deux problèmes.

## 8. ⛔ Sur-ingénierie — à NE PAS faire (Fable + moi d'accord)
Renommer `SUPABASE_SERVICE_ROLE_KEY` dans les 11 fichiers (échanger la **valeur** suffit) · scrubber les
logs/backups/transcripts (**inutile : la clé sera morte**) · clés par fonction · automatisation de
rotation / expiry · refondre les 2 routes tennis · toucher aux fonctions elles-mêmes.
**L'ennui de ce plan est sa qualité principale.**
