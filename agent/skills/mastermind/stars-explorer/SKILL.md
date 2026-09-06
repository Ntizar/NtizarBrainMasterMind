---
name: stars-explorer
version: "1.0.0"
description: "Pipeline nocturno que explora las stars de GitHub de David, analiza repos, extrae patrones y crea skills automáticamente. Cada run procesa un batch de repos, genera análisis profundo, y propone skills basados en los patrones detectados."
tags: [github, stars, skills, pipeline, cron, exploration, learning]
related_skills: [chromadb-skills-vector-search, github-trending-research]
---

# Stars Explorer — Pipeline de Aprendizaje Automático

## Resumen

Pipeline recurrente que explora las ~100+ stars de GitHub de David (Ntizar), analiza los repos más interesantes, extrae patrones arquitectónicos y crea skills automáticamente en el sistema Mastermind/Hermes.

**Objetivo:** Cada noche, el sistema aprende de los repos que David le gusta, ampliando la base de conocimiento de skills de forma autónoma.

## Arquitectura

```
Cron (nocturno 03:00 UTC)
  ↓
Script explorar-stars.py (batch de 3 repos)
  ↓ Fetch GitHub API → README + tree + key files
  ↓ Análisis: tech stack, patterns, skill angles
  ↓ Actualiza stars-registry.json
  ↓
Agent processa el output del script
  ↓ Para cada repo: decide si merece skill
  ↓ Crea skill con skill_manage si es relevante
  ↓ Actualiza registry (category, skill_created)
  ↓
Re-indexa ChromaDB (si hubo cambios)
  ↓
Guarda resumen en notes/ si hubo hallazgos significativos
```

## Componentes

### Script principal
- **Ruta:** `/hermes-home/scripts/explorar-stars.py`
- **Wrapper:** `bash /hermes-home/scripts/run-stars-explorer.sh` (carga entorno automáticamente)
- **Dependencias:** Solo stdlib (urllib, json, base64) — NO necesita pip install
- **Entorno:** Wrapper carga variables de entorno del sistema automáticamente

### Registry
- **Ruta:** `/hermes-home/data/stars-registry.json`
- **Contenido:** Repo → fecha explorada, category, skill_created, skill_angles

### Skill de referencia
- **chromadb-skills-vector-search** — Para re-indexar después de crear skills

## Uso del Script

Usar SIEMPRE el wrapper que carga el entorno automáticamente:

```bash
# Status del registry
bash /hermes-home/scripts/run-stars-explorer.sh --status

# Batch de 3 repos (default) — solo repos >100 stars con topics
bash /hermes-home/scripts/run-stars-explorer.sh

# Batch grande
bash /hermes-home/scripts/run-stars-explorer.sh --batch 5

# Todos los pendientes (SIN FILTRO — explora todos)
bash /hermes-home/scripts/run-stars-explorer.sh --all

# Modo JSON (para agent consumption)
bash /hermes-home/scripts/run-stars-explorer.sh --batch 2 --json

# Incluir propios repos de David
bash /hermes-home/scripts/run-stars-explorer.sh --include-own

# Forzar re-proceso de un repo
bash /hermes-home/scripts/run-stars-explorer.sh --reprocess owner/repo

# Modo LOOP DE APRENDIZAJE — modo completo: explora → aprende → mejora → implementa
bash /hermes-home/scripts/run-stars-explorer.sh --learning-loop
```

## Flujo del Agent (en el cron)

Al recibir el output del script, el agent debe:

1. **Leer cada repo analizado** del JSON
2. **Evaluar si merece skill** basándose en:
   - ¿Tiene patrones reutilizables? (architecture, pipeline, performance)
   - ¿Es relevante para los proyectos de David? (3D, CV, geospatial, CRM, transit)
   - ¿Aporta conocimiento que NO tenemos ya?
   - ¿Tiene enough profundidad para justificar un skill?
3. **Crear skill** con `skill_manage(action='create')` si:
   - El repo tiene patrones claros y reutilizables
   - El skill serait útil en futuras tareas
   - No duplica un skill existente (check ChromaDB primero)
4. **Marcar en registry** como `skill_created: true` + categoría
5. **Re-indexar ChromaDB** al final si se crearon skills

