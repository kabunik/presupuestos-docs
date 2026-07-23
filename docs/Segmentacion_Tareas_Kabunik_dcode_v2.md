# Segmentación de Tareas — Plataforma de Presupuestos de Estructuras Metálicas
### División de responsabilidades para desarrollo en paralelo · Kabunik × dcode · v2

**Propósito:** repartir el trabajo entre dcode (agente + herramientas) y Kabunik (plataforma) para avanzar en paralelo, con el *contrato de integración* como costura. Esta versión incorpora dos definiciones: la percepción se construye propia (Opción A) y el producto se organiza alrededor de un **visor 3D editable conversacionalmente en tiempo real**.

---

## 1. Visión de producto y flujo de interacción (el norte)

La plataforma **no se percibe como un chat**: se percibe como un taller de ingeniería donde el usuario *ve y manipula una solución estructural en 3D*. El chat existe, pero como **volante del modelo**, no como interfaz principal.

**Flujo central:**

1. **Toma de información.** La plataforma captura todas las consideraciones e insumos del sistema del consultor: planos (PDF/DXF), contrato, datos de proyecto y calibración de planta.
2. **Primer output: la solución en un visor 3D.** El agente (percepción propia, Opción A) interpreta los planos y construye la solución estructural propuesta, que se presenta como un **modelo 3D navegable**. Es el primer entregable tangible y el corazón de la experiencia.
3. **Edición conversacional con reconstrucción en tiempo real.** El usuario pide cambios en lenguaje natural ("cambia las columnas del N3 a HSS", "reduce la luz de los pórticos a 8 m", "usa A572 Gr.50 en vigas") y ve cómo la alternativa estructural **se reconstruye en el visor en tiempo real**. Cada cambio pasa por las validaciones y guardarraíles del agente.
4. **Generación de escenarios.** Sobre el modelo validado se generan los distintos escenarios comerciales (precios, flujo de caja, ingeniería de valor).
5. **Validación iterativa de outputs.** El usuario sigue haciendo cambios para comparar y validar los resultados finales antes de emitir la oferta.

**Diferenciador / barrera de clonación:** la edición conversacional de un modelo estructural con reconstrucción en tiempo real —respaldada por la calibración por planta y los guardarraíles de ingeniería— es difícil de replicar. La UI se copia; esta interacción, la calibración y el motor de decisión, no. (Steel Genie by ALLPLAN llega hasta el visor 3D y la edición *previa a exportar*; nuestro flujo va más allá, con edición conversacional en vivo y la capa de costo/decisión que ellos no cubren.)

---

## 2. Principio de división

Arquitectura **agéntica**: un agente orquestador (LLM) resuelve la casuística y llama a **herramientas deterministas** donde vive toda la aritmética; todo sobre una plataforma multi-tenant.

- **dcode es dueño del "cerebro y los músculos":** el agente orquestador (incluida la percepción propia y la edición del modelo) y las herramientas deterministas.
- **Kabunik es dueño de "el cuerpo":** diseño, interfaces, **visor 3D**, formularios, chat de edición, gestión de usuarios, licencias, multitenancy, almacenamiento e integración del agente.
- **La costura es un contrato de API bidireccional** (sección 7) que además maneja **estado de modelo y ediciones incrementales**.

**Regla de frontera (Hoja de Ruta):** el sistema decide y planifica; no administra ni contabiliza. El límite aguas abajo es el puente de exportación a Strumis/SAP.

---

## 3. Decisión de percepción — **Opción A (confirmada)**

dcode construye la percepción propia: el agente lee planos PDF/DXF con visión y genera geometría → BOM → modelo estructural.

- **Plan de contingencia:** durante las pruebas se conserva la opción de **mutar a un esquema híbrido** (aceptar takeoffs externos IFC/Excel de Steel Genie o Tekla como respaldo o complemento) si la percepción propia se complica en algún tipo de proyecto. No es alcance de arranque, pero el diseño del contrato de integración (sección 7) debe dejar la puerta abierta para ingerir un BOM externo sin rehacer la costura.

---

## 4. Workstream A — dcode · Agente + Herramientas

