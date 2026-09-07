# HEGON — Stratégie produit & business

> Ouvert le **2026-09-07**, conversation owner ↔ Opus. **Ce n'est PAS un plan de développement.**
> ⛔ Rien ici ne se construit maintenant — les features sont gelées par le chantier d'audit
> (`hq/audit/plan.md`). Ce fichier existe pour que le jour venu, on n'invente pas sous pression.
>
> Statut : **exploration**. Ce qui est tranché est marqué ✅, ce qui reste ouvert ❓.

---

## 0. ✅✅ LA DÉCISION (2026-09-07) — à lire avant tout le reste

**HEGON reste l'app personnelle de l'owner. Architecture multi-tenant conservée. Le SaaS reste une
option ouverte, jamais un objectif.**

Options écartées explicitement :
- ⛔ **« Multi-utilisateurs entre amis » comme CAP** — ce n'est pas une stratégie, c'est l'état actuel.
  Comme objectif c'est le pire rapport : toute la taxe du multi-utilisateur (RLS, support, données
  d'autrui, migrations qui n'ont plus le droit de casser) sans aucune contrepartie. Très bien comme
  sous-produit, mauvais comme direction.
- ⛔ **Le SaaS comme objectif** — voir §2bis (le terrain est occupé) et l'analyse investisseur.

⭐ **LA RÈGLE DE DÉCISION**, à appliquer chaque fois qu'un choix se présente :

> **« Est-ce que je construirais ça si j'étais le seul utilisateur pour toujours ? »**
> **Non** → ne le construis pas, même si « ça servira un jour ».
> **Oui** → construis-le, et mets `org_id` sur la table.

**Garder l'architecture multi-tenant ne coûte rien. Construire des fonctionnalités multi-tenant coûte
cher.** Toute la différence est là — et c'est exactement ce qui a tué le chemin d'invitation org.
Donc : `org_id` partout ✅, discipline du `service.ts` ✅, exports ✅. Rôles, permissions,
administration d'org, invitations au-delà de Tasks ❌ tant que personne ne les utilise.

⭐ **Ce qui libère de la décision** : le chantier d'audit (sauvegardes, reconstructibilité, tests,
cliquets, doc) sert **les quatre chemins à la fois** — app perso, ouverture aux amis, SaaS, ou actif de
carrière. Il n'y a donc rien à trancher aujourd'hui ; il faut seulement **éviter de payer d'avance pour
un futur incertain**.

⭐ **La quatrième issue, non listée par l'owner et probablement la meilleure en espérance** : HEGON
comme **actif de carrière**. Un projet clonable, exécutable, avec ses décisions d'architecture écrites,
des tests, des cliquets CI et un audit documenté — ça se raconte en entretien. **Coût marginal nul :
c'est exactement le travail en cours.**

---

## 1. ✅ Le recadrage : HEGON est une COLLECTION, pas une app de productivité

L'origine, dans les mots de l'owner : la même obsession que ses cartes Yu-Gi-Oh, les figurines de
chaque anime vu, le maillot de son joueur, la raquette de son joueur de tennis. **Garder ce qui a
compté, sous une forme qu'on peut montrer.** Et l'intention finale : *« ouvrir l'app dans 20 ans et
montrer à mes enfants ce que leur père a vécu »*. **C'est 100 % émotionnel, et c'est le vrai moteur.**

⚠️ `blueprint.md` §2 décrit encore HEGON comme *« la gestion de la vie »* — cadrage **productivité**.
Ce n'est pas ce que HEGON est.

**Pourquoi c'est une bonne nouvelle commercialement.** Les deux marchés n'ont rien à voir :

