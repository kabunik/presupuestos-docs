# Reconciliación: Sistema DM v1.7 × Plataforma Kabunik × dcode

> Las contradicciones detectadas entre el paquete del consultor y nuestros documentos de alcance,
> y cómo se resuelve cada una. Revisión del **2026-08-06**.

## Resumen

Los dos cuerpos documentales describen el mismo producto desde ángulos incompatibles en la forma:

| | **Sistema DM v1.7** | **Plataforma Kabunik × dcode** |
|---|---|---|
| Qué es | Motor de costeo y decisión, determinista | Plataforma SaaS multi-tenant agéntica |
| Autor | Ing. Daniel Michelena (consultor) | Kabunik + dcode |
| Fecha | 2026-08-01 | 2026-07 |
| Contrato de datos | `input_schema.json` E1–E7 + `output_schema.json` — **existen** | `TenantConfig, Model, BOM, Alert, Scenario, Offer, Benchmark` — **por definir** (C1.1, #2) |
| Reglas | 16 invariantes + rúbrica de 13 criterios (6 bloquean emisión) | «las 22 alertas» |
| Aceptación | `golden_test.json`, tolerancia cero, en CI | Eval harness pendiente (D4) |
| Plan | 5 fases / 18 semanas | F0 Toyota → F3 Ferrari |
| Primer entregable | Peso AISC + doble chequeo (F1 del consultor) | Visor 3D navegable (F0) |
| Visor 3D / IFC | **«No construir en v1»** | El «wow» de F0 |

**No son dos productos.** El Sistema DM describe la aritmética; la plataforma describe la
experiencia y el gobierno de los datos. Encajan si el Sistema DM se trata como **especificación
normativa de la capa determinista** — decisión D1, registrada en [README.md](README.md).

Lo que sigue son las tensiones concretas y su resolución.

---

## T1 · Orden de construcción: visor primero vs. motor primero

**Sistema DM:** el README del kit y el Brief §4 dicen explícitamente *«No construir en v1: IFC,
simuladores gráficos…»*. Su F1 es peso AISC + doble chequeo.

**Nuestros documentos:** F0 Toyota es *«intake + percepción one-shot + visor 3D»*, y el visor es el
primer entregable tangible y «el corazón de la experiencia».

**Resolución.** Ambos tienen razón sobre su propio dominio y la contradicción es aparente:

- La prohibición del consultor aplica a **su** alcance: no construir un modelador/simulador gráfico
  *dentro del motor*. El motor es una función determinista pura, sin UI.
- El visor 3D de la plataforma **no es un módulo del motor**: es la superficie de presentación del
  modelo canónico, que vive en Kabunik.
- Se mantiene F0 con visor. **Consecuencia que hay que asumir explícitamente:** en F0 el visor
  muestra geometría, BOM y alertas de percepción, pero **no muestra precio**, porque la aritmética
  no existe todavía. El CTA «Aprobar BOM» es válido en F0; «Generar escenarios» y «Emitir oferta»
  no lo son.

**Acción:** el mockup muestra «Costo estimado $49.5M» en la barra inferior del workspace desde el
primer output. En F0 ese campo va vacío o con estado *«pendiente de motor»*. Registrar en el
inventario de estados de [ALCANCE_v1.md](ALCANCE_v1.md).

---

## T2 · Dos planes de proyecto compitiendo

**Sistema DM:** `Brief_Arranque_Desarrollo.pdf` es un plan de desarrollo completo — 5 fases, 18
semanas, demo quincenal, criterios de aceptación por cifra del golden test, riesgos y mitigación.
Está dirigido «al equipo desarrollador».

**Nuestros documentos:** F0–F3 con 4 milestones en el GitHub Project.

**Resolución.** El Brief del consultor **se absorbe como plan interno del EPIC D3** (herramientas
deterministas, issue #54). Sus 5 fases se convierten en la secuencia de implementación de las tools:

| Fase del Brief | Entrega | Se mapea a |
|---|---|---|
| F1 Contratos y motor de peso (sem. 1–4) | Esquemas + peso AISC + doble chequeo + exclusiones | D3.1 (#55) |
| F2 Costeo (sem. 5–8) | APU AISC, Más X% atado a RSS, costo empresa, reconciliación <1% | D3.1 / D3.2 (#56) |
| F3 Planta y chequeos (sem. 9–12) | Lotes, grúa, S11, inventario neto | D3.2 |
| F4 Programa y compuertas (sem. 13–16) | S12 optimizado, interferencia de familias, bitácora de precios, leyenda | D3.2 |
| F5 Cierre y endurecimiento (sem. 17–18) | Cierre/recalibración, memoria descriptiva | D3.2 + F3 Ferrari |

**No** se adopta la cadencia de «demo quincenal contra el Excel» como cadencia de proyecto: eso es
disciplina interna de quien implemente las tools. Sí se adopta el criterio de aceptación
(golden test en CI) como **condición de cierre de D3**.

**Acción:** la cadencia y el canal quedan en J1.3 (#64). El Brief no se cita como plan de proyecto
en ningún documento nuestro.

---

## T3 · Esquemas de datos: dos vocabularios sin puente

**Sistema DM:** `input_schema.json` organiza la entrada en **E1–E7** (ficha, take-off, precios,
planta/VSM, financieros, especificaciones, bases). La salida son 18 campos *required*.

**Nuestros documentos:** C1.1 pide definir `TenantConfig, Model, BOM, Alert, Scenario, Offer,
Benchmark` — desde cero.

**Resolución.** No se inventan esquemas: se **mapean**. Los esquemas del kit son la fuente; los
nombres de plataforma son la vista que consume el frontend. El mapeo preliminar está en
[CONTRATO_INTEGRACION_v0.md](CONTRATO_INTEGRACION_v0.md).

Puntos de fricción del mapeo, ya identificados:

1. **E1–E7 mezcla tres cosas** que en la plataforma viven separadas y con dueños distintos:
   configuración de tenant (E3 precios, E4 planta, E5 financieros), datos de proyecto (E1, E2, E6),
   y estado operativo del negocio (E7 carga de planta e inventario, que **cambia cada semana**).
   `TenantConfig` no puede absorber E7: necesita su propia entidad con cadencia de actualización.
2. `version_parametros` es *required* en entrada **y** salida. Implica que `TenantConfig` es
   **inmutable y versionada**, no un registro editable. Cada oferta queda ligada a su versión.
3. El `Model` de la plataforma (geometría 3D navegable, con `elementIds`) es **más rico** que E2
   take-off (`peso_planos_t`, `factor_facturable_aisc`, `peso_reconstruido_t`). E2 es el resumen que
   el motor consume; el `Model` es lo que el visor renderiza. Son capas distintas y el `model_delta`
   opera sobre el `Model`, no sobre E2.

**Acción:** C1.1 (#2) se replantea como *«mapear los esquemas del kit a las entidades de
plataforma»*, no como definición desde cero. Comentar en el issue.

---

## T4 · «Las 22 alertas» no tienen fuente

**Verificado:** la cifra **no aparece en ninguno de los 7 entregables del consultor**. Se originó en
nuestros propios documentos y se propagó a 6 lugares:

| Ubicación | Texto |
|---|---|
| `Segmentacion_Tareas_Kabunik_dcode_v2.md:47` | «la lógica de las 22 alertas» |
| `Segmentacion_Tareas_Kabunik_dcode_v2.md:67` | tool `motor_alertas_22` |
| `Segmentacion_Tareas_Kabunik_dcode_v2.md:85` | «panel de validación (22 alertas)» |
| `HANDOFF_GitHub_Projects_Plan.md:106` | «K4.2 Panel de validación — 22 alertas» |
| `HANDOFF_GitHub_Projects_Plan.md:131` | «D1.1 Cerebro/casuística (prompt+skills, 22 alertas…)» |
| `mockup/.../README.md:176` | «Validando — 22 reglas de validación» |

Más los issues #26 y #46, cuyos títulos la incorporan.

**Lo que sí existe y está numerado en el paquete del consultor:**

- **16 invariantes** (reglas de negocio inviolables).
- **Rúbrica de 13 criterios** de auditoría, de los cuales **6 marcados `*` invalidan la emisión**.
- Un conjunto de **alertas nombradas** en el `output_schema`: `alerta_interferencia_familias`,
  `advertencia_bajo_equilibrio`, `alerta_ocupacion`, `alerta_subcontrato_hh`,
  `advertencia_comunicacion_cliente`, `confirmar_precios_mp`, `doble_chequeo → detener_emision`.
- Alertas de **percepción/cómputo** en el material inicial: el resumen del caso CNARCCS lista 7
  advertencias de ingeniería, y el .pptx clasifica alertas en Crítica / Alta / Media / Baja.

**Resolución.** La cifra 22 se retira de todos los documentos. El catálogo real de alertas se
define como trabajo explícito, distinguiendo tres familias con dueños distintos:

| Familia | Origen | Dueño | Bloquea emisión |
|---|---|---|---|
| Invariantes del motor | `motor_calculo_spec.md` | dcode (tools) | 6 de los 13 criterios sí |
| Validación de percepción | Lectura de planos → modelo | dcode (D2) | No, pero condiciona el gate de BOM |
| Guardarraíles de edición | Catálogo del tenant, disponibilidad | dcode (D1) + `TenantConfig` | Rechaza el delta, no la oferta |

**Acción:** issue `needs-definition` para el catálogo de alertas y validaciones, y renombrar la tool
`motor_alertas_22` → `motor_alertas` en el contrato antes de congelarlo.

---

## T5 · Falta una compuerta humana: confirmación de precios de materia prima

**Sistema DM, inv. 8:** antes de emitir el resultado final el sistema presenta la lista completa de
precios de MP considerados (material, precio, fuente, fecha, vigencia restante) y **exige
confirmación explícita registrada en bitácora**. Sin ella, `estado_emision: "bloqueada"` y no hay
`precio_licitacion`. `confirmar_precios_mp` es campo *required* de la salida.

**Nuestros documentos:** solo existe una compuerta humana, la del BOM. El mockup la diseña con
esmero (banner rojo flotante, CTA prominente, «momento deliberado»). La de precios no existe.

**Resolución.** Son **dos compuertas de igual rango** y la v1 lleva las dos:

| | Gate de BOM | Gate de precios de MP |
|---|---|---|
| Qué protege | Que no se cotice sobre un takeoff equivocado | Que no se emita sobre precios caducos |
| Momento | Antes de calcular costos | Antes de emitir la oferta |
| Efecto de no pasarla | No hay escenarios | No hay `precio_licitacion` |
| Registro | Autor + fecha + versión de modelo | Bitácora: material/precio/fuente/fecha/vigencia + autor + fecha + referencia de oferta |
| Reapertura | Cualquier edición que altere el takeoff | Vencimiento de vigencia de cualquier precio |

**Acción:** pantalla nueva en [ALCANCE_v1.md](ALCANCE_v1.md) + issue `needs-definition` para su
diseño (el mockup no la cubre y su lenguaje visual debe ser coherente con el gate de BOM sin
canibalizarlo).

---

## T6 · El plan de montaje es entrada obligatoria y no se pide en el intake

**Sistema DM, inv. 11:** el plan de montaje del cliente **es entrada**, no consecuencia.
`plan_montaje_t_sem` es campo *required* de E6. Lotes, secuencia y memoria se calculan hacia atrás
para cumplirlo **contra la planta cargada**, y el déficit se resuelve explícito (sobretiempo prima
50% / subcontrato all-in), nunca oculto.

**Nuestros documentos:** el wizard de intake P3 tiene 3 pasos — datos generales, carga de archivos,
consideraciones (alcance, grado preferente, tipo de conexión, acabado + texto libre). **No pide el
plan de montaje en ningún campo.**

**Resolución.** El intake gana un paso o un bloque para el plan de montaje (t/semana). Sin él el
motor no puede producir S9 lotes, S11 velocidad ni S12 programa — es decir, no puede producir
`precio_licitacion`. No es un campo más: es una entrada estructural.

Consideración de diseño: el plan de montaje puede llegar como tabla, como archivo del cliente, o no
llegar en absoluto (proyecto sin montaje contratado, como el caso CNARCCS, donde el montaje se
cotiza aparte). El comportamiento cuando **no hay** plan de montaje está sin definir.

**Acción:** issue `needs-definition` — captura del plan de montaje en el intake y comportamiento
cuando no existe.

---

## T7 · El núcleo de decisión de planta no tiene UI

**Sistema DM:** S11 (doble chequeo de velocidad) y S12 (decisión de programa optimizada) son,
según la propia `Evaluacion_Honesta`, lo que *«no existe en software comercial integrado al
costeo»*. Incluyen carga combinada en HH y t/semana contra los contratos en curso, % de ocupación
de habilitado con alerta ≥100%, escenarios A/B/B\*/C con retraso mínimo y pena máxima soportable, y
la advertencia obligatoria de acordar el retraso con el cliente afectado **antes** de comprometer.

**Nuestros documentos:** ninguna pantalla del mockup ni del inventario de 12 pantallas cubre esto.

**Resolución.** Por decisión D2, entra en la v1. Requiere superficies nuevas:

- Carga de planta y ocupación de habilitado (consume E7, que se actualiza semanalmente).
- Escenarios de programa A/B/B\*/C, con su recomendación condicionada.
- Alerta de interferencia de familias, con familias, tonelaje, HH/ton y rentabilidad relativa.

**Riesgo asociado que hay que diseñar, no solo documentar:** la `Evaluacion_Honesta` marca como
riesgo residual **abierto** la *disciplina de datos de E7* — «E7 desactualizado convierte S12 en
ficción bien presentada». La mitigación recomendada es dueño de datos por base, cadencia semanal y
lista de pendientes visible en dirección. Eso es producto, no proceso: hay que diseñar el estado
«E7 desactualizado» y su efecto sobre la emisión.

**Acción:** issues `needs-definition` para las superficies de S12/interferencia y para el modelo de
frescura de E7.

---

## T8 · «3 escenarios» son dos cosas distintas

En el Sistema DM coexisten dos conjuntos de tres/cuatro escenarios que no tienen relación:

| | **Opciones comerciales** | **Escenarios de programa** |
|---|---|---|
| Dónde | `opciones` (3, exactos: `minItems: 3, maxItems: 3`) | `s12_programa.escenarios` |
| Qué varían | **Supuestos, nunca la fórmula** (inv. 6) — velocidad, alcance, condiciones | Cómo resolver el déficit de capacidad |
| Valores | E1 Conservador / E2 Optimizado / E3 Agresivo | A / B / B\* optimizado / C |
| Salida | $/ton, total, plazo, margen, flujo de caja | Costo del déficit, utilidad, retraso en semanas, pena máxima soportable |
| Decisión | Qué se ofrece al cliente | Si se entra al contrato y cómo se programa |
| En el mockup | Sí (P9, P5·C) | **No existe** |

El mockup y nuestros documentos usan «3 escenarios» para el primer conjunto exclusivamente, lo que
ha invisibilizado el segundo.

**Resolución.** Vocabulario fijado en [GLOSARIO.md](GLOSARIO.md): **«opciones de oferta»** para las
comerciales, **«escenarios de programa»** para las de S12. Corregir en documentos y en la UI.

---

## T9 · Recalibración: «el sistema aprende» vs. gobernanza obligatoria

**Sistema DM, inv. 16 (nuevo en v1.7):** cierre de proyecto obligatorio; desviación >±5% genera
**propuesta** de recalibración con evidencia; los parámetros **nunca se recalibran solos** —
aprobación de dirección + nueva versión de parámetros; cada presupuesto queda ligado a su versión;
un proyecto sin cierre no alimenta recalibraciones. El README del kit lo dice sin ambigüedad:
*«no construir en v1: recalibración automática (prohibida por inv. 16)»*.

**Nuestros documentos:** F3 Ferrari habla de *«lazo cotizado-vs-ejecutado»* y *«RAG/memoria
multicliente»*; el material comercial promete *«aprende de cada oferta y se actualiza en tiempo
real»* y *«mejora sola»*.

**Resolución.** La promesa comercial y el invariante son incompatibles tal como están escritos. Rige
el invariante: **hay lazo cerrado, pero con compuerta humana y versionado**. Nada se autoajusta.

Implicaciones de producto:
- `TenantConfig` es versionada e inmutable; `version_parametros` viaja en cada presupuesto.
- Existe una bandeja de **propuestas de recalibración** con evidencia, para dirección.
- Existe una **lista de cierres pendientes** visible.
- El copy del producto no puede decir «se actualiza solo».

**Acción:** issue `needs-definition` para el módulo de cierre y recalibración gobernada. Alinear el
copy comercial es tarea de producto, no de ingeniería, pero queda anotada.

---

## T10 · La percepción: módulo anexo vs. núcleo del producto

**Sistema DM:** el generador de IFC unifilar desde PDF/DXF es un **módulo anexo** que «alimenta
take_off (E2)». El motor no lo contiene y explícitamente no se construye en v1.

**Nuestros documentos:** la percepción propia (Opción A) es el corazón de dcode y de F0.

**Resolución.** No hay contradicción real, solo distinta ubicación de la frontera: para el consultor
el motor es el producto y la percepción es periférica; para nosotros el producto es la plataforma y
la percepción es un componente central. Ambas cosas pueden ser ciertas simultáneamente.

Lo relevante es el **punto de contacto**: la percepción de dcode debe producir, además del `Model`
navegable, exactamente los campos que E2 exige — `fuente` (`ifc` | `pdf_dxf`),
`factor_facturable_aisc`, `peso_reconstruido_t` y `exclusiones[]`. El `peso_reconstruido_t` existe
para el **doble chequeo del inv. 4**: IFC vs. reconstruido, y una diferencia fuera de tolerancia
**detiene la emisión**. Eso obliga a que la percepción emita dos pesos calculados por caminos
independientes, no uno.

**Acción:** requisito explícito en [CONTRATO_INTEGRACION_v0.md](CONTRATO_INTEGRACION_v0.md) y
comentario en D2.1 (#51).

---

## Hallazgo transversal: el mockup no puede emitir una oferta válida

Consecuencia combinada de T5 y T6. Bajo los invariantes, el CTA **«Emitir oferta»** del workspace
(modo C) y de P9 es hoy inalcanzable:

1. Sin `plan_montaje_t_sem` (E6, *required*) no hay S9/S11/S12 → no hay `capitulo_financiero` →
   no hay `precio_licitacion`.
2. Sin `confirmar_precios_mp.confirmado = true` → `estado_emision: "bloqueada"`.
3. Sin `leyenda_alcance` (campo *required*, texto obligatorio) la emisión incumple la regla de
   emisión del paquete.

No es un defecto del mockup: es alcance que nadie había cruzado con los invariantes, porque los
invariantes llegaron después. Queda resuelto por la decisión D2.

---

## Cifras del mockup verificadas contra el caso real

El mockup usa el caso CNARCCS Domo, y los datos son correctos salvo un desvío menor:

| Dato | Mockup | Real (`CNARCCS_DOMO_Oferta_3Escenarios.xlsx`) | |
|---|---|---|---|
| Peso cobrable | 734.0 t | 734.57 t | ✓ |
| Elementos | 1,470 | 1,470 | ✓ |
| Opción E1 $/ton | 73,200 | 73,226 | ✓ |
| Opción E2 $/ton | 67,500 | **67,076** | ✗ desvío |
| Opción E3 $/ton | 61,800 | 61,808 | ✓ |
| Total E1 | $53.7M | $53,789,535 | ✓ |
| Total E2 | $49.5M | $49,271,487 | ≈ |
| Anticipo / retención | 30% / 5% | 30% / 5% finiquito | ✓ |

Un detalle a tener en cuenta al poblar fixtures: el caso real corre a **58 HH/ton** (54 en el
escenario optimizado), valor que en el eje del inv. 2 (24/40/60/90) corresponde a la **clase 60**,
no a 40. El golden test, en cambio, es clase 40. Son dos casos distintos y así deben usarse:
golden test como ley de CI, CNARCCS como caso de evaluación end-to-end y fuente de fixtures de UI.

**Acción:** corregir E2 a 67,076 al implementar, o dejar constancia de que es un valor de mockup.
