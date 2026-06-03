# Spec E1 — Index de code

> Objet : définir **comment on transforme un repo en index exploitable** par les
> agents. Indépendant de Paperclip ; se branchera sur son PostgreSQL et son
> système de projets le moment venu.
>
> Décisions actées : indexation **« totale »** (structure + symboles + résumés +
> **embeddings**) ; documents d'archi/flux **hybrides** (génération depuis le
> code par défaut).

## 1. Portée

- **Entrée** : un repo git (chemin + commit).
- **Sortie** : un index requêtable fournissant les **10 niveaux de contexte** du
  [doc 02](../cadrage/02-modele-de-contexte-par-role.md) (vision/métriques mis à
  part — non dérivés du code).
- **Non couvert ici** : la politique d'accès par rôle → [Spec E2](./E2-politique-de-contexte.md).

## 2. Pipeline d'ingestion

```
repo@commit
   │
   ├─ 1. SCAN        parcours fichiers, filtres (.gitignore, binaires, vendored)
   ├─ 2. PARSE       AST par langage via tree-sitter → symboles
   ├─ 3. ANALYSE     modules, dépendances (imports), points d'entrée
   ├─ 4. RÉSUMÉ      résumés LLM : fichier → module → architecture → flux
   ├─ 5. EMBEDDINGS  chunking + vecteurs (code, résumés, docs)
   └─ 6. STOCKAGE    PostgreSQL (+ pgvector)
```

- **Idempotent & incrémental** : on indexe par `commit_sha`. Réindexation =
  diff de commits ; on ne retraite que les fichiers dont le `content_hash` a
  changé (parse + résumé + embeddings ciblés).
- **Multi-langage** : tree-sitter (grammaires par langage). Un fichier de
  langage non supporté est indexé au niveau fichier (résumé + embedding) sans
  symboles.

## 3. Modèle de données

> Schéma logique (noms indicatifs ; types PostgreSQL ; `vector` = pgvector).

```
code_project(id, repo_url, default_branch, last_commit_sha, indexed_at, status)

code_file(id, project_id→, path, language, size_bytes, content_hash,
          summary TEXT, indexed_at)

code_symbol(id, file_id→, kind ENUM(function|class|interface|type|const|…),
            name, signature, doc TEXT, start_line, end_line)

code_module(id, project_id→, path, name, kind ENUM(package|dir|service|…),
            summary TEXT, is_entrypoint BOOL)

code_dependency(id, project_id→, from_module_id→, to_module_id→,
                kind ENUM(import|call|runtime))

code_doc(id, project_id→, type ENUM(architecture|flow|module),
         title, body MD, source ENUM(generated|manual|hybrid),
         status ENUM(draft|validated), ref_path NULLABLE, updated_at)

code_chunk(id, file_id→ NULLABLE, doc_id→ NULLABLE, owner_kind, content TEXT,
           start_line, end_line, token_count)

code_embedding(id, chunk_id→, model, dim, vector VECTOR, created_at)
```

Notes :
- **`code_doc.source`** porte la décision hybride : `manual` fait autorité,
  `generated` est le défaut, `hybrid` = généré puis raffiné (validé par un agent
  Architecte/Doc). `ref_path` = fichier source (ex. `docs/ARCHITECTURE.md`)
  quand il en existe un, pour réimport/priorité.
- `code_chunk` est l'unité d'embedding (un fichier/doc → N chunks).

## 4. Stratégie d'embeddings

- **Quoi** : (a) chunks de code, (b) résumés de fichiers, (c) résumés de
  modules, (d) corps des `code_doc`. Permet une recherche sémantique aussi bien
  sur le code brut que sur les vues de haut niveau.
- **Chunking** : par frontières de symboles quand dispo (fonction/classe), sinon
  fenêtre glissante (~300–500 tokens, chevauchement ~15 %).
- **Index** : `pgvector` avec index **HNSW** (cosine). Dimension selon le modèle
  d'embedding retenu (paramétrable, stockée par ligne → multi-modèle possible).
- **Provider** : à câbler sur le provider déjà configuré dans le fork (éviter
  une dépendance nouvelle si Paperclip en expose un).

## 5. Production des niveaux de contexte

Comment chaque niveau du doc 02 est matérialisé à partir de l'index :

| Niveau (doc 02) | Source dans l'index |
|---|---|
| 3. Architecture | `code_doc(type=architecture)` (hybride) |
| 4. Flux métier | `code_doc(type=flow)` (hybride) |
| 5. Carte des modules | `code_module` + `code_dependency` |
| 6. Code source | `code_file` / `code_symbol` + recherche sémantique (`code_chunk`/`code_embedding`) |
| 8. Parcours UX | `code_doc(type=flow)` filtré « parcours » + routes/écrans détectés |
| 9. Composants UI | `code_module`/`code_file` filtrés front (heuristique par langage/dossier) |
| 10. Contenu & SEO/GEO | routes publiques, métadonnées, pages détectées (heuristique par framework) |

> Niveaux **1 Vision**, **2 Métriques**, **7 Backlog** ne sont **pas** dérivés du
> code : ils viennent de Paperclip (objectifs, coûts/tokens, tickets).

## 6. API de récupération (contrat logique)

Indépendante du transport (deviendra endpoints serveur + skills). Esquisse :

```
search_code(query, {top_k, scope?}) -> [{path, lines, snippet, score}]
get_file(path, {range?})            -> {path, language, content, symbols[]}
get_module_map({root?})             -> {modules[], dependencies[]}
get_architecture_summary()          -> code_doc(architecture)
get_flows({filter?})                -> [code_doc(flow)]
```

Tout résultat est **traçable** (rattaché au `run_id` du heartbeat — cf. règle
`X-Paperclip-Run-Id`).

## 7. Génération hybride des docs (archi/flux)

1. À l'ingestion, si **aucun** `code_doc` `manual` ne couvre le sujet → on
   **génère** (`source=generated`, `status=draft`) à partir de la carte des
   modules + résumés.
2. Un fichier manuel détecté (`docs/ARCHITECTURE.md`, `*.flow.md`…) est importé
   comme `source=manual`, `status=validated` et **prime**.
3. Un agent **Architecte/Doc** (ou le CTO) peut raffiner un `generated` →
   `hybrid` + `validated`.

## 8. Points à vérifier sur le fork

- Disponibilité de **`pgvector`** dans le setup Docker de Paperclip.
- Provider d'**embeddings** réutilisable côté Paperclip.
- Modèle « projet » de Paperclip : relier `code_project` au projet Paperclip.
- Accès au **worktree/commit** d'un projet pour déclencher l'ingestion.

## 9. Critères d'acceptation (E1)

- Indexer un repo réel produit `code_file`/`code_symbol`/`code_module`/
  `code_dependency` non vides.
- Une réindexation après 1 commit ne retraite que les fichiers modifiés.
- `search_code("…")` renvoie des extraits pertinents (sémantique + chemin).
- `get_architecture_summary()` renvoie un doc non vide (généré si absent).
