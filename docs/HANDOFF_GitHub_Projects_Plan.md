# HANDOFF — Plan de trabajo en GitHub Projects
### Proyecto: Plataforma de Presupuestos de Estructuras Metálicas (Kabunik × dcode)
**Destinatario:** Claude Code · **Autor:** Juan (Kabunik) · **Objetivo:** crear el plan de alto nivel del proyecto en GitHub Projects

---

## 0. Instrucciones para Claude Code

Tu tarea es **crear y poblar un GitHub Project (v2)** con el plan de alto nivel descrito en este documento. Antes de ejecutar nada, confirma con el usuario:

1. **Organización/owner y repositorio(s)** donde vivirá el Project y los issues (¿un repo `platform` de Kabunik, un repo `agent` para tracking de dcode, o un solo repo mono?). Recomendación por defecto: un Project a nivel de organización + issues en el repo de plataforma; los ítems de dcode como issues de tracking (label `workstream:dcode`).
2. **Permisos:** verificar que `gh` está autenticado con scopes `project` y `repo` (`gh auth status`).
3. Si algún epic no aplica todavía, créalo igual como issue "epic" vacío — el objetivo es que el plan completo quede visible.

Usa `gh project create`, `gh project field-create`, `gh issue create` y `gh project item-add`. Crea primero campos y milestones, luego epics, luego issues hijos, y al final asigna campos. No inventes alcance nuevo: si detectas un hueco, márcalo como issue con label `needs-definition` en vez de rellenarlo por tu cuenta.

---

## 1. Contexto del proyecto (resumen autocontenido)

**Qué se construye:** una plataforma SaaS multi-tenant para que fabricantes de estructuras metálicas generen presupuestos calibrados contra su planta real. El sistema parte del know-how de un consultor (Daniel/METALITEC, cliente ancla: Grupo Baysa) que hoy opera como proyecto de Claude, y se productiza en una arquitectura **agéntica**: un agente LLM (percepción + orquestación + edición) que llama **herramientas deterministas** (toda la aritmética; ex-motor `baysa_presupuesto_v4.jsx`).

**Visión de producto (norte):** NO es un chat. Es un taller de ingeniería donde el usuario:
1. Carga información (planos PDF/DXF, contrato, datos de proyecto).
2. Recibe como primer output la **solución estructural en un visor 3D navegable**.
3. **Edita conversacionalmente** el modelo ("cambia columnas del N3 a HSS") y ve la **reconstrucción en tiempo real** en el visor.
4. Genera **escenarios comerciales** (precios, flujo de caja, ingeniería de valor) y valida iterativamente antes de emitir la oferta.

**Reparto de equipos:**
- **dcode** (partner externo, especialista en agentes): agente orquestador, percepción propia (planos→modelo, Opción A confirmada), edición del modelo por deltas, herramientas deterministas (MCP), eval harness.
- **Kabunik** (nosotros): plataforma — diseño/UI, visor 3D con reconstrucción en tiempo real, chat de edición, intake, escenarios, onboarding/calibración por fabricante, usuarios/licencias/multitenancy, backend con **estado canónico del modelo**, puente a Strumis, infraestructura.
- **Conjunto:** contrato de integración (la costura), esquemas versionados, seguridad de tenant, IP.

**Decisiones ya cerradas (no reabrir):**
- Percepción **Opción A** (propia, por dcode), con contingencia híbrida (ingesta de takeoff externo) como puerta abierta en el contrato, no como alcance.
- El **modelo canónico vive en la plataforma**; el agente aplica **deltas**; recálculo **incremental** (solo lo afectado).
- **Compuerta humana** en la validación del BOM; regla "ninguna cifra sin tool".
- Frontera ERP: el sistema decide y planifica; no administra (Strumis/SAP quedan aguas abajo, conectados por un puente de exportación).
- Fases: 0 Toyota → 1 Motor y escenarios → 2 Edición conversacional en vivo → 3 Ferrari.

**Referencia completa:** documento `Segmentacion_Tareas_Kabunik_dcode_v2.md` (fuente de verdad del alcance).

---

## 2. Estructura del GitHub Project

### 2.1 Campos personalizados
| Campo | Tipo | Valores |
|---|---|---|
| `Workstream` | single select | `Kabunik` · `dcode` · `Conjunto` |
| `Fase` | single select | `F0 Toyota` · `F1 Motor` · `F2 Edición viva` · `F3 Ferrari` |
| `Área` | single select | `Agente` · `Percepción` · `Tools` · `Visor 3D` · `Frontend` · `Backend/Estado` · `Integración` · `Calibración` · `Licencias/Multitenancy` · `Infra` · `Producto/Diseño` · `Eval/QA` |
| `Prioridad` | single select | `P0 bloqueante` · `P1 alta` · `P2 media` · `P3 baja` |
| `Estimación` | number | (puntos o días — lo define cada equipo) |