### A.1 · Agente orquestador (con estado de modelo)
- "Cerebro / casuística": system prompt + skills que codifican el proceso de cómputo, la lógica de las 22 alertas, la interpretación de contrato y la lógica de escenarios.
- **Playbook con compuertas** (secuencia obligatoria; no cotizar antes de validar BOM) y guardarraíles duros: **"ninguna cifra sin tool"** y **compuerta humana** en la validación de BOM.
- **Agente iterativo/stateful:** mantiene coherencia con el modelo canónico, interpreta peticiones de edición en lenguaje natural, aplica *deltas* al modelo vía herramientas y **acota el recálculo a lo afectado** (no re-lee todos los planos en cada cambio).
- Orquestación por tool-use / function-calling; modelo Sonnet; manejo de errores y reintentos.
- **Streaming de progreso, alertas y actualizaciones de modelo** hacia la plataforma (para que el visor 3D se reconstruya en vivo).
- RAG sobre memoria histórica; curación del **set de casos reales** (Daniel) como few-shot y base de eval.

### A.2 · Percepción propia (Opción A)
- Planos PDF/DXF → geometría → BOM → modelo estructural inicial.
- Contrato (PDF) → términos estructurados (anticipo, retención, multas, hitos).
- (Contingencia) adaptador para ingerir BOM externo (IFC/Excel) si se activa el modo híbrido.

### A.3 · Edición del modelo (motor de la interacción conversacional)
- Interpretar intención de edición ("cambia sección", "reduce luz", "cambia grado de acero", "empernado en vez de soldado") → **mutación estructurada del modelo**.
- Re-validar y **re-computar solo lo afectado**; devolver el *delta* de modelo + BOM + costos actualizados.
- Clasificar el alcance del cambio (geométrico vs. comercial) para escalar el recálculo correctamente.

### A.4 · Herramientas deterministas (Python, con esquema de I/O validado)
| Grupo | Herramientas |
|---|---|
| Cómputo & validación | `kgm_perfil`, `longitud_3d`, `validar_bom`, `motor_alertas_22` |
| Costo & comercial | `pricer_3_escenarios`, `flujo_caja`, `optimizador_transporte`, `selector_recubrimiento`, `disponibilidad_perfiles`, `ingenieria_valor` |
| Generación | `generar_oferta_docx`, `exportar_ifc` |
| Estado (consumen datos de la plataforma) | `get_config_tenant`, `get_benchmarks`, `registrar_cierre` |

- Exponer las herramientas como **servidor MCP** (o endpoint de function-calling) reutilizable.
- Portar la lógica del motor actual (`baysa_presupuesto_v4.jsx`) conservando la trazabilidad directa al cálculo.

### A.5 · Calidad del agente
- **Eval harness** sobre casos reales de Daniel; medición de regresiones ante cada cambio de prompt/tool y **ante ediciones** (que un cambio no rompa el resto del modelo).
- Logging estructurado de cada llamada a herramienta (soporta la trazabilidad de la plataforma).

---

## 5. Workstream B — Kabunik · Plataforma

### B.1 · Diseño y experiencia
- Sistema de diseño "de plataforma" (no chat-first). El agente es infraestructura invisible; el **visor 3D es el protagonista**.
- Superficies manipulables: workspace de proyecto, **visor 3D**, grilla editable de BOM/takeoff, panel de validación (22 alertas), tarjetas comparativas de escenarios, gráficos de flujo de caja, checklist de ingeniería de valor, preview/descarga de oferta.

### B.2 · Visor 3D y edición en tiempo real (núcleo)
- **Visor 3D / IFC** navegable como primer output.
- **Render de reconstrucción en tiempo real:** consumir los *deltas* / actualizaciones de modelo del agente y actualizar el 3D en vivo.
- **Chat de edición** como superficie de primera clase (la UI es de Kabunik; la inteligencia, del agente): capturar la petición, mostrar el progreso y reflejar el resultado en el visor.
- Compuerta de revisión del BOM antes de continuar el cálculo.

### B.3 · Intake, escenarios y validación
- Formularios y wizard de alta de proyecto + carga de archivos (PDF/DXF/IFC) — capturar **todas** las consideraciones del sistema del consultor.
- UI de **generación y comparación de escenarios**.
- UX de **iteración/validación**: permitir cambios sucesivos y comparar outputs finales antes de emitir.

