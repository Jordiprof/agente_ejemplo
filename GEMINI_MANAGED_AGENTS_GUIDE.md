# Guía completa de Managed Agents en Gemini API (Google)

> Fuente oficial: [blog.google](https://blog.google/innovation-and-ai/technology/developers-tools/managed-agents-gemini-api/) · [ai.google.dev](https://ai.google.dev/gemini-api/docs/custom-agents)

---

## 1. ¿Qué son los Managed Agents?

Los **Managed Agents** en Gemini API te permiten lanzar agentes con una sola llamada. Cada agente razona, usa herramientas y ejecuta código en un **entorno Linux efímero y aislado** gestionado por Google.

**Lanzamiento:** 19 mayo 2026 · Preview pública

---

## 2. Agentes gestionados disponibles

| Agente | Propósito | Modelo |
|--------|-----------|--------|
| **Antigravity** | Propósito general (código, archivos, web) | Gemini 3.5 Flash |
| **Deep Research** | Investigación multi-paso automatizada | — |

---

## 3. Antigravity Agent — capacidades base

Una llamada API provisiona un sandbox Linux remoto donde el agente puede:

| Capacidad | Tipo | Descripción |
|-----------|------|-------------|
| **Code Execution** | `code_execution` | Shell, bash, Python, Node.js con captura stdout/stderr |
| **Google Search** | `google_search` | Búsqueda en web pública |
| **URL Context** | `url_context` | Fetch y lectura de páginas web |
| **Filesystem** | (auto vía `environment`) | Read, write, edit, search, list files |
| **Custom Functions** | `function` | Function calling para APIs propias |
| **Remote MCP Server** | `mcp_server` | Servidores MCP externos |

### Entorno sandbox

- **OS:** Ubuntu con Python 3.12 y Node.js 22 pre-instalados
- **Paquetes:** `pip install` / `npm install` en runtime (persisten con mismo `environment_id`)
- **Lifetime:** 7 días de inactividad → borrado permanente
- **VM spin-down:** Se apaga tras inactividad breve; siguiente request restaura estado (cold start)
- **Compaction automático:** ~135k tokens → compactación para evitar context rot

---

## 4. Cómo extender el Antigravity Agent

Tres formas de personalización:

### 4.1 System instructions (inline)

```python
system_instruction="Eres un agente de análisis financiero. Genera informes en PDF."
```

Compatible con `AGENTS.md`: ambos son **aditivos** (se aplican juntos).

### 4.2 Tools

Por defecto: `code_execution`, `google_search`, `url_context`. Puedes:

- Sobrescribir la lista pasando `tools` en la interacción
- Añadir `mcp_server` (MCP remotos)
- Añadir `function` (function calling propio)

### 4.3 Files & Skills (AGENTS.md + SKILL.md)

Monta archivos directamente en el sandbox:

```
.agents/
  AGENTS.md            # Instrucciones del agente (system prompt versionable)
  skills/
    mi-skill/
      SKILL.md         # Skill específica (auto-descubierta por el harness)
```

---

## 5. AGENTS.md en Gemini API

El agente carga automáticamente `.agents/AGENTS.md` (o `/.agents/AGENTS.md`) del entorno como **system instructions** al arrancar.

**Usos:**
- Persona de largo formato
- Directrices detalladas
- Instrucciones versionables junto al código

```markdown
# AGENTS.md — Financial Analyst Agent

Eres un agente analista financiero.

## Reglas
- Siempre incluye un chart y una tabla resumen en los informes
- Exporta resultados como PDF
- Usa fuentes oficiales (Google Finance, Yahoo Finance)
- No des consejos de inversión personalizados

## Stack
- Python 3.12 con matplotlib, pandas, reportlab
- Datos vía yfinance
```

---

## 6. SKILL.md (Skills)

Las skills son archivos que extienden las capacidades del agente. Se colocan bajo `.agents/skills/<nombre>/SKILL.md` y el harness las auto-descubre y registra.

```markdown
# SKILL.md — PDF Report Generator

Genera informes PDF con matplotlib + reportlab.

## Instrucciones
- Usa reportlab para el layout general
- matplotlib para gráficos embebidos
- Guarda en /home/user/output/report.pdf
```

---

## 7. Llamadas a la API

### 7.1 Interacción simple (nuevo sandbox)

```python
client = genai.Client(api_key="...")

interaction = client.interactions.create(
    model="antigravity-preview-05-2026",
    input_text="Analiza el fichero data.csv y genera un resumen",
    environment="remote",  # sandbox fresco
)

print(interaction.output_text)
```

### 7.2 Multi-turno (mismo sandbox)

```python
# Primer turno — guardar environment_id
interaction_1 = client.interactions.create(
    model="antigravity-preview-05-2026",
    input_text="Instala pandas y crea un script de análisis",
    environment="remote",
)

# Segundo turno — reusa el mismo entorno
interaction_2 = client.interactions.create(
    model="antigravity-preview-05-2026",
    input_text="Ejecuta el script que creaste",
    environment=interaction_1.environment_id,
)
```

### 7.3 Inline customization

```python
interaction = client.interactions.create(
    model="antigravity-preview-05-2026",
    input_text="Analiza el código en src/",
    system_instruction="Eres un revisor de código senior. Busca bugs de seguridad.",
    environment={
        "type": "remote",
        "sources": [
            {
                "type": "repository",
                "source": "https://github.com/tu-org/mi-proyecto",
                "target": "/home/user/project",
            },
            {
                "type": "inline",
                "target": ".agents/AGENTS.md",
                "content": "Revisa como un owner. Prioriza seguridad y tests faltantes.",
            },
        ],
    },
)
```

### 7.4 Crear un Managed Agent persistente

```python
agent = client.agents.create(
    id="mi-revisor",
    base_agent="antigravity-preview-05-2026",
    system_instruction="Eres un revisor de código senior.",
    base_environment={
        "type": "remote",
        "sources": [
            {
                "type": "inline",
                "target": ".agents/AGENTS.md",
                "content": "Siempre incluye ejemplos de código en tus revisiones.",
            },
        ],
    },
)

# Invocar por ID
interaction = client.interactions.create(
    agent="mi-revisor",
    input_text="Revisa el PR #42",
)
```

---

## 8. Schema de la definición del agente

| Campo | Tipo | Obligatorio | Descripción |
|-------|------|:-----------:|-------------|
| `id` | string | Sí | ID único para invocar el agente |
| `base_agent` | string | Sí | Solo `antigravity-preview-05-2026` |
| `description` | string | No | Descripción legible |
| `system_instruction` | string | No | Prompt de sistema |
| `tools` | array | No | Tools habilitadas (default: code_execution, google_search, url_context) |
| `base_environment` | string/object | No | `"remote"`, `environment_id`, o config object con `sources` y `network` |

### Sources disponibles para `base_environment`

| Type | Descripción | Limitaciones |
|------|-------------|--------------|
| `repository` | Clona un repo Git en `target` | — |
| `gcs` | Copia desde Cloud Storage a `target` | — |
| `inline` | Contenido inline escrito en `target` | 1 MB/file, 2 MB total |

No se puede usar `/` (root) como `target`.

---

## 9. Contexto vs Estado

La API gestiona **dos dimensiones de estado independientes**:

| Dimensión | Cómo se controla |
|-----------|-----------------|
| **Contexto de conversación** | `previous_interaction_id` — historial, razonamiento, tools |
| **Estado del entorno** | `environment_id` — archivos, paquetes, sandbox |

```python
# Nueva conversación, mismo entorno (archivos persistentes)
interaction = client.interactions.create(
    model="antigravity-preview-05-2026",
    input_text="Sigue trabajando en el proyecto",
    environment="env_abc123",  # mismo sandbox
    # Sin previous_interaction_id → conversación nueva
)
```

---

## 10. Límites y restricciones (Preview)

| Concepto | Límite |
|----------|--------|
| Máximo de managed agents | 1,000 |
| Entorno inactivo | Borrado tras 7 días |
| Profundidad de subagentes | **No soportado aún** |
| Versionado/Rollback | **No disponible aún** |
| Base agent disponible | Solo `antigravity-preview-05-2026` |
| Facturación | Pay-as-you-go (tokens + tools). Entorno no facturado en preview |

---

## 11. Frameworks compatibles para construir agentes

| Framework | Propósito |
|-----------|-----------|
| **LangChain / LangGraph** | Flujos multi-agente con estado (grafos) |
| **LlamaIndex** | RAG + datos privados |
| **CrewAI** | Agentes colaborativos rol-based |
| **Vercel AI SDK** | UI agents en JS/TS |
| **Google ADK** | Framework open-source para agentes interoperables |
| **Antigravity SDK** | Python SDK con el mismo harness que Antigravity |

---

## 12. Buenas prácticas

### Haz
- ✅ Usa `AGENTS.md` para instrucciones largas y versionables
- ✅ Separa skills en `.agents/skills/<nombre>/SKILL.md`
- ✅ Reusa `environment_id` para sesiones multi-turno con estado
- ✅ Usa `system_instruction` para ajustes rápidos por llamada
- ✅ Compactación automática a ~135k tokens — no te preocupes por context rot

### Evita
- ❌ No uses `previous_interaction_id` si solo quieres mantener archivos (usa `environment_id`)
- ❌ No subas archivos a `/` (root) como target
- ❌ No esperes subagentes anidados (no soportado en preview)
- ❌ No confíes ciegamente en preview — revisa acciones y outputs

---

## 13. Referencia rápida

```
Gemini API Managed Agents
─────────────────────────
Base agent ID: antigravity-preview-05-2026
Modelo: Gemini 3.5 Flash
SDK: Google Gen AI Python SDK

Archivos:
  .agents/AGENTS.md         → System instructions (auto-load)
  .agents/skills/*/SKILL.md → Skills (auto-descubiertas)

Llamadas:
  interactions.create()     → Una interacción (nuevo o mismo sandbox)
  agents.create()           → Persistir agente configurado
  agents.get() / agents.delete()

Parámetros clave:
  model                     → antigravity-preview-05-2026
  environment               → "remote" | environment_id | {...config}
  system_instruction        → Inline prompt adicional
  tools                     → [code_execution, google_search, url_context, ...]
  previous_interaction_id   → Continuar conversación
```
