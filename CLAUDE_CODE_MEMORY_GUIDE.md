# Guía de Memoria en Claude Code — CLAUDE.md y Auto Memory

> Fuente oficial: [code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory)

---

## 1. Sistemas de memoria (dos mecanismos)

| | **CLAUDE.md files** | **Auto memory** |
|---|---|---|
| **Quién escribe** | Tú | Claude |
| **Qué contiene** | Instrucciones y reglas | Aprendizajes y patrones |
| **Ámbito** | Proyecto, usuario u org | Por repositorio (compartido entre worktrees) |
| **Carga en** | Cada sesión | Cada sesión (primeras 200 líneas o 25KB) |
| **Para qué** | Estándares, workflows, arquitectura | Comandos de build, debugging, preferencias |

Ambos sistemas son **contexto**, no configuración forzada. Para bloquear acciones sí o sí, usa un [PreToolUse hook](/en/hooks-guide).

---

## 2. CLAUDE.md — Archivos de instrucciones

### 2.1 Ubicaciones y ámbito (orden de carga)

| Ámbito | Ubicación | Propósito | Compartido |
|--------|-----------|-----------|------------|
| **Managed policy** | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`
Linux/WSL: `/etc/claude-code/CLAUDE.md`
Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` | Instrucciones organizacionales (IT/DevOps) | Todos los usuarios |
| **User** | `~/.claude/CLAUDE.md` | Preferencias personales para todos los proyectos | Solo tú |
| **Project** | `./CLAUDE.md` o `./.claude/CLAUDE.md` | Instrucciones compartidas del proyecto | Team (git) |
| **Local** | `./CLAUDE.local.md` | Preferencias personales del proyecto (gitignored) | Solo tú |

### 2.2 Cómo se cargan

Claude Code **camina hacia arriba** desde el directorio actual, recogiendo `CLAUDE.md` y `CLAUDE.local.md` en cada directorio.

Ejemplo: si ejecutas en `foo/bar/`:
1. `foo/CLAUDE.md` (primero en contexto)
2. `foo/CLAUDE.local.md`
3. `foo/bar/CLAUDE.md` (después → más específico)
4. `foo/bar/CLAUDE.local.md`

**Orden dentro de un directorio:** `CLAUDE.md` → `CLAUDE.local.md`

**Subdirectorios:** `CLAUDE.md` en subdirectorios se cargan **on-demand** cuando Claude lee archivos en ese subdirectorio.

**`--add-dir`:** por defecto no carga CLAUDE.md de directorios adicionales. Para activarlo:
```bash
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ../shared-config
```

### 2.3 Cuándo añadir a CLAUDE.md

- Claude comete el mismo error por segunda vez
- Una code review detecta algo que Claude debería saber
- Escribes la misma corrección en el chat que la sesión anterior
- Un nuevo miembro del equipo necesitaría ese contexto

### 2.4 Cómo escribir instrucciones efectivas

**Tamaño:** < 200 líneas por archivo. Archivos más grandes reducen adherencia.

**Estructura:** headers y bullets. Secciones organizadas > párrafos densos.

**Especificidad:**
- ✅ "Usa indentación de 2 espacios" en vez de "Formatea el código bien"
- ✅ "Ejecuta `npm test` antes de commitear" en vez de "Prueba tus cambios"
- ✅ "Los handlers van en `src/api/handlers/`" en vez de "Mantén archivos organizados"

**Consistencia:** si dos reglas se contradicen, Claude elige una arbitrariamente.

### 2.5 Importar archivos con `@path`

```markdown
# CLAUDE.md
See @README for project overview.
@docs/git-instructions.md
```

- Rutas relativas se resuelven desde el archivo que contiene el import
- Máximo **4 niveles** de imports recursivos
- Para mencionar sin importar: `` `@README` `` (con backticks)
- Para instrucciones locales no versionadas: `@~/.claude/mi-proyecto.md`

**⚠️** La primera vez que Claude encuentra imports externos, muestra un diálogo de aprobación.

### 2.6 AGENTS.md

Claude Code **no** lee `AGENTS.md`, solo `CLAUDE.md`. Para reutilizar:

```markdown
# CLAUDE.md
@AGENTS.md

## Claude Code
Usa plan mode para cambios en src/billing/.
```

O un symlink: `ln -s AGENTS.md CLAUDE.md`

### 2.7 `/init` — Generar CLAUDE.md automáticamente

```bash
/init
```

Claude analiza el codebase y genera CLAUDE.md con commands, tests y convenciones. Si ya existe, sugiere mejoras.

Con `CLAUDE_CODE_NEW_INIT=1` se activa un flujo interactivo multi-fase.

### 2.8 Comentarios HTML (ahorran tokens)

```markdown
<!-- maintainer notes: esto no se inyecta en contexto -->
Los comentarios HTML block-level se eliminan antes de inyectar.
Los comentarios dentro de bloques de código se conservan.
```

---

## 3. `.claude/rules/` — Reglas organizadas

### 3.1 Estructura

```
.claude/
├── CLAUDE.md
└── rules/
    ├── code-style.md
    ├── testing.md
    ├── security.md
    └── frontend/
        └── react-patterns.md
```

Todas las reglas se cargan al inicio (si no tienen `paths`).

### 3.2 Path-specific rules (frontmatter YAML)

```markdown
---
paths:
  - "src/api/**/*.ts"
  - "lib/**/*.ts"
  - "tests/**/*.test.ts"
---

# Reglas de API
- Todos los endpoints deben tener validación de entrada
- Usa el formato de error estándar
```

| Patrón | Coincide |
|--------|----------|
| `**/*.ts` | Todos los `.ts` en cualquier directorio |
| `src/**/*` | Todo bajo `src/` |
| `*.md` | Markdown en la raíz |
| `src/**/*.{ts,tsx}` | Múltiples extensiones (brace expansion) |