### B.4 · Onboarding y calibración por fabricante
- UI para capturar los parámetros de planta (layout, eficiencias/HH·ton, capacidad, precios, catálogo de perfiles, umbrales de alertas) → persistir como **config del tenant**.

### B.5 · Usuarios, licencias y multitenancy
- Autenticación, roles y permisos.
- Integración con el **sistema de licencias existente de Kabunik** (acceso por organización, suscripción SaaS, monitoreo de sesiones y dispositivos).
- **Aislamiento de datos por tenant**; almacén de config por tenant y de **benchmarks agregados anonimizados**.

### B.6 · Backend, estado de modelo e integración del agente
- API de plataforma que consume el frontend.
- **Estado canónico del modelo/BOM por sesión de proyecto** (la plataforma es el sistema de registro; el agente lo muta vía herramientas; el visor renderiza desde este estado).
- **Orquestación de llamadas al agente** (crear job, ediciones incrementales, streaming, compuerta humana, resultados).
- Servicios de datos que las herramientas de dcode invocan (`get_config_tenant`, `get_benchmarks`, `registrar_cierre`).
- Almacenamiento de archivos, exportables (oferta, IFC) y **puente a Strumis/SAP**.

### B.7 · Infraestructura
- Hosting (AWS), seguridad, backups, CI/CD, observabilidad de plataforma.

---

## 6. Workstream C — Conjunto (Kabunik + dcode + Daniel)

- **Contrato de integración** (sección 7): definir y congelar antes de desarrollar en paralelo, incluyendo **estado de modelo y ediciones**.
- **Esquemas de datos compartidos y versionados:** `TenantConfig`, `Model`, `BOM`, `Alert`, `Scenario`, `Offer`, `Benchmark`.
- **Fuente de verdad del modelo:** la plataforma mantiene el modelo canónico; el agente aplica deltas. Acordar el formato del delta.
- **Set de parámetros de calibración** (Daniel define contenido; Kabunik el almacén; dcode el consumo).
- **Casos de evaluación** (Daniel aporta verdad de dominio; dcode construye el harness; Kabunik consume para QA).
- **Seguridad del cruce:** el agente y sus herramientas respetan la frontera de tenant.
- **Definición de "Terminado"** por fase e **IP:** metodología compartida (cerebro) vs. calibración privada por tenant.

---

## 7. El contrato de integración (la costura)

Permite el trabajo en paralelo. Ahora gira en torno a una **sesión de proyecto con estado de modelo**.

### 7.1 · Sesión y modelo
- `POST /project-session` — abre sesión: `{ tenant_id, project_id, inputs (PDF/DXF/IFC/contrato) }`.
- La plataforma mantiene el **modelo canónico**; el agente devuelve *deltas* que la plataforma aplica y el visor renderiza.

### 7.2 · Plataforma → Agente (Kabunik llama a dcode)
- `POST /session/{id}/generate` — genera la solución inicial (percepción → modelo + BOM).
- `POST /session/{id}/edit` — petición de cambio (lenguaje natural o estructurado) → el agente muta el modelo, re-valida, re-computa lo afectado y devuelve `model_delta` + BOM + costos.
- `POST /session/{id}/scenarios` — genera los escenarios sobre el modelo validado.
- `POST /session/{id}/bom-approval` — devuelve el BOM aprobado/editado por el usuario.
- **Eventos de streaming:** `progress`, `tool_call.*`, `alert.raised`, `model.updated` (delta para el visor), `human_gate.bom_review_required`, `result.ready`.
- **Ciclo de vida:** `perceiving → awaiting_bom_review → computing → ready → editing → …` (iterable).

### 7.3 · Agente → Plataforma (herramientas de dcode → servicios de datos de Kabunik)
- `get_config_tenant(tenant_id)` → parámetros de calibración.
- `get_benchmarks(filtros)` → referencias agregadas anonimizadas.
- `disponibilidad_perfiles(...)` → stock/inventario del tenant.
- `registrar_cierre(project_id, actuals)` → datos reales para el lazo cotizado-vs-ejecutado.
- Lectura/escritura de artefactos del proyecto en el almacenamiento de la plataforma.

