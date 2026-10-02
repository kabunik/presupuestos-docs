# Reconciliación de la reunión del 19-ago

> El hito se reordenó, apareció la v0.0 funcionando, y el sistema pasó a ser una familia de motores.
> **20-ago** · issue #127.

## El hito 1 es el modelo geométrico, y eso ordena todo lo demás

> «Objetivo inmediato: construir el modelo canónico geométrico correcto como base de todo lo demás. Una
> vez logrado, **la capa determinística de Daniel se encadena directamente**.»

Es la mejor noticia de la reunión, y no por lo que añade sino por lo que **descarta**: durante semanas el
riesgo era que se intentara avanzar en paralelo en percepción, motor y visor, con el modelo canónico como
un pendiente que nadie bloqueaba. Ahora es el hito, y el resto se declara explícitamente aguas abajo.

Coincide con la cadena que [FLUJO_v1](../00-Fundamentos/FLUJO_v1.md) derivó del motor: **B1 primero, y
sin B1 no hay nada**. Lo que la reunión añade es el corte concreto: **geometría hasta el BOM**. No
propiedades, no costos, no programa. Solo la capa 1 y la 2 de las cuatro que dcode listó.

Y tiene una consecuencia práctica para nosotros: **el hito 1 es exactamente donde vive la pregunta que
llevamos abierta**. Si el modelo canónico se define bien en su parte geométrica, la arquitectura de
proyecciones queda probada; si se define mal, lo descubrimos ahora y no en F2.

## El sistema dejó de ser un motor y es una familia de motores

En tres semanas pasamos de uno a **seis**. Conviene tener el registro en un solo sitio, porque el
alcance se vuelve indefendible si cada entrega añade un motor sin decir en qué etapa entra.

