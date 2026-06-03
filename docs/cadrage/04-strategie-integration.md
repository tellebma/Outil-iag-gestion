# 04 — Stratégie d'intégration

> **Décision : Option A — forker Paperclip et étendre dedans.**
> La couche *code-aware* exige des changements **côté serveur** (stockage de
> l'index, injection dans le `heartbeat-context`) qu'un plugin externe ne
> permet pas. On capitalise sur tout l'acquis de Paperclip (auth, budgets,
> tickets, worktrees, heartbeats, gouvernance).

## 1. Statut

⏳ **Le fork n'est pas encore créé** (action côté utilisateur). Tant qu'il
n'existe pas, on **ne touche pas** au code de Paperclip. Ce document prépare le
terrain pour démarrer **immédiatement** une fois le fork disponible.

## 2. Logistique du fork (à trancher)

Deux montages possibles :

- **(a) Repo de fork dédié** — forker `paperclipai/paperclip` dans un repo à
  part, et garder `Outil-iag-gestion` comme repo de **pilotage/cadrage** (docs,
  specs, suivi). ✅ Reçoit facilement les mises à jour upstream (`git remote add
  upstream`). ✅ Sépare clairement notre extension du moteur.
- **(b) Intégration dans `Outil-iag-gestion`** — importer le code Paperclip
  dans ce repo. ✅ Tout au même endroit. ⚠️ Plus difficile de suivre l'upstream.

**Recommandation : (a)**, pour pouvoir `merge`/`rebase` les évolutions de
Paperclip sans douleur. À confirmer.

> Rappel licence : Paperclip est **MIT** → fork et modification autorisés, à
> condition de conserver la mention de licence d'origine.

## 3. Architecture de Paperclip (rappel, à vérifier sur le fork)

Monorepo **TypeScript** (pnpm), **PostgreSQL**, UI **React**. Packages connus :
`server/`, `ui/`, `cli/`, `packages/`, `skills/`, `.agents/`, `.claude/`.

Flux d'un **heartbeat** : identité (`GET /api/agents/me`) → inbox
(`/api/agents/me/inbox-lite`) → **checkout** d'un ticket
(`POST /api/issues/{id}/checkout`) → **contexte** (heartbeat-context API) →
exécution (via **adapter**) → mise à jour (`PATCH /api/issues/{id}`) →
délégation (création de sous-tickets). Chaque appel mutant porte
`X-Paperclip-Run-Id` (traçabilité).

## 4. Carte des points d'extension (préliminaire)

Où brancher chaque epic. **À valider en lisant le code du fork.**

| Epic | Point d'extension Paperclip | Nature du changement |
|------|-----------------------------|----------------------|
| **E1** Ingestion & index | Nouveau package `packages/code-index` + **migration PostgreSQL** (table d'index + colonnes `pgvector`) | Ajout (peu invasif) |
| **E2** Contexte par rôle | Config de politique (matrice rôle×niveau) + couche de sélection | Ajout + lecture du rôle d'agent |
| **E3** Skills de récupération | Système de **company skills** (`POST /api/agents/{id}/skills/sync`) → skills `search_code`, `get_file`, `get_module_map`… | Ajout de skills + contrôle d'accès par rôle |
| **E4** Injection contexte | **Hook dans le builder du `heartbeat-context`** (server-side) | Modification ciblée du serveur |
| **E5** Cadrage assisté | **Routines** + **AGENTS.md templates** (rôles CEO/CTO/PO/DEV/SEO-GEO/UI-UX) | Ajout de templates/routines |
| **E6** Session Claude | **Adapter** type « Claude Code » étendu (transcript, diff, `--resume`) | Nouvel adapter + modèle « session » |
| **E7** Cockpit UI | Package `ui/` (React) | Ajout de vues |

### Points sensibles à vérifier sur le fork
- Comment le **rôle** d'un agent est exposé au moment du build du contexte
  (nécessaire pour appliquer la matrice du doc 02).
- Le `heartbeat-context` est-il **extensible** proprement (hook/plugin) ou
  faut-il modifier le cœur ?
- PostgreSQL supporte-t-il **`pgvector`** dans leur setup Docker (pour les
  embeddings) ?
- Format exact des **adapters** et du **workspace/worktree** (pour E6).
- Système de **plugins** : jusqu'où permet-il d'éviter de modifier le cœur ?

## 5. Plan de démarrage (dès le fork disponible)

1. Cloner le fork, lancer le projet en local (docker + dev server).
2. **Vérifier la carte du §4** en lisant le code → figer le doc 04.
3. **E1** : migration `pgvector` + package `code-index` + pipeline d'ingestion.
4. **E2** : matérialiser la matrice rôle×niveau en config.
5. **E3 ∥ E4** : skills de récupération + hook d'injection.
6. Démo de bout en bout (exemple du doc 02, §7).

## 6. Ce qu'on peut faire **sans** le fork (en attendant)

- ✅ Cadrage (ce dossier).
- ⬜ **Spec de l'index de code** (E1) : schéma de données, champs, format des
  niveaux de contexte, stratégie d'embeddings — indépendant de Paperclip.
- ⬜ **Spec de la politique de contexte** (E2) : format de configuration de la
  matrice rôle×niveau.
- ⬜ **Prototype d'ingestion autonome** : un petit module qui indexe un repo et
  produit la carte des modules + résumés, testable hors Paperclip, puis branché
  sur le fork le moment venu.