Las reglas sin `paths` se cargan siempre. Las reglas con `paths` se activan cuando Claude lee archivos que coinciden.

### 3.3 Symlinks para compartir reglas

```bash
ln -s ~/shared-claude-rules .claude/rules/shared
ln -s ~/company-standards/security.md .claude/rules/security.md
```

### 3.4 User-level rules

`~/.claude/rules/` — reglas personales para todos los proyectos. Se cargan **antes** que las de proyecto.

---

## 4. Auto Memory — Memoria automática

Claude guarda notas automáticamente entre sesiones: comandos de build, debugging, preferencias, arquitectura.

**Requiere:** Claude Code v2.1.59+

### 4.1 Activar/desactivar

```bash
/memory                        # Toggle desde la UI
```

```json
{ "autoMemoryEnabled": false }  # En settings.json
```

```bash
CLAUDE_CODE_DISABLE_AUTO_MEMORY=1  # Env var
```

### 4.2 Almacenamiento

```
~/.claude/projects/<project>/memory/
├── MEMORY.md          # Índice (200 líneas o 25KB cargadas en cada sesión)
├── debugging.md       # Notas detalladas (on-demand)
├── api-conventions.md # Decisiones de API
└── ...                # Otros archivos que Claude crea
```

**Compartido entre todos los worktrees** del mismo repo. **Local a la máquina** (no se comparte entre equipos).

### 4.3 Cómo funciona

- `MEMORY.md`: primeras 200 líneas o 25KB se cargan al inicio de cada sesión
- Claude mantiene `MEMORY.md` conciso moviendo notas detalladas a archivos temáticos
- Archivos temáticos (`debugging.md`) se cargan **on-demand** cuando Claude los necesita
- Claude lee y escribe durante la sesión. Ves "Writing memory" o "Recalled memory" en la UI

### 4.4 Directorio personalizado

```json
{ "autoMemoryDirectory": "~/my-custom-memory-dir" }
```

Debe ser ruta absoluta o empezar con `~/`.

---

## 5. Comando `/memory`

Lista todos los archivos de memoria cargados en la sesión actual:
- `CLAUDE.md`, `CLAUDE.local.md`, y reglas de `.claude/rules/`
- Toggle de auto memory on/off
- Enlace para abrir la carpeta de auto memory

Si le pides a Claude "recuerda que siempre usamos pnpm, no npm", lo guarda en auto memory. Para guardar en CLAUDE.md, dile "add this to CLAUDE.md".

---

## 6. Troubleshooting

### Claude no sigue mi CLAUDE.md

1. Ejecuta `/memory` para verificar que se está cargando
2. Revisa que esté en la ubicación correcta
3. Haz las instrucciones más específicas
4. Busca instrucciones contradictorias entre archivos
5. Si es una acción que debe ejecutarse siempre, usa un **hook** en vez de CLAUDE.md
6. Para instrucciones a nivel de system prompt, usa `--append-system-prompt`

### No sé qué guardó auto memory

Usa `/memory` y abre la carpeta de auto memory. Todo es markdown plano: lee, edita o borra.

### Mi CLAUDE.md es demasiado grande

- Más de 200 líneas → reduce adherencia
- Usa reglas con `paths` para cargar solo cuando toca
- `@path` imports **no** reducen contexto (se cargan al inicio igualmente)

### Instrucciones perdidas después de `/compact`

- CLAUDE.md de raíz: sobrevive a compactación (Claude lo re-lee de disco)
- CLAUDE.md anidados: se recargan cuando Claude vuelva a leer archivos en ese subdirectorio

---

## 7. Managed CLAUDE.md (organizacional)

### Archivo físico
- macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`
- Linux/WSL: `/etc/claude-code/CLAUDE.md`
- Windows: `C:\Program Files\ClaudeCode\CLAUDE.md`

### Config en managed-settings.json

```json
{
  "claudeMd": "Always run `make lint` before committing.\nNever push directly to main."
}
```

**No se puede excluir** con `claudeMdExcludes`.

### `claudeMdExcludes` — Excluir archivos específicos

```json
{
  "claudeMdExcludes": [
    "**/monorepo/CLAUDE.md",
    "/home/user/monorepo/other-team/.claude/rules/**"
  ]
}
```

Los patrones se emparejan contra rutas absolutas. Los arrays se fusionan entre capas de settings.

---

## 8. Referencia rápida

```
Comando:         /memory         → Ver/editar memoria cargada
                  /init           → Generar CLAUDE.md automático

Settings:
  autoMemoryEnabled: true|false   → Activar auto memory
  autoMemoryDirectory: "~/.claude/custom"  → Directorio personalizado
  claudeMdExcludes: ["**/vendor/**"]  → Excluir CLAUDE.md

Env vars:
  CLAUDE_CODE_DISABLE_AUTO_MEMORY=1
  CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1

Ubicaciones CLAUDE.md:
  ~/.claude/CLAUDE.md            → User (todos los proyectos)
  ./CLAUDE.md                    → Project (versionado)
  ./.claude/CLAUDE.md            → Project (alternativo)
  ./CLAUDE.local.md              → Local (no versionado)

Reglas:
  .claude/rules/*.md             → Reglas del proyecto
  .claude/rules/**/*.md          → Reglas organizadas
  ~/.claude/rules/*.md           → Reglas personales

Auto memory:
  ~/.claude/projects/<project>/memory/MEMORY.md

Managed policy:
  /etc/claude-code/CLAUDE.md     → Linux/WSL
  /Library/Application Support/ClaudeCode/CLAUDE.md  → macOS
  C:\Program Files\ClaudeCode\CLAUDE.md  → Windows
```