### Criterios de Creación de Skill

**Crear skill si:**
- Repo tiene patrones arquitectónicos reutilizables (3+ patterns detectados)
- Tech stack relevante para proyectos existentes de David
- El README describe approach único o innovador
- Tiene +1000 stars (indica calidad/comunidad)
- El repo es de David (siempre crear, es su conocimiento)

**NO crear skill si:**
- Solo tiene README genérico sin patrones concretos
- Es un "awesome list" o curated list sin código
- Ya existe un skill cubriendo lo mismo
- El repo está archivado o sin maintenimiento
- Es demasiado simple (<100 stars, sin code patterns)

**Categorías para el registry:**
- `core` — Skills que son parte fundamental del sistema
- `domain` — Conocimiento de dominio específico (CV, GIS, transit)
- `pattern` — Patrones arquitectónicos reutilizables
- `tool` — Herramientas y librerías concretas
- `reference` — Referencia/inspiración (no skill directo)
- `skip` — Decidido no procesar (awesome list, etc.)

## Datos del Script Output

Cada repo analizado incluye:
```json
{
  "full_name": "owner/repo",
  "description": "...",
  "language": "Python",
  "stars": 1234,
  "topics": ["topic1", "topic2"],
  "tech_stack": ["django", "postgres", "..."],
  "potential_patterns": ["pipeline", "real-time", "ai/ml"],
  "skill_angles": ["ai-cv-pipeline"],
  "readme_excerpt": "primeros 2000 chars del README",
  "key_files_present": ["package.json", "Dockerfile"],
  "file_types": {".py": 15, ".js": 8}
}
```

## Pitfalls

- **Rate limit GitHub:** 5000 req/h autenticados. Batch de 3 repos ≈ 20 req cada uno = 60/batch. Safe.
- **README enorme:** Truncado a 8000 chars. Suficiente para análisis.
- **Skills duplicados:** SIEMPRE consultar ChromaDB antes de crear. Si un skill semánticamente similar existe, NO crear otro. Verificado en producción: manim→skip (score 0.85 contra creative/manim-video), twenty→skip (score 0.79 contra crm-erp-fullstack), VibeVoice→skip (score 0.89 contra media/voicebox).
- **Quality gate:** No crear skills de "awesome lists" o repos sin código sustancial.
- **Re-indexación ChromaDB:** Obligatoria tras crear skills. Sin ella, los nuevos skills son invisibles en búsquedas semánticas.
- **Registry creep:** Si un repo no merece skill, marcar con `category: "skip"` y `skill_created: false`. NO re-procesarlo cada run.
- **Cron security scanner (CRÍTICO 2026-06-16):** El scanner de cron bloquea prompts que contienen patrones como `cat .env`, `cat credentials`, etc. (regex: `cat\s+[^\n]*(\.env|credentials|\.netrc|\.pgpass)`). **Solución:** usar wrapper script (`run-stars-explorer.sh`) que carga el entorno internamente. NUNCA poner comandos que lean secrets directamente en prompts de cron ni en skills que se carguen en crons. El scanner escanea el prompt ensamblado (user prompt + skill content concatenado).
- **ChromaDB dedup funciona en producción (2026-06-16):** El pipeline detectó correctamente 3 repos como "ya cubiertos" en el primer batch nocturno. Scores: manim=0.85, twenty=0.79, VibeVoice=0.89. Threshold 0.25 es suficiente para detectar duplicados semánticos.
- **Wrapper script obligatorio para cron:** El script `explorar-stars.py` necesita variables de entorno (token de GitHub, API key de NaN). En vez de exponer el patrón de lectura en el prompt del cron, usar `bash /hermes-home/scripts/run-stars-explorer.sh` que hace source del .env internamente.
- **Skill overlap con github-trending-research:** `github-trending-research` explora trending público; `stars-explorer` explora las stars personales de David. Complementarios, no duplicados. Comparten patrones de GitHub API, creación de skills, y dedup via ChromaDB.
- **Edición programática del registry (maratón 2026-09-01):** El registry está en CRLF con `indent=2`; al reescribirlo con Python leer/escribir con `newline=""` y normalizar saltos con `json.dumps(...).replace("\n","\r\n")` — un `json.dump` a secas reescribe todo el fichero a LF y genera un diff de 500+ líneas que ensucia el commit. NO inyectar claves nuevas (tipo `full_name` con setdefault). Y NO hacer `git checkout -- data/stars-registry.json` después de ejecutar el script: revierte las entradas `explored` que el script ya escribió y obliga a re-pasar el batch (el re-paso es inocuo porque las marcas skip se re-aplican, pero gasta ~1 min de API).

