# Catálogo de alertas y validaciones

> Cierra el hueco #68. Sustituye la cifra **«22 alertas»**, que no tenía fuente en ninguno de los
> 7 entregables del consultor.
> **2026-08-11** · insumo de C1.1 (#2) y C1.4 (#5), habilita el congelamiento de C1.6 (#7).

## Qué congela este documento y qué no

El contrato v0 **no necesita enumerar todas las alertas**. Necesita dos cosas, y esas sí quedan
cerradas aquí:

1. **La forma de `Alert`** — el esquema compartido.
2. **Las alertas con comportamiento contractual** — las que bloquean la emisión y las que no se
   pueden suprimir. Son las derivadas de invariantes, y están completamente respaldadas por
   `output_schema.json` y la rúbrica de la `Guia_Ejemplos` §4.

Lo que **no** se congela y queda como catálogo extensible de dcode: los códigos de validación de
percepción (familia B) y los guardarraíles de edición (familia C). Añadir un código nuevo a esas dos
familias **no es un cambio de contrato** — por eso el contrato puede congelarse sin esperar a que
estén completas.

## Los dos ejes: bloquea vs. severidad

Distinción deliberada, y es la mitigación de la **fatiga de alertas** que el consultor marca como
riesgo residual abierto (*«advertencias que nunca se apagan pueden normalizarse y dejar de leerse»*):

| Eje | Valores | Para qué sirve | En la UI |
|---|---|---|---|
| **`bloquea_emision`** | `true` \| `false` | Es la única distinción que cambia lo que el usuario **puede hacer** | Eje visual primario: rojo bloquea · ámbar exige decisión · neutro informa |
| **`severidad`** | `critica` \| `alta` \| `media` \| `baja` | Orden y filtrado dentro de cada grupo | Pill de severidad, secundaria |

Se adoptan **los cuatro niveles del consultor** (`Como_Funciona_el_Software.pptx`, bloque 2.2), no los
tres del mockup. El nivel `alta` que el mockup había perdido es justo el que él usa para *«factores
que mueven el costo: soldadura CJP especial, clasificación de correas»* — una banda real y distinta
de `media`.

## Esquema `Alert`

```json
{
  "codigo":              "INV-08",
  "familia":             "motor | percepcion | guardarrail",
  "severidad":           "critica | alta | media | baja",
  "bloquea_emision":     true,
  "supresible":          false,
  "titulo":              "Precios de materia prima sin confirmar",
  "detalle":             "Sin confirmación registrada en bitácora no se emite precio de licitación.",
  "invariante":          8,
  "dueno":               "dcode",
  "elementos_afectados": ["..."],
  "payload":             { }
}
```

Reglas del esquema:

- `codigo` — patrón `^(INV|VAL|GRD)-[0-9]{2}[a-z]?$`. El prefijo determina la familia.
- `invariante` — entero 1–16, o `null` si la alerta no deriva de un invariante.
- `supresible: false` significa que **la UI no puede ofrecer «Ignorar» ni «Ocultar»**. Sí puede
  ofrecer una decisión registrada (p. ej. «Aceptar el impacto documentado»), que es otra cosa.
- `elementos_afectados` — `elementIds` del `Model` canónico, para el enlace bidireccional con el
  visor. Vacío cuando la alerta no es geométrica.
- `payload` — libre por código, documentado en este catálogo. `INV-03` es el caso con payload más
  rico (ver abajo).

## Familia A · Motor — cerrada y contractual

`dueno: dcode` (tools). Derivada de los 16 invariantes y de la rúbrica de 13 criterios. Prefijo `INV`.

### A1 · Compuertas de emisión — el esquema las modela con estado explícito

Son las dos únicas donde **un valor calculado bloquea por sí mismo**. Ambas tienen un campo de estado
en `output_schema.json`, no solo una alerta:

| Código | Condición | Campo del esquema | Inv. | Rúbrica | Severidad |
|---|---|---|---|---|---|
| `INV-04` | Peso IFC vs. reconstruido fuera de tolerancia | `peso.doble_chequeo = "detener_emision"` | 1, 4 | 1\* | crítica |
| `INV-08` | Precios de MP sin confirmación registrada | `confirmar_precios_mp.estado_emision = "bloqueada"` | 8 | 6\* | crítica |

### A2 · Advertencias no suprimibles — se emiten siempre que apliquen

**No bloquean el cálculo**, pero condicionan la recomendación y **no tienen switch de silencio**:

| Código | Condición | Campo del esquema | Inv. | Severidad | Efecto |
|---|---|---|---|---|---|
| `INV-03` | Familia de HH altas satura habilitado y baja la productividad de otra más rentable | `alerta_interferencia_familias.estado = "activa"` | 3 | alta | Exige decisión documentada |
| `INV-12a` | Pico de montaje excede la velocidad con sobretiempo | `s11_velocidad.alerta_subcontrato_hh > 0` | 12 | alta | Costo de subcontrato visible |
| `INV-12b` | Planta operando bajo el punto de equilibrio | `s11_velocidad.advertencia_bajo_equilibrio.activa` | 12 | alta | Visible también en la oferta |
| `INV-14a` | Ocupación de habilitado ≥100% en alguna semana | `s12_programa.alerta_ocupacion[i] = true` | 14 | alta | — |
| `INV-14b` | Hay retraso y no consta el acuerdo con el cliente afectado | `s12_programa.advertencia_comunicacion_cliente` | 14 | crítica | Sin acuerdo la recomendación cae al escenario A |

### A3 · Verificaciones de evidencia — su fallo invalida la emisión

Aquí está el matiz que conviene tener claro antes de implementar: **la mayoría de las condiciones
bloqueantes de la rúbrica no son «un número malo», son la ausencia de evidencia obligatoria.** Se
comprueban antes de emitir; no son alertas de tiempo de ejecución:

| Código | Verifica | Inv. | Rúbrica | Severidad |
|---|---|---|---|---|
| `INV-02` | Clase correcta, sin promedios; salto ≥1 → subproyectos por familia | 2, 3 | 2\* | crítica |
| `INV-05` | Exclusiones declaradas con leyenda; ningún cero silencioso | 5 | 3\* | crítica |
| `INV-06a` | % de crecimiento registrado y justificado si difiere del default por fase RSS | 6 | 5\* | crítica |
| `INV-11` | Déficit resuelto de forma explícita, con costo visible, contra planta cargada | 11 | 8\* | crítica |
| `INV-14c` | S11/S12 con sus advertencias presentes y retraso mínimo calculado | 12, 14 | 9\* | crítica |
| `INV-15a` | $/HH desde financieros reales dentro de tolerancia 2%; sin IVA/ISR en el motor | 15 | 11\* | crítica |

### A4 · Avisos que no bloquean

| Código | Condición | Inv. | Rúbrica | Severidad |
|---|---|---|---|---|
| `INV-06b` | Reconciliación APU vs. Más X% ≥ 1% a margen comparable | 6 | 4 | alta |
| `INV-08b` | Precio de MP con vigencia ≤7 días o vencida | 8 | 6 | alta |
| `INV-10` | Desviación de concurso registrada sin impacto costeado | 10 | 7 | media |
| `INV-15b` | Cargas de planta > overhead disponible | 15 | 11 | alta |
| `INV-16a` | Propuesta de recalibración pendiente de aprobación de dirección | 16 | 13 | media |
| `INV-16b` | Proyecto emitido sin cierre registrado — no alimenta recalibraciones | 16 | 13 | baja |

**19 códigos en la familia A** — 2 compuertas + 5 advertencias no suprimibles + 6 verificaciones de
evidencia + 6 avisos. Los 8 criterios bloqueantes de la rúbrica quedan cubiertos por A1 (2) y A3 (6).

### Payload de `INV-03`

El único con payload normado por `output_schema.json`. Se reproduce porque la UI (P16) depende de él:

```json
{
  "familia_alta_hh": "Misceláneos", "familia_afectada": "Armaduras",
  "tonelaje_alta_t": 80, "tonelaje_afectada_t": 320,
  "hh_ton_alta": 90, "hh_ton_afectada": 24,
  "hh_conflicto_sem": 210,
  "rentabilidad_alta_por_hh": 31, "rentabilidad_afectada_por_hh": 58,
  "impacto_estimado": "Armaduras pierden 12% de productividad → 380 HH extra",
  "opciones": ["repriorizar", "partir_lote", "subcontratar_familia_pesada", "aceptar_impacto"]
}
```

## Familia B · Validaciones de percepción — semilla extensible

`dueno: dcode` (D2, #51). Prefijo `VAL`. **Ninguna bloquea la emisión**, pero condicionan el gate de
BOM: el humano decide si aprueba con ellas abiertas.

La semilla no es inventada: son las **7 advertencias de ingeniería reales** del resumen de cómputo de
CNARCCS Domo, clasificadas con la taxonomía del propio consultor.

| Código | Advertencia | Severidad |
|---|---|---|
| `VAL-01` | Cantidades estimadas por conteo típico según espaciamientos de planos; confirmar contra planos de detalle | alta |
| `VAL-02` | Subconjunto modelado de forma agregada por marco tipo; su tonelaje puede variar al detallar | alta |
| `VAL-03` | Sin peso de referencia del ingeniero de cálculo; recomendado calibrar (tolerancia objetivo ≤5%) | alta |
| `VAL-04` | Partida cold-formed incluida en el peso por decisión del cliente; comercialmente es AISC 303-22 §2.2 y conviene cotizarla como sub-partida | media |
| `VAL-05` | Grilla de ejes inferida donde el plano no la declaraba | crítica |
| `VAL-06` | Cómputo derivado de lectura de planos; no sustituye un modelo de detalle de fabricación | media |
| `VAL-07` | Representación de eje en el IFC (`IfcPolyline` Axis); asignar perfil de catálogo al importar para visualizar sólidos | baja |

Taxonomía de referencia del consultor para clasificar códigos nuevos:

| Severidad | Qué cae aquí |
|---|---|
| **crítica** | Supuestos que **cambian el alcance**: grilla inferida, partidas no incluidas |
| **alta** | Factores que **mueven el costo**: soldadura CJP especial, clasificación de correas |
| **media** | Decisiones técnicas **a confirmar**: tipo de material, marco contractual |
| **baja** | Detalles menores **ya resueltos**: tipo de tornillería, sistema de unidades |

> Los códigos `VAL-07`, `VAL-12` y `VAL-19` que usa el mockup son **ilustrativos** y no coinciden con
> esta numeración. Al implementar, la UI toma los códigos del catálogo, no del mock.

## Familia C · Guardarraíles de edición — semilla extensible

`dueno: dcode` (D1) + `TenantConfig`. Prefijo `GRD`. **Rechazan el delta, no la oferta**: nunca
bloquean la emisión porque el cambio no llega a aplicarse.

| Código | Condición | Severidad |
|---|---|---|
| `GRD-01` | Perfil fuera del catálogo calibrado del tenant | alta |
| `GRD-02` | Perfil en catálogo pero sin disponibilidad en el horizonte del proyecto | media |
| `GRD-03` | Grado de acero no admitido por las especificaciones del concurso | alta |
| `GRD-04` | La edición dejaría el modelo fuera de una clase del eje sin partir en subproyectos | alta |

Todo rechazo `GRD` debe devolver **alternativas equivalentes del catálogo** con su impacto, como ya
modela el mockup: *Armado 900x14 A572 (Ix equivalente 98% · +$2,140/t)*.

## Renombre de la tool

`motor_alertas_22` → **`motor_alertas`**. Aplicado en
`docs/Diseño de producto/Segmentacion_Tareas_Kabunik_dcode_v2.md` §A.4. Congelar un nombre que
contenía una cifra sin fuente lo habría vuelto permanente. Ver #78 para la cobertura completa del
inventario de tools.

## Lo que queda abierto

Nada bloquea el congelamiento del contrato. Sí quedan dos ampliaciones, ambas **aditivas**:

| Pendiente | Dueño | Por qué no bloquea |
|---|---|---|
| Completar familia B con los casos que aparezcan al desarrollar percepción | dcode (D2, #51) | Añadir un `VAL-nn` no cambia el esquema `Alert` |
| Completar familia C con los guardarraíles que exija la calibración real | dcode (D1) + K6 (#35) | Ídem con `GRD-nn` |
| Confirmar con Daniel que A3 se implementa como verificaciones pre-emisión y no como alertas de runtime | Daniel (J1.3, #64) | Es decisión de implementación; el esquema no cambia |
