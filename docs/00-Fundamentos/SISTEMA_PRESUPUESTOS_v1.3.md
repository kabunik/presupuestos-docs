# Sistema de Presupuesto Automático v1.3 — reconciliación

> Entrega del consultor del **14-ago**, recibida el 19-ago dentro de la reorganización de
> `docs/doc inicial/Contexto completo Michelena/Modulo presupuestos/`. Issue #119.
> Es la primera vez que tenemos la aritmética **calibrada contra la planta real**, no contra un caso
> didáctico.

## Qué es, y qué relación tiene con el kit

Dos numeraciones distintas que conviene no confundir: el **kit** es *Sistema DM v1.7*; esto es
*Sistema de Presupuesto Automático v1.3*. No es una versión anterior del mismo documento — **son dos
capas distintas del mismo sistema**.

| | El kit (v1.7) | Esta entrega (v1.3) |
|---|---|---|
| Qué especifica | El **presupuesto de un proyecto**: E1–E7 → 18 salidas *required* | La **economía de la planta**: capacidad, costo de la hora, rentabilidad, carga, P&L |
| Su unidad | La oferta | El mes y la línea de producción |
| Su oráculo | `golden_test.json`, caso Nave 100 t | `costos_reales_zapotlan_2026_04.json`, abril de 2026 real |
| Autoridad | Normativa — máxima | Normativa sobre lo suyo |

Se solapan en un punto y hay que decirlo: **el costo de la hora hombre**. El kit lo trae del golden
test; esta entrega lo reconcilia contra la contabilidad de abril. Ver el conflicto I más abajo.

`cascada_nave_100t` en las semillas reproduce el golden test cifra por cifra —100 → 113.7 → 121.25 t,
factor 1.213, precio $5,977,353—. **El golden test sigue en pie**, que era la duda razonable al abrir
un paquete nuevo.

## Los siete motores

| # | Motor | Entrada | Salida |
|---|---|---|---|
| **M1** | Take-off IFC | Modelo / planos | Piezas, pesos, familias, cantidades de trabajo |
| **M2** | HH por pieza | M1 + rendimientos | HH por proceso, HH/ton, **banda de incertidumbre** |
| **M3** | Costeo / APU | M2 + precios + cascada de pesos | Costo, precio, 3 opciones de oferta |
| **M4** | Carga de planta | M2 + programa + capacidad | Utilización semanal, ventanas, sobrecargas |
| **M5** | Rentabilidad | HH ganadas + gastos + tarifa | Curva, equilibrio, proyección de cierre, P&L |
| **M6** | Flujo de caja | M3 + condiciones comerciales + avance | Caja mensual y acumulada, validación ≥ 0 |
| **M7** | Ingeniería de valor | IFC + inventario + carga | Propuestas `{alternativa, ahorro, impacto_plazo, funcion_preservada, aprobador}` |

M1–M6 mapean casi uno a uno sobre nuestros bloques B1–B3 del [FLUJO_v1](FLUJO_v1.md). **M7 no tiene
superficie en la plataforma**, y es el único motor cuya salida es una *propuesta a una persona* en
lugar de una cifra: homologación laminado↔armado, contraflecha solo donde el servicio la exige,
sustitución de perfil por disponibilidad. Es material de pantalla propia.

## Tres cosas que esta entrega cierra

### 1 · El mapeo de tipologías, que llevaba semanas abierto

Estaba como pendiente 2 de [MOTOR_MONTAJE](MOTOR_MONTAJE.md) y como ámbar «sin mapeo» en el paso E del
wizard. `parametros_modelo_y_verificador.json` lo da, y la **regla R5 lo hace normativo** con acción
definida —alerta + revisión humana si un HH/ton se sale de su banda:

| Tipología del consultor | Banda HH/ton | Nuestro eje |
|---|---|---|
| **pesada** | 14 – 26 | **24** |
| **mediana** | 40 – 60 | **40 y 60** |
| **liviana** | 80 – 999 | **90** |

**La nomenclatura es la contraria a la intuitiva y por eso no había que adivinarla:** «pesada» es
sección pesada, que rinde **menos** HH por tonelada; «liviana» es perfil ligero y muy troceado, que
rinde muchas más. Un mapeo ingenuo habría puesto pesada en 90 y habría invertido todo el costeo.

Confirmación independiente: las tarifas del motor de montaje traen pesada 20, mediana 50, liviana 95
—las tres dentro de su banda—. Dos archivos distintos, entregados en días distintos, concuerdan.

### 2 · La tolerancia del invariante 4 tenía un número y no lo teníamos

Nuestros documentos decían «IFC vs. reconstruido **en tolerancia**», sin cifra. La regla **R7 dice
Δ < 1%**, con acción «alerta alta». Y R8 fija las bandas del semáforo RSS: **verde < 6%, ámbar 6–10%,
rojo > 10%**, con «rojo: no cotizar sin decisión humana».

Con esto quedan separadas las **tres** tolerancias que se venían confundiendo:

| Chequeo | Compara | Tolerancia |
|---|---|---|
| **Inv. 4 / R7** · doble chequeo de peso | Peso del modelo vs. peso reconstruido | **Δ < 1%** |
| **G1** · motor de montaje | Modelo vs. tonelaje contractual **firmado** | ±3% |
| **Cierre** · conciliación de 8 rubros | Presupuestado vs. ejecutado | ±5% |

