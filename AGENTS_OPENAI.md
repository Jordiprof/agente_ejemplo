# AGENT.md

## Propósito

Eres un agente de desarrollo que trabaja sobre este repositorio.

Tu objetivo es implementar funcionalidades, corregir errores y mejorar el código respetando la arquitectura y convenciones existentes.

Prioriza:

1. Correctitud
2. Seguridad
3. Mantenibilidad
4. Simplicidad
5. Rendimiento

---

## Contexto del proyecto

### Descripción

[Explicación breve del producto]

### Usuarios objetivo

[Quién utiliza el sistema]

### Objetivos principales

* [Objetivo 1]
* [Objetivo 2]
* [Objetivo 3]

---

## Stack tecnológico

### Lenguaje

* [Lenguaje y versión]

### Frameworks

* [Framework principal]

### Base de datos

* [Tecnología]

### Testing

* [Framework de tests]

### Infraestructura

* [Docker, AWS, Railway, Vercel, etc.]

---

## Comandos importantes

```bash
# Desarrollo
[comando]

# Tests
[comando]

# Lint
[comando]

# Build
[comando]

# Typecheck
[comando]
```

Todos los tests y validaciones deben pasar antes de considerar una tarea terminada.

---

## Arquitectura

### Principios

* Mantener separación de responsabilidades.
* Evitar duplicación innecesaria.
* Favorecer composición sobre herencia.
* Mantener bajo acoplamiento.

### Reglas

* Controllers → reciben peticiones.
* Services → contienen lógica de negocio.
* Repositories → acceso a datos.
* Models/Entities → representan el dominio.

No saltarse capas.

---

## Estructura del repositorio

```text
src/
├── controllers/
├── services/
├── repositories/
├── models/
├── utils/
└── tests/
```

### Descripción

* controllers/: entrada de la aplicación
* services/: lógica de negocio
* repositories/: persistencia
* models/: entidades y tipos
* utils/: utilidades compartidas
* tests/: pruebas

---

## Convenciones de código

### Estilo

* camelCase para variables y funciones
* PascalCase para clases y tipos
* UPPER_SNAKE_CASE para constantes globales

### TypeScript

* strict mode obligatorio
* evitar any
* preferir tipos explícitos

### Manejo de errores

* usar errores tipados
* nunca silenciar excepciones
* registrar errores relevantes

### Logging

* logs estructurados
* no registrar secretos

---

## Alcance de los cambios

Haz el cambio mínimo necesario.

Evita:

* mover archivos sin necesidad
* cambiar APIs públicas sin motivo
* modificar arquitectura existente
* introducir nuevas dependencias

Salvo que la tarea lo requiera explícitamente.

---

## Seguridad

Nunca:

* expongas secretos
* hardcodees credenciales
* subas archivos .env
* desactives validaciones de seguridad

Valida siempre cualquier entrada externa.

---

## Cuando falte contexto

Antes de asumir:

1. Busca en el repositorio.
2. Revisa código similar.
3. Revisa documentación existente.

Si la incertidumbre sigue siendo alta, pregunta.

No inventes:

* APIs
* tablas
* esquemas
* endpoints
* variables de entorno

---

## Flujo de trabajo

### Cambios pequeños

Puedes ejecutarlos directamente.

### Cambios medianos o grandes

Presenta primero:

* objetivo
* plan
* archivos afectados
* riesgos

Y espera confirmación.

---

## Definition of Done

Una tarea está terminada cuando:

* compila correctamente
* tests pasan
* lint pasa
* typecheck pasa
* no introduce warnings nuevos
* respeta las convenciones
* actualiza documentación cuando corresponde

---

## Zonas protegidas

No modificar:

* [ruta]
* [ruta]
* [ruta]

Salvo autorización explícita.

---

## Documentación adicional

* docs/
* ADRs
* RFCs
* README.md
* especificaciones de producto
