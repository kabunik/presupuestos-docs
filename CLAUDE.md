# Presupuestos AI — Plataforma de Presupuestos de Estructuras Metálicas

> **Núcleo común.** Las secciones *Reglas inviolables (universales)*, *Flujo de trabajo*,
> *Definición de hecho*, *Antes de tocar código* y *Validación automática* son idénticas en
> todos los proyectos: no las edites por proyecto, edítalas en la plantilla compartida.
> El flujo completo (modelo, plan mode, sesiones paralelas) vive en `docs/modelo-trabajo-agentico.md`.

## Qué es

Plataforma SaaS multi-tenant para que fabricantes de estructuras metálicas generen presupuestos
calibrados contra su planta real. Arquitectura **agéntica**: un agente LLM (percepción de planos,
orquestación, edición del modelo) llama **herramientas deterministas** donde vive toda la aritmética.
El producto **no es un chat**: es un taller de ingeniería cuyo protagonista es un **visor 3D
navegable** con edición conversacional y reconstrucción en tiempo real.
Origen: know-how del consultor Daniel Michelena (METALITEC); cliente ancla: Grupo Baysa.

La aritmética está especificada por el **Sistema DM v1.7** del consultor: 16 invariantes, contratos
en JSON Schema y un golden test que es ley. **Lee `docs/00-Fundamentos/` antes de tocar cualquier
cosa** — ahí vive la reconciliación entre ese sistema y esta plataforma.

## Equipos y reparto (workstreams)

| Workstream | Responsable | Alcance |
|---|---|---|
| **dcode** (partner externo) | Agente + herramientas | Agente orquestador, percepción propia (Opción A: planos PDF/DXF → modelo), edición por deltas, tools deterministas vía MCP, eval harness |
| **Kabunik** (nosotros) | Plataforma | Diseño/UI, visor 3D + reconstrucción en tiempo real, chat de edición, intake, escenarios, onboarding/calibración, usuarios/licencias/multitenancy, backend con **estado canónico del modelo**, puente a Strumis, infra |
| **Conjunto** | Ambos + Daniel | Contrato de integración (la costura), esquemas versionados, seguridad de tenant, IP, casos de evaluación |

## Decisiones cerradas — NO reabrir

- Percepción **Opción A** (propia, de dcode); contingencia híbrida (ingesta de takeoff externo) solo como puerta abierta en el contrato, no como alcance.
- El **modelo canónico vive en la plataforma** (Kabunik); el agente aplica **deltas**; recálculo **incremental** (solo lo afectado).
- **Compuerta humana** en la validación del BOM; regla **"ninguna cifra sin tool"**.
- Frontera ERP: el sistema decide y planifica, **no administra** (Strumis/SAP aguas abajo vía puente de exportación). No crear alcance ERP.
- Fases: **F0 Toyota** (intake + percepción one-shot + visor 3D) → **F1 Motor** (tools MCP, alertas, opciones de oferta) → **F2 Edición viva** (chat + deltas + rebuild en tiempo real) → **F3 Ferrari** (benchmarks, nesting, Strumis, RAG multicliente).
- **(2026-08-06) Sistema DM v1.7 es la especificación normativa de la capa determinista.** Sus 16
  invariantes son guardarraíles de producto; `golden_test.json` es compuerta de CI; sus esquemas son
  el punto de partida del contrato C1. El Brief de 18 semanas del consultor se absorbe como plan
  interno de D3, **no** como plan de proyecto.
- **(2026-08-06) Alcance v1 = mockup + Sistema DM completo.** Incluye el núcleo de decisión de planta
  (S11/S12, ocupación de habilitado, escenarios de programa, interferencia de familias) y el cierre
  gobernado con versionado de parámetros. Detalle en `docs/00-Fundamentos/ALCANCE_v1.md`.
- **Recalibración automática de parámetros: prohibida** (inv. 16). No es diferida — nunca entra.

## Fuentes de verdad

**Empieza siempre por `docs/00-Fundamentos/README.md`.** Es la puerta de entrada y contiene el orden
de resolución de conflictos completo.