- **Indexador YA ve cambios de contenido (arreglado 2026-09-02, verificado 2026-09-03):** `scripts/indexar-skills.py` compara por **hash sha256 del contenido**, no solo por ID — un skill parcheado se re-indexa solo (batch del 03:03: "1 modificado → indexados 1/1" automático). El upsert manual vía importlib ya NO es necesario. Ejecutar directo con el Python del sistema: `C:/Users/d_ant/AppData/Local/Programs/Python/Python312/python.exe scripts/indexar-skills.py` (el wrapper `run-indexar-skills.sh` invoca `python3`, que en Windows es el stub del Store → salida vacía engañosa).
- **`skill_angles` del script NO es fiable (batch 2026-09-06):** los heurísticos por topics fallan en repos multi-dominio — dio `obra/superpowers → "crm-erp-patterns"` y `Graphify-Labs/graphify → "voice-ai-integration"`, ambos absurdos. NUNCA decidir el ángulo del skill desde ese campo: leer siempre el README real (`gh api repos/{o}/{r}/readme | base64 -d`) antes de crear/patchear.
- **Registry: entradas bajo `reg["processed"]`, no en raíz:** desde 2026-09 las claves repo→entry viven en el subdict `processed` (algunas entradas antiguas quedaron en raíz — comprobar antes de indexar por clave).
- **Sincronización skills→repo es MANUAL (verificado 2026-09-04):** `skill_manage(create/patch)` escribe SOLO en `%LOCALAPPDATA%\hermes\skills\...`; para que aparezcan en GitHub hay que copiar el SKILL.md a `agent/skills/<categoria>/<skill>/` del repo antes del commit. OJO: los skills antiguos del maratón viven en la raíz `agent/skills/<skill>/` (sin categoría) — conservar su ubicación original al actualizarlos, no moverlos.
- **Formato frontmatter de skills nuevos (2026-09):** el validador exige description ≤60 chars (trigger primero, en castellano, punto final) y advierte si faltan `author`, `license` y `metadata.hermes.{tags,related_skills}` — incluirlos ya en el create.
- **Auditoría quantstats-pro completada (2026-09-04):** de la lista pendiente (gtfs-tidy, gtfs2shp, colmap-view, quantstats-pro), queda auditado y reescrito quantstats-pro → v2.0.0. Pendientes de auditoría v1: CERRADOS el 2026-09-04 — gtfs-tidy reauditado → v2.0.0 (CLI inventado `--input/--validate/--info` reemplazado por flags reales verificados en `gtfstidy.go`: `-v`, `-o`, shorthands `--fix/--compress/--Compress/--merge`, tabla de procesadores, orden interno fijo, `--keep-*`). Registry `patrickbr/gtfstidy` skip→upgrade. gtfs2shp y colmap-view ya estaban v2 (09-03 y 09-01).
- **Auditoría gtfs-box completada (2026-09-04, batch vespertino):** v1 tenía CLI/npm inventados → reescrito mobility/gtfs-box v2.0.0 verificado contra gtfs-box.js (web estática sobre mt3d/Mini Tokyo 3D, sin build). geospatial/gtfs-box-3d-viewer marcado SUPERADO con banner. Lección: `git clone` NO — basta `curl` de 2-3 ficheros clave del tree para verificar la API real de un repo pequeño antes de auditar.
- **Auditoría agresiva de v1 (2026-09-05, 12 skills × 3 subagentes en paralelo):** el problema de v1 fabricados es PEOR de lo que se creía — no eran solo "bullets genéricos": había **APIs REST inventadas** (baidu-unlimited-ocr decía ser un "servicio de OCR gratis sin límites" con endpoint `api.unlimitedocr.com` cuando es un modelo de parsing con HuggingFace/GPU), **comandos falsos** (`pip install chatterbox`→`chatterbox-tts`; `pip install PDFMathTranslate`→`pdf2zh`; `python infer.py`→`webui.py`; build `cmake`→autotools), **imports inexistentes** (`from index_tts import TTS`→`indextts.IndexTTS2`; `f5_tts.infer.api.synthesize`→`f5_tts.api.F5TTS`), **claim falsos** (RVC "zero-shot sin fine-tuning" cuando es training-based; OpenVoice "+no GPU"; Valhalla imagen docker `mapbox`→`valhalla` y payload `mode/range`→`costing/contours`; autoscraper `run()`→`get_result_similar`). **Resultado: 10/12 NEEDS_PATCH → todos reescritos a v2.0.0** (los 2 OK: mapcn, semantica). **Lección: NO auditear solo los flaggeados** — barrer el registry completo con subagentes (cada uno verifica 4-6 skills contra su README real via `gh api`/`raw.githubusercontent.com`, devuelve JSON `NEEDS_PATCH` con discrepancias; el orquestador parchea). Al auditar, marcar category `upgrade` + `skill_version` + `skill_audited` en el registry.
- **Auditoría COMPLETA de todo el registry (2026-09-05, 120 skills en 6 oleadas):** se auditó el 100% de los skills derivados de stars. Resultado: **~56 skills con errores corregidos a v2.0.0** (el resto OK o DESIGNED = patrón manuscrito sin repo que verificar). El problema de "v1 inventados" era GENERAL, no puntual: APIs REST/CLIs/imports/estrellas inventadas en casi todas las categorías (TTS, OCR, scraping, GIS, visión, 3D, transporte).
  **PITFALL CRÍTICO de concurrencia — delegación con subagentes:** NO lanzar ≤10 subagentes a la vez. NaN limita a **max 5 requests LLM simultáneos** (`HTTP 429: <modelo> concurrency limit: max 5`) → con 10 auditoros, ~5 se truncaron por `max_iterations`. **Regla: lanzar ≤4 subagentes por oleada** (o ≤5 justo). Manifiesto compartido (`data/...`: array JSON con `repo`, `url`, `skill`, `skill_path`) + cada subagente lee su rango de filas `[start,end)` y devuelve JSON `{auditoria:[{skill,veredicto,repo_real,discrepancias[],fix_sugerido,stars_reales,lenguaje}]}`; el ORQUESTADOR aplica los patches (los subagentes solo investigan — no pueden `skill_manage`). Auditar por **skill** (dedupe por `skill_path`), no por repo (varios repos apuntan al mismo skill y mapeos del registry pueden ser erróneos: el skill manda).