| | Productivité | Collection |
|---|---|---|
| On se bat sur | l'efficacité | l'attachement |
| Abandon | massif (on décroche dès que la vie s'accélère) | faible (quitter = perdre sa collection) |
| Exemples | Notion, Todoist, Sunsama | Letterboxd, Goodreads, Discogs, Strava, Untappd |
| Les gens paient ? | difficilement | **oui** (Letterboxd Pro, Discogs, Goodreads racheté) |

**L'instinct de l'owner pointe vers le meilleur des deux marchés.**

## 2. ❓ La tension non résolue : deux moitiés opposées

Watching · Books · Sport = **collection**. Tasks · Goals · Habits = **productivité**.

Les deux logiques sont émotionnellement inverses : une collection **accumule** (rien ne s'efface, le
passé est le sujet) ; la productivité **vide** (une tâche finie disparaît, le sujet est le futur).
Elles attirent aussi des publics différents.

⚠️ **Si HEGON devient un produit payant, il faudra probablement choisir laquelle on vend.**
Note : la séparation existe déjà dans la nav (perso / pro / life) — elle n'est pas qu'un rangement,
c'est peut-être la ligne de fracture commerciale. **Non tranché.**

## 2bis. ⚠️ LE TERRAIN EST DÉJÀ OCCUPÉ — recherche du 2026-09-07

**La moitié « médias » de HEGON n'est pas neuve.** Au moins six produits agrègent déjà films, séries,
livres, jeux, musique et anime au même endroit : **Tastral, Medialib, Media Tracker App, OnTrack,
Achriom, Floppy**. Deux détails qui piquent :
- **Ils ont déjà les imports** Letterboxd / Goodreads / Trakt / AniList / IMDb — c'est-à-dire la
  réponse au démarrage à froid du §5, livrée avant nous.
- **Achriom embarque un « AI librarian » qui trouve les motifs à travers les médias** — l'idée Jarvis,
  déjà en production.

**L'autre territoire** : `Exist.io` et `Gyroscope` agrègent la vie depuis 15 ans, mais des
**métriques** (pas, sommeil, humeur), pas des artefacts culturels. Vivants, et leurs utilisateurs se
plaignent du manque de sources et de personnalisation.

**Le cimetière est réel** : *Numerous* (tableau de bord personnel, 2 fondateurs, 700 k$ levés) a fermé
en 2016, et le mouvement *quantified self* est largement déclaré mort. **Pourquoi ils meurent** — et ça
concerne HEGON directement : la valeur est proportionnelle à la complétude des données, la complétude
exige des intégrations, et les intégrations cassent. HEGON porte déjà **neuf dépendances externes**
(TMDB, football-data, TheSportsDB, jolpica, AniList, Wikidata, OpenWeather, Google Books, OpenLibrary)
— **et Ergast est déjà mort.**

**Où HEGON se situe vraiment** : entre deux territoires occupés. Les trackers de médias agrègent des
**artefacts** mais ignorent le sport-fan, les objectifs, les habitudes, le journal, les voyages.
Les tableaux quantified-self agrègent des **métriques** mais aucun artefact culturel. **HEGON fait les
deux — personne d'autre.** ❓ Reste la question sans réponse : **ce vide est-il un marché ou un no
man's land ?** Quinze ans que les deux camps existent sans se rejoindre. La charge de la preuve est du
côté de « c'est un marché ».

⚠️ **Contre-intuitif mais décisif** : un HEGON vendable aurait **MOINS de modules, pas plus.** Chaque
module ajouté rend le produit plus dur à expliquer, plus dur à démarrer à froid, et ajoute une
intégration à maintenir. Le chemin vers un produit payant est plus **étroit**, pas plus large.

⭐ **Ce qui survit à tout ça** : ces produits sont des **trackers** — ils enregistrent ce qu'on a
consommé. L'intention de l'owner est autre : **mémoire et transmission** (« montrer à mes enfants dans
20 ans »). Personne ne vend ça. Le piège est jumeau : **la transmission ne se monétise pas** — personne
ne paie aujourd'hui pour une valeur qui arrive dans vingt ans. C'est exactement pourquoi le créneau est
libre.

## 3. ✅ Le seul différenciateur défendable : l'agrégation TRANSVERSALE

Ce n'est ni le « personal OS », ni Jarvis, ni le catalogue.

⚠️ **Formulation corrigée après la recherche du §2bis.** La version initiale disait « personne n'agrège
à travers les domaines » — **c'est faux pour les médias** : au moins six produits le font. La version
exacte est plus étroite :

> **Personne n'agrège les ARTEFACTS CULTURELS et les MÉTRIQUES DE VIE dans le même objet.**
> Les trackers de médias (Tastral, Medialib, OnTrack…) s'arrêtent aux médias.
> Les tableaux quantified-self (Exist, Gyroscope) s'arrêtent aux métriques.
> HEGON fait les deux, plus le sport-fan, les objectifs, les habitudes et le journal.

⚠️ Corollaire inchangé et renforcé : **attaquer verticale par verticale est suicidaire.** Chaque
vertical a un incumbent avec vingt ans d'effet de réseau — et désormais on sait qu'il y a même des
concurrents sur l'agrégat média. Le seul terrain tenable est l'agrégat **complet**.

## 4. ✅ La promesse à 20 ans EST le produit

Si le pitch est « ta collection de vie qui dure », alors la **durabilité et l'export ne sont pas des
options, ce sont la fonctionnalité centrale**. Une app de collection qui perd les données n'a pas un
bug, elle a rompu son contrat.

⭐ **Conséquence** : le chantier d'audit (sauvegardes, reconstructibilité, cliquets) n'est pas de la
plomberie détournée du produit — **c'est le travail le plus fidèle à HEGON qui ait été fait.**
Rappel : jusqu'au 2026-09-06, la base était sur un plan gratuit **sans aucune sauvegarde**.

## 5. ✅ La réponse au démarrage à froid : RAPPELER, puis REFLÉTER

**Le problème.** La valeur de HEGON vient de l'accumulation (904 médias, 2 085 matchs chez l'owner).
Un inconnu ouvre une coquille vide. *« Ceux qui partagent le sentiment utiliseront HEGON »* est le
maillon faible du raisonnement : la question n'est pas s'ils partagent le **sentiment**, mais s'ils
feront le **travail**.

