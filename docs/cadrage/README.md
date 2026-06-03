# Documents de cadrage — Outil IAG de gestion

Ce dossier contient les documents de cadrage du projet : une extension de
[Paperclip](https://paperclip.ing) visant à rendre les agents IA **conscients
du code applicatif**, afin que leurs propositions, priorisations et décisions
soient ancrées dans la réalité de la base de code — et non génériques.

> **Statut** : cadrage en cours. Aucune décision technique n'est figée tant
> qu'elle n'est pas validée explicitement (voir « Points ouverts » ci-dessous).

## Sommaire

| Doc | Contenu |
|-----|---------|
| [01 — Vision et objectifs](./01-vision-et-objectifs.md) | Le problème, la vision, ce qu'on ajoute à Paperclip, le périmètre, les non-objectifs |
| [02 — Modèle de contexte par rôle](./02-modele-de-contexte-par-role.md) | La « pyramide de contexte » : quelle altitude d'information pour chaque rôle d'agent (CEO → CTO → Coder → QA) |
| [03 — Epics](./03-epics.md) | Le découpage en epics priorisés, avec valeur, périmètre et livrables |

## Principe directeur

Paperclip orchestre une **équipe d'agents** (org chart, budgets, tickets,
heartbeats, gouvernance) mais reste **abstrait sur le code** : un agent dispose
d'un git worktree, mais ne *comprend* pas l'application. On comble ce manque par
une **couche d'intelligence du code** qui sert, à chaque rôle, le **bon niveau
de contexte**.

## Points ouverts (à valider)

1. **Mode d'intégration** : fork de Paperclip et extension dedans (**Option A**,
   recommandée) vs plugin externe (**B**) vs outil autonome inspiré (**C**).
   → *cf. 01-vision, §« Stratégie d'intégration »*.
2. **Profondeur d'indexation initiale** : structure + résumés d'archi seuls, ou
   d'emblée recherche sémantique (embeddings) ?
3. **Source des documents d'archi/flux** : générés automatiquement depuis le
   code, rédigés à la main, ou hybride entretenu par les agents ?