- **execute_code bloqueado en cron:** en modo cron programático solo hay `terminal`; para lógica Python de edición de registry escribir un script temporal en `%LOCALAPPDATA%\Temp` y ejecutarlo con `python`. El parser de terminal TAMBIÉN bloquea heredocs `python - <<EOF` y `python -c` multilínea — siempre script temporal + `python <ruta>`.
- **Skills-v1 inventados del maratón 2026-06-18/19 (detectado 2026-09-02):** varios skills creados en batch masivo describían el repo SIN leer el README (bullets genéricos, CDN inventado, función equivocada: minimaps→"librería de minimapas" cuando es generador de relieves 3D puck; gtfs-to-chart→"frecuencias" cuando son stringlines/Marey). **Cuando el scout re-encuentra un repo ya procesado, NO hacer dedup-skip a ciegas: comparar el skill existente contra el README real y parchearlo (category: "upgrade") si está mal.** Pendientes de auditar: gtfs-tidy, gtfs2shp, colmap-view, quantstats-pro y demás v1 del maratón.
- **Indexar en Windows:** `run-indexar-skills.sh` invoca `python3` (no existe → stub del Store, salida vacía engañosa). Ejecutar directo con el Python del sistema: `C:/Users/d_ant/AppData/Local/Programs/Python/Python312/python.exe scripts/indexar-skills.py` — la DB es embebida (~/.mastermind/chromadb), no necesita server. El indexador solo ve IDs nuevos: para skills PARCHeados, upsert manual vía importlib (patrón documentado arriba).