**La réponse : on ne démarre pas une collection, on récupère celle qu'on a déjà.**
Personne n'arrive vide — il a déjà vu 300 films, lu 40 livres, suivi son club depuis 15 ans. Aucune
app qui réussit dans cette catégorie ne démarre à zéro : Letterboxd demande *quels films tu as déjà
vus*, Goodreads pareil, Strava importe la montre, le Wrapped de Spotify est entièrement rétrospectif.

**Concrètement, en deux temps :**

1. **Le rappel** — pas un formulaire, un **rituel**. Une grille d'affiches iconiques, on tape celles
   qu'on a vues. ~90 secondes → ~30 entrées. Peu coûteux : TMDB fournit déjà le classement de
   popularité. *(Les imports Letterboxd/Goodreads CSV viennent plus tard : le rituel marche pour tout
   le monde, y compris ceux qui n'ont jamais rien utilisé.)*
2. **Le miroir** — et c'est la moitié qui compte. Immédiatement après, montrer quelque chose qu'il ne
   savait pas sur lui : *« ta décennie, c'est les années 2010 »*, *« tu as vu 4 des 9 derniers Oscars
   du meilleur film »*. **Il donne 90 secondes de mémoire, on lui rend un portrait.**

⭐ **Pourquoi c'est plus fort chez HEGON que chez Letterboxd** : après 5 minutes sur trois modules,
HEGON peut croiser films + livres + club. **L'agrégation transversale (§3) apparaît dès le jour 1 au
lieu du jour 30** — le démarrage à froid est résolu par ce qui rend le produit unique.

⚠️ **Ne jamais appeler ça « onboarding » ni le présenter comme de la saisie.** C'est le **déballage de
la collection** — le même geste que sortir ses cartes d'une boîte.

**Nuance à ne pas oublier** : le musée / la navigation encyclopédique (Oscars, palmarès, fiches
équipes) répond au **jour 30**, pas au jour 3. Et il n'est intéressant que **croisé** : *« les
meilleurs films de 1996 »* → Wikipédia le fait mieux ; *« 1996 : tu en as vu 4 sur 5 »* → inimitable.
⛔ **Ne pas copier un Top 250 façon IMDb** : ça exige une foule, et à 5 utilisateurs c'est du bruit.

## 6. ❓ Le modèle de prix : par PROFONDEUR, pas par module

Idée initiale de l'owner : certains modules gratuits, d'autres payants.
**Contre-argument** : ça pose une décision d'achat à chaque module et surtout ça **fragmente
précisément ce qui différencie** (§3 — l'agrégation transversale). Vendre les domaines séparément
détruit l'argument de vente.

**Proposition** : gratuit **limité en profondeur** (ex. 50 entrées, ou 1 an d'historique), payant pour
la vie entière. Le prix suit alors la valeur émotionnelle — l'accumulation — et le moment de payer
tombe quand l'attachement est maximal. **Non tranché.**

## 7. ⚠️ RISQUE BUSINESS NON VÉRIFIÉ — les licences des API

Le « musée » repose sur **TMDB, football-data et Wikidata**. Rien de cette couche n'appartient à HEGON.
Double conséquence :

- **Pas de différenciation possible dessus** — n'importe qui appelle les mêmes API. La valeur d'IMDb
  et Letterboxd n'est pas le catalogue, c'est 20 ans de contenu produit par leurs utilisateurs.
- ⚠️ **Les conditions d'utilisation encadrent l'usage commercial.** Le jour où quelqu'un paie, HEGON
  change de catégorie juridique. **À VÉRIFIER AVANT toute décision SaaS** — ça peut déterminer si un
  SaaS est possible sur la stack actuelle.

---

## Sujets encore à traiter (liste établie le 2026-09-07)

1. ✅ Le problème que HEGON résout pour quelqu'un d'autre — traité ci-dessus.
2. ✅ **Est-ce que ça existe déjà ?** — traité au §2bis. Réponse : **oui, la moitié médias**.
3. ⬜ **Le pari Jarvis.** L'owner croit qu'il lui manque la technologie ; à examiner — son avantage
   est peut-être déjà dans ses mains.
4. ⬜ **L'économie.** Qui paie, combien — et le chiffre que personne ne regarde : ce que **coûte** un
   utilisateur sur la stack actuelle.
5. ⬜ **Le partage : fonctionnalité ou moteur de croissance ?** (voir aussi l'architecture esquissée
   ci-dessous.)
6. ⬜ **L'issue alternative** : le SaaS n'est pas la seule sortie, ni forcément la plus rentable.

---

## Annexe — l'architecture de partage esquissée le 2026-09-07

⛔ **Ne rien construire.** Cible écrite pour ne pas l'inventer sous pression.

**Le constat qui change tout** : ce que l'owner appelle « multi-tenant » pour le partage n'en est pas.
Le multi-tenant, c'est de l'**isolation** ; partager, c'est **traverser la frontière du tenant**.
Vérifié dans le schéma : **36 tables** portent `org_isolation` avec `USING (org_id IN (SELECT
my_orgs()))`, **sans aucune clause d'exception**, et **zéro mécanisme de partage n'existe**.
⇒ `org_id` n'est pas une préparation au partage, c'est l'obstacle à contourner.

**Publication ≠ collaboration.** Les exemples de l'owner (top 10, un livre, un match, un goal terminé)
sont des choses qu'on **montre**, pas qu'on modifie à deux. La collaboration exige de réécrire les 36
policies ; la publication n'exige **rien** de tout ça. *(Tasks est la vraie exception : on y édite
ensemble — d'où workspaces + membres. C'est le bon découpage : Tasks collabore, le reste publie.)*

**Cible** : une table unique `shares (resource_type, resource_id, owner_org_id, token, visibility,
expires_at)` + une route publique en lecture seule `/s/[token]` rendant un **instantané**. Ajouter un
module partageable = une valeur de `resource_type` + un composant de rendu. Zéro RLS touchée.

⭐ **Instantané plutôt que vue vivante** : si je partage mon top 10 aujourd'hui et que j'ajoute un film
demain, le lien montre ce que j'ai partagé. Correspond à l'intention **et** supprime toute une classe
de fuites. *(Contre-argument à trancher : un profil public façon Letterboxd voudrait du vivant.)*

⭐ **Ce qui prépare vraiment le partage n'est pas `org_id`** — c'est la discipline modulaire : chaque
lecture passe par le `service.ts` du module, donc un futur rendu de partage réutilisera
`getMediaList()` telle quelle. **C'est l'axe 3 de l'audit par module.**
⇒ **L'audit EST la préparation du masterplan, pas un détour.**

⚠️ **Leçon à ne pas oublier** : le chemin d'invitation **au niveau organisation** avait été « préparé
pour plus tard » exactement dans cet esprit — 5 fonctions écrites, jamais branchées, mortes, et
faisant doublon avec l'invitation workspace qui, elle, fonctionne. **Une préparation qu'on n'exerce
pas ne reste pas prête, elle pourrit.** Ce qui reste prêt, c'est la discipline structurelle.
