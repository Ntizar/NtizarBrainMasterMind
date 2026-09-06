# 2026-09-06 — Batch stars: tríada de infraestructura para agentes IA

Batch nocturno (run 57): 3 repos gigantes restantes del registry, todos >69K⭐.

| Repo | ⭐ | Veredicto | Destino |
|---|---|---|---|
| obra/superpowers | 282K | dedup → upgrade | comparativa en `addyosmani-agent-skills` (misma liga: frameworks de skills SDLC; este además ejecutable vía `hermes plugins install obra/superpowers --enable`) |
| Graphify-Labs/graphify | 115K | skill nuevo | `ia/graphify-codebase-graph` v2.0.0 — grafo de conocimiento desde código, AST tree-sitter local, sin vector store ni LLM al indexar |
| headroomlabs-ai/headroom | 69K | skill nuevo | `ia/headroom-context-compression` v2.0.0 — compresión de contexto (proxy/librería/MCP), complementa a token-tracking |

## Aprendizajes clave

1. **El ángulo de `skill_angles` del script falló estrepósamente en este batch** (superpowers→"crm-erp-patterns", graphify→"voice-ai-integration"): los heurísticos de topics no distinguen dominos. El análisis manual del README real sigue siendo obligatorio — nunca fiarse del angle.
2. **Superpowers se instala nativamente en Hermes** (`hermes plugins install obra/superpowers --enable`) — candidato real a plugin del sistema Mastermind. Su pipeline brainstorm→worktree→plan→subagent-TDD→review coincide casi 1:1 con nuestros niveles de ejecución y el human loop. Caveat del README: Hermes no tiene hook post-compaction; tras compactar largo, sesión nueva.
3. **Graphify**: paquete PyPI es `graphifyy` (doble y). Cero coste de embeddings al indexar código — encaja con audit-html-project y auditorías de repos.
4. **Headroom**: compresión reversible (CCR) <1 ms; útil para sesiones con dumps grandes (ESIOS, GTFS). Caso de prueba: 55.957→24.340 tokens (57%) en debugging SRE.

Registry al día (300 repos explorados), ChromaDB re-indexada (461 skills), push cea6392.
