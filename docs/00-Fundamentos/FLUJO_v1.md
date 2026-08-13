# El flujo de la v1, derivado de la cadena de dependencias del motor

> Cierra #85. **El orden del flujo no es una decisión de diseño: lo impone `motor_calculo_spec.md`.**
> **2026-08-11** · corrige el ciclo de vida de `CONTRATO_INTEGRACION_v0.md` (afecta C1.3 y C1.4).

## La dependencia que nadie había trazado

El flujo diseñado ponía las **opciones de oferta** justo después del chat de edición. Eso es
imposible bajo el motor, y la razón está en una sola cláusula de §4.3:

> §4.3 APU AISC (material E3×peso; MO propio; **MO subcontrato del pico S11**; indirectos; margen).

El APU necesita el pico de S11. Y S11 no es un dato de entrada:

> §4.9 S11 doble velocidad (`alerta_subcontrato_hh = max(0, pico − v_ot) × HH/ton`)
> §4.7 Lotes y secuencia por plan de montaje **contra planta CARGADA**; déficit explícito

Encadenando con §4.13 —*«precio de licitación = APU + financiero»*— y con §4.14 —*el RSS gobierna
la contingencia y el % por defecto del Más X%*— la cadena queda cerrada:

```
plan de montaje (E6) ──┐
carga de planta (E7) ──┴─→ §4.7 lotes ─→ §4.9 S11 ──┐
                                                     │
peso §4.1 ─→ boom §4.2 ──────────────────────────────┴─→ §4.3 APU ─┐
                                                                    ├─→ §4.13 precio de licitación
precios E3 ─→ costo empresa §4.5 ──────────────────────────────────┤
RSS §4.14 ─→ §4.4 Más X% ──────────────────────────────────────────┘
```

**Conclusión operativa: no existe un precio antes de haber resuelto la planta.** La carga de planta
no es una pantalla lateral de consulta — es una **precondición del costeo**.

## Los cinco bloques del motor

Las 16 secciones se agrupan en cinco bloques con dependencia estricta entre ellos. Cada bloque
termina donde hay una compuerta humana o un artefacto entregable:

| Bloque | Secciones | Necesita | Produce | Termina en |
|---|---|---|---|---|
| **B1 · Modelo y peso** | §4.1, §4.2 | E2 take-off (percepción) | Peso planos/facturable/comprar, factor, boom por familias, exclusiones, doble chequeo | **Gate: aprobar BOM** |
| **B2 · Planta y programa** | §4.7–§4.12 | B1 + plan de montaje (E6) + carga e inventario (E7) + VSM (E4) | Lotes y déficit, grúa, S11, requisición neta, S12 con escenarios A/B/B\*/C, interferencia de familias | **Gate: decisión de programa** · **Gate: decisión de familias** |
| **B3 · Costo y precio** | §4.3–§4.6, §4.13, §4.14 | B1 + **el pico de S11 de B2** + precios (E3) + financieros (E5) | APU AISC, Más X%, reconciliación, costo empresa, desviaciones costeadas, flujo de caja, capítulo financiero, RSS, **precio de licitación** | Arranca en **Gate: confirmar precios de MP**; termina en las opciones de oferta |
| **B4 · Emisión** | §4.15 | B3 | Leyenda de alcance, oferta, IFC, memoria descriptiva | **Revalidación de vigencia** de los precios confirmados |
| **B5 · Cierre** | §4.16 | Proyecto ejecutado | Cierre en 8 rubros, propuesta de recalibración | **Gate: aprobar recalibración** |

## El flujo corregido

```
P3  Intake                    datos + clase HH/ton · archivos · consideraciones + desviaciones
    (4 pasos)                 · plan de montaje + fase RSS
     ↓
P4  Generación                streaming de B1
P20 Ingesta asistida          el humano confirma sobre el plano: clasificación de
     ↓                        páginas, grilla de ejes, niveles, marcas, cotas y
                              SIMBOLOGÍA DE SOLDADURA — compuerta de la fusión (#89)
──── B1 · MODELO Y PESO ──────────────────────────────────────────────────
P5·A Workspace · revisión     visor + BOM + alertas de percepción + exclusiones
P6   Grilla BOM               takeoff a fondo, selección bidireccional
P5·B Workspace · edición      chat, deltas, guardarraíles   ⟲ reabre el gate
     ↓                        ▸ GATE 1 · Aprobar BOM
──── B2 · PLANTA Y PROGRAMA ──────────────────────────────────────────────
P14  Carga de planta          carga combinada, ocupación ≥100%, S11, requisición neta
P16  Interferencia            solo si aplica  ▸ GATE 4 · Decisión de familias
P15  Decisión de programa     escenarios A/B/B*/C  ▸ GATE 3 · Acuerdo de retraso
P19  Requisición y lotes      déficit, grúa
     ↓
──── B3 · COSTO Y PRECIO ─────────────────────────────────────────────────
P13  Gate de precios de MP    ▸ GATE 2 · Confirmar precios · los precios son
                              ENTRADA de §4.3: se confirman antes de costear
P9   Opciones de oferta       3 opciones, flujo de caja, ingeniería de valor,
                              reconciliación APU vs. Más X%, v1 vs v2
     ↓
──── B4 · EMISIÓN ────────────────────────────────────────────────────────
P10  Emisión                  revalidación de vigencia · leyenda de alcance,
                              exclusiones, sello de versión
     ↓
──── B5 · CIERRE ─────────────────────────────────────────────────────────
P17  Cierre y recalibración   ▸ GATE 5 · Aprobar recalibración
```

