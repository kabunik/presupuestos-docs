# Reconciliación de las dos reuniones — equipo y dcode

> Notas de la reunión de equipo (con Daniel) y de la reunión posterior **solo con dcode**.
> **19-ago** · issue #119.
> Se leen juntas porque en un punto se contradicen, y la contradicción importa.

## Lo primero: la contradicción sobre planos escaneados

| Reunión de equipo | Reunión con dcode |
|---|---|
| «Tres formatos soportados en ingesta: **PDF, DXF e imagen**» | «**Descartar planos escaneados** en esta etapa, trabajar solo con vectorizados (exportados desde AutoCAD como PDF vectorial)» |
| Pipeline con extracción de primitivos, **OCR** para cotas y ángulos, visión por computador y **VLM** de verificación | «Simplifica el flujo: **no se necesita OCR ni redes neuronales** para esta fase» |

Y hay un tercer dato: el **12-ago** la dirección respondió que el escaneado *«está en el campo de dcode,
ya nos presentaron su estrategia y van por soportarlo en la v1 y van a medir el impacto de un VLM»*.

**Las tres afirmaciones no pueden ser verdad a la vez.** Mi lectura es que no es un cambio de opinión
sino un cambio de *alcance de fase*: dcode está acotando la **fase 1** a vectorizado para no pelear dos
problemas al mismo tiempo, y el escaneado sigue siendo objetivo de la v1. Pero eso hay que decirlo
explícitamente, porque:

- **#92 está escrito asumiendo escaneado en v1.** Si la fase 1 es solo vectorizado, el issue necesita
  fases, no una redacción nueva.
- **El prototipo muestra «imagen» como tercer formato de ingesta en P20.** Si en fase 1 no existe, la
  pantalla promete algo que no habrá.
- Es la diferencia entre necesitar OCR y CV desde el principio o no necesitarlos nunca en F0.

**Esto necesita una frase de la dirección**, no una interpretación mía. Es lo único de esta
reconciliación que no puedo cerrar solo.

## El modelo canónico: las dos notas dicen cosas distintas y ambas son ciertas

| Reunión de equipo | Reunión con dcode |
|---|---|
| «**Formato base del modelo: GLTF 2.0.** Almacena geometría 3D con propiedades» | «Modelo canónico **propio (no IFC, no OBJ)** como base de datos central. Formato personalizable a nivel de campos» |

Quien lea solo la primera nota concluirá que el modelo canónico *es* un `.glb`, y eso llevaría a un
diseño equivocado. La lectura que las reconcilia, y que es la que [CAPA_MODELO](../00-Fundamentos/CAPA_MODELO.md)
ya sostenía: **el modelo canónico es nuestro esquema, y glTF 2.0 es el formato de su carga
geométrica**. Las propiedades, las capas y las relaciones viven en el esquema; la geometría viaja en
glTF porque es lo que el visor consume sin conversión.

Dicho de otro modo: glTF no es la fuente de verdad, es **el formato del campo `geometría` de la fuente
de verdad**. Es una distinción que suena sutil y no lo es — decide si el BOM se deriva del `.glb` o el
`.glb` se deriva del modelo.

**Esa frase es la que hay que validar con Juan Pablo**, y es justamente el próximo paso que quedó
anotado a nuestro nombre.

### Y las cuatro capas confirman la cadena de dependencias

dcode listó las capas que el modelo debe soportar:

1. Geometría 3D · 2. Propiedades derivadas · 3. Programación y capacidad de planta · 4. Costos y partidas

**Ese orden no es una lista, es la cadena del motor.** Es B1 → B2 → B3 de [FLUJO_v1](../00-Fundamentos/FLUJO_v1.md),
derivada por ellos desde el modelo de datos mientras nosotros la derivábamos desde §4.3 de la
especificación. Dos caminos, la misma secuencia. Y su frase *«las modificaciones geométricas deben
propagarse en cadena a todos los outputs»* es nuestra invalidación en cascada con otras palabras.

## La frontera de trabajo, otra vez, y ahora con código

De las notas de dcode:

> «Front y back ya conectados con base de datos inicial, **basados en los mockups de Juan**.»

Y el flujo que ya implementaron: ingesta de datos generales, carga de IFC con extracción de unidades,
jerarquía espacial, longitudes, material, peso y relaciones; constraints de alcance; **capacidad de
planta conectada a 23 t/semana con alerta si se supera**.

Eso es un avance real y rápido, y conviene reconocerlo antes de señalar el problema: **implementaron
nuestras pantallas y nuestro estado**. La [Segmentación v2](Segmentacion_Tareas_Kabunik_dcode_v2.md)
asigna a Kabunik el frontend (K2), el backend con el estado canónico (K3) y el intake (K1); a dcode, el
agente, la percepción y las tools.