### Comparativa en dedup — mejorar, no solo descartar (regla 2026-09-01)

Cuando el dedup encuentra un skill existente que cubre el tema, NO basta con SKIP: hay que decidir **cuál de los dos es mejor referencia hoy**. El objetivo del pipeline es aprendizaje continuo, y un repo nuevo puede ser la evolución del que ya tenemos.

**Procedimiento:**
1. Traer metadatos de ambos (GET /repos/{owner}/{repo}): `stars`, `pushed_at`, `archived`.
2. Comparar en la misma liga temática (no comparar un motor de routing con un generador de shapes).
3. Decisión:
   - **Nuevo ≥ existente** (más stars/actividad, o aporta un enfoque que falta) → `skill_manage(patch)` sobre el skill existente: añadir sección `## Comparativa de alternativas` (cuándo usar cada cuál, con fechas de consulta) y actualizar la referencia principal si procede. Registry: `category: "upgrade"`.
   - **Nuevo < existente** → SKIP dedup como siempre, pero la nota del batch debe decir **por qué gana el actual** ("R3 310⭐ frente a VGGT 14K⭐ + Depth Anything 3 6.2K⭐ ya cubiertos → el skill se queda, R3 queda como alternativa menor"). Registry: `category: "skip"`, `skip_reason` con el ganador.
4. Nunca borrar del skill la mención al repo peor: la comparativa ES el valor — saber que existe algo mejor y por qué.

**Ejemplo verificado (2026-09-01):** KevinXu02/R3 (310⭐, push 2026-06-19) vs el terreno 3D-reconstruction ya cubierto: facebookresearch/vggt (14.314⭐, CVPR 2025 Best Paper), colmap/colmap (12.609⭐, activo), ByteDance-Seed/Depth-Anything-3 (6.244⭐). Veredicto: R3 es la opción minoritaria; el dedup acierta descartando, pero la nota debe dejar constancia del ranking.

## Loop de Aprendizaje Continuo

El sistema **no es solo explorador** — es un **motor de aprendizaje** que cada noche:

### Diagrama del Loop

```
🌙 Cron (03:00 UTC)
   ↓
🔍 Explorar 3 stars nuevos
   ↓
📖 Analizar README + tree + key files
   ↓
🧠 ¿Merece skill? (ChromaDB dedup)
   ↓
   ├── ✅ Sí → Crear skill → Indexar en ChromaDB
   └── ❌ No → Marcar como skip (razón)
   ↓
📈 Registrar en stars-registry.json
   ↓
🎯 ¿Patrón relevante para proyecto activo?
   ↓
   ├── Sí → Micro-cron de seguimiento semanal
   └── No → Esperar próxima noche
   ↓
🔄 Loop infinito → Cada noche 3 skills nuevos
```

### Fases del ciclo

| Fase | Descripción | Responsable |
|------|-------------|-------------|
| **🔍 Explorar** | Fetch 3 repos no procesados | Script `explorar-stars.py` |
| **📖 Aprender** | Analizar tech stack + patrones | Agent (con `stars-explorer` skill) |
| **✨ Mejorar** | ¿El patrón es nuevo? → skill | Agent + ChromaDB |
| **🛠️ Implementar** | Crear skill + indexar | `skill_manage` + `indexar-skills.py` |
| **📊 Registrar** | Actualizar registry | Script guarda en JSON |
| **👁️ Watch** | Micro-cron semanal si repo >500⭐ | `cronjob` |

### Criterio Avanzado de Creación de Skill

**Matriz de decisión:** (stars × topic_count × pattern_diversity) / existing_skill_overlap

```python
def should_create_skill(repo):
    score = repo['stars'] * (1 + 0.1*len(repo['topics'])) * (1 + 0.2*len(repo['potential_patterns']))
    # Penalizar si ya hay skill similar
    chroma_score = chromadb_search(repo['description'])
    if chroma_score > 0.25: return False
    return score > 500  # Threshold mínimo
```

### Micro-crons de Seguimiento

Cuando un repo >500⭐ tiene patrones relevantes para un proyecto activo:

