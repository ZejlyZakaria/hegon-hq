# Configuration Claude Code — copie de sauvegarde

⚠️ **`.claude/` est dans le `.gitignore` du dépôt principal.** Ce qui vit là-bas n'est versionné
nulle part et disparaît avec la machine. Ce dossier en est la copie, dans le dépôt privé `hegon-hq`.

## Restaurer

```bash
cp hq/claude-config/agents/challenger.md            .claude/agents/
cp -r hq/claude-config/skills/challenge             .claude/skills/
```

## Ce que contient ce dossier

**`agents/challenger.md`** — le contre-examen. Un sous-agent démarre **à froid**, sans le contexte de
la conversation : il ne peut donc pas s'ancrer sur le raisonnement qu'il doit vérifier. C'est la
mécanique qui applique le principe, pas une consigne. Lecture seule (`Read`, `Grep`, `Glob`, `Bash`),
il ne modifie jamais rien.

**`skills/challenge/SKILL.md`** — le protocole autour : transmettre le **résultat** et jamais le
raisonnement, relayer les objections **verbatim** y compris celles qui visent l'auteur, concéder ou
réfuter **avec preuve**, et faire atterrir le résultat dans un fichier.

## Pourquoi ça existe

Testé le **2026-09-10** sur une analyse du cache. Le contre-examen a sorti **11 objections, toutes
prouvées par exécution**, dont : un comptage de mots pris pour un comptage de déclarations (131 vs
125), une règle de doctrine **déjà écrite** et re-dérivée sans être citée, une règle ESLint proposée
qui aurait été **rouge sur 123 sites sur 125** — donc désactivée le premier jour, l'anti-motif que le
projet documente pourtant lui-même — et une classe de bugs jamais vue : **19 sites où `gcTime`
plafonne silencieusement le `staleTime`**, dont un écart de 144× avec un commentaire faux.

⚠️ **La limite** : les deux côtés partagent le même modèle, donc les mêmes angles morts. Le dispositif
attrape les omissions et les affirmations non vérifiées, **pas** une erreur commise à l'identique des
deux côtés. L'arbitrage reste à l'owner — c'est délibéré.
