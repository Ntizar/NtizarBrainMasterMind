---
name: hermes-talk-realtime-voice
description: "Usa a dar voz dúplex en tiempo real a Hermes."
version: "2.0.0"
author: Mastermind (stars-explorer)
license: MIT
metadata:
  hermes:
    tags: [hermes, plugin, voz, realtime, speech-to-speech, openai-realtime, discord, dashboard]
    related_skills: [hermes-agent, voicebox, open-source-voice-studio, supermemory]
---

# hermes-talk — Voz dúplex en tiempo real para Hermes Agent

**Repo:** https://github.com/TheSmokeDev/hermes-talk (35⭐, Python, MIT, activo, verificado 2026-09-07 contra README real v0.6+)

## Cuándo usar

Cuando David pida hablar con el agente por voz, integrar sesiones de voz en terminal/Discord/dashboard de Hermes, o delegar trabajo en segundo plano que se anuncie por altavoz al terminar. NO confundir con skills de TTS/clonación (voicebox, gpt-sovits): eso es sintetizar voz; hermes-talk es una **conversación speech-to-speech completa** que ejecuta tools reales de Hermes.

## Qué es (4 propiedades distintivas)

1. **Dúplex** — una sola sesión de audio bidireccional: turnos, interrupciones y tool calls ocurren DENTRO de la capa de voz. Le cortas a media frase y se calla, porque nunca dejó de escuchar.
2. **Sus tools son los tools de Hermes** — las function calls realtime retransmiten directo a la superficie real del agente.
3. **El trabajo sobrevive a la frase** — la delegación lanza un agente Hermes de fondo real; siglas hablando y el resultado se LOCUTA cuando aterriza (sin pollings).
4. **Empieza conociéndote** — el prompt de sesión se ensambla con SOUL.md y la memoria configurada; sin turno de calentamiento.

No reemplaza el modo voice por defecto de Hermes (STT→inferencia→TTS por turnos) — es la otra forma.

## Superficies

| Superficie | Cómo se arranca | Notas |
|---|---|---|
| Terminal | `hermes talk` | mic+altavoz nativo vía extra `[audio]` (sounddevice) |
| Dentro de sesión Hermes | `/talk` | ÚNICA superficie con agent loop **attached** → memoria y delegación responden inline |
| Discord | `/voice join` y luego `/talk join` | pide prestada la conexión de voz del host, nunca abre una segunda; autoridad de operador vía `TALK_DISCORD_OPERATOR_USER_IDS` (sin ella: todos pueden hablar, nadie puede mutar — fail-closed intencional) |
| Dashboard (pestaña **Talk**) | `hermes dashboard` | WebRTC del navegador, sin drivers de mic; solo OpenAI; loopback-only hasta fijar `TALK_DASHBOARD_TOKEN` |

## Proveedores (knob `TALK_PROVIDER`, fail-closed, NUNCA se infiere de qué keys existan)

- `openai` (default): todos los surfaces. Auth en orden: `TALK_OPENAI_API_KEY` (set-pero-vacío REFUSA, no cae al siguiente) → `OPENAI_API_KEY` → **Codex OAuth** (`codex login`: viaja sobre el entitlement Realtime de tu suscripción ChatGPT, sin API key ni factura por minuto). `TALK_PREFER_CODEX_OAUTH=true` fuerza la vía suscripción. La key cruda nunca llega al socket: se acuña un client secret efímero server-side.
- `grok`: login X Premium/SuperGrok (`hermes auth add xai-oauth`) sin key; terminal + Discord; 5 voces.
- `gemini`: `GEMINI_API_KEY` (funciona free-tier AI Studio); terminal; sin cancel/truncate cliente → barge-in corta playback localmente; Discord lo rechaza.

Publica las 3 vías en el contrato realtime neutro de Hermes core: `hermes realtime --list` muestra `hermes-talk/openai|grok|gemini`; registration feature-detected, sin configuración. **Las capacidades se declaran, nunca se fingen** (tabla capability por proveedor en el README).