## El flujo paralelo de definición de planta

*Añadido el 12-ago (#94).* La planta **no se configura dentro del proyecto**: tiene su propio flujo,
su propio ciclo de vida y su propio versionado. El proyecto **se asocia** a una planta ya configurada.

```
FLUJO DE PLANTA  (paralelo, por administrador de planta)
  P11·A  Alta de planta          identidad, ubicación, layout
  P11·B  Capacidad y VSM         estaciones, velocidades base y con OT,
                                 cuello de botella, capacidad de habilitado
  P11·C  Calibración por clase   eje 24/40/60/90 · desperdicio, soldadura,
                                 conexiones, tornillería
  P11·D  Financieros             nómina de la planta → $/HH propio y subcontrato
                                 + overhead asignado desde TenantConfig según su reparto
                                 ▸ valida tolerancia 2% y cargas < overhead POR PLANTA (inv. 15)
  P11·E  Publicar versión        genera PlantConfig vN inmutable (inv. 16)
     ║
     ║  se enlaza en →  P3 paso 1 · selector de planta y versión, con acceso a esta definición
     ▼
FLUJO OPERATIVO  (semanal, por planeación y almacén de la planta)
  P14·A  Carga de planta         proyectos en curso, HH libres/sem  ← E7
  P14·B  Inventario              disponible / asignado / libre, préstamos ← E7
                                 ▸ marca de frescura; si está viejo, S12 no es fiable
```

### El segundo flujo paralelo: montaje

*Añadido el 13-ago (#112).* Mismo patrón, distinto disparador. El de planta se elige siempre; **el de
montaje se activa solo si el alcance del proyecto incluye montaje** — si es solo suministro, se apaga.

```
FLUJO DE MONTAJE  (paralelo, activado por la opción de alcance en P3 paso 3)
  P22·A  Alcance y plazo         T_MAX_SEM
  P22·B  Loteo y transporte      LOTE_TON ≤50 t · costo de cambio de lote · lote chico
  P22·C  Soldadura de campo      kg de aporte · reparto por posición · % de UT
                                 ▸ COMPUERTA: sin verificación el motor no corre
  P22·D  Condiciones de obra     obra falsa · lluvia · festivos · sitio
  P22·E  Tarifas y BOM contractual  heredado de PlantConfig + tonelaje contractual FIRMADO
     ║
     ╚═→ alimenta el motor de montaje, que devuelve lotes, frontera costo–tiempo
         y señales de ingeniería de valor. Ver MOTOR_MONTAJE.md
```

Por qué importa que sea paralelo y no un paso del proyecto:

- **Se reutiliza.** Un proyecto nuevo apunta a la misma `PlantConfig` sin reconfigurar nada.
- **Se versiona.** «Usar una definición nueva» es publicar `vN+1`; los proyectos anteriores siguen
  ligados a la suya. Es el mecanismo del inv. 16, ahora **por planta**.
- **Un tenant puede tener varias plantas.** El intake elige a cuál se fabrica.
- **Los ritmos son distintos.** La configuración es trimestral; la carga y el inventario, semanales.
  Meterlos en el mismo flujo obligaría a revisar todo cada semana.

Sin `PlantConfig` publicada, un proyecto **no puede pasar de B1**: B2 necesita capacidad y carga.

---

### Lo que cambia respecto al flujo anterior

| | Antes | Ahora |
|---|---|---|
| Posición de P9 | Justo tras el chat de edición | **Al final de B3**, después de toda la planta |
| P14 / P15 / P16 | Pantallas sueltas, sin lugar en la secuencia | **Bloque B2 completo**, entre el gate de BOM y el precio |
| P13 | Sin ubicar | **Arranque de B3**, antes de costear (corregido el 12-ago) |
| Orden de las compuertas | 1 → 2 → 3 → 4 → 5 | **1 → 4 → 3 → 2 → 5** |

> **Por qué el gate de precios está en B3 y no al final (corregido el 12-ago).** El inv. 8 exige que
> la confirmación esté **registrada y con vigencias vigentes al emitir** — no que sea el último paso.
> Y los precios son **entrada de §4.3**: mostrar opciones con $/ton calculados sobre precios sin
> validar convierte la compuerta en un trámite, y si el usuario rechaza un precio, todo lo que acaba
> de revisar estaba mal. Después del gate de BOM ya se sabe qué materiales lleva la oferta, así que la
> confirmación cabe ahí. En B4 queda una **revalidación de vigencia** que puede reabrirla — algo que ya
> era necesario porque la vigencia caduca sola.
>
> Daniel escribió «compuerta final» en §14 del Documento Maestro. El requisito se cumple igual, pero
> el cambio de ubicación interpreta su invariante: **pendiente de confirmar con él** (#94).

> **Aviso sobre la numeración de las compuertas.** La que usa
> [INVARIANTES_Y_COMPUERTAS.md](INVARIANTES_Y_COMPUERTAS.md) es de **catálogo**, no de flujo. En
> ejecución ocurren en el orden 1 → 4 → 3 → 2 → 5. Al implementar, guiarse por los bloques, no por el
> número.

## Cadena de CTAs del workspace

El botón primario contextual de la barra superior es el que hace legible el flujo. Hoy tiene tres
estados y le faltan tres:

| Estado de la sesión | CTA primario | Color | Lleva a |
|---|---|---|---|
| BOM sin aprobar | **Aprobar BOM** | `--red` (gate) | B2 |
| Aplicando cambios | *(deshabilitado)* | — | — |
| BOM aprobado | **Analizar planta** | `--navy` | P14 |
| Interferencia activa sin decidir | **Decidir familias** | `--amber` | P16 |
| Programa sin decidir | **Decidir programa** | `--amber` | P15 |
| Programa decidido | **Confirmar precios** | `--red` (gate) | P13 |
| Precios confirmados | **Generar opciones** | `--teal` | P9 |
| Opciones vigentes | **Emitir oferta** | `--teal` | P10 |
| Oferta emitida | **Registrar cierre** | `--navy` | P17 |

Los dos CTAs en rojo son los dos gates bloqueantes. Los dos en ámbar exigen decisión registrada pero
no bloquean el cálculo. Coherente con la regla de color del handoff.

## Ciclo de vida corregido

El borrador del contrato tenía `awaiting_price_confirmation` **antes** de
`awaiting_program_decision`. Está invertido: la confirmación de precios es la última compuerta antes
de emitir, y la decisión de programa ocurre en B2, mucho antes.

```
perceiving
  → awaiting_bom_review            ▸ GATE 1                          (fin de B1)
  → computing_plant                                                   (B2)
  → awaiting_family_decision       ▸ GATE 4  · solo si INV-03 activa
  → awaiting_program_decision      ▸ GATE 3  · solo si hay retraso
  → awaiting_price_confirmation    ▸ GATE 2 · arranque de B3
  → computing_price                bloque B3 · los precios son entrada de §4.3
  → ready
  → emitted
  → awaiting_closure                                                  (B5)
  → closed
       ↳ recalibration_proposed → approved | rejected                ▸ GATE 5
  ⟲ editing  desde cualquier estado posterior a B1; si altera el takeoff
             reabre GATE 1 y **invalida B2 y B3**
```

**Consecuencia de la reapertura que hay que diseñar:** una edición que cambie el takeoff no solo
reabre el gate de BOM — **invalida el bloque de planta y el de precio**. Los lotes, S11, S12 y el APU
se calcularon sobre un peso que ya no es el vigente. Hoy el mockup solo marca los escenarios como
«desactualizados»; debe marcar también la decisión de programa.

## Impacto en las etapas de progreso (P4)

Las cuatro etapas actuales —*Leyendo planos → Construyendo geometría → Generando BOM → Validando*—
cubren **solo B1**. Están bien para lo que son, pero el usuario no puede inferir de ahí que después
vienen tres bloques más de cómputo.

Propuesta: mantener las cuatro etapas de B1 en P4 (es el job de percepción), y que **B2 y B3 tengan
su propio streaming** al lanzarse desde el workspace, con las secciones del motor como etapas:
`Lotes y secuencia → Chequeo de grúa → Velocidad S11 → Requisición → Programa S12` para B2, y
`APU AISC → Más X% → Costo empresa → Flujo de caja → RSS` para B3. Cada etapa es una tool: encaja con
la trazabilidad de *«ninguna cifra sin tool»*.

## Consecuencia para F0

Más nítida de lo que estaba dicho: **F0 solo puede entregar B1.** No es que «el visor no muestra
precio» — es que ni el precio ni el programa existen, porque B2 y B3 necesitan tools que en F0 no
están construidas.

Lo que F0 entrega: visor navegable, BOM, exclusiones, doble chequeo de peso y el gate de aprobación.
Es un entregable coherente y demostrable por sí mismo — el «wow» de ver la solución en 3D y validar
el takeoff — y encaja con la promesa de F0 Toyota sin prometer un número que no puede calcular.

## Qué no cambia

- El visor 3D sigue siendo el protagonista y el chat el volante del modelo.
- Los tres supuestos ⚙️ del handoff (`chatMode`, `bomApproval`, `costUpdate`) siguen sin congelar.
- El supuesto `costUpdate: vivo` gana un matiz: durante la edición se recalcula lo que **existe**.
  Si aún no se ha corrido B3, no hay costo que recalcular y el total va en «pendiente de motor».
