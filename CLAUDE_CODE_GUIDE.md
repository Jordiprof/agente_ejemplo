# Guía completa de Claude Code — Agentes, Tools y Configuración

> Fuentes oficiales: [docs.anthropic.com](https://docs.anthropic.com/en/docs/claude-code/overview) · [platform.claude.com](https://platform.claude.com/docs/en/agents-and-tools/tool-use/build-a-tool-using-agent)

---

## 1. Arquitectura general

Claude Code es el agente de código de Anthropic. Su configuración se basa en archivos en el proyecto o en `~/.claude/`.

| Característica | Ubicación usuario | Ubicación proyecto | Ubicación local |
|---------------|-------------------|-------------------|-----------------|
| **Settings** | `~/.claude/settings.json` | `.claude/settings.json` | `.claude/settings.local.json` |
| **Subagents** | `~/.claude/agents/` | `.claude/agents/` | — |
| **MCP servers** | `~/.claude.json` | `.mcp.json` | `~/.claude.json` (per-project) |
| **CLAUDE.md** | `~/.claude/CLAUDE.md` | `CLAUDE.md` o `.claude/CLAUDE.md` | `CLAUDE.local.md` |
| **Skills** | `~/.claude/skills/` | `.claude/skills/` | — |
| **Output Styles** | `~/.claude/output-styles/` | `.claude/output-styles/` | — |
| **Commands** (legacy) | — | `.claude/commands/` | — |
| **Plugins** | `~/.claude/settings.json` | `.claude/settings.json` | `.claude/settings.local.json` |

---

## 2. CLAUDE.md — Memoria del proyecto

Archivo Markdown que Claude Code lee al inicio de cada sesión. Instrucciones persistentes y versionables.

**Ubicaciones (orden de carga):**
1. `~/.claude/CLAUDE.md` — global del usuario
2. `CLAUDE.md` o `.claude/CLAUDE.md` — raíz del proyecto
3. `CLAUDE.local.md` — local (no versionado)
4. `CLAUDE.md` anidados en subdirectorios
5. `.claude/rules/*.md` — reglas condicionales con frontmatter

```markdown
# CLAUDE.md

## Stack
- TypeScript estricto, Node 22, Express 5
- PostgreSQL con Prisma
- Tests: Vitest

## Comandos
- `npm run dev` — desarrollo
- `npm test` — ejecutar tests
- `npm run lint` — lint
- `npm run build` — compilar

## Convenciones
- camelCase para variables y funciones
- PascalCase para clases y tipos
- Tests colocalizados: `foo.ts` + `foo.test.ts`
- Errores tipados y estructurados

## No hagas
- No instalar dependencias sin permiso
- No modificar src/legacy/
```

---

## 3. Subagentes

Agentes especializados que manejan tareas específicas en su propio contexto.

### 3.1 Subagentes nativos

| Agente | Modelo | Tools | Propósito |
|--------|--------|-------|-----------|
| **Explore** | Haiku | Solo lectura | Búsqueda y exploración rápida |
| **Plan** | Heredado | Solo lectura | Investigación para plan mode |
| **General-purpose** | Heredado | Todas | Tareas complejas multi-paso |
| **statusline-setup** | Sonnet | — | Configurar status line |
| **claude-code-guide** | Haiku | — | Preguntas sobre Claude Code |

### 3.2 Archivo de subagente (`*.md` con frontmatter YAML)

```markdown
---
name: code-reviewer
description: Revisa código buscando calidad, seguridad y bugs
tools: Read, Glob, Grep
disallowedTools: Write, Edit
model: sonnet
permissionMode: read-only
maxTurns: 20
skills:
  - api-conventions
mcpServers:
  - github
memory: project
effort: high
isolation: worktree
color: blue
background: false
initialPrompt: "Revisa el código y busca problemas"
---

Eres un revisor de código senior. Busca problemas de seguridad,
rendimiento y mantenibilidad. Proporciona ejemplos concretos.
```

### 3.3 Campos del frontmatter

| Campo | Obligatorio | Descripción |
|-------|:-----------:|-------------|
| `name` | Sí | ID único (minúsculas + guiones) |
| `description` | Sí | Cuándo debe delegar Claude a este agente |
| `tools` | No | Tools permitidas (hereda todas si se omite) |
| `disallowedTools` | No | Tools denegadas |
| `model` | No | `sonnet`, `opus`, `haiku`, `fable`, ID completo o `inherit` |
| `permissionMode` | No | `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan` |
| `maxTurns` | No | Máximo de turnos antes de parar |
| `skills` | No | Skills a precargar en contexto |
| `mcpServers` | No | Servidores MCP para este subagente |
| `hooks` | No | Lifecycle hooks |
| `memory` | No | `user`, `project` o `local` (memoria persistente) |
| `background` | No | `true` para siempre ejecutar en background |
| `effort` | No | `low`, `medium`, `high`, `xhigh`, `max` |
| `isolation` | No | `worktree` para clon aislado del repo |
| `color` | No | `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan` |
| `initialPrompt` | No | Primer turno auto-enviado al iniciar sesión con `--agent` |

### 3.4 Ubicaciones y prioridad

| Ubicación | Ámbito | Prioridad |
|-----------|--------|:---------:|
| Managed settings | Organización | 1 (máxima) |
| `--agents` CLI flag | Sesión actual | 2 |
| `.claude/agents/` | Proyecto actual | 3 |
| `~/.claude/agents/` | Todos los proyectos | 4 |
| Plugin `agents/` | Donde esté el plugin | 5 |

### 3.5 Cómo invocar subagentes

**Automático:** Claude delega según la `description`.

**@-mention:** `@"code-reviewer (agent)" revisa los cambios de auth`

**Por nombre en CLI:**
```bash
claude --agent code-reviewer
```

**En settings.json (sesión completa):**
```json
{ "agent": "code-reviewer" }
```

**Subagentes anidados:** desde v2.1.172, un subagente puede spawnear sus propios subagentes.

---

## 4. Skills

Skills son capacidades especializadas que Claude usa automáticamente o invocas con `/nombre`.

### 4.1 Estructura

```
.claude/skills/
  mi-skill/
    SKILL.md         # Obligatorio
    otros-archivos   # Opcional (scripts, referencias, etc.)
```

### 4.2 SKILL.md — frontmatter

```markdown
---
name: mi-skill
description: Genera informes PDF con matplotlib
disable-model-invocation: false
model: sonnet
effort: high
context: fork
agent: general-purpose
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate.sh"
paths: "src/**/*.py"
---

# Mi Skill

Instrucciones detalladas para Claude sobre cómo usar esta skill.
```

### 4.3 Campos del frontmatter (SKILL.md)

| Campo | Descripción |
|-------|-------------|
| `name` | Máx 64 chars, solo minúsculas/números/guiones |
| `description` | Máx 1024 chars, no XML tags |
| `disable-model-invocation` | `true` = solo invocación manual con `/nombre` |
| `model` | Modelo específico para la skill |
| `effort` | Nivel de esfuerzo |
| `context` | `fork` para ejecutar en subagente |
| `agent` | Tipo de subagente cuando `context: fork` |
| `hooks` | Lifecycle hooks |
| `paths` | Glob patterns para activación automática |

### 4.4 Comportamiento

1. **Metadata pre-cargada:** name + description van al system prompt al inicio
2. **Archivos leídos on-demand:** Claude lee SKILL.md solo cuando los necesita
3. **Scripts ejecutados eficientemente:** solo el output consume tokens
4. **Sin penalización de contexto:** archivos grandes no consumen tokens hasta leerse

---

## 5. Tool Use API — Lifecycle completo

Tutorial oficial: 5 anillos (rings) de complejidad creciente.

### Ring 1: Single tool, single turn

```python
import anthropic

client = anthropic.Anthropic()

tools = [{
    "name": "create_calendar_event",
    "description": "Create a calendar event.",
    "input_schema": {
        "type": "object",
        "properties": {
            "title": {"type": "string"},
            "start": {"type": "string", "format": "date-time"},
            "end": {"type": "string", "format": "date-time"},
            "attendees": {
                "type": "array",
                "items": {"type": "string", "format": "email"}
            },
        },
        "required": ["title", "start", "end"]
    }
}]

# Enviar mensaje + tools
response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    tools=tools,
    tool_choice={"type": "auto", "disable_parallel_tool_use": True},
    messages=[{"role": "user", "content": "Agenda una reunión..."}],
)

# Claude responde con stop_reason="tool_use"
tool_use = next(b for b in response.content if b.type == "tool_use")
print(tool_use.name, tool_use.input)

# Ejecutar tool y devolver resultado
result = {"event_id": "evt_123", "status": "created"}

followup = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    tools=tools,
    messages=[
        {"role": "user", "content": "Agenda una reunión..."},
        {"role": "assistant", "content": response.content},
        {"role": "user", "content": [{
            "type": "tool_result",
            "tool_use_id": tool_use.id,
            "content": json.dumps(result)
        }]}
    ],
)

# stop_reason="end_turn", respuesta en lenguaje natural
```

### Ring 2: Agentic loop

```python
messages = [{"role": "user", "content": "..."}]
response = client.messages.create(..., messages=messages)

while response.stop_reason == "tool_use":
    tool_use = next(b for b in response.content if b.type == "tool_use")
    result = run_tool(tool_use.name, tool_use.input)

    messages.append({"role": "assistant", "content": response.content})
    messages.append({"role": "user", "content": [{
        "type": "tool_result",
        "tool_use_id": tool_use.id,
        "content": json.dumps(result)
    }]})

    response = client.messages.create(..., messages=messages)
```

### Ring 3: Error handling

```python
try:
    result = run_tool(tool_use.name, tool_use.input)
except Exception as e:
    result = {"error": str(e)}

# Mandar error como tool_result con is_error=True
messages.append({"role": "user", "content": [{
    "type": "tool_result",
    "tool_use_id": tool_use.id,
    "content": json.dumps({"error": str(e)}),
    "is_error": True
}]})
```

### Ring 4: Multiple tools, parallel calls

```python
tool_choice = {"type": "auto"}  # parallel_tool_use activado por defecto
# Claude puede llamar varias tools en una misma respuesta
```

### Ring 5: Tool Runner SDK

```python
from anthropic import ToolRunner

runner = ToolRunner(client, tools)
result = runner.run(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=[...],
)
```

---

## 6. Settings — `settings.json`

### 6.1 Archivos de configuración

```json
{
  "$schema": "https://raw.githubusercontent.com/anthropics/claude-code/main/schema.json",
  "agent": "code-reviewer",
  "permissions": {
    "allow": ["Read", "Write", "Edit", "Bash"],
    "deny": ["Agent(Explore)", "WebFetch"]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "./scripts/validate.sh" }]
      }
    ],
    "SubagentStart": [
      {
        "matcher": "db-agent",
        "hooks": [{ "type": "command", "command": "./scripts/setup.sh" }]
      }
    ]
  },
  "outputStyle": "concise",
  "claudeMd": "Instrucciones gestionadas por la organización..."
}
```

### 6.2 Campos principales

| Campo | Descripción |
|-------|-------------|
| `agent` | Subagente por defecto para la sesión |
| `permissions.allow` | Tools permitidas (allowlist) |
| `permissions.deny` | Tools denegadas (denylist) |
| `hooks` | Lifecycle hooks globales |
| `outputStyle` | Estilo de output por defecto |
| `claudeMd` | (Managed) Instrucciones organizacionales |

---

## 7. MCP Servers

```json
// .mcp.json (proyecto)
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"]
    },
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

---

## 8. Output Styles

Archivos Markdown en `.claude/output-styles/` o `~/.claude/output-styles/`.

```markdown
---
name: concise
description: Respuestas breves y directas
keep-coding-instructions: false
---

Responde de forma concisa. Usa listas. Sin rodeos.
```

| Campo | Descripción |
|-------|-------------|
| `name` | Nombre del estilo (del filename si se omite) |
| `description` | Descripción para el picker de `/config` |
| `keep-coding-instructions` | `false` para borrar instrucciones de ingeniería de software |
| `force-for-plugin` | (Plugin) Forzar estilo automáticamente |

---

## 9. Hooks — Lifecycle events

| Evento | Matcher | Cuándo se dispara |
|--------|---------|-------------------|
| `PreToolUse` | Tool name | Antes de usar una tool |
| `PostToolUse` | Tool name | Después de usar una tool |
| `SubagentStart` | Agent type | Cuando un subagente empieza |
| `SubagentStop` | Agent type | Cuando un subagente termina |
| `InstructionsLoaded` | — | Cuando se carga un CLAUDE.md |
| `Stop` | — | Cuando el subagente termina (→ `SubagentStop`) |

---

## 10. Resumen de archivos y ubicaciones

```
# Raíz del proyecto
CLAUDE.md                          # Memoria del proyecto
CLAUDE.local.md                    # Memoria local (no versionada)

.claude/
├── settings.json                  # Config del proyecto (versionable)
├── settings.local.json            # Config local (gitignored)
├── agents/
│   ├── code-reviewer.md           # Subagente
│   └── research/
│       └── security.md            # Subagente organizado en carpetas
├── skills/
│   └── mi-skill/
│       ├── SKILL.md               # Skill (frontmatter + body)
│       └── script.sh              # Script auxiliar
├── output-styles/
│   └── concise.md                 # Estilo de output
├── commands/                      # Comandos legacy
│   └── deploy.md
└── rules/                         # Reglas condicionales
    └── react-patterns.md

.mcp.json                          # MCP servers del proyecto

# Home del usuario
~/.claude/
├── settings.json                  # Config global
├── CLAUDE.md                      # Memoria global
├── agents/                        # Subagentes globales
└── skills/                        # Skills globales
```

---

## 11. Buenas prácticas

### Haz
- ✅ Usa `CLAUDE.md` para convenciones del proyecto y comandos
- ✅ Crea subagentes para tareas repetitivas (review, test, deploy)
- ✅ Usa `tools` allowlist para restringir subagentes a solo lectura
- ✅ Skills con `context: fork` para aislar tareas complejas
- ✅ Hooks `PreToolUse` para validar comandos antes de ejecutarlos
- ✅ Memory `project` en subagentes para aprendizaje cross-sesión

### Evita
- ❌ No dupliques info de linters en CLAUDE.md
- ❌ No documentes todos los comandos posibles
- ❌ No uses `bypassPermissions` sin entender los riesgos
- ❌ No mezcles responsabilidades en un mismo subagente

---

## 12. Referencia rápida

```
Comandos CLI:
  claude --agent <nombre>          # Sesión como subagente
  claude --agents '<json>'         # Subagentes inline
  claude --disallowedTools "Agent(Explore)"  # Bloquear subagente

Dentro de Claude:
  /agents                          # Gestionar subagentes
  /config                          # Configuración
  /btw <pregunta>                  # Pregunta rápida (sin tools)

API:
  anthropic.Anthropic()            # Cliente
  client.messages.create()         # Llamada con tools
  ToolRunner(client, tools)        # SDK para agentic loop

Modelos: claude-opus-4-8, claude-sonnet-4-6, haiku, fable
```