### 2.2 Milestones (mapean las fases)
- **M0 — Toyota:** contrato de integración v0 congelado · percepción mínima one-shot · intake + visor 3D del primer output · validación con 1–2 clientes.
- **M1 — Motor y escenarios:** herramientas deterministas completas vía MCP · panel de alertas · escenarios y flujo de caja · grilla BOM editable · streaming.
- **M2 — Edición conversacional en vivo:** agente stateful con deltas y recálculo incremental · chat de edición · reconstrucción del visor en tiempo real · UX de validación iterativa.
- **M3 — Ferrari:** lazo cotizado-vs-ejecutado · benchmarks agregados · optimizador de nesting · puente a Strumis · analítica.

### 2.3 Labels
`workstream:kabunik` · `workstream:dcode` · `workstream:conjunto` · `epic` · `needs-definition` · `blocked` · `integration-contract` · `security-tenant`

### 2.4 Vistas
1. **Board por Fase** (columnas = Fase) — vista de roadmap.
2. **Tabla por Workstream** (agrupada por Workstream, ordenada por Prioridad) — vista de coordinación con dcode.
3. **Board por Estado** (Todo / In progress / Blocked / Done) — vista de ejecución.

---

## 3. Backlog inicial (epics → issues)

> Convención: `[EPIC]` en el título de los epics; los issues hijos se enlazan con task-lists en el cuerpo del epic. Los ítems de dcode se crean como **tracking issues** (dcode planifica el detalle en su propio espacio; aquí se sigue el hito y la interfaz).

### EPIC C1 · Contrato de integración (Conjunto · F0 · P0) `integration-contract`
El artefacto que desbloquea el paralelismo. **Todo lo demás depende de esto.**
- C1.1 Definir esquemas compartidos versionados: `TenantConfig`, `Model`, `BOM`, `Alert`, `Scenario`, `Offer`, `Benchmark`.
- C1.2 Definir formato de `model_delta` (mutaciones incrementales del modelo canónico).
- C1.3 Definir API de sesión: `POST /project-session`, `/generate`, `/edit`, `/scenarios`, `/bom-approval`.
- C1.4 Definir eventos de streaming: `progress`, `tool_call.*`, `alert.raised`, `model.updated`, `human_gate.bom_review_required`, `result.ready`.
- C1.5 Definir servicios de datos plataforma→agente: `get_config_tenant`, `get_benchmarks`, `disponibilidad_perfiles`, `registrar_cierre`.
- C1.6 Congelar contrato v0 + política de versionado (sin cambios unilaterales).
- *Criterio de terminado:* documento de contrato v0 firmado por ambos equipos y publicado en el repo.

### EPIC K1 · Intake y workspace de proyecto (Kabunik · F0 · P1)
- K1.1 Wizard de alta de proyecto + carga de archivos (PDF/DXF/IFC, contrato).
- K1.2 Workspace de proyecto (layout base, navegación, estados de sesión).
- K1.3 Notificaciones de progreso del job.

### EPIC K2 · Visor 3D (Kabunik · F0→F2 · P0)
- K2.1 Selección tecnológica del visor (IFC.js / Three.js / otro) — spike con criterio de latencia de F2. `needs-definition`
- K2.2 Visor navegable del primer output (F0).
- K2.3 Render desde el estado canónico del modelo (F1).
- K2.4 Aplicación de `model_delta` en vivo — reconstrucción en tiempo real (F2).

### EPIC K3 · Backend de plataforma y estado canónico (Kabunik · F0→F1 · P0)
- K3.1 API de plataforma (proyectos, sesiones, archivos).
- K3.2 Almacén del **modelo canónico** por sesión + aplicación de deltas.
- K3.3 Orquestación del agente dcode (jobs, streaming, compuerta humana).
- K3.4 Servicios de datos para tools de dcode (`get_config_tenant`, `get_benchmarks`, `registrar_cierre`).
- K3.5 Gestión de exportables (oferta docx, IFC).

### EPIC K4 · Superficies de estimación (Kabunik · F1 · P1)
- K4.1 Grilla editable de BOM + compuerta de revisión (human gate UI).
- K4.2 Panel de validación — 22 alertas.
- K4.3 Tarjetas comparativas de escenarios.
- K4.4 Gráficos de flujo de caja.
- K4.5 Checklist de ingeniería de valor + preview/descarga de oferta.

