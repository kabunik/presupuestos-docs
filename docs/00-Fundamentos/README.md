# Fundamentos del proyecto

> Base documental y referencial de la Plataforma de Presupuestos de Estructuras Metálicas.
> Estos documentos son la **capa de reconciliación** entre el sistema del consultor
> (Sistema DM v1.7) y la arquitectura de plataforma Kabunik × dcode.
> Última revisión: **2026-08-12**.

## Por qué existe esta carpeta

El 1 de agosto de 2026 el consultor (Ing. Daniel Michelena, Grupo Baysa / METALITEC) entregó el
paquete **Sistema DM v1.7**: un sistema de costeo cerrado, versionado y testeable, con 16
invariantes, contratos de datos en JSON Schema, un golden test que es ley y su propio plan de
desarrollo de 18 semanas.

Ese paquete no encajaba directamente con nuestros documentos de alcance ni con el primer mockup
de producto: había contradicciones de orden de construcción, de esquemas de datos y de
vocabulario. Esta carpeta resuelve ese desencaje y fija el punto de partida para construir la v1.

## Documentos

| # | Documento | Qué resuelve |
|---|---|---|
| 1 | [MAPA_DOCUMENTAL.md](MAPA_DOCUMENTAL.md) | Inventario de todas las fuentes del repo y **quién manda en qué dominio** |
| 2 | [RECONCILIACION_SistemaDM_x_Plataforma.md](RECONCILIACION_SistemaDM_x_Plataforma.md) | Las 10 tensiones detectadas y su resolución |
| 3 | [INVARIANTES_Y_COMPUERTAS.md](INVARIANTES_Y_COMPUERTAS.md) | Los 16 invariantes traducidos a requisitos de producto y UI |
| 4 | [CONTRATO_INTEGRACION_v0.md](CONTRATO_INTEGRACION_v0.md) | Borrador del mapeo de esquemas E1–E7 ↔ `Model`/`BOM`/`Offer` (insumo de C1) |
| 5 | [ACTIVOS_REUTILIZABLES.md](ACTIVOS_REUTILIZABLES.md) | Qué del material entregado se usa tal cual, y en qué ítem del backlog |
| 6 | [ALCANCE_v1.md](ALCANCE_v1.md) | Qué entra en la v1 y su inventario de pantallas |
| 7 | [GLOSARIO.md](GLOSARIO.md) | Vocabulario común (resuelve la ambigüedad «3 escenarios») |
| 8 | [CATALOGO_ALERTAS.md](CATALOGO_ALERTAS.md) | Catálogo de alertas y esquema `Alert`. Sustituye la cifra «22 alertas», que no tenía fuente |
| 9 | [PLAN_DE_MONTAJE.md](PLAN_DE_MONTAJE.md) | El plan de montaje como entrada obligatoria y **qué hace el sistema cuando el cliente no lo entrega** |
| 10 | [FLUJO_v1.md](FLUJO_v1.md) | **El orden del flujo, derivado de la cadena de dependencias del motor.** Los 5 bloques, las 5 compuertas en orden de ejecución y el ciclo de vida |
| 11 | [MODEL_DELTA_propuesta.md](MODEL_DELTA_propuesta.md) | **Propuesta** de formato de `model_delta` para C1.2. Pendiente de validar con dcode |
| 12 | [CAPA_MODELO.md](CAPA_MODELO.md) | **Frente abierto de la capa del modelo.** Lo decidido, la arquitectura de proyecciones y las decisiones pendientes |

Léelos en ese orden la primera vez.

- **8 y 9** (11-ago) desbloquearon el congelamiento del contrato v0: eran los dos huecos que cambiaban
  la forma del payload.
- **10** (11-ago) corrigió el orden del flujo, que violaba la cadena de dependencias del motor.
- **11 y 12** (11 y 12-ago) cubren el último hueco de forma del contrato y declaran el frente abierto
  de la capa del modelo.

## Decisiones de dirección — 2026-08-06

