---
name: graphify-codebase-graph
description: "Usa a generar un grafo de conocimiento desde código."
version: "2.0.0"
author: Mastermind (stars-explorer)
license: Apache-2.0
metadata:
  hermes:
    tags: [knowledge-graph, graphrag, tree-sitter, ast, code-analysis, mcp, rag, codigo]
    related_skills: [rag-knowledge-base, semantica, memory-context-engine]
---

# Graphify — Codebase → Grafo de Conocimiento Consultable

**Repo:** https://github.com/Graphify-Labs/graphify (115K⭐, Python, verificado 2026-09-06 contra README real v8)

## Qué es y por qué importa

Convierte cualquier codebase (con docs, esquemas SQL, configs, PDFs) en un grafo de conocimiento **queryable**. Diferenciadores clave frente a RAG vectorial:

- **Parsing AST determinista y local** con tree-sitter (~40 lenguajes) — cero créditos LLM al construir el grafo, nada sale de la máquina.
- **Sin vector store**: se consulta el grafo, no similarity de chunks.
- **Cada arista lleva tag de confianza**: `EXTRACTED` (explícito en el fuente) vs `INFERRED` (derivado por resolución).
- **Más allá del código**: docs, PDFs, imágenes y vídeo/audio van al mismo grafo (el pase semántico de docs/media sí usa backend LLM, opcional).
- Benchmarks del README (mismo harness, n=300): LOCOMO recall@10 = 0.497 (mem0 0.048, supermemory 0.149); LongMemEval-S QA 76%.

## Instalación (verificada del README)

```bash
# ⚠️ El paquete PyPI es graphifyy (doble y). El comando sigue siendo graphify.
uv tool install graphifyy        # recomendado (entorno aislado)
pipx install graphifyy
# uvx: hay que nombrar el paquete, no el comando:
uvx --from graphifyy graphify install
```

Registrar el skill del asistente: `graphify install` (acepta `--project` y `--platform codex|claude|...`; escribe `.claude/skills/graphify/SKILL.md` o `.agents/skills/...`).

## Uso

```bash
graphify .                     # en CLI; /graphify . dentro del agente
# salida: graphify-out/ → graph.html (viz interactiva), GRAPH_REPORT.md, graph.json

graphify query "qué conecta auth con la BBDD"   # subgrafo acotado por pregunta
graphify path "UserService" "DatabasePool"       # ruta más corta entre dos conceptos
graphify explain "APIRouter"                     # nodo + conexiones con tags

graphify extract ./raw --code-only    # solo código, AST local, SIN API key
graphify . --update                   # re-extraer solo ficheros cambidos
graphify . --cluster-only             # re-clustering (Leiden) sin re-extraer
graphify . --wiki                     # wiki markdown para crawl de agentes
graphify . --neo4j                    # genera cypher.txt para Neo4j
graphify hook install                 # auto-rebuild en git commit
graphify merge-graphs a.json b.json
```

Como servidor MCP para acceso repetido por tool-calls:

```bash
python -m graphify.serve graphify-out/graph.json                 # stdio (default)
python -m graphify.serve graphify-out/graph.json --transport http --port 8080 --api-key "$SECRET"  # compartido equipo
```

Herramientas MCP: `query_graph`, `get_node`, `get_neighbors`, `shortest_path`, `list_prs`, `get_pr_impact`, `triage_prs`.

## Conceptos del grafo

- **God nodes**: conceptos más conectados (por dónde fluye todo); `--exclude-hubs 99` excluye super-hubs utilitarios del ranking.
- **Comunidades**: subsistemas vía Leiden, etiquetadas sin LLM.
- **Aristas**: `calls` / `imports` / `inherits` / `mixes_in` resueltas entre ficheros.
- **Rationale nodes**: comentarios `# NOTE:` / `# WHY:` y citas ADR/RFC se vuelven nodos enlazados al código.
- **Work memory**: `graphify save-result --question Q --answer A --outcome useful|dead_end|corrected` + `graphify reflect` → `LESSONS.md` con notas recency-weighted.

## Pitfalls (del propio README, verificados)

- **`pip install graphify` NO** — el paquete es `graphifyy`; paquetes `graphify*` ajenos en PyPI no están afiliados.
- En **PowerShell**: `graphify .` (sin `/` inicial — es separador de rutas).
- `graphify: command not found` tras uv/pipx → `uv tool update-shell` / `pipx ensurepath` + terminal nueva.
- Mezclar entornos: el skill resuelve Python desde `graphify-out/.graphify_python`; si apunta a otro entorno → `ModuleNotFoundError` (uv tool/pipx lo evitan).
- Filtros: `.graphifyignore` (sintaxis .gitignore, se fusiona con el `.gitignore` del repo y gana el primero en conflictos); `--no-gitignore` en `extract` para incluir código generado.
- Headless/CI (`graphify extract`): requiere backend vía env vars (`--backend openai|claude|ollama|gemini|azure|bedrock|deepseek|kimi`; `OPENAI_BASE_URL` sirve para endpoints OpenAI-compatible tipo vLLM/llama.cpp — funcionaría con NaN.builders).

## Cuándo usarlo en Mastermind

- Auditar proyectos HTML grandes (audit-html-project) o repos ajenos: grafo en vez de leer ficheros uno a uno.
- Alimentar un pipeline de conocimiento de proyecto sin coste de embeddings.
- Comparativa: `semantica` (grafo contexto+provenance general) y `rag-knowledge-base` (RAG vectorial) — graphify es específico de **inteligencia sobre código**, local y determinista.

## Verificación

`graphify . --no-viz` sobre una carpeta pequeña → comprobar `graphify-out/graph.json` + `GRAPH_REPORT.md`; luego `graphify explain "<nodo>"` responde con conexiones taggeadas.
