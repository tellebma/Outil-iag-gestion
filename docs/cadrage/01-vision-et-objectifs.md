# 01 — Vision et objectifs

## 1. Le constat

[Paperclip](https://paperclip.ing) est un *control plane* open-source (Node.js +
React + PostgreSQL) pour faire tourner une **équipe d'agents IA comme une
entreprise** : org chart, rôles, budgets, tickets, **heartbeats** (réveils
planifiés), workspaces en git worktrees, gouvernance et approbations.

Sa limite, du point de vue de notre usage : Paperclip raisonne au niveau
**organisationnel / ticket**. Un agent reçoit bien un git worktree, mais il
reste **« aveugle » au code** — il n'a pas de compréhension globale de
l'application (architecture, flux, conventions, dépendances). Résultat : ses
propositions risquent d'être **génériques** plutôt qu'**ancrées dans le réel**.

## 2. La vision

> Étendre Paperclip pour que ses agents **exploitent le code applicatif** et
> deviennent **pertinents** dans leurs propositions, priorisations, découpages
> et décisions — chaque rôle recevant le contexte **à la bonne altitude**.

Deux capacités nouvelles, complémentaires :

1. **Comprendre le code** — une couche d'intelligence qui cartographie
   l'application et sert, à chaque agent, le niveau d'information adapté à son
   rôle (du document d'archi de haut niveau jusqu'au fichier source).
2. **Produire du code** (prolongement) — piloter des sessions Claude attachées
   à un ticket et son worktree : transcript, diff produit, reprise de session,
   revue/validation.

## 3. Le principe central : le contexte à la bonne altitude

Tous les agents **n'ont pas besoin du code complet**.

- Le **CEO** a besoin de **documents d'archi, de flux, d'objectifs** — une vue
  *suffisante* pour **prioriser et déléguer**.
- Le **CTO** descend dans le détail pour **cadrer précisément** la demande
  avant de la confier à un agent codeur.
- Le **Coder** a besoin du code complet et de son worktree pour produire.
- La **QA** a besoin des specs, tests et diffs.

C'est une **pyramide de contexte par rôle**, alimentée à partir du code réel.
Détaillée dans [02 — Modèle de contexte par rôle](./02-modele-de-contexte-par-role.md).

## 4. Périmètre

### Dans le périmètre
- Couche d'**ingestion & cartographie** du code (structure, résumés d'archi,
  documents de flux).
- **Distribution du contexte par rôle** (la pyramide).
- **Skills de récupération** exposés aux agents (rechercher / lire / résumer).
- **Injection de contexte** dans le heartbeat des agents.
- Boucle de **cadrage assistée** (CEO → CTO → epics → tickets).
- (Prolongement) **Adapter de session Claude** pilotable et sa **vue cockpit**.

### Hors périmètre (pour l'instant)
- Refonte du moteur d'orchestration de Paperclip (heartbeats, budgets,
  gouvernance) — on **réutilise** l'existant.
- Support multi-fournisseurs de modèles au-delà de ce que Paperclip gère déjà.
- Hébergement / offre cloud.

## 5. Stratégie d'intégration (à valider)

**Recommandation : Option A — forker Paperclip et étendre dedans.** La couche
de contexte nécessite des changements **côté serveur** (stockage de l'index,
injection dans le `heartbeat-context`) qu'un simple plugin externe ne permet
probablement pas. On capitalise ainsi sur tout l'acquis (auth, budgets,
tickets, worktrees, gouvernance).

Alternatives documentées : **B** (plugin/skill externe, non-invasif mais
limité) et **C** (outil autonome inspiré, liberté totale mais on reperd
l'orchestration de Paperclip).

## 6. Objectifs mesurables (cibles initiales)

- Un repo ajouté est **cartographié automatiquement** (structure + résumé
  d'archi) sans intervention manuelle.
- Un agent CEO peut **prioriser et déléguer** à partir des seuls documents
  d'archi/flux, sans charger le code complet.
- Un agent CTO peut **cadrer un epic en tickets** en s'appuyant sur le contexte
  code injecté.
- Mesure de pertinence : les propositions citent des **éléments réels** du code
  (fichiers, modules, flux) plutôt que des généralités.

## 7. Priorités de démarrage

1. **D'abord** : documents de cadrage + epics (ce dossier).
2. **Ensuite** : la couche *code-aware* (ingestion, contexte par rôle, skills,
   injection). — *priorité confirmée*
3. **Enfin** : l'adapter de session Claude et son cockpit.
