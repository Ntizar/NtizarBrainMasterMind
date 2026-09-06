# 2026-09-06 — Scout de stars: la era del navegador como herramienta de agente

Batch de 3 stars (quedan 2 pendientes de procesar). Dos de tres repos son infraestructura de
browser-automation para agentes IA con +40.000⭐ cada uno — la categoría se está consolidando
muy rápido y antes no teníamos ningún skill que la cubriera (dogfood es metodología QA,
browser-use-ai es scraping con IA, page-agent es in-page; ninguno cubre CLI/MCP de DevTools).

## Skills creados

1. **mcp/chrome-devtools-mcp** — ChromeDevTools/chrome-devtools-mcp (Google, 51.131⭐, TS).
   El servidor MCP oficial de DevTools: trazas de rendimiento con insights accionables,
   red, consola source-mapped, Lighthouse, heapsnapshots. Utilidad directa para auditar
   los dashboards HTML de David (DataHubEspana, SolMAD) antes de desplegar. Ojo: telemetría
   de Google ON por defecto (`--no-usage-statistics`) y CrUX envía URLs de trace.

2. **autonomous-ai-agents/agent-browser** — vercel-labs/agent-browser (42.047⭐, Rust, enero 2026).
   CLI nativo: snapshot de accesibilidad con refs `@eN` → actuar por ref (patrón sin visión),
   batch, sesiones aisladas, reuso de perfiles Chrome, servidor MCP con perfiles de tools,
   `read` con soporte llms.txt. Es el patrón que Hermes ya aplica en browser_exec.

## Skip

- **rarf/hermes-quota-plugin** (61⭐): plugin de terceros para la barra de quota de Hermes
  Desktop. Interesante como referencia (pitfall de symlinks de plugins por perfil), pero sin
  patrones reutilizables más allá de `hermes-agent/references/desktop-plugins.md`.

## Lecciones

- Confirmado de nuevo: los `skill_angles` del script siguen siendo ruido ("pattern-real-time"
  para un MCP de DevTools). Decidir siempre desde el README real vía `gh api`.
- Dedup ChromaDB: mejor score 0.78 (dogfood) y 0.86 para el ángulo CLI/browser — pero al leer
  dogfood es QA exploratorio metodológico, no cobertura de estas herramientas → crear fue correcto.
