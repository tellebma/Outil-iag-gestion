# Spec E2 — Politique de contexte par rôle

> Objet : définir **comment la matrice rôle × niveau** du
> [doc 02](../cadrage/02-modele-de-contexte-par-role.md) devient une
> **configuration** appliquée à chaque heartbeat — quoi injecter, quoi exposer
> en drill-down, comment respecter le budget tokens.
>
> Dépend de [Spec E1](./E1-index-de-code.md) (source des niveaux de contexte).

## 1. Concepts

- **Rôle** : `CEO | CTO | PO | DEV | UI_UX | SEO_GEO` (+ optionnels
  `ARCHITECTE_DOC | QA | DEVOPS`).
- **Niveau de contexte** : les 10 niveaux du doc 02 (clés ci-dessous).
- **Mode** : `inject` (◎ poussé d'office) · `on_demand` (○ accessible via skill)
  · `none` (— interdit).
- **Budget** : plafond de tokens du contexte injecté, par rôle/heartbeat.

Clés de niveaux :
`vision · metrics · architecture · flows · module_map · source_code ·
backlog · ux_flows · ui_components · content_seo`

## 2. Format de configuration

Fichier versionné (ex. `config/context-policy.yaml`), surchargeable par
société/projet. Encode **exactement** la matrice du doc 02.

```yaml
version: 1
defaults:
  budget_tokens: 6000          # plafond par défaut du contexte injecté
  truncation: priority         # voir §4
roles:
  CEO:
    budget_tokens: 5000
    levels:
      vision: inject
      metrics: inject
      architecture: inject
      flows: inject
      module_map: on_demand
      source_code: none
      backlog: inject
      ux_flows: on_demand
      ui_components: none
      content_seo: on_demand
  CTO:
    budget_tokens: 9000
    levels:
      vision: inject
      metrics: inject
      architecture: inject
      flows: inject
      module_map: inject
      source_code: inject
      backlog: inject
      ux_flows: on_demand
      ui_components: on_demand
      content_seo: on_demand
  PO:
    levels:
      vision: inject
      metrics: inject
      architecture: on_demand
      flows: inject
      module_map: on_demand
      source_code: none
      backlog: inject
      ux_flows: inject
      ui_components: inject
      content_seo: inject
  DEV:
    budget_tokens: 9000
    levels:
      vision: on_demand
      metrics: none
      architecture: inject
      flows: on_demand
      module_map: inject
      source_code: inject
      backlog: inject
      ux_flows: on_demand
      ui_components: on_demand
      content_seo: on_demand
  UI_UX:
    levels:
      vision: inject
      metrics: on_demand
      architecture: on_demand
      flows: inject
      module_map: none
      source_code: on_demand
      backlog: inject
      ux_flows: inject
      ui_components: inject
      content_seo: on_demand
  SEO_GEO:
    levels:
      vision: inject
      metrics: inject
      architecture: on_demand
      flows: on_demand
      module_map: none
      source_code: on_demand
      backlog: inject
      ux_flows: inject
      ui_components: on_demand
      content_seo: inject
```

> Cette config est la **source de vérité** de la matrice. Le doc 02 en est la
> vue lisible ; tout changement doit rester synchronisé entre les deux.

## 3. Algorithme de résolution (au heartbeat)

Entrée : `agent.role`, `issue`, `project`, `budget`.

```
1. policy   = merge(defaults, roles[agent.role], overrides[company/project])
2. inject   = { niveau | mode == inject }
3. pour chaque niveau de `inject` :
      bloc = fetch_level(niveau, issue, project)   # via index E1
4. ordonner les blocs par PRIORITÉ (§4)
5. assembler en respectant policy.budget_tokens (troncature §4)
6. exposer les niveaux `on_demand` comme SKILLS autorisés (cf. §5)
7. retourner { injected_context, allowed_skills }
```

Le résultat alimente le `heartbeat-context` de Paperclip (hook à brancher —
*à vérifier sur le fork*).

## 4. Budget & troncature

- **Stratégie `priority`** : on remplit le budget par ordre de priorité jusqu'à
  saturation ; le dernier bloc est tronqué proprement (jamais coupé en plein
  milieu d'un symbole/section).
- **Ordre de priorité par défaut** (du plus prioritaire au moins) :
  `backlog (ticket courant) > flows > architecture > module_map > source_code
  (ciblé sur le ticket) > ux_flows > ui_components > content_seo > vision >
  metrics`.
  → On garantit que **le ticket et le « pourquoi » fonctionnel** passent
  toujours ; les vues larges sont rognées en premier.
- **Pertinence** : pour les niveaux volumineux (`source_code`, `module_map`), on
  ne pousse que ce qui est **relié au ticket** (recherche sémantique E1 sur le
  titre/description du ticket), pas tout le repo.

## 5. Drill-down (niveaux `on_demand`)

- Chaque niveau `on_demand` est exposé à l'agent comme **skill** (E3) :
  `architecture→get_architecture_summary`, `module_map→get_module_map`,
  `source_code→search_code`/`get_file`, etc.
- **Contrôle d'accès** : un appel de skill est refusé si le niveau est `none`
  pour le rôle de l'agent. (Un DEV ne peut pas tirer les `metrics` ; un CEO ne
  peut pas tirer le `source_code`.)
- Chaque appel est tracé via `X-Paperclip-Run-Id`.

## 6. Exemples de contexte assemblé

**CEO sur l'epic « facturation »** → injecté : vision, métriques, résumé
d'archi, flux facturation, ticket(s) backlog. **Pas** de code. Peut, à la
demande, demander la carte des modules (`on_demand`).

**DEV sur un ticket « ajouter TVA »** → injecté : résumé d'archi, carte des
modules de facturation, **extraits de code reliés au ticket**, le ticket cadré.
Peut tirer les flux (`on_demand`) si besoin.

## 7. Extensibilité

- **Surcharges** par société puis par projet (merge en cascade sur `defaults`).
- **Rôles optionnels** (`ARCHITECTE_DOC`, `QA`, `DEVOPS`) : ajout d'entrées
  `roles:` sans changer le moteur.
- Ajout d'un **nouveau niveau** = nouvelle clé + un `fetch_level` correspondant
  (E1) ; rétro-compatible (absent ⇒ `none`).

## 8. Points à vérifier sur le fork

- Comment le **rôle** d'un agent est disponible au moment du build du contexte.
- Le `heartbeat-context` est-il extensible par **hook/plugin** ou faut-il
  modifier le cœur ?
- Mécanisme d'**enregistrement des skills** par rôle (E3) et de refus d'accès.

## 9. Critères d'acceptation (E2)

- Un agent CEO ne reçoit **jamais** `source_code` en injection et se voit
  **refuser** un `get_file`.
- Un agent DEV reçoit du code **ciblé sur son ticket**, pas tout le repo.
- Le contexte injecté **respecte** `budget_tokens` (troncature par priorité).
- Modifier `context-policy.yaml` change le contexte **sans** redéploiement de
  code (rechargement de config).
