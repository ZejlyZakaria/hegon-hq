---
name: challenge
description: Lance un contre-examen indépendant sur une conclusion, une architecture ou une stratégie. À utiliser quand une décision est incertaine ou coûteuse à défaire — pas pour du travail de routine. Sans argument, challenge la conclusion qui vient d'être produite ; avec un argument, challenge ce qui est fourni (par exemple un retour collé depuis une autre conversation).
---

# /challenge — contre-examen

Tu déclenches l'agent **`challenger`** (`.claude/agents/challenger.md`) sur un résultat, et tu en
restitues le rapport. Ce skill encode le **protocole**, pas seulement le lancement.

## 1. Identifie l'artefact

**Sans argument** → la conclusion que tu viens toi-même de produire dans cette conversation.
**Avec un argument** → ce que l'owner fournit (souvent un retour collé d'une autre conversation, ou un
chemin de fichier).

## 2. ⚠️ La règle qui fait toute la valeur : transmets le RÉSULTAT, jamais le RAISONNEMENT

Reformule l'artefact en **affirmations vérifiables** — des faits chiffrés d'un côté, des jugements de
l'autre. **Ne transmets pas comment tu y es arrivé.** Si le challenger lit ton cheminement, il s'y
ancre et finit par le confirmer ; il ne vaut que s'il refait le chemin par un autre bout.

Format qui marche (éprouvé le 2026-09-10) :

> **Affirmations factuelles à vérifier :** 1. … 2. … *(chiffres, chemins de fichiers, valeurs)*
> **Conclusions de jugement à challenger :** A. … B. … *(les décisions, les recommandations)*
> Termine par : « Vérifie les points factuels en exécutant. Puis dis-moi lesquelles des conclusions ne
> tiennent pas, et ce qui manque au diagnostic. »

⛔ Ne dis jamais au challenger que tu es l'auteur, ni ce dont tu es le moins sûr. Tu orienterais sa
recherche.

## 3. Lance-le

Un seul agent, en **premier plan** (son rapport est le seul livrable ; rien d'autre d'utile ne peut se
faire pendant). Compte plusieurs minutes : il lit, il exécute, il compte.

## 4. Restitue — et c'est là que l'honnêteté se joue

⭐ **Relaie les objections VERBATIM, en particulier celles qui te visent.** Le rapport passe par toi :
c'est le point faible du dispositif, et le seul garde-fou est que tu ne l'adoucisses pas. Une objection
prouvée qui t'embarrasse est exactement celle qui a le plus de valeur.

Organise en trois temps : **ce que ça change pour l'owner** (l'actionnable d'abord, c'est ce qui
l'intéresse) · **où tu avais tort** (concis, sans t'excuser, sans ruminer) · **ce qui reste ouvert**.

Puis, pour chaque objection : **concède ou réfute — avec une preuve exécutée, pas une opinion.**
Ce qui survit à ça, **c'est l'owner qui tranche.** Ne clos jamais un désaccord par lassitude.

## 5. Fais-le atterrir

Règle du chantier : **rien n'atterrit en prose seule.** Si le contre-examen change une décision, une
règle ou un chiffre, écris-le dans le fichier concerné — `hq/audit/plan.md`, `hq/rules/*.md`,
`hq/product-strategy.md` — pas seulement dans la conversation. Sinon on refera le tour dans trois
semaines.

## Quand NE PAS l'utiliser

Chaque lancement recharge tout le contexte à froid : c'est le chemin coûteux. **Réserve-le aux
décisions incertaines ou chères à défaire** — une architecture, une doctrine, une stratégie, un
diagnostic qui va déclencher du travail. Pas pour une correction de bug ni une passe de routine.

## La limite à garder en tête

Les deux côtés partagent le **même modèle, donc les mêmes angles morts**. Ce dispositif attrape très
bien les **omissions** et les **affirmations non vérifiées**. Il n'attrape pas une erreur commise des
deux côtés à l'identique. **La question naïve de l'owner reste irremplaçable** — et l'automatisation ne
doit jamais lui retirer l'arbitrage.
