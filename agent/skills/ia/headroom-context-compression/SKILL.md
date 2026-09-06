---
name: headroom-context-compression
description: "Usa a comprimir tokens de contexto antes de llegar al LLM."
version: "2.0.0"
author: Mastermind (stars-explorer)
license: Apache-2.0
metadata:
  hermes:
    tags: [context-engineering, compresion, tokens, proxy, mcp, rag, coste]
    related_skills: [token-tracking, memory-context-engine, dspy]
---

# Headroom — Capa de Compresión de Contexto para Agentes IA

**Repo:** https://github.com/headroomlabs-ai/headroom (69K⭐, Python+Rust, Apache-2.0, verificado 2026-09-06 contra README real)

## Qué es

Comprime salidas de herramientas, logs, ficheros y chunks RAG **antes** de que lleguen al LLM: ~20% menos tokens en salidas de coding agents, 60-95% en JSON repetitivo, con las mismas respuestas. Corre local (los datos no salen de la máquina) y es **librería + proxy + servidor MCP** a la vez. Encaja con token-tracking de Mastermind: reduce coste real de sesiones Hermes/NaN.

Diferenciadores (README, sección "Compared to"): scope total de contexto (no solo historia), proxy local reversible — los originales se guardan en CCR y el modelo los recupera con `headroom_retrieve`.

## Arquitectura (pipeline por request)

```
CacheAligner → ContentRouter → {SmartCrusher (JSON), CodeCompressor (AST), Kompress-v2-base (prosa, HF)}
```
- **ContentRouter** detecta el tipo de contenido y elige el compresor.
- **SmartCrusher**: JSON universal — conserva items de error, valores fuera de rango estadístico y bordes primero/último (por varianza de campos, no por keywords).
- **CodeCompressor**: AST-aware Python/JS/TS/Go/Rust/Java/C/C++/Perl.
- **CacheAligner**: marca contenido volátil que rompería el prefijo KV-cache del proveedor; NUNCA reescribe prompts.
- **Live-zone**: solo se comprimen bytes nuevos; el prefijo congelado queda byte-idéntico → la cache del proveedor sobrevive.
- Compresión <1 ms (0.21 ms p50 en 10K tokens JSON) — no añade latencia.

## Instalación y modos de uso (verificados del README)

```bash
pip install "headroom-ai[all]"          # Python — incluye la CLI `headroom`
uv tool install --python 3.13 "headroom-ai[all]"   # CLI aislada
npm install headroom-ai                 # SOLO SDK TypeScript, sin CLI
```

```bash
headroom proxy --port 8787     # proxy drop-in, cero cambios de código
headroom wrap claude           # envuelve un agente (arranca proxy + Serena + config)
headroom deploy                # despliegue local llave en mano
headroom doctor                # health check del routing
headroom dashboard             # ahorros en vivo
headroom savings               # nº de ahorro sobre TU tráfico
```

Librería Python:

```python
from headroom import compress
result = compress(messages, model="gpt-4o")   # result.messages, result.tokens_saved, result.compression_ratio
```

Integraciones: Anthropic/OpenAI SDK (`withHeadroom(new OpenAI())`), LangChain (`HeadroomChatModel`), LiteLLM (`HeadroomCallback`), ASGI (`CompressionMiddleware`), multi-agente (`SharedContext`), MCP (`headroom mcp install`).

## Reducción de tokens de SALIDA (proxy, opcional)

```bash
export HEADROOM_OUTPUT_SHAPER=1
headroom proxy --port 8787
```
- **Verbosity steering**: añade nota "sé breve" al final del system prompt (la cache sigue impactando).
- **Effort routing**: baja `reasoning_effort` (OpenAI) / `thinking.budget_tokens` (Anthropic) en turnos rutinarios post-tool-result.
- `headroom learn --verbosity` infiere la concisión deseada de sesiones pasadas.

## Pitfalls

- **Python 3.13 para el dólar**: el dashboard "Proxy $ Saved" usa LiteLLM, que no se instala en 3.14+. Con 3.14 los tokens se ahorran pero la cifra $ queda en 0.
- npm `headroom-ai` es solo librería — no provee el comando `headroom`.
- Clientes MCP sin PATH heredado (Codex): poner la ruta absoluta de `command -v headroom` en la config TOML.
- Prosa ya densa y payloads cortos apenas se comprimen (bloque bajo `min_input_words` vuelve byte-idéntico) — el ahorro escala con la repetitividad.
- ONNX (detección Magika + relevancia embebida) requiere AVX2 en x86; sin él cae a BM25/heurísticas (no rompe).
- Beacon de telemetría ON por defecto: `HEADROOM_BEACON=off` o `DO_NOT_TRACK=1`.
- En redes con inspección SSL corporativa puede fallar el build maturin/rustup: instalar Rust antes o usar wheel: `pip install --only-binary headroom-ai headroom-ai`.

## Cuándo usarlo en Mastermind

- Sesiones Hermes largas con outputs de tool masivos (scraping, dumps API ESIOS/GTFS) → `headroom proxy` + `--1m`-style savings.
- Dashboards/pipelines JSON pesados: el caso SRE del README pasa 55.957 → 24.340 tokens (57%).
- Comparativa: `memory-context-engine`/`supermemory` = memoria y retrieval; `dspy` = optimización de prompts; **headroom = compresión del contexto ya assembled** — son complementarios, no sustitutos.

## Verificación

`headroom doctor` → routing OK; correr un `compress()` sobre un JSON de 10K tokens propio y comprobar `compression_ratio` y que las líneas de error sobreviven byte a byte.
