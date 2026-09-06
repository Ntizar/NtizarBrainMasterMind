---
name: addyosmani-agent-skills
description: Catálogo de Addy Osmani de skills engineering para AI coding agents — 8 slash commands mapeados al SDLC (spec → plan → build → test → review → ship).
category: development
---

# Addy Osmani — Agent Skills

## Qué es

Repositorio `addyosmani/agent-skills` (92K+⭐, verificado 2026-09-05) que packagea skills de ingeniería para que agentes IA sigan workflows consistentes en todas las fases del desarrollo.

## Estructura SDLC

8 slash commands que mapean al ciclo de vida completo:

| Comando | Fase | Principio |
|---------|------|-----------|
| `/spec` | Definir | Spec before code |
| `/plan` | Planificar | Small, atomic tasks |
| `/build` | Construir | One slice at a time |
| `/test` | Verificar | Tests are proof |
| `/review` | Revisar | Improve code health |
| `/webperf` | Auditoría | Measure before optimize |
| `/code-simplify` | Simplificar | Clarity over cleverness |
| `/ship` | Producción | Faster is safer |

Flujo: `Idea → Spec → Code → Test → QA → Go Live` con `/build auto` para ejecución autónoma.

## Patrón clave

Los skills encode workflows, quality gates, and best practices que senior engineers usan. Cada command activa las skills correctas automáticamente.

## Relevancia para Mastermind

- Validación del enfoque de skills procedurales
- Referencia para mejora del SOUL.md y orquestación
- `/build auto` como analogía del patrón `mastermind-orchestration`
- `/code-simplify` como práctica transversal

## Pitfalls

- Este repo NO es un framework ejecutable — es un catálogo de documentos/skills
- No confundir con las skills de Hermes (que sí son ejecutables con `skill_view()`)
- Los slash commands son para herramientas como Claude Code, Codex, Cursor — no para Hermes directamente

## Comparativa de alternativas (actualizado 2026-09-06)

**obra/superpowers (282K⭐, MIT, Jesse Vincent / Prime Radiant)** es el referente del mismo espacio y aquí SÍ es ejecutable — a diferencia de este catálogo de Addy Osmani. Pipeline SDLC completo con skills auto-triggered: brainstorming (socrático, spec en chunks) → using-git-worktrees → writing-plans (tareas de 2-5 min con rutas y verificación) → subagent-driven-development (dos revisiones: spec + calidad) → test-driven-development (RED-GREEN-REFACTOR estricto, borra código escrito antes que tests) → requesting-code-review → finishing-a-development-branch.

**Instalación nativa en Hermes Agent** (verificada del README):

```bash
hermes plugins install obra/superpowers --enable
```

Sin hook post-compaction en Hermes: tras una compactación muy larga hay que abrir sesión nueva si los skills dejan de disparar.

**Decisión de dedup:** superpowers > addyosmani (ejecutable, multi-harness con 13 integraciones, workflow con checkpoints humanos que coincide con el human-loop de Mastermind; sus skills `subagent-driven-development`/`dispatching-parallel-agents` son la versión formal de `mastermind-orchestration`). Queda cubierto por esta sección en vez de skill aparte para no fragmentar el dominio.

## Referencias

- Repo: https://github.com/addyosmani/agent-skills
- Superpowers: https://github.com/obra/superpowers
- Licencia: MIT
- Temas: agent-skills, claude-code, codex, cursor, skills