| Dominio | Autoridad |
|---|---|
| Aritmética, invariantes, parámetros | `docs/Contexto completo Michelena/Kit_Desarrollo_Sistema_DM.zip` → `motor_calculo_spec.md` + los 2 JSON Schema · luego `Ejemplo_Presupuesto_Sistema_DM.xlsx` · luego la narrativa del Documento Maestro |
| Cifras del caso de referencia | `golden_test.json` — **ley, tolerancia cero** |
| Reparto de trabajo y arquitectura | `docs/Diseño de producto/Segmentacion_Tareas_Kabunik_dcode_v2.md` |
| Alcance y plan de la v1 | `docs/00-Fundamentos/ALCANCE_v1.md` → GitHub Project [kabunik/projects/2](https://github.com/orgs/kabunik/projects/2) |
| Contrato de integración | Contrato v0 congelado (pendiente, C1.6) → `docs/00-Fundamentos/CONTRATO_INTEGRACION_v0.md` como borrador |
| Diseño visual y comportamiento de UI | `docs/mockup/design_handoff_plataforma_presupuestos/README.md` + `tokens.css` |
| Frontera de alcance (ERP) | `docs/doc inicial/Hoja_de_Ruta_del_Producto.pdf` |
| Vocabulario | `docs/00-Fundamentos/GLOSARIO.md` |

Este workspace es el **repositorio documental** del proyecto: aquí viven specs, handoffs y decisiones.
El documento rector del consultor (`claude/instrucciones-sistema-dm.md` v1.7) **no está aquí** —
vive en su proyecto de Claude y está pedido.

## Arquitectura — repos en este workspace

| Repo | Lenguaje | Rol |
|---|---|---|
| (este workspace) | Markdown | Repositorio documental: specs, handoffs, contrato de integración |
| *(pendiente)* repo de plataforma | TBD | Frontend + backend Kabunik (visor 3D, intake, estado canónico) |
| *(dcode, espacio propio)* | Python | Agente + tools MCP — aquí solo tracking issues (`workstream:dcode`) |

## El contrato de integración (la costura)

Bloqueante para el paralelismo (EPIC C1, F0, P0). Gira en torno a una **sesión de proyecto con estado de modelo**:
- Esquemas compartidos versionados: `TenantConfig`, `Model`, `BOM`, `Alert`, `Offer`, `Benchmark` —
  **mapeados desde `input_schema.json`/`output_schema.json` del kit, no diseñados de cero.**
  Sin cambios unilaterales.
- Plataforma→Agente: `POST /project-session`, `/generate`, `/edit`, `/scenarios`, `/bom-approval`,
  `/price-confirmation` (inv. 8), `/program-decision` (inv. 14); streaming `progress`, `tool_call.*`,
  `alert.raised`, `model.updated`, `human_gate.*`, `emission.blocked`, `result.ready`.
- Agente→Plataforma: `get_config_tenant(tenant_id, version?)`, `get_benchmarks`,
  `disponibilidad_perfiles`, `registrar_cierre`, `get_plant_load`, `get_price_list`.
- Formato de `model_delta`: si no se acuerda, visor y agente se desincronizan (riesgo #1).
  **`model_delta` opera sobre `Model`; el take-off E2 se deriva de él, nunca al revés.**

Borrador de mapeo y de la API ampliada: `docs/00-Fundamentos/CONTRATO_INTEGRACION_v0.md`.

## Reglas inviolables (universales)

- **No commits a `main`/`master` directos.** Branch + PR, siempre.
- **Nunca commitear secrets ni `.env`.** Sensibles vía `direnv`/entorno, jamás en el repo.
- **Tests pasan antes de pedir review.** Si no pasan, dilo explícitamente.

## Reglas inviolables (de este proyecto)

- **No inventar alcance:** si se detecta un hueco, issue con label `needs-definition`; no rellenarlo por cuenta propia.
- **No detallar el backlog interno de dcode:** sus ítems son tracking issues; ellos planifican su detalle.
- **No inventar estimaciones:** el campo `Estimación` lo llena cada equipo.
- **Cambios al contrato de integración** solo por el canal de decisiones acordado (J1.3), nunca unilaterales.
- **Los 16 invariantes del Sistema DM son ley.** Violar uno es un defecto, no una preferencia.
  Traducción a requisitos verificables: `docs/00-Fundamentos/INVARIANTES_Y_COMPUERTAS.md`.
- **Ninguna cifra sin tool.** El agente no hace aritmética; toda cifra sale de una herramienta
  determinista y es trazable a ella.
- **Ninguna alerta no-supresible lleva control de silencio.** `advertencia_bajo_equilibrio`,
  `alerta_ocupacion`, `alerta_interferencia_familias`, `advertencia_comunicacion_cliente` y
  `confirmar_precios_mp` no tienen switch. Las dos últimas **bloquean la emisión**.
- **Nada de «cero silencioso»:** las exclusiones (escaleras, F1554, Nelson studs) se declaran
  siempre de forma explícita, nunca como cero (inv. 5).
- **Todo el motor trabaja antes de impuestos.** IVA/ISR solo en la capa de presentación de la oferta.
- **No inventar cifras de dominio.** Si un número no está en el golden test, en un caso real o en el
  Excel del consultor, no lo escribas. La cifra «22 alertas» que circula en documentos antiguos
  **no tiene fuente** — no la propagues.

## Flujo de trabajo

- **Una tarea = una rama = un PR.** No mezcles tareas en una rama.
- **Nomenclatura:** `feature/<issue>-desc`, `fix/<issue>-desc`, `chore/<issue>-desc`.
  El `<issue>` enlaza la rama con el GitHub Project.
- **Toda tarea parte de un issue.** Si no existe, créalo antes de empezar.
- **Cada rama sale de un `main` actualizado y el PR va contra `main`.** Antes de crear la rama: `git checkout main && git pull`. La `base` del PR es **siempre `main`**.
- **Un PR a la vez (secuencial).** Mergea el PR → `git checkout main && git pull` → recién entonces arranca el siguiente. **No** empieces una tarea brancheando desde otra rama de feature aún sin mergear.
- **No apilar PRs.** Si una tarea depende de otra sin mergear, espera a que se integre. Si apilar es inevitable: la `base` del PR dependiente debe ser **`main`** (no la rama padre); mergea primero el padre, **rebasa el hijo sobre el `main` actualizado** y verifica que el diff sea **solo tu cambio** antes de mergear.
- **⛔ Nunca mergees un PR cuya `base` sea otra rama de feature** — se integra en esa rama, no en `main`. (Pasó con `validar-desktop` #19/#20: quedaron dentro de la rama del backbone y hubo que hacer un PR de consolidación a `main`.)
- **Tras mergear:** borra la rama integrada (local y remota) y haz `git pull` de `main` antes de la siguiente tarea.
- **Tareas en paralelo sobre el mismo repo → git worktree**, nunca dos sesiones sobre el mismo working tree:
  ```bash
  git worktree add ../<repo>-<feature> -b feature/<issue>-<desc>
  # al terminar:
  git worktree remove ../<repo>-<feature>
  ```
- **Ante ambigüedad o bloqueo, detente y márcalo** en el issue/PR. No improvises una solución que se aleje de la spec.

## Definición de "hecho" (DoD)

Un PR está listo para review cuando:
- [ ] Tests verdes y lint limpio (ver hooks).
- [ ] Pasa la revisión de calidad (code-reviewer / skill `senior-qa`).
- [ ] La descripción enlaza el issue (`Closes #<issue>`) y la spec en docs.
- [ ] Si tocaste un contrato entre servicios, su spec se actualizó en este mismo PR.
- [ ] PR pequeño y enfocado (una sola tarea).

## Stack y tests

- Sin código todavía — fase de definición y planificación. Se actualizará al crear los repos de plataforma.
- **Recomendación del handoff de diseño** (no decisión: pendiente del spike K2.1, issue #13):
  React + TypeScript + Vite/Next, tabla virtualizada tipo TanStack Table, visor 3D con
  three.js + web-ifc / IFC.js. Tokens listos en
  `docs/mockup/design_handoff_plataforma_presupuestos/tokens.css`.
- **Cuando exista el repo de tools:** `golden_test.json` corre en CI en cada commit. Un número
  distinto sin cambio aprobado de parámetros = **build roto**. Es criterio de aceptación permanente,
  no una prueba más.

## Cómo levantar entorno local

```bash
# Pendiente: sin entorno de ejecución aún (workspace documental).
```

## Convenciones cross-repo

- El **contrato de integración** (esquemas + API + eventos) se documenta en este repo documental y se versiona; todo cambio requiere acuerdo Kabunik×dcode.
- Los issues del plan viven en GitHub Project [kabunik/projects/2](https://github.com/orgs/kabunik/projects/2) con campos `Workstream` / `Fase` / `Área` / `Prioridad` y labels `workstream:*`, `epic`, `needs-definition`, `blocked`, `integration-contract`, `security-tenant`.

## Antes de tocar código

Lee siempre `docs/` para entender el feature. Orden recomendado:

1. `docs/00-Fundamentos/README.md` — puerta de entrada, decisiones cerradas y orden de autoridad.
2. `docs/00-Fundamentos/RECONCILIACION_SistemaDM_x_Plataforma.md` — por qué las cosas son como son.
3. `docs/00-Fundamentos/INVARIANTES_Y_COMPUERTAS.md` — si el feature toca cálculo, alertas o emisión.
4. `docs/Diseño de producto/Segmentacion_Tareas_Kabunik_dcode_v2.md` — alcance y reparto.
5. `docs/mockup/design_handoff_plataforma_presupuestos/README.md` — si el feature toca UI.

Si una spec contradice el código, o dos documentos se contradicen entre sí, **marca el conflicto**
en lugar de asumir cuál tiene razón: usa la tabla de autoridad de *Fuentes de verdad* y, si no la
resuelve, abre un issue `needs-definition`.

## Validación automática

Hooks que corren al editar: linter por archivo (PostToolUse) y tests al cerrar turno (Stop). Si recibes feedback de lint/test tras una edición, corrígelo antes de continuar.
