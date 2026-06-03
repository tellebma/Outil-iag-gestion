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

**Décision : (a) repo de fork dédié.** Montage retenu :

| Repo | Rôle | Contenu |
|------|------|---------|
| **`Outil-iag-gestion`** (celui-ci) | **Pilotage / produit** — *conservé* | Cadrage, specs, plan, suivi, et les **modules fork-indépendants** (ex. prototype d'ingestion) |
| **Fork de Paperclip** (repo séparé, à créer) | **Moteur** | Code Paperclip + notre extension *code-aware* |

`Outil-iag-gestion` **n'est pas abandonné** : il reste la **source de vérité
produit** (vision, décisions, specs) et l'atelier des composants réutilisables.
Le fork **consomme** ces specs/modules (import direct, sous-module git, ou
package publié).

> ✅ Reçoit facilement l'upstream (`git remote add upstream …` sur le fork).
> ✅ Sépare clairement notre extension du moteur Paperclip.

> Rappel licence : Paperclip est **MIT** → fork et modification autorisés, à
> condition de conserver la mention de licence d'origine.

## 3. Architecture de Paperclip — **vérifiée** sur le fork

> Source : lecture du fork `tellebma/paperclip-plusplus` @ `70b1a91`.

Monorepo **TypeScript** (pnpm). **ORM = Drizzle**. DB : **PostgreSQL 17** en
prod (docker), **PGlite embarqué** en dev local par défaut. UI **React**.

**Packages** (`packages/`) :

| Package | Rôle |
|---------|------|
| `db` | Schéma Drizzle, migrations, clients DB |
| `shared` | Types API, validators, constantes |
| `adapter-utils` | Utilitaires adapters (skills, env vars…) |
| `adapters/*` | Implémentations d'adapters (`claude-local`, `cursor`…) |
| `plugins` | SDK de plugins + exemples |
| `skills-catalog` | Catalogue de skills (CLI + métadonnées) |
| `mcp-server` | Intégration **MCP** (exposition d'outils aux agents) |

**Serveur** (`server/src/`) : `routes/` (REST), `services/` (logique métier,
dont `heartbeat.ts`), `adapters/` (registry + plugin-loader), `middleware/`.

**Flux d'un heartbeat** : la route `GET /api/issues/:id/heartbeat-context`
(`server/src/routes/issues.ts:2188`) assemble le contexte ; l'exécution
(`server/src/services/heartbeat.ts:7016`, `executeRun`) parse
`run.contextSnapshot`, récupère l'agent (`getAgent(run.agentId)` → `agent.role`)
et passe le tout à l'**adapter** via `AdapterExecutionContext`.

## 4. Carte des points d'extension — **vérifiée**

Légende impact : 🟢 ajout orthogonal · 🟡 petite modif du cœur · 🔴 chantier infra.

| Epic | Point d'ancrage (fichier réel) | Impact |
|------|--------------------------------|--------|
| **E1** Index de code | Nouveau schéma `packages/db/src/schema/code_index.ts` (+ export `index.ts`, `pnpm db:generate`) + service `server/src/services/code-index.ts` | 🟢 |
| **E1bis** Embeddings | `pgvector` **absent** du `docker/docker-compose.yml` (postgres:17-alpine) + dev en PGlite → init script / image custom (ou extension vector PGlite) | 🔴 |
| **E2** Contexte par rôle | Service `context-policy` lisant `agent.role` (`packages/db/src/schema/agents.ts:20`), appliqué **dans** `server/src/routes/issues.ts:~2220` avant `res.json()` | 🟡 |
| **E3** Skills de récupération | **Voie recommandée : `packages/mcp-server`** → outils `search_code`/`get_file`/`get_module_map` exposés en MCP. Alt. : adapter custom enregistré via `registerServerAdapter()` (`server/src/adapters/registry.ts:619`) | 🟢 |
| **E4** Injection contexte | Même hook que E2 (le `heartbeat-context` n'a **pas** de hook de plugin → modif ciblée de la route) | 🟡 |
| **E5** Cadrage assisté | Templates `AGENTS.md` + routines, rôles CEO/CTO/PO/DEV/SEO-GEO/UI-UX | 🟢 |
| **E6** Session Claude | Wrapper `packages/adapters/claude-contextual/` autour de `claude-local` (`packages/adapters/claude-local/src/server/execute.ts`) + `sessionManagement` | 🟢/🟡 |
| **E7** Cockpit UI | Package `ui/` (React) | 🟢 |

### Constats clés (issus de la lecture du code)
- **Pas de hook d'enrichissement du `heartbeat-context`** dans le système de
  plugins → E2/E4 imposent une **petite modification du cœur** (route issues).
  ➡️ **Confirme que le fork (Option A) était nécessaire** ; un plugin externe
  n'aurait pas suffi.
- Le **rôle d'agent existe déjà** (`agents.role`, défaut `"general"`) → il faut
  juste une **convention de nommage** des rôles (CEO/CTO/PO/DEV/SEO_GEO/UI_UX).
- **Aucun contrôle d'accès aux skills par rôle** aujourd'hui → à créer (E2/E3).
- **`mcp-server`** existe → voie idéale pour les skills de récupération (E3).
- **pgvector** = vrai chantier infra (E1bis) : décider PGlite-vector vs image
  Postgres avec `pgvector`.

## 5. Plan de démarrage (dès le fork accessible en écriture)

1. Installer & lancer le fork en local (`pnpm i`, dev server, DB).
2. **Convention de rôles** : mapper CEO/CTO/PO/DEV/SEO_GEO/UI_UX sur `agents.role`.
3. **E1** : schéma `code_index` (Drizzle) + service `code-index` + pipeline
   d'ingestion (cf. [spec E1](../specs/E1-index-de-code.md)).
4. **E1bis** : trancher l'infra embeddings (pgvector) et la câbler.
5. **E2/E4** : service `context-policy` (cf. [spec E2](../specs/E2-politique-de-contexte.md))
   branché dans `routes/issues.ts` (heartbeat-context).
6. **E3** : outils `search_code`/`get_file` via `mcp-server`.
7. Démo de bout en bout (exemple du doc 02, §7).

## 6. Statut d'accès (cette session)

- ✅ Fork **lisible/clonable** (réseau public OK).
- ❌ Fork **non poussable** depuis cette session : le proxy git répond
  *« repository not authorized »* et l'outil `add_repo` est indisponible ici.
  Le périmètre est verrouillé sur `Outil-iag-gestion`.
- ➡️ **Pour coder ET pousser sur le fork** : l'ajouter aux repos/sources de
  l'environnement sur claude.ai/code, ou démarrer une session ciblant
  `paperclip-plusplus`.
