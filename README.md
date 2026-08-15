# mcp-graph-store

**Gerenciado automaticamente** pelo servidor MCP [graphifyy-mcp-persistent-based-context](https://github.com/pedrothor/graphifyy-mcp-persistent-based-context). **Não edite arquivos aqui à mão.**

Este repositório é um **data store** versionado de grafos de conhecimento extraídos de outros repos GitHub. Cada `graph.json.gz` é o output determinístico da lib [graphifyy](https://pypi.org/project/graphifyy/) rodando sobre a AST (tree-sitter) do repo alvo.

## Estrutura

```
.
├── README.md
├── .gitignore              # ignora artefatos internos do graphify
├── index.json              # metadados leves de todos os repos (~KB por entrada)
└── repos/
    └── {owner}__{repo}/
        ├── graphify-out/
        │   └── graph.json.gz     # grafo comprimido (networkx JSON)
        └── meta.json             # url, commit_sha, extracted_at_utc, num_nodes, num_links
```

## Como um repo é adicionado

O MCP tem uma tool `index_repo(url)` que:

1. Faz clone efêmero do url em `%TEMP%`
2. Roda `graphify extract --code-only` (sem LLM, 100% AST local)
3. Comprime `graph.json` em `graph.json.gz`
4. Copia para `repos/{slug}/graphify-out/`
5. Regenera `index.json`
6. `git commit` e `git push` aqui

## Como um repo é removido

Tool `remove_repo(slug)` do MCP: apaga a pasta + regenera index + push.

## Como consultar

Não é feito diretamente aqui — o MCP mantém um clone local e responde consultas do Claude Code (ou qualquer cliente MCP) via as tools:

- `list_repos()`, `describe_repos()`, `get_repo_summary(repo)`
- `search_symbol(pattern, repo?)` (cross-repo se `repo=None`)
- `get_node(repo, id)`, `get_neighbors(repo, id, depth)`, `shortest_path(repo, src, tgt)`
- `list_communities(repo, top_n)`

## Formato do `graph.json.gz`

Formato JSON do networkx (`node-link`):

```json
{
  "directed": true,
  "multigraph": true,
  "nodes": [
    {"id": "...", "label": "...", "source_file": "...", "source_location": "L42",
     "file_type": "code", "community": 3, "_origin": "ast"}
  ],
  "links": [
    {"source": "...", "target": "...",
     "relation": "calls|contains|imports|references|uses|inherits|imports_from|method",
     "confidence": "EXTRACTED|INFERRED", "confidence_score": 1.0}
  ]
}
```

## Licença dos grafos

Cada `graph.json.gz` é derivado do código do repo `url` (em `meta.json`). O grafo é uma representação estrutural (símbolos + relações) — não contém o código-fonte em si. Ainda assim, respeite a licença do repositório original.
