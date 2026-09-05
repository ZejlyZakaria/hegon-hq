# <MODULE> — Architecture

> **Ce fichier ne décrit pas le code. Il énonce ce que le code doit garantir.**
> Copie ce dossier pour un nouveau module. Quatre fichiers, quatre raisons d'être mises à jour.

## Les invariants

Un invariant est une phrase qui doit rester vraie quoi qu'on ajoute. Chacun a un **point
d'étranglement** : l'endroit unique par lequel il est IMPOSÉ. Un invariant sans point
d'étranglement est un vœu — la prochaine surface l'oubliera par défaut, et personne ne le verra.

| # | L'invariant | Son point d'étranglement | La faute qu'il évite |
|---|---|---|---|
| a | *ex : le statut est dérivé, jamais assemblé à la main* | `lib/<x>.ts`, fonction pure | trois surfaces, trois règles |
| b | *ex : toute écriture est parsée avant d'entrer* | le parse DANS le service, pas dans le hook | une porte future qui contourne |

**La règle qui décide si une ligne appartient à ce tableau :** peux-tu nommer l'endroit qui
l'empêche d'être violée ? Si non, ce n'est pas un invariant, c'est un commentaire.

## Ce qui est délibérément EXCLU

Les exceptions assumées, avec leur raison. Sans cette section, chaque relecture les redécouvre
comme des bugs et quelqu'un « corrige » un choix.

## Performance · Tests · Sécurité

Pas de fichiers séparés : ce sont des invariants de ce module, ils vivent dans le tableau
ci-dessus. Les règles GLOBALES (la méthode de mesure, la philosophie de test, le modèle de
menace) sont dans `hq/rules/`.
