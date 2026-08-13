# `model_delta` — propuesta para C1.2

> **Propuesta de Kabunik para #3.** No es acuerdo: el formato del delta es un ítem bilateral y se
> cierra por el canal de decisiones (J1.3, #64). Este documento existe para que dcode tenga algo
> concreto que validar o corregir en vez de partir de cero.
> **2026-08-11** · último hueco de forma que impide congelar C1.6 (#7).

## Por qué importa tanto

Es el **riesgo #1** declarado en la Segmentación v2 §9: *«si no se acuerda el formato del delta, el
visor y el agente se desincronizan»*. Y es la costura de la Fase 2, el diferenciador del producto.

## Cinco restricciones que el formato debe respetar

Ninguna es negociable; salen de decisiones ya cerradas.

| # | Restricción | De dónde viene |
|---|---|---|
| 1 | **Opera sobre `Model`, nunca sobre `e2_take_off`.** E2 se **deriva** del `Model` tras aplicar el delta | `CONTRATO_INTEGRACION_v0.md` §3 |
| 2 | **El delta no trae los dos pesos.** Trae el cambio del modelo; la reconstrucción se recalcula por su camino independiente | Inv. 4 — si ambos pesos vinieran de la misma fuente, el doble chequeo sería teatro |
| 3 | **Ninguna cifra sin tool.** Toda cifra del delta declara qué tool la produjo | Guardarraíl de producto |
| 4 | **Declara qué bloques invalida.** Una edición del takeoff invalida B2 y B3 completos | `FLUJO_v1.md` |
| 5 | **Idempotente y reversible.** Revertir crea una versión nueva; nunca borra historial | Handoff de diseño §6 |

## Forma propuesta

```json
{
  "delta_id":             "d_01JQ8F3ZK2",
  "model_version_from":   1,
  "model_version_to":     2,
  "origen":               "agente",
  "intencion":            "Cambia las columnas del nivel 3 a HSS 10x10x½",
  "revierte_delta":       null,

  "ops": [
    {
      "op":       "set_property",
      "elementos": ["e_1042", "e_1043", "…24 ids"],
      "propiedad": "perfil",
      "valor":     "HSS 10x10x1/2",
      "procedencia": { "tool": "kgm_perfil", "kg_m": 92.3, "catalogo": "METALITEC v4" }
    }
  ],

  "afecta": {
    "elementos": ["e_1042", "…"],
    "marcas":    ["C-115"],
    "familias":  ["Columnas"]
  },

  "invalida": ["B1", "B2", "B3"],
  "requiere_doble_chequeo": true,
  "reabre_gates": [1],

  "resumen": {
    "tonelaje_delta_t": -8.2,
    "elementos":        24,
    "procedencia":      { "tool": "weight_calc" }
  }
}
```

`resumen` **no incluye el costo.** El impacto en costo no es del delta: sale de volver a correr B3, y
solo existe si B3 ya había corrido antes. La tarjeta de delta del mockup muestra «Costo −$14,300»
porque representa una edición iterativa posterior a una pasada completa; en la primera revisión ese
campo no existe.

## Las cinco operaciones

Conjunto cerrado. Si dcode necesita expresar una edición que no cabe en estas cinco, es señal de que
falta una op — no de que haya que meterla a presión en `set_property`.

| Op | Payload | Qué cambia | Invalida |
|---|---|---|---|
| `set_property` | `elementos`, `propiedad`, `valor` | Perfil, grado, acabado, tipo de conexión | Según la propiedad, ver tabla abajo |
| `set_geometry` | `elementos`, `nodos: [{id, xyz}]` | Coordenadas — reducir luz, cambiar altura | B1, B2, B3 |
| `add_elements` | `elementos: [{…}]` | Elementos nuevos | B1, B2, B3 |
| `remove_elements` | `elementos` | Elementos eliminados | B1, B2, B3 |
| `set_family` | `elementos`, `familia`, `clase_hh_ton` | Asignación de familia y partición en subproyectos | B2, B3 |
| `add_cost` | `elementos`, `concepto`, `importe`, `procedencia` | Adjuntar costo al modelo. **Propuesto por dcode** (#112) | B3 |

> **Convergencia con dcode (13-ago, #112).** En la reunión del 12-ago dcode listó sus tools de delta:
> `set_geometry`, `set_property`, `add_elements`, `remove_elements`, `add_cost`. **Cuatro de las cinco
> coinciden exactamente** con esta propuesta, hecha el 11-ago sin haber visto la suya. La quinta
> difiere y las dos hacen falta, así que el conjunto pasa a **seis operaciones**.
>
> `add_cost` tiene una condición que no es negociable: **debe llevar procedencia de tool**. Si el
> agente pudiera escribir un costo que él mismo calculó, «ninguna cifra sin tool» se cae.

`set_family` existe por el **inv. 3**: partir el proyecto en subproyectos por familia es una mutación
del modelo, no un ajuste de configuración, y tiene que quedar en el historial de versiones como
cualquier otra edición.

### Invalidación por propiedad en `set_property`

No toda edición invalida lo mismo, y tratarlas igual haría que cada cambio de acabado tirase el
programa entero:

| Propiedad | Cambia peso | Cambia HH | Invalida |
|---|---|---|---|
| `perfil` | Sí | Sí | B1, B2, B3 |
| `grado` | No | No | **B3** (cambia el precio del material, no el peso) |
| `conexion` | **Sí** | **Sí** | **B1, B2, B3** |
| `acabado` | No | No | **B3** |

**Corregido el 12-ago.** `conexion` figuraba como «no cambia el peso» y es falso: según la hoja 7 del
informe de factores, las **conexiones apernadas pesan 8%–16% del peso AISC frente a 5%–11% las
soldadas** —más placas, atiesadores y splice—, y el factor de tornillería también se mueve. Cambiar el
tipo de conexión invalida los tres bloques.

Es la propiedad que más fácil se implementa mal. El ejemplo del mockup —*«186 conexiones del anillo
perimetral pasan a empernadas — −214 HH de taller, +$3,900 en tornillería»*— ya muestra las dos
mitades: bajan las HH de soldadura y sube la tornillería. Lo que faltaba ver es que el **peso de las
placas de conexión también cambia**.

## Selector: siempre IDs resueltos

El agente interpreta *«las columnas del nivel 3»*, pero **el delta que emite lleva los `element_id`
resueltos**, nunca la consulta.

Razón: un delta con una consulta (`{nivel: 3, tipo: "columna"}`) no es reproducible — al reaplicarlo
sobre un modelo que cambió, selecciona un conjunto distinto. Con IDs explícitos el delta es
idempotente y replayable, que es lo que permite el historial de versiones y el revertir.

Coste: deltas más verbosos. Aceptable, y se mitiga con el chunking de abajo.

## Concurrencia

`model_version_from` funciona como control optimista: **si no coincide con la versión vigente del
modelo canónico, la plataforma rechaza el delta** con `409` y el agente debe re-leer estado.

Esto cubre el riesgo de *«concurrencia de ediciones»* que la Segmentación v2 §9 marca como punto de
integración delicado, y es lo que K3.3 (#21) tiene que implementar.

## Reversión

```json
{ "origen": "reversion", "revierte_delta": "d_01JQ8F3ZK2",
  "model_version_from": 4, "model_version_to": 5 }
```

Genera una versión **nueva**. Las `ops` son las inversas, calculadas por la plataforma a partir del
estado guardado — no las manda el agente, porque la plataforma es la dueña del estado canónico.

## Streaming en dos tiempos

El evento `model.updated` se emite **en dos partes**, y no es un detalle de implementación: es lo que
hace posible la UX que el handoff pide —*«durante "aplicando" el visor no se recarga: solo cambian los
elementos afectados»*—.

```
model.updated.header   { delta_id, afecta, invalida, resumen, ops_total }
                       → el visor ya puede pintar en violeta y mostrar
                         «24 elementos afectados · reconstruyendo»
model.updated.ops      { delta_id, chunk: {i, n}, ops: [...] }
                       → se aplican por lotes
model.updated.commit   { delta_id, model_version_to }
                       → atómico: hasta aquí el modelo canónico no cambió
```

Con `ops_total` en la cabecera la UI puede mostrar progreso real en ediciones grandes. Un delta de
tres elementos manda cabecera, un chunk y commit.

## Qué necesita decidir dcode

Cuatro puntos, y los cuatro son suyos:

1. **¿Las cinco ops cubren su motor de edición?** Es la pregunta central. Si falta una op, mejor
   saberlo antes de congelar que después.
2. **¿Puede resolver siempre el selector a IDs** antes de emitir, o hay casos donde necesita emitir la
   consulta y que la plataforma la resuelva? Si es lo segundo, cambia la garantía de idempotencia y
   hay que discutirlo.
3. **Tamaño típico y máximo** de un delta en elementos, para dimensionar el chunking.
4. **¿La tabla de invalidación por propiedad es correcta** desde el lado del cómputo? Sobre todo el
   caso de `conexion`.

## Lo que Kabunik se compromete a hacer

- Mantener el `Model` como estado canónico y aplicar los deltas de forma atómica.
- **Derivar `e2_take_off`** del `Model` después de cada commit, y **recalcular el peso reconstruido
  por su camino independiente** para que el doble chequeo del inv. 4 siga siendo real.
- Rechazar deltas con `model_version_from` desfasado.
- Reabrir los gates que el delta declare y **marcar como desactualizados los bloques invalidados** —
  no solo las opciones de oferta, también la decisión de programa.
- Guardar el historial completo y calcular las ops inversas al revertir.
