# Batch stars-explorer — 2026-09-07

**Repos explorados:** 2 (quedan **0 pendientes** en el registry — el queue de stars está a cero tras este batch).

| Repo | ⭐ | Decisión | Skill |
|---|---|---|---|
| `NousResearch/hermes-plugin-backsearch` | 38 | CREATE (tool) | `autonomous-ai-agents/hermes-plugin-backsearch` v2.0.0 |
| `TheSmokeDev/hermes-talk` | 35 | CREATE (tool) | `autonomous-ai-agents/hermes-talk-realtime-voice` v2.0.0 |

## Por qué criar skills con <1000 estrellas

El criterio general pide >1000⭐, pero ambos son **plugins del propio runtime Hermes** que ejecuta Mastermind: conocimiento de alta frecuencia de uso para David aunque la comunidad sea joven (repos de agosto 2026, ambos activos, push ≤4 días). El valor no es el patrón arquitectónico sino la operatividad directa: voz dúplex con delegación locutada y búsqueda web point-in-time son capacidades que este agente puede instalar y usar.

## Verificación de dedup (ChromaDB)

- backsearch → mejor score 0.61 (quantstats-pro) por temas de backtest, pero **ningún skill cubre point-in-time search** → no duplicado.
- hermes-talk → `media/voicebox` 0.81 en la misma liga de voz, pero voicebox es TTS/clonación local (síntesis) y hermes-talk es conversación speech-to-speech con tools + delegación (otro dominio). Decisión: CREATE, con la distinción explícita en la sección "Cuándo usar" del nuevo skill para que no se confundan al consultar.

## Lecciones del batch

- `skill_angles` volvió a fallar: a backsearch le asignó `pattern-ai/ml` (absurdo — es una herramienta de búsqueda). Confirmado: decidir siempre desde el README real, nunca desde el campo heurístico.
- El script de exploración ya había marcado ambos como `explored` con `category: pending`; el agente los promovió a `tool` + `skill_created: true`.

## Estado del queue

Registry completo: **todas las stars de David procesadas** a fecha 2026-09-07. Los próximos batches del scout solo tendrán trabajo cuando David dé nuevas stars.
