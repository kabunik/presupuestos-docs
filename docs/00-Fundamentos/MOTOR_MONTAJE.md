# El motor de montaje: reconciliación y wizard

> El consultor entregó el **12-ago** un motor de montaje funcional en Python, con su especificación de
> integración declarada vinculante. Dirección decidió el **13-ago** incluirlo en la v1.
> Cierra #112. Material en `docs/doc inicial/Contexto completo Michelena/Modulo montaje/`.

## La decisión y su reparto

**El motor de montaje entra en el alcance de la v1.** Revierte lo que decía
[ALCANCE_v1.md](ALCANCE_v1.md), que lo tenía fuera con la interfaz reservada.

La razón es de coherencia, no de ambición: **ya tenemos vistas que muestran prácticamente su output**
—P19 requisición, lotes y grúa; P15 escenarios de programa—. Mantenerlo fuera dejaría esas pantallas
alimentadas por nada.

| Pieza | Quién |
|---|---|
| El motor, envuelto como **tool del agente** | dcode. El código ya es funcional, no se reescribe |
| **Wizard dedicado y paralelo** para sus entradas | Kabunik. Mismo patrón que la definición de planta |
| **Opción de alcance** que lo activa | Kabunik, en especificaciones de proyecto. Si el alcance es solo suministro, el wizard se desactiva |

## Qué es, en concreto

Un pipeline de Python 3.10+ con smoke test, sin servicios externos, todo por archivos JSON. Convierte
un IFC en presupuesto de montaje **en USD** con dispersión declarada, y cierra un **lazo de
co-optimización con fabricación**: lotes ≤50 t y frontera costo–tiempo.

```
extract_montaje.py    IFC → dataset canónico (tonelaje, histograma de izaje, pernos de campo)
preparar_montaje.py   dataset + params → pre-figuras, con factores de realidad
motor_lazo.py         pre-figuras + tarifas → frontera del lazo + lotes de retorno
retro_montaje.py      plantilla de retroalimentación → calibración propuesta (con firma)
```

Los «factores de realidad» son lo que lo hace creíble: reposicionamientos de grúa, rampa de
aprendizaje, ausentismo, PF&D, doble manejo, obra falsa, calendario con lluvia y festivos, NCR, y
bandas por partida que producen una `dispersion_esperada_pct`.

---

## Los cinco conflictos, y su resolución

### 1 · Moneda: el motor trabaja en USD, el Sistema DM en MXN

El motor exige **USD en todo importe** y declara `fx_usado` (18.5 en el ejemplo). El golden test y las
ofertas de CNARCCS están en **MXN**.

**Resolución.** El puente ya existe en el esquema: `fx_usado` en `tarifas_taller.json`. Pero hay una
consecuencia que no se puede dejar implícita: **una oferta emitida queda ligada a un tipo de cambio,
igual que a una versión de parámetros.** Si el FX no se sella junto a la oferta, la trazabilidad se
rompe — el mismo proyecto recalculado meses después daría otra cifra sin que nada lo explique.

Así que el FX es un **parámetro versionado del tenant**, no una constante, y viaja en el sello de la
oferta junto a `version_parametros`.

### 2 · Tolerancia ±3% vs. ≤5% — no es un conflicto

Su compuerta **G1 detiene el pipeline** si `bom_contractual.toneladas_contractuales` difiere del IFC
más de **±3%**. El inv. 4 y el resumen de CNARCCS hablan de **≤5%**.

**Resolución: no son el mismo chequeo, y confundirlos sería el error.**

| | Compara | Tolerancia |
|---|---|---|
| **Inv. 4 · doble chequeo** | Dos cálculos internos: peso del modelo vs. peso reconstruido | **Δ < 1%** (regla R7 de v1.3) |
| **G1 del motor de montaje** | El modelo contra el **tonelaje contractual firmado** | ±3% |

