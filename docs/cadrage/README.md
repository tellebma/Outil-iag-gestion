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
| [04 — Stratégie d'intégration](./04-strategie-integration.md) | Fork de Paperclip, carte des points d'extension, logistique |

## Principe directeur

Paperclip orchestre une **équipe d'agents** (org chart, budgets, tickets,
heartbeats, gouvernance) mais reste **abstrait sur le code** : un agent dispose
d'un git worktree, mais ne *comprend* pas l'application. On comble ce manque par
une **couche d'intelligence du code** qui sert, à chaque rôle, le **bon niveau
de contexte**.

## Décisions actées

1. **Mode d'intégration** : **fork de Paperclip et extension dedans (Option A)**.
   → *cf. [04 — Stratégie d'intégration](./04-strategie-integration.md)*.
2. **Profondeur d'indexation** : **« la totale » dès le départ** — structure +
   symboles + résumés + **recherche sémantique (embeddings)**.
3. **Source des documents d'archi/flux** : **hybride** — génération **depuis le
   code par défaut** (quand rien n'existe), puis raffinage par un agent
   Architecte/Doc.

## Pré-requis / en attente

- ⏳ **Le fork de Paperclip n'est pas encore créé.** Tant qu'il n'existe pas, on
  n'écrit pas de code dans le moteur Paperclip ; on prépare le terrain (cadrage,
  carte des points d'extension, specs). Reste à décider la **logistique du
  fork** : fork dans un repo dédié, ou intégration du code Paperclip dans
  `Outil-iag-gestion` ? → *cf. 04, §logistique*.