Estas dos decisiones gobiernan todo lo demás y **no se reabren** sin pasar por el canal de
decisiones (J1.3, issue #64):

### D1 · Sistema DM v1.7 es la especificación normativa de la capa determinista

El Sistema DM **no es un producto paralelo ni un insumo suelto**: es la especificación de la
aritmética que vive en las herramientas deterministas de dcode.

- Los **16 invariantes** son guardarraíles de producto, no recomendaciones. Se traducen a
  requisitos verificables en [INVARIANTES_Y_COMPUERTAS.md](INVARIANTES_Y_COMPUERTAS.md).
- `golden_test.json` es **compuerta de CI** de las tools (D3): un número distinto sin cambio
  aprobado de parámetros rompe el build.
- `input_schema.json` / `output_schema.json` son el **punto de partida del contrato de
  integración** (C1.1, issue #2). No se inventan esquemas desde cero: se mapean.
- El roadmap **F0 Toyota → F3 Ferrari se mantiene**. El Brief de Arranque del consultor
  (5 fases / 18 semanas) se absorbe como **plan interno de D3**, no como plan de proyecto.
- Consecuencia operativa: el visor 3D de F0 **no cotiza nada** hasta que existan las tools de F1.
  Lo que muestra en F0 es geometría y BOM, no precio.

### D2 · Alcance v1 = mockup + Sistema DM completo

La v1 cubre lo diseñado en el mockup **y** el núcleo de decisión de planta, que es donde el propio
consultor sitúa el diferenciador (*«no existe en software comercial integrado al costeo»*):

- Plan de montaje como entrada obligatoria (inv. 11).
- Compuerta de confirmación de precios de materia prima, con bitácora (inv. 8).
- Carga combinada de planta, ocupación de habilitado y escenarios de programa A/B/B\*/C (inv. 14).
- Alerta de interferencia de familias y subproyectos por familia (inv. 3).
- Cierre de proyecto y recalibración gobernada, con versionado de parámetros (inv. 16).
- Leyenda de alcance obligatoria en toda emisión.

Detalle en [ALCANCE_v1.md](ALCANCE_v1.md).

## Orden de resolución de conflictos

Cuando dos documentos se contradigan, manda el de más arriba. **No asumas cuál tiene razón:
si el conflicto no se resuelve con esta tabla, márcalo como issue `needs-definition`.**

| Dominio | Autoridad (de mayor a menor) |
|---|---|
| **Aritmética, invariantes, parámetros** | `motor_calculo_spec.md` → esquemas JSON del kit → `Ejemplo_Presupuesto_Sistema_DM.xlsx` → narrativa del Documento Maestro |
| **Cifras del caso de referencia** | `golden_test.json` (ley, tolerancia cero) |
| **Reparto de trabajo y arquitectura** | `Segmentacion_Tareas_Kabunik_dcode_v2.md` |
| **Contrato de integración** | El contrato v0 congelado (pendiente, C1.6) → [CONTRATO_INTEGRACION_v0.md](CONTRATO_INTEGRACION_v0.md) como borrador |
| **Alcance de la v1** | [ALCANCE_v1.md](ALCANCE_v1.md) → GitHub Project [kabunik/projects/2](https://github.com/orgs/kabunik/projects/2) |
| **Orden del flujo** | [FLUJO_v1.md](FLUJO_v1.md) — derivado del motor, no de preferencia de diseño |
| **Capa del modelo** | [CAPA_MODELO.md](CAPA_MODELO.md) — decidido vs. abierto |
| **Diseño visual y comportamiento de UI** | `docs/mockup/.../README.md` (hi-fi, medidas y copy finales) → `tokens.css` |

Nota: el consultor define su propio orden interno (*rector → kit → Excel → narrativa*). El
documento rector (`claude/instrucciones-sistema-dm.md` v1.7) **no está en este repo** — vive en el
proyecto de Claude del consultor. Está pedido; ver [ACTIVOS_REUTILIZABLES.md](ACTIVOS_REUTILIZABLES.md).

## Huecos abiertos

Los huecos de alcance detectados en esta revisión están abiertos como issues con label
`needs-definition`. No los rellenes por cuenta propia — ese es el punto de la etiqueta.
Lista viva: [issues con `needs-definition`](https://github.com/kabunik/presupuestos-docs/issues?q=is%3Aissue+label%3Aneeds-definition).
