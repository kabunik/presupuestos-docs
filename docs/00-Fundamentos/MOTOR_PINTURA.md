# El motor de pintura

> `Motor_Pintura_Desperdicios (1).xlsx`, 15 hojas, recibido el 19-ago. Issue #119.
> Cierra la parte de pintura de #114 — pero no como la habíamos planteado.

## Lo que cambia respecto a #114

Abrimos #114 diciendo que a las especificaciones del proyecto **le faltaba un bloque de pintura**:
sistema, capas, espesor, ambiente. Eso era quedarse corto por un orden de magnitud. Lo que hay es un
**motor entero**, con su tabla de factores, su cascada de cálculo, su APU por familia y **su propio
contrato de datos** (hoja `14_INTERFAZ_MOTOR`, con rangos de entrada y salida declarados).

Es el **cuarto motor** del sistema, junto al de presupuesto, el de planta y el de montaje.

## La frontera, que viene dicha y es tajante

Del análisis de la marca AG22, palabra por palabra:

> «La pintura **no entra en el total de fabricación bajo ninguna circunstancia**.»

Así que la pintura es **partida propia**, nunca un componente del costo de fabricar. Coincide con el
Budget Engine, que declara su alcance como «corte + armado + soldadura, **sin montaje, sin pintura**».
Dos fuentes independientes trazan la misma línea, y eso ordena el modelo de costos: fabricación,
pintura y montaje son tres partidas paralelas, no una con dos añadidos.

## Cómo calcula

La base es la fórmula SSPC de rendimiento teórico, y sobre ella se apilan tres correcciones
independientes. Lo interesante es que **ninguna es un porcentaje global**: cada una tiene su tabla con
criterio explícito.

```
rendimiento teórico (m²/L) = 10 × %sólidos / EPS(µm)
       │
       ├─ TABLA A · factor de forma F1–F7      placa 5% · perfil pesado 10% · mediano 15%
       │                                        ligero 25% · celosía 40% · HSS 12% · miscelánea 30%
       ├─ TABLA · ambiente y método A1–A6      pérdida por sobreaspersión y viento
       └─ TABLA · perfil de anclaje R0–R4      la rugosidad del granallado se come pintura
       │
       ▼
litros prácticos → cuñetes a comprar → colchón de redondeo → costo
```

El factor de forma es el que más manda: una **celosía pierde 40%** contra el 5% de una placa, porque el
abanico de pintura atraviesa la estructura abierta. Es el mismo tipo de razonamiento físico que sostiene
el eje de HH/ton, y por eso encaja con nuestro modelo en lugar de pelearse con él.

## Tres piezas que valen para el producto más allá de la pintura

**1 · Estimar m² desde el peso, y auditarse.** La hoja `12_ESTIMACION` deriva el área pintable con
`m²/ton ≈ 255 / espesor_promedio_mm`, le descuenta el % no pintable por familia, y **compara contra los
m² reales del modelo con tolerancia del 15%**. Si se sale, no devuelve el número: devuelve
**⚠ REEVALUAR** con una guía de revisión por familia —«caras embebidas en dados», «patín superior con
contralosa», «HSS cerrados: el interior no se pinta»—.

Es exactamente la disciplina que queremos en todas partes: el sistema estima, se compara contra el
modelo, y **cuando no cuadra lo dice y explica dónde mirar**.

**2 · Un panel de control que declara qué falta.** La hoja `13_PANEL` es una checklist con estados
`OK / FALTA / PENDIENTE / 5 FAMILIAS EN ALERTA`. No es un formulario que se pueda enviar a medias sin
que se note: la incompletitud es visible antes de calcular. Es el mismo patrón del semáforo de la
soldadura de campo en P22·C.

**3 · La zona copiloto.** La hoja de instrucciones la nombra así: *«la IA propone los factores»*. El
consultor ya está diseñando para que el agente sugiera y la persona confirme, en su propio Excel. La
división de trabajo que definimos en el producto es la que él ya usa.

## Lo que el motor necesita como entrada

De la hoja `14_INTERFAZ_MOTOR`, los seis rangos de entrada. Tres los deriva la plataforma y tres los
introduce una persona — la misma partición que ordenó el wizard de montaje:

| Entrada | De dónde |
|---|---|
| Catálogo de productos con su TDS: % sólidos, EPS mín/recom/máx, cuñete, intumescente | **Del usuario** — es dato del proveedor |
| Precios de pintura y de abrasivo | **Del usuario** — y son precios, así que caen bajo la compuerta del inv. 8 |
| Partidas de pintura por capa: área, forma, ambiente, anclaje, producto, espesor, franjeo | **Mixto** — el área se deriva del modelo, el sistema de pintura lo decide el proyecto |
| Listado de piezas con área y peso unitarios | **Derivado** del modelo canónico |
| Espesor promedio y % no pintable por familia | **Del usuario**, con la auditoría del 15% encima |
| Partidas de abrasivo y su sistema | **Del usuario** |

Ojo con la tabla de abrasivo, que tiene una trampa de las que cuestan dinero: **la arena es de un solo
uso y consume 30–55 kg/m²; la granalla de acero recircula 150–250 ciclos y consume 0.35–0.75 kg/m²**.
Dos órdenes de magnitud de diferencia según el equipo instalado en la planta — o sea que **es un dato de
`PlantConfig`**, no del proyecto.

## Lo que propongo, y lo que hay que decidir

La pintura necesita **su propio flujo paralelo**, igual que planta y montaje, activado por el acabado
del proyecto. Repetir el patrón tres veces ya no es coincidencia: es la forma que tiene este producto.

Lo que **no** propongo es meterla en el paso 3 del intake como cuatro campos. Sería mentir sobre su
tamaño y garantizaría rehacerlo.

| # | Abierto | Quién |
|---|---|---|
| 1 | ¿La pintura entra en el alcance de la v1, con flujo propio? | Dirección |
| 2 | El consumo de abrasivo es dato de planta: ¿nuevo campo de `PlantConfig` o paso propio? | Kabunik |
| 3 | Ignífugo por Hp/A: exige la relación superficie/masa **por pieza**, que hoy el `Model` no calcula | C1 |
| 4 | ¿Los precios de pintura caen bajo la compuerta de confirmación de precios (inv. 8)? | Conjunto |

El punto 3 es el que tiene consecuencia técnica: el espesor de intumescente se lee de una tabla por
`Hp/A` —perímetro expuesto sobre área de sección—, y ese es un derivado geométrico por pieza que hay que
añadir a la capa de propiedades del modelo canónico.
