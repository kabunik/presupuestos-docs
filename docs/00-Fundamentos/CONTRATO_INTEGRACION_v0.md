# Contrato de integración v0 — borrador de mapeo

> **Estado: BORRADOR.** No es el contrato congelado. Es el insumo de trabajo para C1
> (issues #1–#7) que evita partir de cero: mapea los esquemas que **ya existen** en el kit del
> consultor a las entidades de plataforma.
>
> **Nada de este documento es acuerdo bilateral hasta que C1.6 (#7) lo congele.**
> Cambios al contrato solo por el canal de decisiones (J1.3, #64), nunca unilaterales.

## Principio de partida

C1.1 pedía «definir esquemas compartidos versionados» desde cero. No hace falta: el kit del
consultor trae `input_schema.json` y `output_schema.json` v1.7 completos y validados contra un
golden test. **El trabajo real es de mapeo, no de diseño.**

La regla de propiedad se mantiene: Kabunik posee el almacenamiento y los esquemas de `TenantConfig`,
`Model`, `Benchmark` y artefactos; dcode posee la lógica de tools y del agente; los esquemas se
definen conjuntamente y se versionan.

---

## 1 · La separación que E1–E7 no hace

El `input_schema.json` organiza la entrada en siete bloques pensados para **una corrida del motor**.
La plataforma necesita separarlos por **dueño y por ciclo de vida**, porque no cambian al mismo ritmo
ni los edita la misma persona:

| Bloque del kit | Contenido | Entidad de plataforma | Quién lo mantiene | Cadencia |
|---|---|---|---|---|
| `e3_precios.materiales` | Precios de MP con fuente/fecha/vigencia | **`PriceList`** (versionada, `plant_id` nullable) | Compras del tenant | Continua; **caduca** |
| `e4_planta_vsm` | Velocidades, cuello de botella, ahorro Lean | **`PlantConfig`** (versionada) | Admin de la planta | Trimestral |
| `e5_financieros` — nómina | Nómina, prestaciones, CCT, mezcla propio/subcontrato → `$/HH` | **`PlantConfig`** | Dirección/finanzas de la planta | Trimestral |
| `e5_financieros` — overhead | Overhead corporativo **y su regla de reparto entre plantas** | **`TenantConfig`** | Dirección corporativa | Trimestral |
| *(factores por clase)* | Desperdicio, soldadura, conexiones por eje 24/40/60/90 | **`PlantConfig`** | Calibración | Versionada, inmutable |
| `e1_ficha` | Proyecto, cliente, peso de planos, clase, fase RSS, subproyectos, **`plant_id` + versión** | **`Project`** | Estimador | Por proyecto |
| `e2_take_off` | Fuente, factor facturable AISC, peso reconstruido, exclusiones | **`Takeoff`** (derivado de `Model`) | Agente (percepción) | Por versión de modelo |
| `e6_especificaciones` | Desviaciones de concurso, **plan de montaje** | **`Project`** | Estimador | Por proyecto |
| `e7_bases.carga_planta` | Proyectos en curso, HH libres/sem, capacidad de habilitado | **`PlantLoad`** (por planta) | Planeación de la planta | **Semanal** |
| `e7_bases.inventario` | Disponible / asignado / libre, préstamos | **`Inventory`** (por planta) | Almacén de la planta | **Semanal** |
| `e7_bases.registro_cierres` | Cierres de proyecto con su versión de parámetros | **`ProjectClosure`** | Dirección | Al cerrar proyecto |
| `version_parametros` | Versión con la que se calcula | atributo de **`PlantConfig`** + sello en `Offer` | Sistema | Inmutable |

### Consecuencias de diseño

0. **La planta es una entidad de primera clase, y un tenant puede tener varias** *(corregido el
   12-ago, #94)*. La planta se configura en un **flujo paralelo** al del proyecto y con su propio
   ciclo de vida; el proyecto **se asocia** a una planta ya configurada, en el intake. Consecuencia
   fuerte: **`version_parametros` es por planta, no por tenant** — la calibración HH/ton sale del VSM
   de *esa* planta.

   ```
   Tenant
    └─ Plant[] 1..N
         ├─ PlantConfig  versionada e inmutable ← E4 VSM + factores por clase (inv. 16)
         ├─ PlantLoad    semanal                ← E7 carga
         └─ Inventory    semanal                ← E7 inventario
   Project → referencia (plant_id, config_version)
   ```

   **Resuelto el 12-ago (#100): `e5_financieros` va partido.** La **nómina en `PlantConfig`**
   —prestaciones, CCT y mezcla propio/subcontrato son de esa planta y producen su `$/HH`— y el
   **overhead en `TenantConfig`**, porque es corporativo.

   **Consecuencia derivada, y no es menor.** El inv. 15 exige que *«el overhead cubra las cargas de
   planta»* (golden test: cargas 99.61 < overhead 115.45). Con las cargas por planta y el overhead
   por tenant, comparar el overhead total contra las cargas de **una** planta no dice nada. Por eso
   `TenantConfig` guarda el overhead **y su regla de reparto** entre plantas —por tonelaje, por HH o
   fijo—; cada planta recibe su `overhead_asignado_hh` y **el chequeo del inv. 15 sigue corriendo por
   planta**. Con una sola planta el reparto es el 100% y el chequeo es idéntico al del golden test.

   **`PriceList` arranca con alcance de tenant** *(decisión del 12-ago, #102)*, asumida como
   definición inicial que puede mutar: cada fabricante tiene su realidad. Para que revertirla sea
   aditivo, el alcance se modela **como campo, no como padre**:

   ```json
   "PriceList": { "id": "…", "tenant_id": "…", "plant_id": null }
   ```

   `plant_id: null` significa «aplica a todas las plantas del tenant». Mañana, una lista específica de
   planta es **una fila más** con `plant_id` puesto y la resolución es «gana la más específica»: sin
   migración y sin cambio de contrato. Es el mismo patrón que `plan_montaje_meta.procedencia` — la
   variación se modela como dato, no como estructura.

   Tres consecuencias que hay que respetar desde ya:
   - `get_price_list(tenant_id, plant_id?)` — la firma admite el futuro sin cambiar (ver §5).
   - El proyecto referencia un **`price_list_id`**, no «la lista del tenant», para que el consumidor no
     dependa del alcance.
   - La bitácora del inv. 8 registra el **`price_list_id`** confirmado, no solo `referencia_oferta`: si
     mañana hay varias listas, queda inequívoco cuál se confirmó.

   **Evolución probable, y no es duplicar la lista.** Lo que varía entre plantas no es el precio
   negociado sino el **flete de entrada**: dos plantas comprando al mismo molino pagan distinto precio
   entregado por transporte. Así que cuando toque, la opción recomendada es **precio de tenant + factor
   de flete de entrada por planta**, no una lista por planta — mantiene una sola fuente del precio y
   modela lo que realmente cambia. Hoy ese flete **no está modelado en ningún sitio**:
   `optimizador_transporte` es para el transporte de salida del proyecto, no para la materia prima.

   Fuera de alcance de v1, anotado para no cerrar la puerta: un proyecto **repartido entre varias
   plantas**. Hoy la referencia es a una sola.

1. **`TenantConfig` no puede absorber E7.** La carga de planta y el inventario son **estado
   operativo semanal**, no configuración. Necesitan entidad propia, con **marca de frescura** —la
   `Evaluacion_Honesta` marca la disciplina de datos de E7 como riesgo residual **abierto**:
   *«E7 desactualizado convierte S12 en ficción bien presentada»*.
2. **`PriceList` caduca.** `vigencia_dias` por material implica que la lista tiene fecha de muerte y
   que la compuerta del inv. 8 puede reabrirse **sin que nadie toque nada**, solo por paso del tiempo.
3. **`TenantConfig` es inmutable y versionada** (inv. 16). No es un registro que se edita: cada
   cambio produce una versión nueva (`v2026.1` → `v2026.2`) y los presupuestos anteriores quedan
   ligados a la suya. Esto contradice el modelo mental habitual de «pantalla de ajustes».

---

## 2 · Mapeo de la salida

El `output_schema.json` tiene 18 campos *required*. Se agrupan así hacia el frontend:

| Campos del kit | Entidad de plataforma | Superficie de UI |
|---|---|---|
| `boom`, `peso` | `BOM` + `Takeoff` | Grilla de BOM (P6), barra de totales |
| `apu_aisc`, `mas6`, `capitulo_financiero`, `precio_licitacion` | `Pricing` | Opciones de oferta (P9) |
| `opciones` (3, exactos) | `Scenario[]` — **opciones comerciales** | Tarjetas E1/E2/E3 (P9, P5·C) |
| `flujo` | `CashFlow` | Gráfico de flujo de caja (P9) |
| `leyenda_alcance` | atributo de `Offer` | Oferta emitida (P10) |
| `requisicion_neta` | `Requisition` | *(sin UI diseñada)* |
| `secuencia_grua`, `lotes` | `FabricationPlan` | *(sin UI diseñada)* |
| `s11_velocidad` | `PlantCheck` | Panel de planta *(nuevo)* |
| `s12_programa` | `ProgramDecision` — **escenarios de programa** | Pantalla de programa *(nueva)* |
| `rss` | `Uncertainty` | Semáforo en el workspace |
| `lean` | `LeanSaving` | Ingeniería de valor (P9) |
| `confirmar_precios_mp` | `PriceConfirmation` | **Gate de precios** *(nuevo)* |
| `alerta_interferencia_familias` | `Alert` (familia A) | Panel de alertas + vista dedicada *(nueva)* |
| `propuesta_recalibracion` | `RecalibrationProposal` | Bandeja de dirección *(nueva)* |
| `memoria_descriptiva` | artefacto de `Offer` | Descarga |
| `version_parametros` | sello en `Offer` | Chip de versión |

**Ojo con `Scenario`.** El nombre está sobrecargado: `opciones` (comerciales) y
`s12_programa.escenarios` (de programa) son decisiones distintas. Ver [GLOSARIO.md](GLOSARIO.md).
Propuesta para el contrato: `OfferOption` y `ProgramScenario`, evitando `Scenario` a secas.

---

## 3 · `Model` vs. `e2_take_off`: dos capas, no una

Distinción crítica que el contrato debe dejar explícita:

| | `Model` (plataforma) | `e2_take_off` (motor) |
|---|---|---|
| Qué es | Geometría 3D navegable con identidad por elemento | Resumen agregado que el motor consume |
| Campos | Elementos, marcas, `elementIds`, coordenadas, cámara | `fuente`, `factor_facturable_aisc`, `peso_reconstruido_t`, `exclusiones[]` |
| Quién lo posee | Kabunik (**estado canónico**) | Derivado, viaja al motor |
| Quién lo muta | El agente, vía `model_delta` | Se recalcula desde el `Model` |
| Para qué | Renderizar, seleccionar, resaltar | Calcular |

**`model_delta` opera sobre el `Model`, no sobre E2.** E2 se **deriva** del `Model` tras aplicar cada
delta. Si esto no queda escrito, visor y motor se desincronizan — es el riesgo #1 declarado en la
Segmentación v2 §9.

### Requisito derivado del inv. 4 para la percepción

El doble chequeo del peso exige que la percepción produzca **dos pesos calculados por caminos
independientes**:

- el peso del modelo/IFC, y
- el `peso_reconstruido_t`, reconstruido desde el listado de perfiles (Σ longitud × kg/m del catálogo).

Un solo cálculo expuesto dos veces **no satisface el invariante** y hace inútil la compuerta. Debe
constar como requisito en D2.1 (#51).

---

## 4 · API de sesión — sin cambios de forma, con dos endpoints nuevos

La API de la Segmentación v2 §7.2 sigue siendo válida. Los invariantes añaden dos compuertas que
necesitan endpoint propio:

```
POST /project-session                    abre sesión { tenant_id, project_id, inputs }
POST /session/{id}/generate              percepción → Model + BOM
POST /session/{id}/edit                  petición NL/estructurada → model_delta + BOM + costos
POST /session/{id}/scenarios             opciones comerciales sobre modelo validado
POST /session/{id}/bom-approval          gate 1 — BOM aprobado/editado

POST /session/{id}/price-confirmation    gate 2 — inv. 8 · NUEVO
                                         { confirmado, por, fecha, referencia_oferta }
                                         sin esto: estado_emision = bloqueada
POST /session/{id}/program-decision      inv. 14 · NUEVO · ocurre en B2, ANTES del precio
                                         decisión sobre escenarios de programa +
                                         acuse de advertencia_comunicacion_cliente
```

### Eventos de streaming

A los cinco acordados (`progress`, `tool_call.*`, `alert.raised`, `model.updated`,
`human_gate.bom_review_required`, `result.ready`) hay que añadir los de las compuertas nuevas:

```
human_gate.price_confirmation_required   inv. 8
human_gate.program_decision_required     inv. 14
emission.blocked                         { motivo, invariante }   ← estado explícito
parameters.version_pinned                { version_parametros }   ← inv. 16
```

`emission.blocked` merece ser evento de primera clase: hay **cinco** razones distintas por las que la
emisión puede estar bloqueada (inv. 1/4, 3, 8, 11, 15) y la UI debe poder explicar cuál sin adivinar.

### Ciclo de vida ampliado

El de la Segmentación v2 era:

```
perceiving → awaiting_bom_review → computing → ready → editing → …
```

Con los invariantes, y **en el orden que impone la cadena de dependencias del motor** (ver
[FLUJO_v1.md](FLUJO_v1.md)):

```
perceiving
  → awaiting_bom_review            gate 1 · fin del bloque B1 modelo y peso
  → computing_plant                bloque B2 · §4.7–§4.12
  → awaiting_family_decision       gate 4 · solo si INV-03 activa
  → awaiting_program_decision      gate 3 · solo si hay retraso
  → awaiting_price_confirmation    gate 2 · arranque de B3 · corregido 12-ago
  → computing_price                bloque B3 · §4.3–§4.6, §4.13, §4.14
  → ready
  → emitted
  → awaiting_closure               bloque B5 · inv. 16
  → closed
       ↳ recalibration_proposed → approved | rejected      gate 5
  ⟲ editing  desde cualquier estado posterior a B1
```

**Dos correcciones sucesivas de este ciclo de vida:**

1. *(11-ago)* `awaiting_price_confirmation` estaba **antes** de `awaiting_program_decision`. La
   decisión de programa ocurre en B2 porque el APU necesita el pico de S11 (§4.3) y S11 necesita la
   planta cargada (§4.7).
2. *(12-ago)* La confirmación de precios estaba al **final**, en B4. Se mueve al **arranque de B3**:
   los precios son entrada de §4.3, y confirmarlos después de mostrar las opciones convierte la
   compuerta en trámite. En B4 queda una **revalidación de vigencia** que puede reabrirla. Ver
   [FLUJO_v1.md](FLUJO_v1.md) y #94.

**Regla de invalidación en cascada.** Una edición que altere el takeoff no solo reabre el gate 1:
**invalida B2 y B3 completos**. Lotes, S11, S12 y APU se calcularon sobre un peso que ya no es el
vigente. El evento `model.updated` debe indicar qué bloques quedan invalidados, para que la UI marque
como desactualizados no solo los escenarios sino también la decisión de programa.

Estados terminales que antes no existían: `emitted`, `closed`. El invariante 16 convierte el cierre
en parte del ciclo de vida del proyecto, no en un apéndice.

---

## 5 · Servicios de datos agente → plataforma

Los cuatro acordados, más lo que exigen los invariantes:

| Servicio | Estado | Devuelve |
|---|---|---|
| `get_config_tenant(tenant_id, version?)` | Acordado — **añadir `version`** | `TenantConfig` de la versión pedida (inv. 16: no siempre la vigente) |
| `get_benchmarks(filtros)` | Acordado | Referencias agregadas anonimizadas |
| `disponibilidad_perfiles(...)` | Acordado | Stock/inventario del tenant |
| `registrar_cierre(project_id, actuals)` | Acordado | Datos reales para el lazo cotizado-vs-ejecutado |
| `get_plant_load(tenant_id, semanas)` | **Nuevo** | E7 carga: proyectos en curso, HH libres/sem, capacidad de habilitado, **+ marca de frescura** |
| `get_price_list(tenant_id, plant_id?)` | **Nuevo** | E3 precios con fuente/fecha/vigencia restante por material. `plant_id` se acepta desde ya aunque hoy siempre resuelva a la lista del tenant (#102) |

Los dos nuevos existen porque S12 (inv. 14) y la compuerta de precios (inv. 8) no pueden resolverse
con `get_config_tenant`: leen estado operativo semanal, no configuración.

`get_config_tenant` gana el parámetro `version` porque el inv. 16 exige poder **recalcular un
presupuesto histórico con los parámetros con los que se calculó**, no con los vigentes.

---

## 6 · Tool renombrada

`motor_alertas_22` → **`motor_alertas`**. Aplicado en la Segmentación v2 §A.4. La cifra 22 no tenía
fuente (ver T4 en la reconciliación) y congelar un nombre de tool con un número inventado lo habría
vuelto permanente.

---

## 7 · Esquema `Alert` — cerrado

Definido en **[CATALOGO_ALERTAS.md](CATALOGO_ALERTAS.md)** (cierra #68). Lo que el contrato congela:

- La **forma** de `Alert`: `codigo`, `familia`, `severidad`, `bloquea_emision`, `supresible`,
  `titulo`, `detalle`, `invariante`, `dueno`, `elementos_afectados`, `payload`.
- Los **dos ejes independientes**: `bloquea_emision` —lo único que cambia qué puede hacer el
  usuario— y `severidad` en 4 niveles, para orden y filtrado. Se adopta la taxonomía del consultor
  (crítica/alta/media/baja), no la de 3 niveles del mockup.
- Las **19 alertas de familia A** (motor), derivadas de los invariantes y de la rúbrica de 13
  criterios. Son las que tienen comportamiento contractual: 2 compuertas con estado propio en el
  esquema, 5 advertencias no suprimibles, 6 verificaciones de evidencia y 6 avisos.

Lo que **no** congela, y por eso no bloquea: los códigos `VAL-nn` (validaciones de percepción) y
`GRD-nn` (guardarraíles de edición). Añadir un código a esas dos familias **no es un cambio de
contrato**.

---

## 8 · Ritmo de demanda y plan de montaje — cerrado

Definido en **[PLAN_DE_MONTAJE.md](PLAN_DE_MONTAJE.md)** (cierra #70).

`plan_montaje_t_sem` **sigue siendo *required***: el invariante no se relaja. Se añade
`plan_montaje_meta` con la **procedencia** del ritmo — `cliente` | `despacho` |
`derivado_de_plazo` — más `fuente`, `confirmado_por` y `fecha`. Todo aditivo.

Motivo nuevo de `emission.blocked`: **`ritmo_no_confirmado`**, cuando falta `confirmado_por` y la
procedencia no es `cliente`.

---

## 9 · Checklist para C1

| Ítem | Issue | Estado |
|---|---|---|
| C1.1 Esquemas compartidos versionados | #2 | **Replantear**: es mapeo, no diseño. Base en §1, §2, §7 y §8 |
| C1.2 Formato de `model_delta` | #3 | **Propuesta en [MODEL_DELTA_propuesta.md](MODEL_DELTA_propuesta.md)** (11-ago), pendiente de validar con dcode. Es el único hueco de forma que queda |
| C1.3 API de sesión | #4 | Base en §4 y §8; añadir los 2 endpoints de compuerta |
| C1.4 Eventos de streaming | #5 | Base en §4; añadir los 4 eventos nuevos + el motivo `ritmo_no_confirmado` |
| C1.5 Servicios de datos | #6 | Base en §5; añadir los 2 servicios nuevos y el parámetro `version` |
| C1.6 Congelar v0 + versionado | #7 | **Desbloqueado respecto a #68 y #70.** Depende ahora solo de C1.2 |

Los dos huecos que impedían congelar —catálogo de alertas y comportamiento sin plan de montaje— están
resueltos, y **ninguna de las dos resoluciones cambia la forma del payload de manera no aditiva**.

Queda **C1.2 (`model_delta`)** como último elemento de forma pendiente. Es el riesgo #1 declarado en
la Segmentación v2 §9: si su formato no se acuerda, visor y agente se desincronizan.