| Motor | Estado | Etapa | Dónde está especificado |
|---|---|---|---|
| **Presupuesto** (M1–M6) | Especificado, calibrado con planta real | v1 | [SISTEMA_PRESUPUESTOS_v1.3](../00-Fundamentos/SISTEMA_PRESUPUESTOS_v1.3.md) + el kit |
| **Planta** (capacidad, VSM, S11/S12) | Especificado | v1 | El kit + P11 |
| **Montaje** | Código funcional + spec vinculante | v1 | [MOTOR_MONTAJE](../00-Fundamentos/MOTOR_MONTAJE.md) |
| **Pintura** | Excel completo con contrato de datos | **por decidir** (#122) | [MOTOR_PINTURA](../00-Fundamentos/MOTOR_PINTURA.md) |
| **Piezas complejas** (minería) | Dos tipologías analizadas; entrega el viernes | **etapa 2** | `Elementos estructurales/` |
| **Puentes** | Desarrollado, pendiente de compartir | **etapa 2** *(por confirmar)* | — |
| **M7 ingeniería de valor** | Especificado, sin superficie | por decidir (#125) | v1.3 §9.4 |

**Daniel aclaró que buena parte del material enviado es complementario y explicativo del sistema
completo**, no alcance nuevo. Eso baja la presión: no son seis frentes de construcción, son seis
dominios de los cuales tres son v1.

### El motor de piezas complejas es más interesante de lo que parecía

Lo habíamos catalogado como «dos oráculos de HH». Es un motor propio, de etapa 2, y lo que analiza va
más allá del costo: **armado, soldadura y accesibilidad de antorcha y de llaves**. Cubre los casos en que
la geometría obliga a **cambiar de proceso** —electrodo doblado en lugar de Fluxor porque la antorcha no
entra—.

Eso no es un factor de corrección: es una decisión de método que cambia el rendimiento. Y su unidad de
trabajo es el **plano de taller**, no el modelo — por eso Daniel tiene en curso un convertidor de PDF a
planos de taller.

## La v0.0 contra nuestro flujo

`presupuestado.app` está en un servidor de red, con seis cuentas, y el flujo implementado se lee casi
como nuestro inventario de pantallas. Lo que sigue es la comparación, y lo último es lo único que falta.

| Lo que implementaron | Nuestra pantalla | Estado |
|---|---|---|
| Dashboard de proyectos por fábrica | **P2** | Coincide. Hardcodeado a Metalitec: la multitenancy sigue pendiente |
| Nombre, cliente, destino **con integración a Maps para los km** | **P3 paso 1** | Coincide, y Maps es una idea buena que no teníamos: el destino gobierna el transporte |
| Complejidad de HH, tipo de conexión, acabado, consideraciones libres, desviaciones bajo norma | **P3 paso 3** | Coincide pieza por pieza |
| Plan de montaje: toneladas por semana con avisos de capacidad | **P3 paso 4** | Coincide, y el aviso es el inv. 11 funcionando |
| Procesamiento del BOM en 3–4 min, con catálogo genérico | **P4 → P6** | Coincide. El catálogo por región es #123 |
| Identifica elementos con **geometría o material faltante** y permite corrección manual | **P20** capa de no resueltos | Coincide con `VAL-08` y el filtro de #91 |
| **Estado de ingeniería: completo, en curso, preliminar o personalizado** | *no existe en nuestro mock* | **Hueco nuestro** |

### El estado de ingeniería es un hueco de nuestro lado

Ellos lo tienen y nosotros no, y no es un campo cosmético: en `Informe_Factores...xlsx` hoja 10 la
**contingencia depende de la fase de ingeniería** —anteproyecto +15%, básica +8–10%, detalle 30% +5–7%,
detalle 70% +3–5%, detalle 100% 0–2%—. Si el proyecto declara su fase, la contingencia sale de una tabla
en lugar de un criterio.

Es el mismo patrón que celebramos en el resumen de Ampliación Bodega 50 —«Clase 4/5, banda ±15–20%»— y
que dijimos que debería viajar en el producto. Aquí está el campo que lo hace posible, y lo puso dcode.

**Hay que añadirlo al paso 1 o 3 del intake**, y con él la contingencia derivada.

### Y una advertencia sobre la multitenancy

«Usuarios aislados entre sí; compartir proyectos entre cuentas queda para etapa futura.» Aislar usuarios
no es aislar tenants. Un usuario aislado no puede ver el proyecto de su propio compañero de empresa, que
es justo lo que un fabricante necesita; y en cambio no dice nada sobre si el tenant A puede alcanzar
datos del tenant B. **Son dos ejes distintos** y el que nos importa por contrato es el segundo.

## Los formatos de ingesta: la contradicción se resuelve, y no era de alcance

Veníamos con tres afirmaciones incompatibles sobre planos escaneados (#92). La reunión da el encuadre que
las reconcilia, y es de **prioridad**, no de alcance:

| Vía | Estado |
|---|---|
| **IFC** | Vía **principal**, ya operativa en la v0.0 |
| **PDF** | En desarrollo. Requiere **intervención humana** para validar capas: columnas, vigas, elevaciones |
| **IFC + PDF juntos** | **Lo ideal**: el PDF aporta el detalle de soldadura y tornillería que el IFC no refleja |

Y la frase de Héctor que lo cierra: *«el IFC geométricamente suele estar bien; el PDF añade el detalle de
fabricación»*.

Así que el desacuerdo nunca fue sobre si el escaneado entra en la v1: era sobre **qué se construye
primero**. Se construye la vía IFC, luego la vía PDF vectorial con validación humana, y la imagen o el
escaneado no están en el camino cercano. Eso es compatible con las tres notas anteriores y **desbloquea
#92** — con una consecuencia de diseño: el chip de «imagen» en P20 no debe prometer lo que no habrá.

Un dato de dominio que conviene no perder: **para minería el detalle previo al presupuesto es
obligatorio**, por la precisión que exige la interfaz entre estructura y máquina. Encaja con que el motor
de piezas complejas sea justamente el de minería.

## El feedback que nos toca a nosotros

> «Dashboards parecen generados por Claude: recomendación de diferenciar visualmente la UI.»

Hay que recibirlo sin defenderse, porque es feedback sobre **nuestro** diseño: la v0.0 se construyó desde
nuestros mockups. Dos lecturas posibles y conviene distinguirlas antes de tocar nada:

1. **La implementación no está usando el sistema de diseño.** `tokens.css` existe precisamente para que
   la UI no parezca genérica. Si la v0.0 partió de las capturas y no de los tokens, el parecido genérico
   es de la implementación, no del diseño.
2. **El diseño sí es genérico.** Posible: el prototipo prioriza densidad de información y claridad
   funcional, y su personalidad visual es deliberadamente sobria.

No se puede saber cuál es sin ver la aplicación, y el acceso está pedido. Lo que sí acordó la reunión es
el orden: **primero la estructura de datos correcta en el backend, la UI se retoca después.** Correcto,
y encaja con que el hito 1 sea el modelo geométrico.

Dos ítems concretos y pequeños:

- **Contador de tiempo estimado** durante el procesamiento del IFC. Con 3–4 minutos de proceso, P4
  necesita ETA además de etapas
- El feedback llega por WhatsApp con foto y el equipo lo sube a GitHub como ticket. Funciona, pero
  conviene que los tickets de diseño lleven referencia a la pantalla (`P3 paso 3`) para poder cruzarlos
  con el handoff

## La tesis comercial de Héctor, que es una pregunta de alcance disfrazada

> «Producto muy completo pero percibido como muy **premium**. Riesgo: requiere consultoría previa y
> conocimiento profundo del cliente para implementar. Propuesta: **módulo básico vendible a todos +
> módulo premium** para empresas grandes (>1.000 t/mes).»

Esto no es una observación de mercado: **es una tensión con la premisa del producto.** Todo lo que
construimos se apoya en que el presupuesto está *calibrado contra tu planta real* — y la calibración es
exactamente lo que exige consultoría, datos históricos y una definición de planta versionada. Un módulo
básico que se venda sin eso tendría que renunciar a la calibración, y sin calibración el diferenciador
desaparece.

Hay salidas y merecen análisis, no una respuesta rápida: arrancar con parámetros por defecto de la tabla
de referencia y mejorar con los cierres; vender el take-off y el BOM sin la capa de planta; o tratar la
consultoría como onboarding pagado en lugar de barrera. **Cuál de las tres es una decisión de dirección**
y cambia el inventario de pantallas de la v1.

Dato de referencia de la propia reunión: SiteGenuine/Allplan ~**$9,000/año** de suscripción; Allplan
Professional ~$300. Y un competidor nuevo que hay que mirar: **Bitforge**, proyecto reciente con IA y
flujo similar de subida de planos y estimación.

## Y una funcionalidad que ya apareció dos veces

**RFIs automáticas al cliente cuando el modelo no puede resolver dudas del plano** — valorada en la
reunión. Es el mismo mecanismo que el análisis de AG22 usa para levantar incumplimientos verificables
contra AISC y AWS, y el mismo patrón de «el agente declara su incertidumbre» que adoptamos para la
simbología de soldadura (#89).

Tres apariciones independientes es señal de que es una pieza del producto y no una idea suelta. Todavía
no tiene issue.

## Infraestructura: cambia lo decidido hace un día

**Instancia AWS temporal** para desbloquear el entorno de producción mientras se resuelve la
infraestructura con Héctor. No invalida la evaluación de Hetzner —sigue siendo más barata— pero sí el
carácter de decisión cerrada que le dimos en #124: hoy es **AWS temporal + decisión pendiente**.

## Compromisos

**De dcode:** compartir las credenciales de `presupuestado.app` (6 cuentas) y el acceso al repositorio y
la base de datos de la v0.0.

**De Daniel:** motor de piezas complejas y convertidor PDF → planos de taller *(estimado viernes)* ·
módulo de puentes · explicativo de **BCM**, análisis de layouts y video de calibración por clase.

**Nuestros:** levantar la instancia AWS temporal · organizar la documentación en el repositorio ·
convocar la reunión interna de modelo canónico y user flow · revisar **Bitforge** y la comparativa de
precios · pasar los planos de Michelena al equipo.