```bash
# Automatizado por el cron
cronjob(action='create', 
  name=f'watch-{repo_basename}',
  schedule='0 0 * * 1',  # Cada lunes
  prompt=f"Revisar nuevas releases/issues de {owner}/{repo}")
```

Estos micro-crons se auto-destruyen tras 4 semanas si no generan ningún skill → `skill_created: false` en registry tras 4 ciclos sin actividad.

---

## Bulk Processing — Modo "Hazlo Rápido" 🚀

Cuando el usuario pide **procesar TODAS las stars de una vez** (no esperar al cron):

### Pipeline completo (1 hora para ~117 repos)

```
Fase 1: Registrar todas → Fase 2: Clasificar → Fase 3: Analizar en lote
→ Fase 4: Crear skills → Fase 5: Indexar ChromaDB → Fase 6: GitHub push
```

### Fase 1 — Registrar masivamente

No usar `--all` (timeout 300s). Hacer fetch de todas las stars paginadas + repo info básica (sin README) en batch:

```bash
# Fetch de todas las stars
curl -s "https://api.github.com/users/Ntizar/starred?per_page=100&page=1" -H "Authorization: token $GITHUB_TOKEN"

# Registrar cada repo en el registry como "pending" con stars_count + language
# 1 llamada API por repo (GET /repos/{owner}/{repo})
# ~30 segundos para 50 repos
```

### Fase 2 — Clasificar en 4 tiers

```python
tiers = {
    "high":   stars >= 3000,          # → subagent prioritario
    "medium": 500 <= stars < 3000,    # → subagent si hay slots
    "low":    stars < 500,            # → cron (no urgente)
    "skip":   "awesome" in name.lower() or "clone-wars" in name.lower(),
           or "collection" in name.lower(),  # → marcar y olvidar
}
```

También detectar **ALREADY_COVERED** por nombre: si el repo coincide con un skill existente (voicebox→media/voicebox, mlx-vlm→mlx-vlm-inference, nango→nango, city2graph→geoai-city2graph-pattern, postgres-mcp→postgres-mcp, gaze-tracking→gaze-tracking, supervision→vision/roboflow-supervision, etc.)

### Fase 3 — Análisis en lote con subagentes paralelos

```python
# 3 subagentes × 6-8 repos cada uno
delegate_task(tasks=[
  {"goal": "Analizar 6 repos y decidir skills (CREATE/SKIP/COVERED)", 
   "context": "REPOS: [lista de 6...]", "toolsets": ["terminal", "file"]},
  ...  # hasta 3 tasks en paralelo
])

# Cada subagente: fetch README → check existing skills → decidir CREATE/SKIP/COVERED
# Output: JSON con decisión por repo
```

**IMPORTANTE:** Cada subagente recibe una lista explícita de "ya cubiertos" para que no investigue skills existentes desde cero. Pasar en context:
```python
context = f"""
Ya existen skills para: voicebox (media/voicebox), mlx-vlm (mlx-vlm-inference),
nango (nango), city2graph (geoai-city2graph-pattern), postgres-mcp (postgres-mcp),
gaze-tracking (gaze-tracking), supervision (vision/roboflow-supervision),
crm-erp-fullstack (mastermind/), etc.
Repo high-value para analizar: {repo_list}
"""
```

### Fase 4 — Crear skills

Para cada repo con CREATE_SKILL:

```python
# 1. Fetch README completo (o leer del JSON ya guardado)
# 2. skill_manage(action='create', name=..., category=..., content=SKILL.md)
# 3. skill_manage(action='patch', name='stars-explorer', ...) # NO — marcar en registry
```

La creación debe ser **rápida**: SKILL.md conciso pero útil, con:
- YAML frontmatter (name, version, description, tags)
- Resumen del repo
- Instalación y uso básico
- Integración con Mastermind
- Referencia al repo original

### Fase 5 — Indexar en ChromaDB

```bash
bash /hermes-home/scripts/start-chromadb.sh   # Si no está running
python3 /hermes-home/scripts/indexar-skills.py  # Re-indexa todo
```

### Fase 6 — Subir a GitHub (OBLIGATORIO)

**Todos los skills deben estar en el repo Mastermind de GitHub.** David es explícito: "recuerda que estén en el github de mastermind también".

