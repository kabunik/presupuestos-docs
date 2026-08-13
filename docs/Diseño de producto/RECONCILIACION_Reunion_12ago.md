# Reconciliación de la reunión del 12-ago

> Qué de lo acordado en la reunión coincide con lo que teníamos, qué lo corrige y qué es nuevo.
> **13-ago** · issue #112

## La convergencia que más dice

En la reunión dcode listó las tools de `model_delta` que tienen definidas:

> `set_geometry`, `set_property`, `add_elements`, `remove_elements`, `add_cost`

Yo había propuesto, el 11-ago y sin haber visto eso:

> `set_property`, `set_geometry`, `add_elements`, `remove_elements`, `set_family`

**Cuatro de cinco coinciden exactamente.** Dos equipos llegaron al mismo conjunto de operaciones por
caminos separados, lo que es la mejor señal posible de que el formato del delta va bien encaminado.

La quinta difiere, y las dos hacen falta:

| Op | Quién la tenía | Para qué |
|---|---|---|
| `add_cost` | dcode | Adjuntar costo al modelo. **Debe llevar procedencia de tool** — si el agente pudiera escribir un costo que él mismo calculó, «ninguna cifra sin tool» se cae |
| `set_family` | Kabunik | Asignar familia y partir en subproyectos (**inv. 3**). Es mutación del modelo, no configuración, y tiene que quedar en el historial |

**Propuesta: seis operaciones.** Y con ello #3 queda prácticamente cerrado en su punto más difícil —
la pregunta de si las ops cubren el motor de edición se responde sola cuando los dos lados proponen lo
mismo.

## Lo que confirma lo que ya teníamos

| De la reunión | Lo nuestro |
|---|---|
| **glTF 2.0 como formato base**, con propiedades por elemento | Decidido el 12-ago |
| **IFC es output derivado, no fuente de verdad** | La arquitectura de proyecciones de `CAPA_MODELO.md`, palabra por palabra |
| Modificaciones del agente **directamente sobre el modelo canónico vía deltas** | Decisión cerrada desde el principio |
| **Loop de confirmación humana antes de generar el modelo 3D** | Es P20, la ingesta asistida |
| **Versiones de planta ligadas a proyectos históricos «para no contaminar el aprendizaje»** | Exactamente el inv. 16 y lo que muestra el paso E |
| Conciliación en **ocho rubros con tolerancia ±5%** | P17, tal cual |
| Recalibración **basada en proyectos cerrados de la misma clase** | P17, tal cual |
| Confirmación de precios de MP **con alerta de vigencia** | P13, tal cual |
| Si el proyecto es **solo suministro**, el plan de montaje se desactiva | Es la opción de alcance que activa el wizard de montaje |

## Lo que corrige lo que teníamos

### Los pasos de la definición de planta no son los que mockeé

La reunión lista cinco pasos y **no son los cinco del prototipo**:

| # | En la reunión | En el prototipo hoy |
|---|---|---|
| A | Datos generales de identidad | ✅ Identidad |
| B | **Capacidad y flujo de valor (VSM/BCM)** | Capacidad y VSM |
| C | **Procesos y estaciones: productividad, utilización, velocidades** | ❌ *(mi paso C es «Calibración por clase»)* |
| D | Financieros: nómina, overhead, **HH directas vs. indirectas** | ✅ Financieros, sin la distinción directas/indirectas |
| E | Publicación **y comparación** de versiones | Publicación, sin comparación |

Dos cosas a resolver:

1. **«Procesos y estaciones» es un paso propio** en su lectura; yo lo fusioné dentro de capacidad. Y
   la **calibración por clase**, que en el prototipo es el paso C, en las notas aparece como
   *«integrada en la definición de planta»* sin ser uno de los cinco. **Puede que sean seis pasos.**
2 . Aparece **BCM** junto a VSM como término. No está en ningún documento que tengamos, y Daniel se
   comprometió a enviar el explicativo. Hasta entonces no sé qué modela.

### La ingesta soporta tres formatos, no dos

**PDF, DXF e imagen.** Nuestros documentos decían PDF/DXF. La imagen como entrada de primera clase es
nueva, y encaja con lo que ya se resolvió en #92 sobre escaneados: el pipeline lleva extracción de
primitivos, OCR de cotas y ángulos, visión por computador tradicional para la composición inicial, y un
**VLM experimental** para verificar las estructuras identificadas.

Nótese el orden: **el VLM verifica, no interpreta desde cero**. Eso es más robusto de lo que asumí y
refuerza que el humano cierre el lazo.

### El modelo canónico puede exportar a STP

Además de IFC. Una proyección de entregable más, y confirma que la arquitectura de proyecciones era el
encuadre correcto: se añade un formato sin tocar el modelo.

## Lo que es nuevo y hay que incorporar

| Hallazgo | Por qué importa |
|---|---|
| **El empalme de piezas afecta el desperdicio de forma significativa** | Si una pieza admite empalme o no mueve el desperdicio, y **el `Model` no lo registra hoy**. Es de los que cambian cifras |
| **Falta la parte de pintura** en las especificaciones del proyecto | El paso 3 de P3 no la tiene, y el recubrimiento es una desviación de concurso típica —en CNARCCS, C2 vs C3 movía −$235,061 |
| **Instrucciones particulares** en especificaciones del proyecto | Tampoco están |
| **Formato de la requisición a fabricación: pendiente** | P19 muestra la requisición pero su formato de salida no está definido |
| **Las versiones de planta son escenarios de capacidad instalada** | Máquinas dañadas, máquinas nuevas. Es un significado más rico que «una versión por cambio de parámetro» y conviene reflejarlo en el copy del paso E |
| **HH directas vs. indirectas** en financieros | Distinción que el paso D no hace |

## La frontera que hay que hablar

De las notas: *«Frontend ya en ThreeJS: renderización rápida, manejo de concurrencias, dockerizado.
Próximamente expuesto en web para pruebas por Daniel.»*

**dcode ya tiene un frontend con visor 3D funcionando.** El reparto dice que el visor y la plataforma
son de Kabunik (K2, K3), y que dcode es el agente y las tools.

No es un problema si es un banco de pruebas de su percepción — lo necesitan para validar lo que
extraen. Sí lo es si acaba siendo el visor del producto, porque entonces hay dos visores y el modelo
canónico tiene dos dueños. **Es exactamente el mismo aviso que levanté con la base de datos del modelo
(#20), y sigue sin resolverse.**

Y encaja con el próximo paso que quedó anotado: *«alinear arquitectura del modelo canónico con Juan
Pablo»*. Ese es el sitio para cerrarlo.

## Compromisos de la reunión

**De Daniel** — y por primera vez con lista concreta: layouts de plantas · explicativo de BCM,
análisis de layouts y **video de calibración por clase** · análisis del informe financiero para definir
el valor de la hora hombre · **planos del proyecto Space X de referencia**.

Ese último resuelve el bloqueo que veníamos arrastrando: **necesitábamos pares IFC + planos** y no los
teníamos.

**Nuestros:** revisar los mockups completos y **corregir nombres de campos** · exportar los perfiles de
Tekla a CSV para nutrir la biblioteca de perfilería · alinear la arquitectura del modelo canónico con
Juan Pablo.