El primero pregunta «¿calculé bien?». El segundo, «¿lo que voy a montar coincide con lo que se
contrató?». Son dos compuertas distintas y ambas legítimas. Hay que nombrarlas separadas en el
catálogo de alertas para que nadie las funda en una.

### 3 · La verificación de soldadura de campo es una compuerta nueva

El README lo dice sin ambigüedad: *«Soldadura de campo: verificación SIEMPRE obligatoria — estado
INGRESADA / PENDIENTE / NO DISPONIBLE. Integrar en UI como semáforo bloqueante.»*

**Sería la sexta compuerta humana.** Y resuelve de paso una ambigüedad que arrastrábamos: el esquema
`computos_soldadura.json` separa explícitamente **`campo`** de **`taller`**, mientras nosotros
tratábamos «soldadura» como una sola cosa en #89. Son dos datos distintos, con dos fuentes distintas y
dos consecuencias distintas.

Regla suya que conviene adoptar tal cual: **`campo.kg_aporte = 0` es un dato; la ausencia del archivo
es una bandera bloqueante.** Es la misma disciplina del «cero silencioso» del inv. 5, aplicada a otro
sitio.

### 4 · Identidad de pieza `{marca, guid_ifc}` — y una trampa

Exige que toda pieza se identifique por el par **`{marca, guid_ifc}`**. Eso valida nuestra identidad
por elemento y la restringe: el `elementId` del `Model` debe poder mapear a ese par.

**La trampa está en que decidimos que IFC es proyección derivada, no fuente de verdad.** Si el
`guid_ifc` se leyera del IFC, cada regeneración produciría GUIDs nuevos y la identidad de pieza se
rompería entre corridas del motor de montaje.

Por tanto: **el `guid_ifc` lo asigna y lo persiste el modelo canónico**, y la exportación a IFC lo
reutiliza. Es un atributo del `Model`, no del archivo.

### 5 · Cómo se envuelve como tool: son cuatro pasos con estado

El motor no es una función: es un **pipeline con estado entre pasos**
—`dataset → prefiguras → frontera → lotes`—, y cada paso escribe un archivo que el siguiente lee.

Eso no encaja en una tool sin estado. Hay dos formas y hay que elegir:

| Opción | Implicación |
|---|---|
| **A · Una tool con estado** | dcode expone un solo `motor_montaje` que orquesta los 4 pasos internamente. Simple de llamar, opaco de depurar |
| **B · Cuatro tools encadenadas por la plataforma** | La plataforma guarda los artefactos intermedios y los pasa. Más trazable —cada paso es auditable— y encaja mejor con «ninguna cifra sin tool» |

**Recomendación: B.** Los artefactos intermedios son evidencia, y guardarlos es lo que permite
explicar una cifra después. Pero la decisión es de dcode y va al canal.

---

## El contrato de intercambio

**Cinco archivos FAB → MONTAJE.** Y aquí hay una distinción que ordena el wizard: **tres se derivan de
lo que ya tenemos, dos los introduce el usuario.**

| # | Archivo | De dónde sale |
|---|---|---|
| 1 | `despiece_tornilleria.json` | **Derivado** de la percepción y las tools. Mata la estimación de pernos por tonelaje |
| 2 | `computos_soldadura.json` | **Derivado**, con la verificación humana de campo del wizard |
| 3 | `cronograma_fab.json` | **Derivado** del bloque B2: lotes, planta y fecha de liberación |
| 4 | `tarifas_taller.json` | **Del usuario** — y buena parte ya vive en `PlantConfig` |
| 5 | `bom_contractual.json` | **Del usuario**, y va **firmado**: `firmado_por` + `fecha_firma` |

**Tres archivos MONTAJE → FAB**, que tenemos que ingerir: `lotes_montaje.json` (reprogramar
producción), `senales_ingenieria_valor.json` (a diseño) y `calibracion_hist.json` (NCR a calidad).

Nota importante sobre el último: **`calibracion_hist.json` es compartido con fabricación** y acumulativo
multiproyecto. Y su calibración **no se auto-aplica: requiere firma**. Es exactamente el inv. 16
operando en el otro motor — dos sistemas distintos llegaron a la misma regla de gobernanza.

