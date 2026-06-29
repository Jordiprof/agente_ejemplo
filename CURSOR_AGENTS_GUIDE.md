# Guía completa de Rules & Agents para Cursor

> Fuente oficial: [cursor.com/es/docs/rules](https://cursor.com/es/docs/rules)

---

## 1. Tipos de reglas

Cursor soporta **4 tipos de reglas**:

| Tipo | Ubicación | Ámbito | Versionado |
|------|-----------|--------|------------|
| **Project Rules** | `.cursor/rules/*.mdc` | Proyecto actual | Sí (git) |
| **User Rules** | Personalizar → regla (global) | Todos los proyectos | No |
| **Team Rules** | Dashboard Cursor (Team/Enterprise) | Todo el equipo | Desde dashboard |
| **AGENTS.md** | Raíz o subdirectorios | Proyecto o subdirectorio | Sí (git) |

---

## 2. Cómo funcionan las reglas

Las reglas se inyectan al inicio del contexto del modelo en cada sesión. Proporcionan guía coherente y reutilizable a nivel de prompt. No hay memoria persistente entre respuestas del LLM.

---

## 3. Project Rules (`.cursor/rules/*.mdc`)

Almacenadas en `.cursor/rules/` con extensión `.mdc`. Bajo control de versiones.

**Usos típicos:**

- Codificar conocimiento específico del dominio
- Automatizar flujos de trabajo o plantillas del proyecto
- Estandarizar estilo o arquitectura

**Nota:** Los archivos `.md` simples en `.cursor/rules/` son **ignorados** (no tienen frontmatter). Para Markdown simple usa `AGENTS.md`.

### 3.1 Estructura de archivos

```
.cursor/rules/
  react-patterns.mdc
  api-guidelines.md        # Ignorado (extensión incorrecta)
  frontend/
    components.mdc
```

### 3.2 Anatomía de una regla

Cada regla es un archivo Markdown con **frontmatter** y contenido.

```markdown
---
description: "Descripción para que Agent decida si aplicarla"
globs: "src/**/*.tsx"
alwaysApply: false
---

Contenido de la regla...
```

### 3.3 Campos del frontmatter

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `description` | string | Describe el propósito. Agent la lee para decidir relevancia |
| `globs` | string | Patrón(es) de archivos para activación automática |
| `alwaysApply` | boolean | Si es `true`, se aplica en **toda** sesión de chat |

### 3.4 Comportamiento según configuración

| `alwaysApply` | `description` | `globs` | Comportamiento |
|:---|:---|:---|:---|
| `true` | — | — | Siempre se incluye (ignora globs y description) |
| `false` | — | proporcionado | Se adjunta auto cuando un archivo coincide en el contexto |
| `false` | proporcionado | omitido | Agent lee description e incluye si es relevante |
| `false` | omitido | omitido | Solo se incluye con @ en chat (`@my-rule`) |

### 3.5 Tipos de regla (desde la UI)

| Tipo | Cómo se activa |
|:-----|:---------------|
| **Always Apply** | Cada sesión de chat |
| **Apply Intelligently** | Agent decide según `description` |
| **Apply to Specific Files** | Cuando un archivo coincide con `globs` |
| **Apply Manually** | Cuando mencionas la regla con `@` en el chat |

### 3.6 Ejemplos de patrones glob

| Patrón | Coincidencias |
|:-------|:--------------|
| `*` | Cualquier nombre de archivo (un segmento) |
| `**` | Cualquier número de directorios (recursivo) |
| `*.ts` | Todos los `.ts` en la raíz |
| `**/*.ts` | Todos los `.ts` en cualquier directorio |
| `src/**` | Todo en cualquier subdirectorio de `src/` |
| `src/**/*.tsx` | Todos los `.tsx` bajo `src/` |
| `docs/**/*.md, docs/**/*.mdx` | `.md` y `.mdx` en `docs/` (separados por coma) |
| `tailwind.config.*` | `tailwind.config` con cualquier extensión |

### 3.7 Ejemplos completos de reglas

**Always Apply:**
```markdown
---
alwaysApply: true
---

- Todos los archivos fuente deben incluir encabezado de copyright
- Cuando no estés seguro, lee los archivos fuente antes de proponer cambios
- Nunca modifiques archivos en `dist/` o `build/`
```

**Auto-attach por patrón:**
```markdown
---
globs: src/components/**/*.tsx
alwaysApply: false
---

- Usa named exports, no default exports
- Co-localiza estilos en módulo CSS junto al componente
- Mantén componentes bajo 200 líneas
- Prefiere composición sobre prop drilling
```

**Agent-selected por descripción:**
```markdown
---
description: Convenciones y patrones RPC para el backend
alwaysApply: false
---

- Define cada servicio en su propio archivo bajo `src/services/`
- Valida entradas en el límite del servicio
- Devuelve errores estructurados con `code` y `message`
```

**Manual (solo @-mention):**
```markdown
---
alwaysApply: false
---

- Toda migración debe tener funciones `up` y `down`
- Nunca modifiques el tipo de una columna in-place
- @migration-template.sql
```

### 3.8 Cómo crear reglas

1. **Via chat:** Escribe `/create-rule` en Agent y describe la regla
2. **Desde Personalizar:** Abre Personalizar → Reglas → Añadir regla

---

## 4. AGENTS.md

Archivo Markdown simple para instrucciones de agente. **Alternativa a `.cursor/rules`** para casos sencillos.

- Colócalo en la **raíz** del proyecto
- Sin frontmatter, sin metadatos complejos
- Compatible con AGENTS.md anidados en subdirectorios

### 4.1 AGENTS.md anidados

```
project/
  AGENTS.md              # Instrucciones globales
  frontend/
    AGENTS.md            # Instrucciones específicas de frontend
    components/
      AGENTS.md          # Instrucciones específicas de componentes
  backend/
    AGENTS.md            # Instrucciones específicas de backend
```

Las instrucciones anidadas se **combinan** con las de directorios superiores. Las más específicas tienen **prioridad**.

### 4.2 Estructura recomendada para AGENTS.md

```markdown
# Project Instructions

## Stack
- Lenguaje: [p.ej. TypeScript estricto]
- Framework: [p.ej. Node 22 + Express 5]
- Base de datos: [p.ej. PostgreSQL con Prisma]
- Tests: [p.ej. Vitest]

## Comandos
- `npm run dev` — desarrollo local
- `npm test` — ejecutar tests
- `npm run lint` — revisar estilo
- `npm run build` — compilar producción

## Arquitectura
- Sigue el patrón Controlador → Servicio → Repositorio
- La lógica de negocio va en servicios, no en controladores

## Convenciones
- camelCase para variables y funciones
- PascalCase para clases/types
- Errores tipados y estructurados
- Tests al lado del archivo: `foo.ts` + `foo.test.ts`

## Seguridad
- Validar toda entrada externa
- No hardcodear credenciales
- No subir archivos .env

## No hagas
- Instalar dependencias sin permiso
- Modificar `src/legacy/` (congelado)
- Usar `any` sin justificarlo
```

---

## 5. User Rules

Preferencias globales en **Personalizar → regla**. Afectan a Agent (Chat) en todos los proyectos.

**No afectan a:**
- Cursor Tab
- Inline Edit (Cmd/Ctrl+K)

Ejemplo:
```markdown
Please reply in a concise style. Avoid unnecessary repetition or filler language.
```

---

## 6. Team Rules (Team/Enterprise)

Gestionadas desde el [dashboard de Cursor](https://cursor.com/dashboard/team-content).

- **Habilitar inmediatamente:** Se activa al crearla
- **Imponer regla:** Obligatoria para todo el equipo (no se puede desactivar)
- **Globs:** Admiten patrones glob para aplicar por tipo de archivo
- **Sin globs:** Se aplican a todas las conversaciones

**Precedencia:**
```
Team Rules → Project Rules → User Rules
```

Cuando hay conflictos, la fuente anterior tiene prioridad.

---

## 7. Importar reglas (Remotas via GitHub)

1. Abre **Personalizar** en la barra lateral
2. Ve a **Regla** → **Añadir regla**
3. Selecciona **Remote Rule (Github)**
4. Pega la URL del repositorio

Las reglas se colocan en:
```
.cursor/rules/imported/<repoName>/
```

Conservan rutas relativas: `dir/rule.mdc` → `.cursor/rules/imported/<repoName>/dir/rule.mdc`

---

## 8. Mejores prácticas

### Haz
- ✅ Mantén reglas con **menos de 500 líneas**
- ✅ Divide reglas grandes en **varias reglas componibles**
- ✅ Proporciona **ejemplos concretos** o archivos de referencia
- ✅ **Referencia archivos** en lugar de copiar su contenido
- ✅ Escribe reglas como **documentación interna clara**
- ✅ Reutiliza reglas cuando repitas indicaciones en chat
- ✅ Haz **commit de las reglas** para compartir con el equipo

### Evita
- ❌ Copiar guías de estilo completas (usa un linter)
- ❌ Documentar todos los comandos posibles (Agent conoce npm, git, pytest)
- ❌ Instrucciones para casos límite que raramente aplican
- ❌ Duplicar lo que ya existe en la base de código

### Estrategia recomendada
Empieza simple. Añade reglas **solo cuando notes que Agent comete el mismo error repetidamente**. No sobreoptimices antes de entender tus propios patrones.

---

## 9. Preguntas frecuentes

**¿Por qué no se aplica mi regla?**
- Verifica el tipo de regla. Para `Apply Intelligently`, asegúrate de tener `description`. Para `Apply to Specific Files`, verifica que `globs` coincida con los archivos.

**¿Pueden las reglas referenciar otras reglas o archivos?**
- Sí. Usa `@filename.ts` para incluir archivos. Usa `@rule-name` en el chat para aplicar reglas manualmente.

**¿Puedo crear una regla desde el chat?**
- Sí, pídeselo al Agent.

**¿Las reglas afectan a Cursor Tab?**
- No. Solo afectan a Agent (Chat).

**¿Las User Rules afectan a Inline Edit (Cmd/Ctrl+K)?**
- No. Solo las usa Agent (Chat).

---

## 10. Plantilla rápida: `.cursor/rules/mi-regla.mdc`

```markdown
---
description: "Describe cuándo aplica esta regla"
globs: "src/**/*.ts"
alwaysApply: false
---

# [Nombre de la regla]

## Instrucciones
- [Regla concreta 1]
- [Regla concreta 2]

## Referencias
@archivo-de-ejemplo.ts
```
