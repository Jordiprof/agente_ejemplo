# Guía completa de Subagents en OpenAI Codex

> Fuente oficial: [developers.openai.com/codex/subagents](https://developers.openai.com/codex/subagents)

---

## 1. ¿Qué son los Subagents?

Codex puede ejecutar **workflows con subagentes**: agentes especializados que se lanzan en paralelo y cuyos resultados se consolidan en una sola respuesta. Ideal para tareas complejas y altamente paralelizables como:

- Exploración de codebase
- Implementación de planes multi-paso
- Revisiones de PR en paralelo

Codex **solo lanza subagentes cuando se lo pides explícitamente**.

---

## 2. Agentes nativos incorporados

| Agente | Propósito |
|--------|-----------|
| **default** | Propósito general (fallback) |
| **worker** | Ejecución, implementación y correcciones |
| **explorer** | Exploración de codebase (solo lectura) |

---

## 3. Workflow típico

1. Codex orquesta: lanza subagentes, enruta instrucciones, espera resultados y cierra hilos
2. Cuando todos los agentes terminan, Codex devuelve una respuesta consolidada
3. Cada subagente consume sus propios tokens (modelo + tools)

**Ejemplo:**
```
Quiero revisar los siguientes puntos del PR actual. Lanza un agente por punto,
espera a todos y resume el resultado de cada uno:
1. Seguridad
2. Calidad del código
3. Bugs
4. Condiciones de carrera
5. Tests flaky
6. Mantenibilidad
```

---

## 4. Gestión de subagentes

- `/agent` en CLI — cambia entre hilos activos e inspecciona el hilo en curso
- Pregunta directamente a Codex para dirigir, detener o cerrar un subagente

### Controles de aprobación y sandbox

- Los subagentes heredan la política de sandbox de la sesión actual
- En sesiones CLI interactivas, las solicitudes de aprobación pueden aparecer desde hilos inactivos. Usa `o` para abrir ese hilo antes de aprobar/rechazar
- En flujos no interactivos, si una acción necesita aprobación y no se puede mostrar, falla y Codex notifica al workflow padre
- Codex re-aplica los overrides de runtime de la sesión padre al lanzar un hijo (sandbox, permisos, `--yolo`)

---

## 5. Custom Agents

Define tus propios agentes en archivos **TOML** bajo:

| Ubicación | Ámbito |
|-----------|--------|
| `~/.codex/agents/` | Personal (global) |
| `.codex/agents/` | Proyecto específico |

### 5.1 Campos obligatorios

| Campo | Tipo | Propósito |
|-------|------|-----------|
| `name` | string | Nombre para invocar al agente |
| `description` | string | Guía para que Codex decida cuándo usarlo |
| `developer_instructions` | string | Instrucciones de comportamiento |

### 5.2 Campos opcionales

| Campo | Tipo | Propósito |
|-------|------|-----------|
| `nickname_candidates` | string[] | Nombres para mostrar en UI (útil con múltiples instancias) |
| `model` | string | Modelo específico (ej: `gpt-5.4`) |
| `model_reasoning_effort` | string | `low`, `medium` o `high` |
| `sandbox_mode` | string | Ej: `read-only`, `workspace-write` |
| `mcp_servers` | table | Servidores MCP específicos |
| `skills.config` | array | Skills habilitadas/deshabilitadas |

### 5.3 Schema completo del archivo custom agent

```toml
name = "reviewer"
description = "PR reviewer focused on correctness, security, and missing tests."
developer_instructions = """
Review code like an owner.
Prioritize correctness, security, behavior regressions, and missing test coverage.
"""
nickname_candidates = ["Atlas", "Delta", "Echo"]
model = "gpt-5.4"
model_reasoning_effort = "high"
sandbox_mode = "read-only"

[mcp_servers.openaiDeveloperDocs]
url = "https://developers.openai.com/mcp"

[[skills.config]]
path = "/Users/me/.agents/skills/docs-editor/SKILL.md"
enabled = false
```

> **Nota:** Si un custom agent tiene el mismo nombre que un built-in (ej: `explorer`), el custom agent tiene prioridad.

### 5.4 Display nicknames

Usa `nickname_candidates` para mostrar nombres legibles en vez del nombre técnico. Solo afecta a la presentación. Código ASCII, dígitos, espacios, guiones y guiones bajos.

---

## 6. Configuración global (en `config.toml`)

Sección `[agents]` en el archivo de configuración de Codex:

| Campo | Tipo | Default | Propósito |
|-------|------|---------|-----------|
| `agents.max_threads` | number | `6` | Máximo de hilos concurrentes |
| `agents.max_depth` | number | `1` | Profundidad máxima de anidamiento (0 = raíz) |
| `agents.job_max_runtime_seconds` | number | `1800` | Timeout por worker para `spawn_agents_on_csv` |

**Advertencia sobre `max_depth`:** el default `1` permite un nivel de agente hijo. Subirlo puede provocar *fan-out* recursivo, aumentando tokens, latencia y consumo de recursos.

```toml
[agents]
max_threads = 6
max_depth = 1
```

---

## 7. Ejemplos completos de custom agents

### 7.1 PR Review (3 agentes especializados)

**`pr_explorer` — mapea el codebase (solo lectura):**
```toml
name = "pr_explorer"
description = "Read-only codebase explorer for gathering evidence before changes."
model = "gpt-5.3-codex-spark"
model_reasoning_effort = "medium"
sandbox_mode = "read-only"
developer_instructions = """
Stay in exploration mode.
Trace the real execution path, cite files and symbols, and avoid proposing fixes unless asked.
Prefer fast search and targeted file reads over broad scans.
"""
```

**`reviewer` — busca bugs y riesgos (solo lectura):**
```toml
name = "reviewer"
description = "PR reviewer focused on correctness, security, and missing tests."
model = "gpt-5.4"
model_reasoning_effort = "high"
sandbox_mode = "read-only"
developer_instructions = """
Review code like an owner.
Prioritize correctness, security, behavior regressions, and missing test coverage.
Lead with concrete findings, include reproduction steps when possible.
"""
```

**`docs_researcher` — verifica APIs y frameworks:**
```toml
name = "docs_researcher"
description = "Documentation specialist that uses the docs MCP server to verify APIs."
model = "gpt-5.4-mini"
model_reasoning_effort = "medium"
sandbox_mode = "read-only"
developer_instructions = """
Use the docs MCP server to confirm APIs, options, and version-specific behavior.
Return concise answers with links or exact references when available.
Do not make code changes.
"""

[mcp_servers.openaiDeveloperDocs]
url = "https://developers.openai.com/mcp"
```

**Prompt de ejemplo:**
```
Review this branch against main. Have pr_explorer map the affected code paths,
reviewer find real risks, and docs_researcher verify the framework APIs that the
patch relies on.
```

### 7.2 Frontend debugging (3 agentes)

**`code_mapper` — localiza código relevante:**
```toml
name = "code_mapper"
description = "Read-only codebase explorer for locating frontend/backend code paths."
model = "gpt-5.4-mini"
model_reasoning_effort = "medium"
sandbox_mode = "read-only"
developer_instructions = """
Map the code that owns the failing UI flow.
Identify entry points, state transitions, and likely files before editing.
"""
```

**`browser_debugger` — reproduce bugs en navegador:**
```toml
name = "browser_debugger"
description = "UI debugger using browser tooling to reproduce issues."
model = "gpt-5.4"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"
developer_instructions = """
Reproduce the issue in the browser, capture exact steps, and report what the UI does.
Use browser tooling for screenshots, console output, and network evidence.
Do not edit application code.
"""

[mcp_servers.chrome_devtools]
url = "http://localhost:3000/mcp"
startup_timeout_sec = 20
```

**`ui_fixer` — aplica la corrección mínima:**
```toml
name = "ui_fixer"
description = "Implementation-focused agent for small, targeted fixes."
model = "gpt-5.3-codex-spark"
model_reasoning_effort = "medium"
developer_instructions = """
Own the fix once the issue is reproduced.
Make the smallest defensible change, keep unrelated files untouched.
"""

[[skills.config]]
path = "/Users/me/.agents/skills/docs-editor/SKILL.md"
enabled = false
```

**Prompt de ejemplo:**
```
Investigate why the settings modal fails to save. Have browser_debugger reproduce it,
code_mapper trace the responsible code path, and ui_fixer implement the smallest fix.
```

---

## 8. Procesamiento por lotes CSV (`spawn_agents_on_csv`) [Experimental]

Útil para auditorías repetitivas: un worker por fila del CSV.

**Parámetros:**

| Parámetro | Tipo | Propósito |
|-----------|------|-----------|
| `csv_path` | string | Ruta al CSV fuente |
| `instruction` | string | Template del worker con placeholders `{columna}` |
| `id_column` | string | Columna para IDs estables |
| `output_schema` | object | Schema JSON que debe devolver cada worker |
| `output_csv_path` | string | Ruta del CSV de salida |
| `max_concurrency` | number | Concurrencia máxima |
| `max_runtime_seconds` | number | Timeout por worker |

**Regla:** cada worker debe llamar a `report_agent_job_result` exactamente una vez.

**Ejemplo de prompt:**
```
Create /tmp/components.csv with columns path,owner and one row per component.

Then call spawn_agents_on_csv with:
- csv_path: /tmp/components.csv
- id_column: path
- instruction: "Review {path} owned by {owner}. Return JSON with keys path, risk,
  summary, and follow_up via report_agent_job_result."
- output_csv_path: /tmp/components-review.csv
- output_schema: an object with required string fields path, risk, summary, follow_up
```

---

## 9. Buenas prácticas

### Haz
- ✅ Agentes **especializados y opinados**: un trabajo claro por agente
- ✅ Instrucciones que eviten que se desvíen a tareas adyacentes
- ✅ Modelos más baratos (`mini`) para tareas de exploración/lectura
- ✅ Modelos más potentes para revisión y debugging
- ✅ Sandbox `read-only` para agentes que solo deben explorar

### Evita
- ❌ Subir `max_depth` sin necesidad (riesgo de fan-out recursivo)
- ❌ Mezclar responsabilidades en un mismo agente
- ❌ Instrucciones vagas — sé concreto sobre el comportamiento esperado

---

## 10. Referencia rápida

```
~/.codex/agents/          # Custom agents globales
.codex/agents/            # Custom agents del proyecto
.codex/config.toml        # Config global [agents]

Comandos:
  /agent                    # Gestionar hilos activos en CLI
  spawn_agents_on_csv       # Procesar lotes CSV experimental

Modelos recomendados:
  gpt-5.3-codex-spark      # Ejecución rápida (worker)
  gpt-5.4-mini             # Exploración / lectura
  gpt-5.4                  # Revisión / debugging complejo
```
