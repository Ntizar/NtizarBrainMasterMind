---
name: hermes-plugin-backsearch
description: "Usa a buscar en el web pasado sin fuga temporal."
version: "2.0.0"
author: Mastermind (stars-explorer)
license: MIT
metadata:
  hermes:
    tags: [hermes, plugin, busqueda-web, point-in-time, backtest, research, reproducibilidad]
    related_skills: [hermes-agent, blocked-page-recovery, research-grounded-citations]
---

# BackSearch — Búsqueda web point-in-time para Hermes Agent

**Repo:** https://github.com/NousResearch/hermes-plugin-backsearch (38⭐, Python, MIT, oficial de NousResearch, verificado 2026-09-07 contra README real)

## Cuándo usar

Usar cuando haga falta buscar o citar información web **tal como era en una fecha dada** sin que se cuele evidencia posterior: backtests, research histórico con fecha de corte, reproducir una decisión pasada, o verificar "qué estaba publicado el día X". No usar para búsqueda web normal (para eso basta `web_search`).

## Qué es

Plugin de Hermes Agent que da **búsqueda y fetch web condicionados por fecha** sobre un archivo de noticias congelado (BackSearch de General Reasoning). Cada petición lleva `as_of`: el buscador solo devuelve documentos *crawlados* en o antes de esa fecha, y el fetch devuelve el texto tal como se archivó entonces. Misma query + mismo `as_of` = mismos resultados para siempre.

**Caso de uso:** backtests de forecasting, investigación cuantitativa, entornos RL, benchmarks reproducibles — cualquier tarea donde la evidencia posterior a la fecha de corte NO debe filtrarse. En Mastermind: investigar "qué se sabía del mercado X en marzo" sin contaminación de noticias posteriores.

## Los dos tools

| Tool | Qué hace |
|---|---|
| `backsearch` | Búsqueda híbrida sobre el corpus congelado: `query`, `as_of`, opcionales `k`, `allowed_domains`/`blocked_domains` |
| `backfetch` | Texto de una página desde la última captura ≤ cutoff: `url`, `as_of`, opcional `prompt` para resumen enfocado |

Ambos están gated en `OPENREWARD_API_KEY`: sin key configurada NUNCA llegan al schema del modelo → footprint cero de tools.

## Instalación

```bash
# 1. Clonar al directorio de plugins de Hermes
#    (~/.hermes/plugins; en esta máquina Windows: %LOCALAPPDATA%\hermes\plugins)
git clone https://github.com/NousResearch/hermes-plugin-backsearch.git ~/.hermes/plugins/backsearch

# 2. Activar
hermes plugins enable backsearch

# 3. API key — se factura contra saldo prepago OpenReward (https://openreward.ai/, formato or_...)
echo 'OPENREWARD_API_KEY=or_...' >> ~/.hermes/.env
```

Las sesiones nuevas recogen los tools automáticamente cuando la key existe.

## Semántica clave (pitfalls)

- **`as_of` gatea por `crawl_date`, no por la fecha de publicación que declara el artículo.** Una página archivada por primera vez después del cutoff jamás se devuelve, aunque afirme ser anterior — así se garantiza cero fuga post-cutoff en un backtest.
- **Ventana del archivo (preview):** dominios de noticias, diciembre 2025 – julio 2026. Un `as_of` fuera de la ventana devuelve lista vacía (no error) — vacío ≠ fallo.
- **Facturación por request exitosa:** fetch sin captura ≤ cutoff → soft 404 y NO cuesta; saldo agotado → 402 con mensaje accionable.
- **Fetch capped a 15K chars** — para artículos largos pasar `prompt` y pedir resumen enfocado en vez del texto completo.
- `OPENREWARD_SEARCH_URL` sobreescribe la URL base (`https://search.openreward.ai`) para testing/self-routing.
- El repo vino primero como PR #71207 a hermes-agent; extrajo a plugin standalone por la política de que integraciones de servicios de terceros se distribuyen como plugins, no como core.

## Tests / verificación

```bash
# Sin red ni key, desde el venv de un checkout de hermes-agent
# (HERMES_AGENT_REPO si no está en ~/.hermes/hermes-agent):
python -m pytest tests/ -q
```

Verificar que el plugin está vivo: `hermes plugins list | grep backsearch` y que la key está en el `.env` del home activo.

## Origen

Aprendido del ciclo stars-explorer de David (Ntizar) — batch 2026-09-07. Registry: `NousResearch/hermes-plugin-backsearch` → category `tool`.