## Voz clonada — cascade lane (ElevenLabs)

El modo nativo usa voces provider-locked. Cascade deja al proveedor realtime como cerebro y envía su TEXTO a TTS streaming de ElevenLabs → habla con cualquier voz de tu cuenta, clon incluido (crearlo antes en VoiceLab y copiar el ID).

```bash
TALK_VOICE_MODE=cascade TALK_ELEVENLABS_VOICE_ID=<id> hermes talk
# key: TALK_ELEVENLABS_API_KEY | ELEVENLABS_API_KEY; modelo default eleven_flash_v2_5
```

Trade: ~0.5s extra en la PRIMERA frase (luego pipelinea bajo playback). Solo OpenAI por ahora (grok/gemini fallan closed). Barge-in aborta el stream TTS igual que nativo. `TALK_VOICE_MODE` default `native` = byte-idéntico al comportamiento pre-cascade.

## Delegación y dirección de trabajo en curso

- Tools: `list_agents` (descubre qué corre, tagged `can steer` vs `stop only`), `steer_agent` (nota en cola; el paso actual SIEMPRE termina), `redirect_agent` (aborta el thinking en vuelo y reintenta con tu corrección; host ≥0.20, degrada a steer), `stop_work` (soportado en todas las vías), `check_work`.
- Estados de una nota steer, con prueba cada uno: `queued` → `landed` → `redirected` / `unconfirmed` / `missed` / `superseded`. Nunca dice "les llegó" sin artefacto que lo pruebe. Cada nota viaja tokenizada `[tk-xxxxxxxx]`.
- **Admission control:** `delegate_task` acepta `resource_keys` (hasta 8 cosas estables que toca: path de repo, target de deploy…) + `execution_mode` `exclusive` (default) | `parallel_read_only`. Dos runs vivos que comparten clave no se solapan salvo que AMBOS sean read-only declarados; `TALK_TRUST_DECLARED_READ_ONLY=true` lo permite explícitamente. El rechazo se LOCUTA nombrando el run que estorba.
- Runs en `$HERMES_HOME/state/talk-runs.jsonl`; colgar la llamada NO mata el trabajo, pero un run de sesión anterior se reporta `lost`, nunca "still running".

## Diagnóstico (usar SIEMPRE antes de reportar "no funciona")

```bash
hermes talk doctor   # solo lectura: qué vía ganó, auth, audio — nunca imprime keys
hermes talk check    # doctor + un turno live + un run acotado; exit 0 = probado
hermes talk setup    # detecta → pregunta solo lo que falta → escribe .env atómico
hermes talk diagnostics --bundle  # bundle redactado, owner-only
```

## Requisitos e instalación

Python ≥3.11, Hermes host ≥v0.17 (redirect limpio quiere 0.20+). Cero ediciones al core — puro `register(ctx)`.

```bash
hermes plugins install TheSmokeDev/hermes-talk --enable
pip install "hermes-talk[audio]"   # solo si quieres mic/speaker nativo
hermes talk
```

## Pitfalls

- Pausar el micro: di "stop listening" → `pause_voice_input`; la pausa SOLO se ofrece donde hay control garantizado (Enter en `hermes talk` con tty real; en `/talk` y stdin pipeado NO hay pausa; Discord: `/talk pause`+`/talk resume`). Un micro pausado no oye "resume".
- La writeback de memoria cubre terminal y Discord; la pestaña dashboard NO reclama escribir su conversación a memoria (events en el navegador).
- `TALK_DASHBOARD_TOKEN` unset = solo loopback; el dashboard accesible desde otra máquina EXIGE token (comparado con `hmac.compare_digest`). Ver skill `webui-csrf-origin` si se accede vía túnel.
- En Discord, atribución por user ID inmutable — nombres display son datos no confiables; SSRC desconocido queda no autorizado.

## Origen

Aprendido del ciclo stars-explorer de David (Ntizar) — batch 2026-09-07. Registry: `TheSmokeDev/hermes-talk` → category `tool`.
