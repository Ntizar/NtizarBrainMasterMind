---
name: chrome-devtools-mcp
description: "Usa al auditar rendimiento y depurar webs con DevTools MCP."
version: "2.0.0"
author: "David Antizar (Ntizar) — vía stars-explorer"
license: "Apache-2.0"
tags: [mcp, browser, chrome, devtools, rendimiento, depuracion, puppeteer, testing]
related_skills: [native-mcp, dogfood, webgl-headless-verification, agent-browser]
metadata:
  hermes:
    tags: [mcp, browser, chrome, devtools, rendimiento, depuracion, puppeteer, testing]
    related_skills: [native-mcp, dogfood, webgl-headless-verification, agent-browser]
---

# Chrome DevTools MCP — DevTools para agentes de código

## Qué es

`chrome-devtools-mcp` (github.com/ChromeDevTools/chrome-devtools-mcp, Google, TypeScript, Apache-2.0, 51.131⭐, push 2026-09-06) es el **servidor MCP oficial de Chrome DevTools**: da a un agente IA (Claude, Cursor, Copilot, Hermes) control e inspección de un Chrome vivo — automatización fiable vía puppeteer, trazas de rendimiento accionables, red, consola con stack traces source-mapped y auditorías Lighthouse. También tiene **CLI** para uso sin MCP.

**Cuándo usarlo:** auditoría de rendimiento de dashboards/HTML de David (DataHubEspana, SolMAD, Kit72h), depuración de errores de consola/red en producción, verificación visual antes de desplegar. Para automatización ligera sin rendimiento: `agent-browser`. Para QA exploratorio de metodología: `dogfood`.

## Instalación / Arranque

Config de cliente MCP (stdio vía npx):

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

- Requiere Node.js LTS + Chrome stable (soporte oficial SOLO Chrome y Chrome for Testing; Chromium tipo Edge/Brave no garantizado).
- `--slim --headless`: modo básico solo con tareas elementales de navegador (menos tools, menos contexto).
- El servidor NO abre Chrome al conectar: la browser se lanza al primer tool que la necesita.
- Test de humo (prompt del propio README): `Check the performance of https://developers.chrome.com`

## Grupos de tools (verificado en docs/tool-reference.md)

| Grupo | Tools clave |
|-------|-------------|
| Input | `click`, `drag`, `fill`, `fill_form`, `hover`, `press_key`, `type_text`, `upload_file`, `click_at`, `handle_dialog` |
| Navegación | `navigate_page`, `new_page`, `list_pages`, `select_page`, `close_page`, `wait_for` |
| Rendimiento | `performance_start_trace`, `performance_stop_trace`, `performance_analyze_insight` (insights accionables, no solo números) |
| Red | `list_network_requests`, `get_network_request` |
| Depuración | `evaluate_script`, `list_console_messages`, `get_console_message`, `take_screenshot`, `take_snapshot`, `lighthouse_audit`, `screencast_start/stop` |
| Memoria | `take_heapsnapshot`, `close_heapsnapshot`, `compare_heapsnapshots`, `get_heapsnapshot_retainers/retaining_paths/dominators`, `query_heapsnapshot_objects` |
| Emulación | `emulate`, `resize_page` |
| Extensiones | `install_extension`, `list_extensions`, `reload_extension`, `trigger_extension_action`, `uninstall_extension` |

## Configuración avanzada (docs/advanced-usage.md)

- **Conexión a Chrome ya abierto** (imprescindible para reutilizar login): `--autoConnect`, o manual con `--browser-url=http://127.0.0.1:9222` lanzando Chrome con `--remote-debugging-port=9222 --user-data-dir=<dir no default>` (Chrome LO EXIGE por seguridad — nunca usa tu perfil por defecto con el puerto abierto). En Windows: `"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir="%TEMP%\chrome-profile-stable"`.
- **Sesiones concurrentes:** `--experimentalPageIdRouting` expone `pageId` en los tools; combinar con `--isolated` para perfiles temporales.
- **Datos de usuario:** profile por defecto en `%USERPROFILE%\.cache\chrome-devtools-mcp\chrome-profile`.
- **Depuración de Chrome en Android** soportada (ver guía advanced).
- Referencia de subagente de navegador integrado: Gemini CLI browser agent (el propio README recomienda construir sobre esto).

## Pitfalls

- **Telemetría ON por defecto:** Google recoge stats de uso (éxito de invocaciones, latencia, entorno). Desactivar con `--no-usage-statistics` o env `CHROME_DEVTOOLS_MCP_NO_USAGE_STATISTICS` (también desactiva si hay `CI`). Es INDEPENDIENTE de las métricas de Chrome: salir del opt-out de Chrome no afecta aquí.
- **CrUX:** las tools de rendimiento envían URLs de trace a la API CrUX de Google (datos de usuarios reales). `--no-performance-crux` para desactivar.
- **Chequeo de actualizaciones npm:** `CHROME_DEVTOOLS_MCP_NO_UPDATE_CHECKS` para silenciar.
- **Exposición de contenido:** el MCP expone TODO lo que ve el navegador al cliente — nunca iniciar sesión en banca/servicios sensibles mientras un agente está conectado.
- **Fijar versión:** `chrome-devtools-mcp@latest` es lo recomendado oficialmente; en pipelines reproducibles pinear versión.
- **APIs de terceros** solo con Chrome/Chrome for Testing; en entornos Linux headless verificar que Chrome (no Chromium del paquete) está instalado.

## Verificación

1. `npx -y chrome-devtools-mcp@latest --version` responde.
2. Prompt "Check the performance of https://developers.chrome.com" → debe abrir Chrome, grabar trace y devolver insights.
3. Si se usa `--browser-url`, confirmar que el Chrome objetivo arrancó con `--remote-debugging-port` y `--user-data-dir` distinto del default.

## Fuentes

- Repo: https://github.com/ChromeDevTools/chrome-devtools-mcp (README + docs/tool-reference.md + docs/advanced-usage.md + docs/configuration.md, verificados 2026-09-06)
