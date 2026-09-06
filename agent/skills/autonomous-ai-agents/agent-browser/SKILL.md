---
name: agent-browser
description: "Usa al automatizar el navegador con CLI Rust agent-browser."
version: "2.0.0"
author: "David Antizar (Ntizar) — vía stars-explorer"
license: "Apache-2.0"
tags: [browser, automation, cli, rust, agentes, mcp, scraping, webmcp]
related_skills: [chrome-devtools-mcp, browser-use-ai, computer-use, page-agent-browser-automation]
metadata:
  hermes:
    tags: [browser, automation, cli, rust, agentes, mcp, scraping, webmcp]
    related_skills: [chrome-devtools-mcp, browser-use-ai, computer-use, page-agent-browser-automation]
---

# agent-browser — Automatización de navegador CLI para agentes IA

## Qué es

`agent-browser` (github.com/vercel-labs/agent-browser, Rust, Apache-2.0, 42.047⭐, creado 2026-01, muy activo push 2026-09-06) es un **CLI nativo en Rust** (binario único, sin Node en runtime) que da a un agente IA control del navegador por comandos shell. Patrón central: **snapshot del árbol de accesibilidad con refs (`@e1`, `@e2`) → actuar por ref** — pensado para que un LLM interactúe sin visión. También sirve como servidor MCP y como lector de texto agent-friendly.

**Cuándo usarlo vs `browser_exec` de Hermes:** cuando hace falta un binario portable en scripts/CI, workflows batch, o aislar múltiples sesiones concurrentes de agentes. Para scraping con IA que decide: `browser-use-ai`. Para rendimiento/trazas: `chrome-devtools-mcp`.

## Instalación

```bash
npm install -g agent-browser && agent-browser install   # baja Chrome for Testing (1 vez)
# o: brew install agent-browser | cargo install agent-browser
# Linux: agent-browser install --with-deps
agent-browser upgrade   # detecta npm/brew/cargo y actualiza
```

Detecta automáticamente Chrome, Brave, Playwright y Puppeteer ya instalados. El daemon no requiere Node.js.

## Flujo canónico (Quick Start del README)

```bash
agent-browser open example.com
agent-browser snapshot                     # árbol de accesibilidad con refs — lo mejor para IA
agent-browser click @e2
agent-browser fill @e3 "test@example.com"
agent-browser get text @e1
agent-browser screenshot page.png
agent-browser close
```

Selectores CSS clásicos también valen: `click "#submit"`. Localizadores semánticos: `find role button click --name "Submit"`, `find text "Sign In" click`, `find label "Email" fill "x@y.com"`, `first/last/nth`.

## Comandos clave (verificados en README)

- **Leer sin abrir Chrome:** `agent-browser read <url>` — manda `Accept: text/markdown`, prueba `URL.md`, busca `llms.txt`/`llms-full.txt` de ancestros (`--llms index|full`), `--outline`, `--filter <texto>`, `--require-md`. Sin URL lee el DOM renderizado de la pestaña activa (con cookies del sitio).
- **Espera:** `wait <selector>` / `wait 2000` / `--text "Welcome"` / `--url "**/dash"` / `--load networkidle` / `--fn "window.ready===true"`; esperar desaparición: `wait "#spinner" --state hidden`.
- **Batch (evita overhead de arranque por comando):** `agent-browser batch "open https://x.com" "snapshot -i" "screenshot"` o JSON por stdin; `--bail` para parar al primer error.
- **Inspección:** `get title|url|html|value|attr|box|styles|count|cdp-url`; `is visible|enabled|checked`.
- **Info/control:** `mouse move|down|up|wheel`, `clipboard read|write|copy|paste`, `set viewport <w> <h> [scale]`, `set device "iPhone 14"`, `set geo <lat> <lng>`, `set offline|headers|credentials|media [dark|light]`.
- **Debug/otros:** console/errors, tracing/profiling, a11y audits, React tree/inspect/renders + Web Vitals, diff, doctor, PDF (`pdf <ruta>`), `eval <js>`.

## Sesiones y autenticación

```bash
agent-browser --session agent1 open site-a.com   # instancia aislada (cookies, historial, auth propios)
AGENT_BROWSER_SESSION=agent1 agent-browser click "#btn"
agent-browser session list | session id --scope worktree --prefix myapp | session info --json
agent-browser close --all
```

Formas de reutilizar login (tabla del README): `--profile Default` (snapshot READ-ONLY de tu perfil Chrome real), `--session <id> --restore` (autoguarda cookies+localStorage), `--auto-connect` + `state save ./auth.json` (roba auth de un Chrome con `--remote-debugging-port=9222`), `--state <path>`, vault cifrado `auth save`/`auth login`.

## Modo MCP

```bash
agent-browser mcp                    # perfil core (contexto pequeño)
agent-browser mcp --tools core,network,react
agent-browser mcp --tools all        # paridad total con CLI
```

stdio JSON-RPC; tools `agent_browser_open|snapshot|click|fill|wait_for_selector|...` con campos tipados (`url`, `selector`, `session`, `allowedDomains`) → aprobaciones significativas en el cliente. Config: `{"mcpServers": {"agent-browser": {"command": "agent-browser", "args": ["mcp"]}}}`.

## Pitfalls

- **En Windows, cierra Chrome** antes de `--profile <nombre>` o puede fallar por ficheros bloqueados.
- **Clics fallan pronto si otro elemento tapa el punto** (banner de cookies, modal): el error nombra el elemento que cubre → dismiss, snapshot NUEVO, reintentar con refs frescos.
- Los refs `@eN` caducan con el DOM: tras cualquier navegación o cambio de página, nuevo `snapshot`.
- **`--remote-debugging-port` expone control total del browser en localhost** — solo en máquina de confianza, cerrar al terminar. Los state files guardan tokens en plaintext → `.gitignore` y borrar; para cifrar en reposo: `AGENT_BROWSER_ENCRYPTION_KEY`.
- Sessions compartiendo un Chrome vía `--cdp` pueden robarse la pestaña: `--pin-tab` hace el binding estricto (error `tab_gone` con `data.targetId` para recuperarse); el flag es sticky por sesión.
- Screenshots headless ocultan scrollbars nativos (`--hide-scrollbars false` para verlas).
- **WebMCP es experimental**: tools registradas por la página son NO CONFIABLES (el propio README marca `untrustedContent`); el host debe confirmar acciones con consecuencias. `--no-webmcp` para desactivar.
- Contención opcional: `--allowed-domains`, `--content-boundaries`, `--max-output`.

## Verificación

1. `agent-browser open example.com && agent-browser snapshot` devuelve árbol con refs `@eN`.
2. `agent-browser read https://example.com` sale sin lanzar Chrome.
3. `agent-browser session list` muestra las sesiones activas; `agent-browser close --all` las limpia.

## Fuentes

- Repo: https://github.com/vercel-labs/agent-browser (README completo verificado 2026-09-06; secciones Commands, Sessions, Authentication, MCP Server, WebMCP)