### 3 · El pipeline de verificación V1–V7

Siete chequeos que **todo número de HH atraviesa antes de publicarse**: rango histórico, coherencia de
soldadura (kg de aporte contra longitud × sección × densidad y tasa de depósito FCAW), balance de
procesos, y cuatro más. Con dos reglas de parada que son de la misma familia que nuestros
guardarraíles:

> Máximo 3 vueltas. Si un chequeo falla 2 veces por la misma causa → **bandera roja + dato faltante,
> nunca ajustar sin causa física.**

«Nunca ajustar sin causa física» es el inv. 16 —prohibición de recalibración automática— aplicado al
nivel de la pieza. El sistema no se acomoda para pasar su propio chequeo.

## Los modos A, B y C — y por qué importan

`M3` se desdobla en tres modos de presupuestar, y **cada presupuesto declara el suyo** (regla R2b):

| Modo | Cómo | Desperdicio |
|---|---|---|
| **A** | Cascada de pesos AISC, la del golden test | % **sobre peso** |
| **B** | Rendimientos por clase del Budget Engine v4.1 | por clase |
| **C** | «Teleférico»: perfil por perfil contra inventario, homologando lo que no hay | % **sobre costo** |

El modo C tiene un caso real completo (`251601_Teleferico_Hacienda`, 15 tipos de perfil, USD, wt_drawing
88.3 t → wt_prop 119.26 t). **Esto es nuevo para nosotros**: nuestro flujo asume implícitamente el modo A.
Un proyecto en modo C aplica reglas distintas —el desperdicio va sobre costo, no sobre peso— y eso
cambia la aritmética, no solo la presentación.

## Lo que hay que abrir: cuatro conflictos de cifras

Los cuatro son entre documentos **del propio consultor**, no entre él y nosotros. Ninguno se resuelve
adivinando.

### F · El factor de obra: ×1.4 o ×1.667

El resumen de Ampliación Bodega 50 aplica **×1.4** al filete de obra frente al de taller. La
especificación v1.3 §11.5 dice `factor_obra = 0.60`, o sea `hh_obra = hh_taller / 0.60` = **×1.667**.
Dos documentos de la misma semana. La diferencia mueve las HH de campo un **19%**.

### G · Dos escalas de HH/ton con las mismas palabras

`rendimientos_budget_engine_v41.json` define las clases por **kg/m** con su HH/ton base: liviana 34,
mediana 25, pesada 19, extra_pesada 17. Las bandas de R5 dicen liviana **80–999**, mediana 40–60,
pesada 14–26. Mismos nombres, escalas distintas: la liviana del Budget Engine es 34, la de R5 arranca
en 80.

La explicación probable es de alcance —el Budget Engine declara «corte + armado + soldadura, sin
montaje ni pintura», y las bandas parecen ser del total— pero **2.4× de diferencia no se asume**.
Y hay un antecedente que obliga a cuidarlo: la regla R4 exige que *todo HH/ton publique su alcance*.
Aquí hay dos cifras publicadas sin que se distinga el alcance de cada una.

### H · Lotes de 50 t contra camiones de 30 t

El motor de montaje arma lotes de `LOTE_TON = 50`. La v1.3 §11.5 dice `transporte: camion estandar
max 30 t/viaje`. No es necesariamente contradictorio —un lote de montaje no es un camión— pero si el
costo de transporte se deriva del loteo, **un lote de 50 t son dos viajes**, y eso no está dicho en
ninguna parte.

### I · Tres costos de la hora hombre

| Cifra | De dónde | Qué es |
|---|---|---|
| **$122.79/HH** | `golden_test.json` | Caso didáctico Nave 100 t. **Sigue siendo ley** para ese caso |
| **$300/HH** | `planta_didactica` de v1.3 | Otro caso didáctico, otra planta ficticia |
| **$265/HH** | Reconciliado con abril 2026 | **Metalitec Zapotlán real**, sin depreciación. Con depreciación, ~$311 |

No se contradicen: son tres plantas distintas. El problema es de rotulado, y nos toca a nosotros: el
prototipo muestra $122.79 bajo el nombre **«METALITEC · Pachuca»**, y la planta real del consultor es
**Zapotlán** con $265. Un número didáctico con el nombre de una planta real es la clase de cosa que
después alguien cita en una reunión.

## Lo que queda abierto y de quién depende

| # | Abierto | Quién |
|---|---|---|
| 1 | Factor de obra ×1.4 o ×1.667 | Daniel |
| 2 | Alcance de cada escala de HH/ton (Budget Engine vs. bandas R5) | Daniel |
| 3 | Relación entre lote de montaje y viaje de camión | Daniel |
| 4 | ¿El modo C entra en la v1? Cambia la aritmética del desperdicio | Dirección + C1 |
| 5 | M7 ingeniería de valor no tiene superficie en la plataforma | Kabunik |
| 6 | El rector del consultor va en **Rev. 3.8**; nuestros documentos citan v1.7 | Juan → Daniel |

Las seis **decisiones pendientes de Daniel** que el propio LEEME declara —factor AWS D1.8, composición
del costo fijo comercial, composición de consumibles, dos valores ESTIMADO del Budget Engine, mapa de
puestos MOD/MOI, reporte de HH de contratistas— se suman a esta lista sin que nosotros las toquemos.