### Solape con `PlantConfig`

`tarifas_taller.json` pide `hh_ton_por_tipologia` (pesada/mediana/liviana), `usd_hh_taller`,
`usd_ton_material_proceso`, `costo_cambio_lote_usd`, `penal_lote_chico_usd_ton` y
`capacidad_taller_ton_sem`.

Tres de esos ya los tenemos en `PlantConfig`: las HH/ton son nuestro eje de complejidad, el $/HH sale
del paso D, y la capacidad semanal del paso B. **No se piden dos veces**: el wizard los muestra
heredados de la planta —en solo lectura, con su procedencia— y solo captura los que faltan, que son
los costos de loteo y la conversión a USD.

Ojo con una diferencia de vocabulario: él usa **pesada / mediana / liviana**; nuestro eje es
**24 / 40 / 60 / 90**. Hay que mapearlos explícitamente, no asumir la correspondencia.

> **Resuelto el 19-ago** (#119). El paquete v1.3 trae las bandas y la regla R5 las hace normativas:
> **pesada 14–26 · mediana 40–60 · liviana 80–999 HH/ton** → nuestro **24 · 40 y 60 · 90**.
> La nomenclatura es la contraria a la intuitiva —«pesada» es sección pesada, que rinde *menos* HH por
> tonelada— y por eso no había que adivinarla. Las tarifas de montaje (20 / 50 / 95) caen las tres
> dentro de su banda: dos archivos distintos concuerdan. Detalle en
> [SISTEMA_PRESUPUESTOS_v1.3.md](SISTEMA_PRESUPUESTOS_v1.3.md).

---

## El wizard de montaje

Paralelo al de planta, activado por la opción de alcance. Lo que tiene que capturar son los
**overrides de `params.json`** más los dos archivos de usuario:

```
P22·A  Alcance y plazo          T_MAX_SEM · si el alcance incluye montaje
                                ▸ si es solo suministro, el wizard no se activa
P22·B  Loteo y transporte       LOTE_TON (≤50 t) · transporte_incluido_en
                                costo de cambio de lote · penalización de lote chico
P22·C  Soldadura de campo       SOLD_CAMPO_KG · reparto por posición (plana/horizontal/
                                vertical/sobrecabeza) · % de UT
                                ▸ COMPUERTA: sin verificación no se corre el motor
P22·D  Condiciones de obra      obra falsa y sus HH · días de lluvia/mes · meses de lluvia
                                festivos · condiciones de sitio
P22·E  Tarifas y BOM contractual  heredado de PlantConfig en solo lectura + lo que falta
                                tonelaje contractual FIRMADO · fx_usado
```

Dos reglas de diseño que salen del propio motor:

- **Toda estimación automática queda marcada** con su fuente y una bandera, y el valor real siempre
  puede sustituirla. Es su convención y coincide con la nuestra de badges `tool ✓` vs. `editado`.
- **Las decisiones de izaje, secuencia y tonelaje llevan firma humana.** *«El software propone, no
  cierra»* — su frase, nuestra misma ley.

---

## Lo que queda abierto

| # | Abierto | Quién |
|---|---|---|
| 1 | ¿Tool con estado o cuatro tools encadenadas? | dcode |
| 2 | ~~Mapeo de tipologías: pesada/mediana/liviana ↔ eje 24/40/60/90~~ · **cerrado el 19-ago** (#119) | — |
| 3 | ¿Dónde vive el FX y cómo se sella en la oferta? | Conjunto |
| 4 | Formato de la requisición a fabricación — declarado pendiente en la reunión | Conjunto |
| 5 | El **empalme de piezas** afecta el desperdicio de forma significativa y no está en nuestro modelo | Daniel + C1 |

El punto 5 salió en la reunión y es de los que cambian cifras: si una pieza admite empalme o no mueve
el desperdicio, y hoy nuestro `Model` no lo registra.