### 7.4 · Propiedad de los esquemas
- **Kabunik** posee almacenamiento y esquemas de `TenantConfig`, `Model`, `Benchmark` y artefactos.
- **dcode** posee la lógica de herramientas y del agente.
- Esquemas definidos **de forma conjunta** y versionados; sin cambios unilaterales.

---

## 8. Secuenciación por fases ("Toyota → Ferrari")

| Fase | dcode | Kabunik | Objetivo |
|---|---|---|---|
| **0 · Toyota** | Percepción propia mínima: planos → modelo + BOM (one-shot, con compuerta humana). Congelar contrato v0. | Intake + carga de archivos + **visor 3D del primer output** (navegable, aún sin edición conversacional). | Entregar el "wow" de ver la solución 3D rápido; validar con 1–2 clientes. |
| **1 · Motor y escenarios** | Herramientas deterministas completas (MCP); agente orquesta cómputo, alertas, escenarios. | Panel de alertas, escenarios, flujo de caja, grilla BOM editable, integración + streaming. | Cotización calibrada y comparación de escenarios. |
| **2 · Edición conversacional en vivo** | Agente iterativo/stateful: edición por lenguaje natural + recálculo incremental + deltas de modelo. | **Chat de edición + reconstrucción del visor 3D en tiempo real**; UX de validación iterativa. | El diferenciador: editar la solución hablando y verla rebuild en vivo. |
| **3 · Ferrari** | RAG/memoria multicliente, lazo cotizado-vs-ejecutado, optimizador de nesting. | Benchmarks agregados, puente a Strumis, analítica. | Aprendizaje y efecto de red. |

*Nota: la edición conversacional en tiempo real (Fase 2) es el corazón de la visión pero también lo más exigente técnicamente (latencia, recálculo incremental). Se llega a ella sobre la base sólida del visor (Fase 0) y el motor (Fase 1).*

---

## 9. Dependencias críticas y riesgos

- **Contrato de integración con estado de modelo (7):** bloqueante para el paralelismo; congelar en Fase 0.
- **Latencia de la reconstrucción en tiempo real:** exige recálculo incremental (solo lo afectado), caché y que el agente mantenga estado en vez de re-percibir. Es el mayor riesgo técnico de la Fase 2.
- **Fuente de verdad del modelo:** plataforma canónica + deltas del agente; si no se acuerda el formato del delta, el visor y el agente se desincronizan.
- **Esquemas compartidos (6):** versionado disciplinado o ambos equipos se rompen.
- **Compuerta humana en BOM y concurrencia de ediciones:** puntos de integración delicados; diseñarlos temprano.
- **Frontera de tenant en el agente:** riesgo de fuga entre organizaciones; validar en seguridad.
- **Casos de evaluación:** sin el set de Daniel, dcode no puede medir calidad ni evitar regresiones.

---

## 10. Resumen de responsabilidad

| Área | Responsable |
|---|---|
| Agente orquestador, playbook, guardarraíles, **estado de modelo** | **dcode** |
| Percepción propia (Opción A) + edición del modelo | **dcode** |
| Herramientas deterministas + MCP | **dcode** |
| Eval del agente (incl. ediciones) | **dcode** (con casos de Daniel) |
| Diseño, UI, **visor 3D + reconstrucción en tiempo real**, chat de edición | **Kabunik** |
| Intake, escenarios, UX de validación | **Kabunik** |
| Onboarding/calibración, usuarios, licencias, multitenancy | **Kabunik** |
| Backend, **estado canónico del modelo**, integración del agente, almacenamiento | **Kabunik** |
| Puente a Strumis, infraestructura | **Kabunik** |
| Contrato de integración, esquemas, seguridad de cruce, IP | **Conjunto** |
| Contenido de calibración y casos de evaluación | **Daniel** |

---

*Documento de trabajo · Kabunik × dcode · v2 · 2026. La estimación de esfuerzo y cronograma la aporta cada equipo sobre su workstream una vez congelado el contrato de integración (7).*
