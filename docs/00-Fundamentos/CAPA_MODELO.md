# La capa del modelo — frente abierto

> **Documento para la reunión de equipo del 12-ago.** Declara la capa del modelo canónico separando
> lo que está decidido de lo que está abierto, y lista las decisiones que hacen falta.
> Cierra #88.

## Por qué es un frente

Hasta ahora la capa del modelo estaba implícita: sabíamos que el modelo canónico vive en la
plataforma y que el agente lo muta por deltas, pero no habíamos declarado **qué contiene**, **en qué
formatos se proyecta** ni **de dónde sale cada dato**.

Tres cosas la volvieron urgente en la última semana:

1. El **flujo resecuenciado** (#85) mostró que el modelo se enriquece por bloques: primero
   geometría y peso, luego planta y programa, luego costo. El modelo no es un artefacto que se
   produce una vez.
2. El demo de dcode encontró **180 elementos sin peso asignado** y perfiles fuera de catálogo. Eso
   es la capa del modelo pidiendo estructura.
3. La reunión detectó que **el IFC no contiene datos de soldadura** y que los planos PDF los
   complementan. Eso convierte el modelo en un problema de **fusión de fuentes**, no de percepción
   secuencial.

---

## Lo decidido — no reabrir

| Decisión | Dónde está registrada |
|---|---|
| **El modelo canónico vive en la plataforma (Kabunik).** El agente lo muta vía `model_delta`; el visor renderiza desde este estado | `CLAUDE.md` · decisiones cerradas |
| **`Model` es estructura propia y semántica.** No es IFC ni OBJ: esos son proyecciones de él | `CONTRATO_INTEGRACION_v0.md` §3 |
| **`model_delta` opera sobre `Model`.** `e2_take_off` se **deriva** del `Model`, nunca al revés | `CONTRATO_INTEGRACION_v0.md` §3 |
| **Dos pesos por caminos independientes** — peso del modelo y peso reconstruido desde el listado de perfiles | Inv. 4 · `MODEL_DELTA_propuesta.md` |
| **Ninguna cifra sin tool.** El agente orquesta e interpreta; las tools calculan | Guardarraíl de producto |
| **Multitenancy y autenticación son de Kabunik** (K7.1, K7.2, K7.3) | `Segmentacion_Tareas_Kabunik_dcode_v2.md` §5 |
| **Un solo agente orquestador**, con varios puntos de interacción en el userflow. Varios agentes en el futuro son viables sin cambiar la arquitectura | Aclaración de dirección, 12-ago |
| **La planta es entidad de primera clase y un tenant puede tener varias.** Se configura en un flujo paralelo, se versiona, y el proyecto se asocia a `(plant_id, config_version)` en el intake. **`version_parametros` es por planta, no por tenant** | Decisión de dirección, 12-ago · `FLUJO_v1.md` · #94 |
| **El gate de precios de MP va al arranque de B3**, no al final: los precios son entrada de §4.3. En B4 queda una revalidación de vigencia | Decisión de dirección, 12-ago · #94 |

---

## La arquitectura de proyecciones

La idea que ordena todo el frente: **hay un solo modelo y varias proyecciones de él.** Confundirlas
es lo que hace que la discusión «IFC o OBJ» parezca irresoluble.

```
                     FUENTES
   planos PDF/DXF ──┐
   IFC del cliente ─┼──→  percepción  ──┐
   simbología de    ─┘                  │
   soldadura (PDF)                      ▼
                              ┌───────────────────┐
                              │  Model canónico   │  ← estado de registro, Kabunik
                              │  semántico        │     mutable solo por model_delta
                              │  por elemento     │
                              └─────────┬─────────┘
                                        │
              ┌─────────────────────────┼─────────────────────────┐
              ▼                         ▼                         ▼
      PROYECCIÓN DE RENDER      PROYECCIÓN DE CÓMPUTO     PROYECCIÓN DE ENTREGABLE
      glTF / .glb               e2_take_off               IFC4
      ligero, navegable         resumen agregado          para Tekla / Navisworks
      identidad por nodo        alimenta el motor         lo que espera el cliente
```

Consecuencias de leerlo así:

- **El formato de render es una decisión reversible.** Cambiarlo no toca el modelo ni el cómputo.
- **Dejar IFC como camino de render no es dejarlo como entregable.** CNARCCS entregó IFC4 para que
  el cliente lo importara a Tekla/Navisworks. Eso es un compromiso comercial.
- **El inv. 4 no depende del formato.** «Doble chequeo IFC vs. reconstruido» trata de *dos cálculos
  independientes*; la palabra IFC solo nombra el que viene del modelo.

---

## El frente abierto

### 1 · Formato de render

**Abierto.** Se planteó OBJ por ser más fácil de renderizar y de mutar que IFC. El instinto es
correcto: IFC es pesado de renderizar y doloroso de mutar en cada edición.

**Pero OBJ es la elección equivocada** para este producto:

- **No lleva atributos.** La interacción central del mockup es la selección bidireccional grilla ⇄ 3D,
  que necesita identidad estable por elemento **dentro del render**. OBJ tiene grupos, no atributos
  tipados → obliga a un mapa lateral grupo→`elementId`.
- Es texto: para la misma geometría pesa **más** que un binario equivalente.
- No tiene semántica de jerarquía.

**Recomendación: glTF 2.0 / `.glb`.** Nombres por nodo, `extras` para alojar el `elementId`, nativo de
web, binario, y three.js lo carga sin librería adicional. Mismo beneficio buscado, sin perder la
identidad por elemento.

**Quién decide:** spike K2.1 (#13), que ya está abierto con label `needs-definition`.

### 2 · Fusión de geometría con conexiones y soldadura

**Abierto y sin dueño.** Es el hallazgo más consecuente de la reunión.

La geometría viene del IFC o de los planos. **La soldadura y las conexiones vienen de la simbología
del plano PDF, y solo de ahí.** No es un dato secundario:

| Concepto | % sobre peso AISC | Fuente |
|---|---|---|
| Soldadura, estructura atornillada en obra | 0.8% → 2.8% según clase | Informe de factores, hoja 7 |
| Soldadura, estructura soldada en obra | 1.8% → 4.5% | ídem |
| Soldadura, perfiles armados 4 placas con CJP | 4.5% → 8% | ídem |
| Perfil cajón con diafragmas internos | hasta **8.5%** | hoja 5 |
| **Conexiones apernadas** | **8% → 16%** | hoja 7 |
| **Conexiones soldadas** | **5% → 11%** | hoja 7 |

Y dos notas técnicas del consultor que cambian el cálculo:

- Los % son de **consumible depositado, no de consumible a comprar**. Para compras hay que dividir
  por la eficiencia del proceso: **SMAW 65% · FCAW 80% · SAW 95% · GMAW 90%**.
- **La soldadura CJP en perfiles cajón cuesta 4–6 veces más en HH** que el filete equivalente. Y la
  inspección UT/RT al 100% incrementa tiempo, no peso.

**Lo que hay que definir:** dónde alojan las conexiones y la soldadura en el `Model`, con
**procedencia distinta** de la geometría, y cómo llegan a `e2_take_off` para que §4.3 las cobre.
Hoy el `Model` tiene un campo `Conexión` por elemento en el mockup —«Placa base 4 anclas»— pero nada
de soldadura.

**Insumo que ya existe:** el `ROL.docx` del consultor especifica los criterios con detalle —verificar
la simbología del plano, metros lineales de filete según tamaño y rendimiento, biseles para
penetración completa y parcial, tratarlos como semiautomáticos, penetración parcial con FCAW alambre
1.6 mm y CO₂, completa con FCAW salvo indicación de SAW con equipo LT7, y tasa de deposición para
soldadores mexicanos certificados—. Es exactamente el material de la sesión interna de simbología.

### 3 · Estructura y persistencia de la base de datos del modelo

**Abierto en el cómo, cerrado en el dónde.** Aparece como objetivo de dcode para esta semana
—«definir persistencia, estructura por sesión y multitenancy»— y es **K3.2 (#20) + K7.3 (#40)**, del
lado de Kabunik por reparto.

No es una objeción al trabajo exploratorio de dcode: es que si cada equipo asume que la construye él,
el visor y el agente se desincronizan, que es el riesgo #1 declarado.

**Lo que hay que definir:** estructura por sesión, versionado del modelo, aislamiento por tenant, y
cómo se guarda el historial de deltas para que revertir sea posible.

### 4 · Clasificación de elementos y filtro de concreto

**Abierto.** La reunión lo pidió: el equipo de acero debe ver solo su alcance.

Es distinto de las **exclusiones declaradas** del BOM (inv. 5), que son alcance comercial —escaleras,
F1554, Nelson studs, steel deck— y van siempre visibles. Esto es **clasificación y visualización**:
la percepción debe etiquetar cada elemento (acero estructural / concreto / fuera de alcance) y el
visor necesita un filtro.

El `ROL.docx` ya lo instruye: *«No tomes en la evaluación las escaleras ni elementos de concreto
armado, ni tornillería, ni pernos tipo nelson stud»*.

### 5 · Secciones sin peso y el fallback de volumen × densidad

**Abierto, y con un riesgo concreto.** El demo encontró 180 elementos sin peso y propone calcular
volumen × 7.850 kg/m³.

**El fallback es necesario y correcto para lo que se necesita: derivar el kg/m de una sección
desconocida o armada** —los tubos personalizados fabricados en planta—. Ese kg/m entra a §4.1 con
normalidad.

**El riesgo es que se convierta en el camino de peso.** AISC 303-22 §9.2 calcula el peso facturable
con las **dimensiones nominales del detallado** y **no descuenta** cortes, biseles, perforaciones ni
soldadura; las tolerancias de laminación de ASTM A6 (±2.5%) **tampoco se descuentan**. Volumen ×
densidad da un peso **neto geométrico**, que es otro número. Si se cuela como base, la cascada
facturable → a comprar → factor 1.2125 queda mal y **el golden test deja de reproducir**.

**Y hay que separar tres cosas** que el reporte del demo agrupa como «perfiles faltantes»:

| | Qué es | Cómo se calcula |
|---|---|---|
| **Placas** | Sí son peso | Al peso del modelo. El `ROL.docx` incluye placas base y planchas de conexión |
| **Tornillería** | **Partida separada** | **Porcentaje sobre el peso**: 2% → 5% según clase (hoja 7). **Nunca elemento por elemento** |
| **Tubos y perfiles armados de planta** | Sección propia | Fórmula de kg/m; aquí sí aplica el fallback |

Tratar la tornillería como perfil faltante al que asignar peso la **duplica** contra la partida
porcentual.

### 6 · Alcance de los PDFs: escaneado vs. vectorial

**Abierto.** La reunión notó que el PDF vectorial exportado de CAD es mucho más fácil de procesar que
el escaneado. Pregunta de alcance: **¿la v1 soporta planos escaneados?**

Si no, es una validación del intake y hay que declararla, no descubrirla con un cliente. Los planos de
CNARCCS son 48 páginas y 62 MB, probablemente vectoriales — pero eso no dice nada del resto del
mercado.

### 7 · La ingesta asistida de planos

**Abierto, y relevante justo esta semana** porque dcode está trabajando en la interpretación de PDFs.

Falta la superficie donde **el humano confirma sobre el plano** lo que el agente interpretó:
clasificación de páginas —planta, elevación, detalle, isométrico—, grilla de ejes, niveles con su cota,
marcas de perfil, cotas y **simbología de soldadura**.

No es una pantalla de comodidad. **Es la compuerta humana de la fusión del punto 2**: la simbología de
soldadura solo vive en el plano, ninguna máquina la lee perfecta al primer intento, y hasta 8.5% del
peso depende de ella. Que el usuario la confirme sobre el plano es el diseño correcto, no un parche.

Declarada como **P20** en `FLUJO_v1.md`, entre la generación y el bloque B1. Sin mockear.

### 8 · `model_delta.origen`

**Menor y cerrable ya.** Hoy el enum es `agente | usuario | reversion`. Con la aclaración de que se
interactúa con el agente en varios puntos del flujo, conviene que `origen` identifique **en qué punto
del flujo** se originó la edición. Cuesta nada y da trazabilidad — y deja la puerta abierta a varios
agentes sin cambiar el esquema.

---

## Lo que bloquea hoy

**No tenemos los planos PDF.** Para validar que el modelo generado desde PDF es correcto —el criterio
que la propia reunión fijó— hacen falta pares IFC + PDF del mismo proyecto:

| | IFC | BOM | Planos PDF |
|---|---|---|---|
| CNARCCS Domo | ✅ 1,470 elementos | ✅ 1,473 filas | ❌ |
| CNAR Metropolitano | ✅ | ✅ 1,625 filas | ❌ |

El resumen de CNARCCS referencia «planos PDF E1-E19» pero no están en el repo. El material de
referencia estaba comprometido para el **sábado 8 de agosto**.

Sin esos pares, el objetivo técnico de esta semana —el flujo de exploración de PDFs— **no se puede
validar**, solo construir a ciegas. Es el pendiente que más urge, por encima del resto de archivos
del consultor.

---

## Decisiones que se piden en la reunión

| # | Decisión | Quién |
|---|---|---|
| 1 | **Formato de render**: confirmar glTF/`.glb` como proyección de render, e IFC4 como proyección de entregable | Kabunik + dcode · spike #13 |
| 2 | **Dueño de la fusión de soldadura y conexiones** en el modelo, y cuándo entra | dcode + Daniel |
| 3 | **Confirmar que la base de datos del modelo la construye Kabunik** (K3.2, K7.3), y qué necesita dcode de ella | Ambos |
| 4 | **Alcance de PDF escaneado en v1**: sí o no | Producto |
| 5 | **Prioridad de los planos PDF de CNARCCS** por encima del resto de insumos de Daniel | Juan → Daniel |
| 6 | **¿`e5_financieros` ($/HH) va en `PlantConfig`, en `TenantConfig` o partido?** La nómina es de la planta; el overhead puede ser corporativo | Dirección + Daniel |
| 7 | **Confirmar con Daniel la reubicación del gate de precios** al arranque de B3. Él escribió «compuerta final»; el requisito se cumple igual, pero es su invariante | Juan → Daniel |

Los puntos 5, 6 y 8 del frente (secciones sin peso, PDF escaneado, `origen` del delta) no necesitan
la reunión: se resuelven con los issues abiertos. El 7 (ingesta asistida) necesita diseño, no decisión.