### EPIC K5 · Chat de edición conversacional (Kabunik · F2 · P1)
- K5.1 UI del chat como "volante del modelo" (no interfaz principal).
- K5.2 Flujo petición → progreso → delta reflejado en visor.
- K5.3 UX de validación iterativa y comparación de outputs.

### EPIC K6 · Onboarding y calibración por fabricante (Kabunik · F2 · P1)
- K6.1 UI de captura de parámetros de planta (layout, HH/ton, capacidad, precios, catálogo, umbrales de alertas).
- K6.2 Persistencia como `TenantConfig` (consumido por tools de dcode).

### EPIC K7 · Usuarios, licencias y multitenancy (Kabunik · F0→F2 · P1) `security-tenant`
- K7.1 Autenticación, roles y permisos.
- K7.2 Integración con sistema de licencias Kabunik (orgs, suscripción, sesiones/dispositivos).
- K7.3 Aislamiento de datos por tenant + revisión de seguridad del cruce agente↔plataforma.
- K7.4 Almacén de benchmarks agregados anonimizados (F3).

### EPIC K8 · Infraestructura (Kabunik · F0→ · P1)
- K8.1 Hosting AWS, entornos (dev/staging/prod).
- K8.2 CI/CD, backups, observabilidad.

### EPIC D1 · Agente orquestador (dcode · F0→F2 · P0) — tracking
- D1.1 Cerebro/casuística (prompt+skills, 22 alertas, escenarios) — F1.
- D1.2 Playbook con compuertas + guardarraíles ("ninguna cifra sin tool", human gate) — F0.
- D1.3 Streaming de progreso/alertas/deltas — F1.
- D1.4 Agente stateful: interpretación de ediciones, deltas, recálculo incremental — F2.

### EPIC D2 · Percepción propia (dcode · F0→F2 · P0) — tracking
- D2.1 Planos PDF/DXF → geometría → BOM → modelo (one-shot, F0).
- D2.2 Contrato PDF → términos estructurados — F1.
- D2.3 Adaptador de contingencia para BOM externo (diseño de la puerta, no implementación) — F1. `needs-definition`

### EPIC D3 · Herramientas deterministas + MCP (dcode · F1 · P0) — tracking
- D3.1 Port del motor `.jsx` a tools Python con I/O validado.
- D3.2 Grupos: cómputo/validación, costo/comercial, generación, estado.
- D3.3 Exposición como servidor MCP.

### EPIC D4 · Eval harness (dcode · F1→ · P1) — tracking
- D4.1 Set de casos reales (insumo de Daniel).
- D4.2 Harness + medición de regresiones (incl. ediciones F2).

### EPIC J1 · Gobernanza y definición (Conjunto · F0 · P1)
- J1.1 Definición de "Terminado" por fase.
- J1.2 Acuerdo de IP: metodología (cerebro compartido) vs. calibración privada por tenant.
- J1.3 Cadencia de sync Kabunik×dcode y canal de decisiones del contrato.
- J1.4 Insumos de Daniel: parámetros de calibración + casos de evaluación (fechas compromiso).

---

## 4. Dependencias a registrar (usar label `blocked` + mención cruzada)

- **C1 bloquea:** K2.3, K2.4, K3.2, K3.3, K3.4, K5.*, D1.3, D1.4, D3.3.
- **D2.1 bloquea:** K2.2 (necesita un modelo que renderizar) — para F0 puede usarse un modelo de muestra como fixture; crear issue K2.2a "fixture de modelo de muestra" P1.
- **K3.2 bloquea:** K2.3 y D1.4.
- **J1.4 (casos de Daniel) bloquea:** D4.*.
- **K6.2 bloquea:** ejecución real de `get_config_tenant` (D3) con datos reales; mientras, mock en el contrato.

---

## 5. Qué NO hacer

- No crear issues de alcance ERP (control de producción, inventario contable, facturación) — fuera de frontera.
- No detallar el backlog interno de dcode más allá de tracking (ellos planifican su detalle).
- No reabrir decisiones cerradas (sección 1).
- No inventar estimaciones: el campo `Estimación` se deja vacío para que cada equipo lo llene.

---

## 6. Resultado esperado

Al terminar, el usuario debe poder ver: un Project con las 3 vistas, 4 milestones, ~14 epics con sus issues hijos enlazados, campos asignados (Workstream/Fase/Área/Prioridad), y las dependencias registradas. Reporta al final un resumen de lo creado (conteo por workstream y fase) y cualquier `needs-definition` pendiente.
