# 02 — Modèle de contexte par rôle

> Principe central du projet : **le contexte servi à un agent doit être à la
> bonne altitude selon son rôle.** Donner le code complet à tout le monde est
> inutile, coûteux (tokens) et contre-productif (bruit). Chaque rôle de
> l'organisation reçoit un **profil de contexte** taillé pour sa mission,
> alimenté à partir du code réel.

## 1. Org chart cible

```
                         ┌───────┐
                         │  CEO  │   vision · priorise · délègue
                         └───┬───┘
              ┌──────────────┴───────────────┐
          ┌───┴───┐                       ┌───┴───┐
          │  CTO  │ cadre technique       │  PO   │ cadre produit
          └───┬───┘                       └───┬───┘
              │                  ┌────────────┴───────────┐
         ┌────┴────┐         ┌───┴────┐               ┌────┴────┐
         │ n × DEV │         │ UI/UX  │               │ SEO/GEO │
         │ produit │         │ design │               │ contenu │
         │ le code │         │ parcours│              │ visibilité│
         └─────────┘         └────────┘               └─────────┘
```

**Lignes de reporting (chaîne de délégation des heartbeats)**
- **CEO** → CTO, PO
- **CTO** → n × DEV
- **PO** → UI/UX, SEO/GEO

**Rôles optionnels (à activer plus tard)**
- **Architecte/Doc** — maintient les documents d'archi & de flux à partir du
  code (alimente la pyramide). À défaut : porté par le **CTO**.
- **QA** — validation des diffs. À défaut : porté par les **DEV** + **CTO**.
- **DevOps** — CI/CD et déploiement du code produit.

## 2. Missions par rôle

| Rôle | Mission | Sortie attendue |
|------|---------|-----------------|
| **CEO** | Vision, priorisation, allocation de budget | Délégation d'intentions claires au CTO / PO |
| **CTO** | Cadrage technique, garant de l'archi | Découpage technique, contraintes, tickets DEV |
| **PO** | Cadrage produit, propriétaire du backlog | Epics & user stories priorisés |
| **DEV** | Implémentation | Diff / branche + tests, work product inspectable |
| **UI/UX** | Expérience & interface | Parcours, composants, cohérence design |
| **SEO/GEO** | Visibilité (moteurs de recherche **et** moteurs génératifs/LLM) | Contenu, métadonnées, structure optimisés |

## 3. Catalogue des niveaux de contexte

La couche *code-aware* expose le code sous forme de **niveaux** adressables,
du plus synthétique au plus brut :

1. **Vision & objectifs** — missions, buts produit.
2. **Métriques** — avancement, coût (tokens), KPIs.
3. **Architecture** — vue d'ensemble des composants (haut niveau).
4. **Flux métier** — parcours fonctionnels de bout en bout.
5. **Carte des modules** — modules, dépendances, points d'entrée.
6. **Code source** — fichiers, symboles, recherche (le plus brut).
7. **Backlog / specs** — epics, user stories, critères d'acceptation.
8. **Parcours utilisateur (UX)** — écrans, enchaînements, états.
9. **Composants UI / design system** — composants front, styles, tokens.
10. **Contenu & SEO/GEO** — pages publiques, métadonnées, routes, sitemap.

## 4. Matrice rôle × niveau de contexte

Légende : **◎** injecté par défaut au heartbeat · **○** accessible à la demande
(drill-down via skill) · **—** non pertinent.

| Niveau \ Rôle | CEO | CTO | PO | DEV | UI/UX | SEO/GEO |
|---|:--:|:--:|:--:|:--:|:--:|:--:|
| 1. Vision & objectifs | ◎ | ◎ | ◎ | ○ | ◎ | ◎ |
| 2. Métriques | ◎ | ◎ | ◎ | — | ○ | ◎ |
| 3. Architecture | ◎ | ◎ | ○ | ◎ | ○ | ○ |
| 4. Flux métier | ◎ | ◎ | ◎ | ○ | ◎ | ○ |
| 5. Carte des modules | ○ | ◎ | ○ | ◎ | — | — |
| 6. Code source | — | ◎ | — | ◎ | ○ | ○ |
| 7. Backlog / specs | ◎ | ◎ | ◎ | ◎ | ◎ | ◎ |
| 8. Parcours utilisateur | ○ | ○ | ◎ | ○ | ◎ | ◎ |
| 9. Composants UI / design | — | ○ | ◎ | ○ | ◎ | ○ |
| 10. Contenu & SEO/GEO | ○ | ○ | ◎ | ○ | ○ | ◎ |

Lecture : le **CEO** délègue à partir de vues synthétiques (◎ sur 1-4, 7) sans
jamais charger le code source (—). Le **DEV** vit dans le code (◎ sur 3, 5, 6).
Le **SEO/GEO** est centré contenu/visibilité (◎ sur 10, 8, 2) et descend dans le
front public à la demande (○ sur 6). L'**UI/UX** vit dans les parcours et les
composants (◎ sur 8, 9, 4).

## 5. D'où viennent les documents d'archi / flux ?

**Décision : hybride.** Par défaut, on **génère depuis le code** (carte des
modules, dépendances → résumés) dès que rien n'existe ; puis un agent
**Architecte/Doc** (ou le CTO) **raffine et valide**. Les documents rédigés à la
main, quand ils existent, font autorité sur la génération automatique.

## 6. Mécanisme d'implémentation (esquisse)

S'appuie sur les points d'extension existants de Paperclip :

- **Index de code** par projet (stockage serveur) : structure, symboles,
  résumés, flux, (option) embeddings.
- **Niveaux de contexte** matérialisés comme artefacts adressables (cf. §3).
- **Politique de contexte par rôle** = la matrice du §4, appliquée à chaque
  heartbeat : on injecte les niveaux **◎** et on autorise les **○** en
  drill-down.
- **Skills de récupération** pour les niveaux **○** : `search_code`, `get_file`,
  `get_module_map`, `get_architecture_summary`…

> Découpage en chantiers : voir [03 — Epics](./03-epics.md).

## 7. Exemple de bout en bout

1. Le **CEO** lit l'archi + les objectifs → priorise l'epic « facturation » et
   **délègue** au CTO et au PO.
2. Le **PO** précise les user stories et les parcours impactés (niveaux 7, 8).
3. Le **CTO** *drill-down* dans la carte des modules de facturation (niveau 5),
   identifie les fichiers (niveau 6) et **cadre** des tickets DEV.
4. Chaque **DEV** implémente dans son worktree → diff.
5. L'**UI/UX** ajuste les écrans concernés ; le **SEO/GEO** vérifie l'impact sur
   les pages publiques et métadonnées (niveau 10).

À aucun moment le CEO n'a chargé le code complet : il a eu une **vue
suffisante pour déléguer**.