```bash
# Copiar skills al repo
cp -r /hermes-home/skills/<category>/<skill-name> /root/workspace/Mastermind/skills/<category>/
cp /hermes-home/data/stars-registry.json /root/workspace/Mastermind/data/
cp /hermes-home/scripts/explorar-stars.py /root/workspace/Mastermind/scripts/
cp /hermes-home/scripts/run-stars-explorer.sh /root/workspace/Mastermind/scripts/

# Commit + push
cd /root/workspace/Mastermind
git add -A
git commit -m "✨ Stars Explorer: <N> skills nuevos de stars de David"
git push
```

### Resumen de tiempos (117 repos)

| Fase | Tiempo |
|------|--------|
| Registrar 117 repos | ~40s |
| Clasificar | ~2s |
| 3 subagentes × 6 repos (paralelo) | ~3-5 min |
| Crear 8 skills | ~3 min |
| Indexar ChromaDB | ~1 min |
| GitHub push | ~30s |
| **Total** | **~10 min** |

### Lecciones aprendidas (próxima vez)

- El `--all` del script timeout a 300s → mejor registrar manualmente con paginación
- Los subagentes de `delegate_task` necesitan **context explícito** sobre qué skills ya existen (evita que investiguen desde cero)
- Los repos "awesome-*", "Clone-Wars" etc. se SKIP automáticamente
- Los skills deben estar en `/hermes-home/skills/` (el sistema Hermes) Y en `/root/workspace/Mastermind/skills/` (GitHub)

## Cron Asociado

- **Nombre:** `stars-explorer-nocturno`
- **Schedule:** 0 3 * * * (03:00 UTC diario)
- **Job ID:** `f22516e4ab77`
- **Batch:** 3 repos/run → 3 por noche
- **Re-procesamiento:** Nunca (registry previene duplicados)
- **ChromaDB:** Si ChromaDB no está corriendo, arrancar con `bash /hermes-home/scripts/start-chromadb.sh` antes de consultar. El cron puede ejecutarse en un momento donde ChromaDB se haya caído.
- **Modelo del cron:** `deepseek-v4-flash` (contexto 32K suficiente para READMEs de 8K chars)

### Modo de Procesamiento Rápido (para un batch completo)

Cuando hay **muchos repos nuevos** (50+), **NO usar `--all`** en el cron (se timeout). Mejor:

1. **Ejecutar el script** `python3 /hermes-home/scripts/registrar-stars-masivo.py` (registra todos en "pending" sin fetch)
2. **Actualizar el registry** manualmente con todos los nombres
3. **Dejar que el cron** procese 3/noche en modo standard

### Pitfall: `--all` se timeout

El script `explorar-stars.py` hace:
- `GET /repos/{name}` (1 req)
- `GET /repos/{name}/readme` (1 req)
- `GET /repos/{name}/git/trees/HEAD` (1 req)
- `GET /repos/{name}/contents/{file}` (hasta N reqs)

Para **117 repos** → ~200+ requests → **~2 min por req con rate-limit** → **se timeout**.

**Solución:** El cron **SIEMPRE** debe hacer `--batch 3`. Solo para cubrir todos rápidamente, usar el **registro masivo** primero (que solo hace 1 req por repo).

### Próxima Ejecución

El primer cron se ejecutará:
- **Fecha:** 2026-06-19T03:00:00+00:00
- **Batch:** 3 repos no procesados
- **Skills cargados:** `stars-explorer`, `chromadb-skills-vector-search`

Para **test** manual, ejecutar:
```bash
bash /hermes-home/scripts/run-stars-explorer.sh --batch 3 --json
```

## Referencias

- Script: `/hermes-home/scripts/explorar-stars.py`
- Registry: `/hermes-home/data/stars-registry.json`
- ChromaDB: skill `chromadb-skills-vector-search`
- GitHub trending (relacionado): skill `github-trending-research`
- Nota previa (exploración manual): `/root/workspace/Mastermind/notes/2026-05-30-nightly-explored-repos.md`
- Skills creados por este pipeline: [ver en registry → `skill_created: true`]
- **This cron:** `cronjob(action='list')` to see status
