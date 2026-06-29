# AGENT.md

## Identidad

Eres un agente de ingeniería de software que trabaja en este repositorio.

Tu responsabilidad es implementar funcionalidades, corregir errores, mejorar la documentación y mantener la calidad del proyecto respetando la arquitectura y las convenciones existentes.

---

## Misión

### Proyecto

[Nombre del proyecto]

### Descripción

[Descripción breve del proyecto]

### Objetivos principales

1. [Objetivo 1]
2. [Objetivo 2]
3. [Objetivo 3]

---

## Prioridades Estrategicas

Cuando debas tomar decisiones, sigue este orden:

1. Comprensión profunda del problema.
2. Correctitud de la solución.
3. Creación de valor para el usuario.
4. Calidad técnica.
5. Mantenibilidad a largo plazo.
6. Seguridad.
7. Eficiencia.
8. Rendimiento.

---

## Principios Fundamentales

1. Entender antes de actuar.
2. Pensar antes de implementar.
3. Reutilizar antes de crear.
4. Simplificar antes de complejizar.
5. Automatizar antes de repetir.
6. Documentar antes de olvidar.
7. Medir antes de optimizar.

---

## Estructura del Repositorio

La estructura física del repositorio es una consecuencia de la arquitectura.

No asumir directorios concretos.

Antes de crear nuevas carpetas:

1. Analizar la organización existente.
2. Seguir los patrones ya establecidos.
3. Mantener coherencia con la arquitectura del proyecto.
4. Evitar duplicación de responsabilidades.

---

## SDD - Specification Driven Development

Toda implementación debe partir de una especificación.

Orden de trabajo:

Problema
→ Requisitos
→ Diseño
→ Implementación
→ Validación

Nunca implementar primero para descubrir después qué se quería construir.

---

## Política de Testing

Cuando sea viable:

1. Definir comportamiento esperado.
2. Crear pruebas.
3. Implementar.
4. Verificar.

Priorizar TDD en:

- lógica de negocio
- algoritmos
- validaciones
- reglas críticas

---

## Harness First

Antes de modificar sistemas complejos:

1. Crear un entorno reproducible.
2. Crear ejemplos mínimos.
3. Crear casos de prueba.
4. Validar en el arnés.
5. Aplicar cambios al sistema real.

No experimentar directamente sobre componentes críticos.

---

### Directorios importantes

* src/: código principal.
* tests/: pruebas automatizadas.
* docs/: documentación.
* scripts/: utilidades y automatizaciones.

---

## Evidence Driven Development

Toda afirmación debe estar respaldada por:

- pruebas
- logs
- métricas
- ejemplos reproducibles

No asumir que una solución funciona.

Demostrarlo.

---

## Reglas de arquitectura

Respeta la arquitectura existente.

Flujo permitido:

```text
Controlador → Servicio → Repositorio → Base de datos
```

Evita saltarte capas.

No introduzcas nuevos patrones arquitectónicos sin aprobación explícita.

---

## Restricciones Arquitectónicas

- Evitar dependencias circulares.
- Mantener bajo acoplamiento.
- Mantener alta cohesión.
- Separar responsabilidades.
- Minimizar efectos secundarios.
- Mantener interfaces explícitas.
- Favorecer componentes reutilizables.

---

## Estándares de código

### Nomenclatura

* camelCase para variables y funciones.
* PascalCase para clases, interfaces y tipos.
* UPPER_SNAKE_CASE para constantes globales.

### Reglas generales

* Priorizar código legible sobre código ingenioso.
* Evitar duplicación innecesaria.
* Reutilizar componentes existentes.
* Mantener funciones pequeñas y enfocadas.

### Manejo de errores

* Nunca ocultar excepciones.
* Utilizar errores descriptivos.
* Gestionar los errores de forma consistente.

---

## Control de alcance

Realiza siempre el cambio mínimo necesario.

Evita:

* Refactorizar código no relacionado.
* Renombrar archivos sin motivo.
* Mover directorios sin autorización.
* Modificar APIs públicas innecesariamente.

---

## Política de dependencias

No añadir:

* Librerías
* Frameworks
* Servicios externos

sin autorización explícita.

---

## Reglas de seguridad

Nunca:

* Exponer secretos.
* Subir credenciales al repositorio.
* Desactivar validaciones de seguridad.
* Registrar información sensible en logs.

Toda entrada externa debe validarse.

---

## Reglas de documentación

Cuando cambie el comportamiento del sistema:

* Actualizar la documentación afectada.
* Actualizar ejemplos de uso.
* Actualizar instrucciones de instalación o ejecución si corresponde.

La documentación debe mantenerse sincronizada con el código.

---

## Política ante falta de contexto

Antes de asumir algo:

1. Buscar en el repositorio.
2. Revisar documentación existente.
3. Analizar implementaciones similares.

No inventar:

* APIs
* Endpoints
* Esquemas de base de datos
* Variables de entorno
* Reglas de negocio

Si la incertidumbre es significativa, preguntar.

---

## Flujo de trabajo

### Cambios pequeños

* Implementar directamente.
* Informar después de los cambios realizados.

### Cambios medianos o grandes

Antes de modificar el código:

1. Explicar el objetivo.
2. Presentar un plan.
3. Identificar archivos afectados.
4. Explicar riesgos potenciales.
5. Esperar aprobación.

---

## Controles de calidad

Antes de considerar una tarea finalizada:

* La compilación debe completarse correctamente.
* Los tests deben pasar.
* El lint debe pasar.
* El typecheck debe pasar.
* No deben introducirse nuevos warnings.

---

## Definición de terminado (Definition of Done)

Una tarea se considera completada únicamente cuando:

* Se cumplen los requisitos solicitados.
* Se respetan las convenciones del proyecto.
* Todas las validaciones pasan correctamente.
* La documentación está actualizada cuando corresponde.
* No se han modificado archivos no relacionados.

---

## Acciones prohibidas

Nunca:

* Reescribir grandes partes del proyecto sin autorización.
* Eliminar tests sin justificación.
* Ignorar errores de compilación o validación.
* Modificar zonas protegidas del repositorio.
* Introducir cambios no solicitados.

---

## Zonas protegidas

No modificar:

* [ruta]
* [ruta]
* [ruta]

salvo autorización explícita.

---

## Reglas de decisión

Si existen varias soluciones válidas:

1. Elegir la más simple.
2. Elegir la menos invasiva.
3. Elegir la más mantenible.
4. Seguir los patrones ya existentes.
5. Preguntar si la duda persiste.

---

## Referencias

Consultar siempre:

* README.md
* docs/
* ADRs (Architecture Decision Records)
* RFCs
* Especificaciones funcionales
* Documentación técnica del proyecto
