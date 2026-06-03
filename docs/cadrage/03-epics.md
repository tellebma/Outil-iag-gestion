# 03 — Epics

Découpage du projet en epics priorisés. Chaque epic = un lot de valeur
autonome. Les priorités suivent les décisions de cadrage : **d'abord la couche
code-aware**, ensuite la production de code.

| # | Epic | Priorité | Dépend de |
|---|------|----------|-----------|
| E0 | Cadrage & fondations | P0 | — |
| E1 | Ingestion & cartographie du code | P0 | E0 |
| E2 | Modèle de contexte par rôle | P0 | E1 |
| E3 | Skills de récupération pour agents | P1 | E1 |
| E4 | Injection de contexte dans le heartbeat | P1 | E2, E3 |
| E5 | Boucle de cadrage assistée (CEO → CTO → tickets) | P2 | E2, E4 |
| E6 | Adapter de session Claude (production de code) | P2 | E4 |
| E7 | Cockpit UI | P3 | E2, E6 |

> Légende priorités : **P0** = socle indispensable, **P1** = cœur de la valeur,
> **P2** = capacités avancées, **P3** = confort/visualisation.

---

## E0 — Cadrage & fondations
**Objectif** : poser la vision, le modèle de contexte, les epics, et trancher
la stratégie d'intégration (fork vs plugin vs autonome).
**Valeur** : tout le monde (humain + agents) partage la même cible.
**Livrables**
- [x] Documents de cadrage (`docs/cadrage/`).
- [x] Décisions actées : **fork (A)**, indexation **totale (embeddings)**,
  docs **hybrides** (génération depuis le code par défaut).
- [ ] **Création du fork Paperclip** (⏳ en attente — côté utilisateur).
- [ ] Carte des points d'extension Paperclip (cf. doc 04).
- [ ] Squelette de l'extension une fois le fork disponible.

## E1 — Ingestion & cartographie du code
**Objectif** : à l'ajout d'un repo, construire automatiquement une
représentation exploitable : arborescence, modules, dépendances, points
d'entrée, conventions, et **résumés d'architecture**.
**Valeur** : la matière première de toute la pertinence des agents.
**User stories**
- En tant que système, à l'ajout d'un projet, je **scanne** la base de code et
  produis une **carte des modules**.
- En tant que système, je génère un **résumé d'architecture** de haut niveau.
- (Option) En tant que système, je calcule des **embeddings** pour la recherche
  sémantique.
**Livrables** : pipeline d'ingestion, schéma de l'index, stockage côté serveur.
**Profondeur (décidée)** : « la totale » — structure + symboles + résumés +
**embeddings** pour la recherche sémantique (ex. `pgvector` sur le PostgreSQL
de Paperclip, à confirmer sur le fork).

## E2 — Modèle de contexte par rôle
**Objectif** : matérialiser la **pyramide de contexte** — définir les niveaux
(`architecture`, `flux`, `carte-modules`, `fichier`, `recherche`) et la
**politique** qui associe chaque rôle aux niveaux autorisés/pertinents.
**Valeur** : le CEO délègue sans charger le code ; le CTO cadre avec le détail.
**User stories**
- En tant que CEO, je reçois **archi + flux + objectifs**, pas le code complet.
- En tant que CTO, je peux **descendre** dans le détail à la demande.
- En tant qu'admin, je **configure** quel rôle voit quel niveau.
**Livrables** : modèle de niveaux, politique de contexte par rôle, configuration.

## E3 — Skills de récupération pour agents
**Objectif** : exposer aux agents des outils de récupération de contexte,
branchés sur le système de **skills** de Paperclip.
**Valeur** : un agent autorisé va chercher *activement* l'info dont il a besoin.
**User stories**
- `search_code(query)` — recherche dans la base de code.
- `get_file(path)` — lecture d'un fichier.
- `get_architecture_summary()` / `get_module_map()` — vues de haut niveau.
**Livrables** : skills installables, contrôle d'accès par rôle, traçabilité
(rattachement au `run_id` du heartbeat).

## E4 — Injection de contexte dans le heartbeat
**Objectif** : au checkout d'un ticket, enrichir automatiquement le
`heartbeat-context` de l'agent avec le contexte pertinent **selon son rôle**.
**Valeur** : les propositions deviennent ancrées dans le réel, sans action
manuelle.
**User stories**
- En tant qu'agent, quand je prends un ticket, le **contexte code pertinent**
  est déjà là.
- En tant que système, je respecte le **budget tokens** en bornant le volume
  injecté.
**Livrables** : hook d'injection, sélection/troncature pertinente, garde-fous
budget.

## E5 — Boucle de cadrage assistée (CEO → CTO → tickets)
**Objectif** : outiller le flux où le CEO **délègue** une intention, le CTO
**cadre** en tickets actionnables, à partir du contexte par rôle.
**Valeur** : matérialise l'exemple de bout en bout du doc 02.
**Livrables** : routines/templates d'agents, format de cadrage, génération de
tickets reliés à un epic.

## E6 — Adapter de session Claude (production de code)
**Objectif** : piloter des **sessions Claude** attachées à un ticket + worktree,
capturer transcript, tool-calls, coût et **diff produit**, permettre la
**reprise** de session (`--resume`).
**Valeur** : les agents ne font plus que proposer — ils **produisent du code**
de façon pilotable et auditable.
**Livrables** : adapter, modèle de domaine « session de code », reprise,
work products (diff/branche).

## E7 — Cockpit UI
**Objectif** : visualiser la pyramide (docs d'archi/flux), suivre une session
Claude en direct, **réviser → approuver / relancer / commenter**.
**Valeur** : le poste de pilotage humain au-dessus des agents.
**Livrables** : vues archi/flux, vue session + diff, actions de revue.

---

## Séquence recommandée
**E0 → E1 → E2 → (E3 ∥ E4) → E5 → E6 → E7**

Les epics E3 et E4 peuvent avancer en parallèle une fois E1/E2 posés.