La semana pasada esto era un banco de pruebas de percepción —legítimo, lo necesitan— y así lo registré.
Ya no lo es: es front + back + base de datos ejecutando el flujo de intake. **Es la tercera vez que
levanto esta frontera** (#20 por la base de datos del modelo, la reconciliación del 12-ago por el visor
ThreeJS) y sigue sin resolverse.

No hace falta que la respuesta sea «paren». Puede ser que su v0.0 es el prototipo funcional y que la
plataforma se construye encima o al lado, con el contrato en medio. Lo que no puede pasar es que la
respuesta siga siendo implícita, porque **cada semana que pasa hay más código que reasignar**.

El acceso al repo y a la base de datos de la v0.0 está pedido y es la condición para opinar con datos.

## Lo que confirma lo que ya teníamos

| De las reuniones | Lo nuestro |
|---|---|
| Versiones de planta = escenarios de capacidad instalada, ligadas a proyectos históricos | Inv. 16 y el paso E de P11 |
| Calibración por clase integrada en la definición de planta | El paso C de P11 |
| Cinco pasos de la definición de planta | P11·A–E, con la discrepancia de #115 aún abierta |
| IFC sin símbolos de soldadura → pasar **IFC + PDF** como contexto al agente | La resolución de #89, y ahora hay un par real para probarlo |
| Loop de confirmación humana antes de generar el 3D | P20 |
| Si es solo suministro, el plan de montaje se desactiva | La bifurcación de alcance de P3 paso 3, ya mockeada |
| Requisición: formato pendiente | Sigue pendiente, #119 no lo resuelve |
| Empalmes afectan el desperdicio | #113 |
| Instrucciones particulares y pintura en especificaciones | #114 — y la pintura resultó ser un motor, ver [MOTOR_PINTURA](../00-Fundamentos/MOTOR_PINTURA.md) |
| Conciliación en 8 rubros con ±5% y recalibración por clase cerrada | P17 |

## Decisiones nuevas que hay que registrar

### Infraestructura: Hetzner en lugar de AWS

Por costo y simplicidad: ~18 USD/mes por 8 núcleos contra un equivalente mucho más caro en AWS. El
equipo de dcode ya opera ahí —mencionan embeddings que pasaron de 10 s a 0.5 s— y la latencia desde
Alemania les resulta aceptable. Recomiendan instancia **AMD, tier «cost optimized»**, para facilitar el
salto a arquitecturas nuevas.

Es una decisión de K8 que no estaba tomada. Con una consecuencia práctica: si la verificación de cuenta
tarda dos o tres días, montan provisionalmente en un servidor local — o sea que **el arranque no queda
bloqueado por esto**, pero la instancia definitiva sí es tarea nuestra.

### Catálogo de perfiles: hay más alcance del que teníamos anotado

Lo que estaba anotado era «exportar los perfiles de Tekla a CSV». De la reunión con dcode salen dos
cosas más:

1. **Tekla tiene perfilería por entorno** —LATAM, España/Europa, americano en pulgadas, internacional
   métrico—. No es un catálogo, son cuatro o más, y el proyecto tiene que saber en cuál está.
2. **Hace falta un módulo para que los talleres suban sus propias librerías de perfiles.** Eso es
   superficie de producto y multitenancy, no un CSV: catálogo por tenant, versionado, y la pregunta de
   qué pasa cuando un perfil propio no tiene equivalencia en la norma.

Hoy dcode trabaja con un CSV de AISC provisional. El catálogo es, además, **precondición del modo C**
del sistema de presupuestos v1.3, que homologa perfil por perfil contra lo que hay en inventario.

### Y una que conviene no dejar pasar

De la reunión de equipo: **Daniel enviará el análisis del informe financiero para definir el valor de la
hora hombre**. Ese entregable ya llegó, dentro del paquete v1.3: `costos_reales_zapotlan_2026_04.json`
reconcilia el costo de la hora contra la contabilidad de abril y da **$265/HH**. Conviene confirmarlo con
él antes de darlo por cerrado, pero el dato está.

## Compromisos abiertos

**De Daniel:** layouts de planta (**llegó** el layout de Zapotlán) · explicativo de **BCM**, análisis de
layouts y video de calibración por clase · análisis del informe financiero (**llegó** dentro de v1.3) ·
planos del proyecto Space X *(llegó otro par: Ampliación Bodega 50, IFC + planos + resumen de HH/ton)*.

**Nuestros:** revisar los mockups completos y corregir nombres de campos · exportar los catálogos de
Tekla · contratar la instancia Hetzner AMD y dar acceso · alinear la arquitectura del modelo canónico con
Juan Pablo · pasar los planos de Michelena al equipo *(ya se pueden pasar)* · pedir acceso al repo y la
base de datos de la v0.0.
