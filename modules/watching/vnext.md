# Watching — V.next

> Le brainstorm de la prochaine version. **Rien ici n'est décidé.** Ce fichier se vide quand une
> version sort : tranché → `decisions.md`, construit → invariant d'`architecture.md`, reporté →
> `backlog.md` avec son déclencheur.
>
> **Vidé le 2026-09-13** (réconciliation contre le code, audit phase 3). La spec V2 de mai — tout
> construit sauf trois lignes sans décision — est archivée avec son verdict feature par feature :
> [`hq/archive/watching-v2-spec-2026-05.md`](../../archive/watching-v2-spec-2026-05.md). L'idée
> Oscars est devenue la ligne « MUSÉE » du `backlog.md`. La prochaine version (« Watching v4 ») s'ouvre
> dans une conversation dédiée **après** la clôture de l'audit — pas ici.

## Idées

### Cartes « Universes » sur la main page → page univers (vu / reste à voir) — 2026-09-04, owner, non tranché

**Source : TMDB `discover` + `company`** — clé déjà en place, logos fournis (`logo_path`). Le
« vu / pas vu » est 100 % notre DB, zéro coût API. Faisabilité **testée en exécution** (repo
`public-apis`, [[reference_public_apis_repo_verdict]]).

✅ **Volumes réels :** Pixar (`with_companies=3`) **138** · Marvel Studios (420) **103** · Ghibli
(10342) **83** · A24 (41077) **172** · Walt Disney Pictures (2) **361**.

⛔ **NETFLIX CASSE L'IDÉE — et c'est structurel.** Netflix est un diffuseur, pas un studio : TMDB le
modélise en **network** (séries) et non en company. `with_networks=213` → **2 852 séries** (un mur) ;
`with_companies=178464` → **32 films** (crédités aux boîtes de prod). Idem HBO (376), Disney (361) =
catalogue, pas univers.

⭐ **La ligne de partage** : excellente sur un **univers d'auteur fini et complétable** (Pixar,
Ghibli, Marvel, A24) où « ce qu'il reste » est atteignable ; s'effondre sur un **catalogue de
diffuseur**. ⇒ Si on la fait, c'est **« Universes »**, pas « Studios », sur une liste **curée à la
main**.

⚠️ **Filtres obligatoires** : `discover` renvoie le **non-sorti** (Pixar : « Coco 2 » daté 2029) et les
**courts-métrages** gonflent les totaux. Borner à `primary_release_date.lte=today` + long-métrage.

**Ce qui la trancherait :** un « go » owner à l'ouverture de Watching v4 — elle passe alors en
`backlog.md` avec un déclencheur, ou en `decisions.md` comme refusée.
