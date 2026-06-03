# 02 — Modèle de contexte par rôle

> Principe central du projet : **le contexte servi à un agent doit être à la
> bonne altitude selon son rôle.** Donner le code complet à tout le monde est
> inutile, coûteux (tokens) et contre-productif (bruit). On construit une
> **pyramide de contexte** alimentée à partir du code réel.

## 1. La pyramide

```
        ┌─────────────────────────────┐
        │            CEO              │  Vision, objectifs, docs d'archi & de
        │   (vue suffisante pour      │  flux, métriques. → PRIORISE & DÉLÈGUE
        │     déléguer)               │
        ├─────────────────────────────┤
        │            CTO              │  Archi détaillée, carte des modules,
        │  (cadre précisément la      │  accès « drill-down » au code.
        │    demande)                 │  → CADRE en tickets
        ├─────────────────────────────┤
        │           Coder             │  Code complet + git worktree.
        │  (produit le code)          │  → IMPLÉMENTE
        ├─────────────────────────────┤
        │            QA               │  Specs, tests, diffs produits.
        │  (valide)                   │  → VÉRIFIE
        └─────────────────────────────┘
```

Plus on monte, plus le contexte est **synthétique et abstrait** ; plus on
descend, plus il est **détaillé et brut**.

## 2. Détail par rôle

### CEO (et management)
- **A besoin de** : vision produit, objectifs/missions, **documents d'archi**
  (vue d'ensemble des composants), **documents de flux** (parcours métier),
  métriques d'avancement et de coût.
- **N'a pas besoin de** : le code source complet.
- **Sortie attendue** : priorisation, allocation de budget, **délégation** à un
  CTO avec une intention claire.

### CTO (et architectes)
- **A besoin de** : l'archi détaillée, la **carte des modules** et leurs
  dépendances, la capacité de **descendre dans le code** à la demande (lire un
  fichier, chercher un symbole), l'historique des décisions techniques.
- **Sortie attendue** : un **cadrage précis** — découpage d'un epic en tickets
  actionnables, contraintes techniques explicitées, fichiers/zones impactés
  identifiés.

### Coder
- **A besoin de** : le **code complet** dans un **git worktree** isolé, le
  ticket cadré par le CTO, les conventions du projet.
- **Sortie attendue** : un **diff** / une branche, des tests, un work product
  inspectable.

### QA
- **A besoin de** : les specs du ticket, la suite de tests, le **diff produit**.
- **Sortie attendue** : verdict de validation, anomalies remontées.

## 3. D'où viennent les « documents d'archi / flux » ?

Trois sources possibles (à arbitrer — *point ouvert n°3*) :

1. **Générés depuis le code** par la couche d'ingestion (carte des modules,
   dépendances, points d'entrée → résumés).
2. **Rédigés à la main** et versionnés.
3. **Hybride entretenu par les agents** : générés puis raffinés/validés par un
   agent (ex. le CTO maintient le document d'archi à jour à chaque epic).

La cible privilégiée est l'**hybride** : génération automatique pour le socle,
entretien par les agents pour la pertinence.

## 4. Mécanisme d'implémentation (esquisse)

S'appuie sur les points d'extension existants de Paperclip :

- **Index de code** par projet (stockage côté serveur) : structure, symboles,
  résumés d'archi, documents de flux, (option) embeddings.
- **Niveaux de contexte** matérialisés comme artefacts adressables :
  `architecture`, `flux`, `carte-modules`, `fichier`, `recherche`.
- **Politique de contexte par rôle** : à chaque heartbeat, on injecte dans le
  `heartbeat-context` de l'agent **seulement** les niveaux autorisés/pertinents
  pour son rôle.
- **Skills de récupération** : un agent autorisé (ex. CTO) peut *descendre*
  explicitement via `search_code`, `get_file`, `get_architecture_summary`.

> Détail du découpage en chantiers : voir [03 — Epics](./03-epics.md).

## 5. Exemple de bout en bout

1. Le **CEO** lit le document d'archi + les objectifs → décide de prioriser
   l'epic « facturation » et **délègue** au CTO.
2. Le **CTO** reçoit l'intention, *drill-down* dans la carte des modules de
   facturation, identifie les fichiers impactés, et **cadre** 3 tickets précis.
3. Chaque ticket part à un **Coder** avec worktree → diff.
4. La **QA** valide le diff contre les specs du ticket.

À aucun moment le CEO n'a chargé le code complet : il a eu une **vue
suffisante pour déléguer**.
