# Presupuestos AI — Plataforma de Presupuestos de Estructuras Metálicas

> **Núcleo común.** Las secciones *Reglas inviolables (universales)*, *Flujo de trabajo*,
> *Definición de hecho*, *Antes de tocar código* y *Validación automática* son idénticas en
> todos los proyectos: no las edites por proyecto, edítalas en la plantilla compartida.
> El flujo completo (modelo, plan mode, sesiones paralelas) vive en `docs/modelo-trabajo-agentico.md`.

## Qué es

Plataforma SaaS multi-tenant para que fabricantes de estructuras metálicas generen presupuestos
calibrados contra su planta real. Arquitectura **agéntica**: un agente LLM (percepción de planos,
orquestación, edición del modelo) llama **herramientas deterministas** donde vive toda la aritmética
(ex-motor `baysa_presupuesto_v4.jsx`). El producto **no es un chat**: es un taller de ingeniería cuyo
protagonista es un **visor 3D navegable** con edición conversacional y reconstrucción en tiempo real.
Origen: know-how del consultor Daniel (METALITEC); cliente ancla: Grupo Baysa.

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
- Fases: **F0 Toyota** (intake + percepción one-shot + visor 3D) → **F1 Motor** (tools MCP, alertas, escenarios) → **F2 Edición viva** (chat + deltas + rebuild en tiempo real) → **F3 Ferrari** (lazo cotizado-vs-ejecutado, benchmarks, nesting, Strumis).

## Fuentes de verdad

- **Alcance:** `docs/Segmentacion_Tareas_Kabunik_dcode_v2.md` (v2 — segmentación completa de tareas).
- **Plan de trabajo:** `docs/HANDOFF_GitHub_Projects_Plan.md` → materializado en el GitHub Project
  [kabunik/projects/2 "Presupuestos AI"](https://github.com/orgs/kabunik/projects/2).
- Este workspace es el **repositorio documental** del proyecto: aquí viven specs, handoffs y decisiones.

## Arquitectura — repos en este workspace

| Repo | Lenguaje | Rol |
|---|---|---|
| (este workspace) | Markdown | Repositorio documental: specs, handoffs, contrato de integración |
| *(pendiente)* repo de plataforma | TBD | Frontend + backend Kabunik (visor 3D, intake, estado canónico) |
| *(dcode, espacio propio)* | Python | Agente + tools MCP — aquí solo tracking issues (`workstream:dcode`) |

## El contrato de integración (la costura)

Bloqueante para el paralelismo (EPIC C1, F0, P0). Gira en torno a una **sesión de proyecto con estado de modelo**:
- Esquemas compartidos versionados: `TenantConfig`, `Model`, `BOM`, `Alert`, `Scenario`, `Offer`, `Benchmark`. Sin cambios unilaterales.
- Plataforma→Agente: `POST /project-session`, `/generate`, `/edit`, `/scenarios`, `/bom-approval`; streaming `progress`, `tool_call.*`, `alert.raised`, `model.updated`, `human_gate.bom_review_required`, `result.ready`.
- Agente→Plataforma: `get_config_tenant`, `get_benchmarks`, `disponibilidad_perfiles`, `registrar_cierre`.
- Formato de `model_delta`: si no se acuerda, visor y agente se desincronizan (riesgo #1).

## Reglas inviolables (universales)

- **No commits a `main`/`master` directos.** Branch + PR, siempre.
- **Nunca commitear secrets ni `.env`.** Sensibles vía `direnv`/entorno, jamás en el repo.
- **Tests pasan antes de pedir review.** Si no pasan, dilo explícitamente.

## Reglas inviolables (de este proyecto)

- **No inventar alcance:** si se detecta un hueco, issue con label `needs-definition`; no rellenarlo por cuenta propia.
- **No detallar el backlog interno de dcode:** sus ítems son tracking issues; ellos planifican su detalle.
- **No inventar estimaciones:** el campo `Estimación` lo llena cada equipo.
- **Cambios al contrato de integración** solo por el canal de decisiones acordado (J1.3), nunca unilaterales.

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

## Cómo levantar entorno local

```bash
# Pendiente: sin entorno de ejecución aún (workspace documental).
```

## Convenciones cross-repo

- El **contrato de integración** (esquemas + API + eventos) se documenta en este repo documental y se versiona; todo cambio requiere acuerdo Kabunik×dcode.
- Los issues del plan viven en GitHub Project [kabunik/projects/2](https://github.com/orgs/kabunik/projects/2) con campos `Workstream` / `Fase` / `Área` / `Prioridad` y labels `workstream:*`, `epic`, `needs-definition`, `blocked`, `integration-contract`, `security-tenant`.

## Antes de tocar código

Lee siempre `docs/` para entender el feature — en particular `Segmentacion_Tareas_Kabunik_dcode_v2.md`
(alcance) y `HANDOFF_GitHub_Projects_Plan.md` (plan). Si una spec contradice el código, marca el
conflicto en lugar de asumir cuál tiene razón.

## Validación automática

Hooks que corren al editar: linter por archivo (PostToolUse) y tests al cerrar turno (Stop). Si recibes feedback de lint/test tras una edición, corrígelo antes de continuar.
