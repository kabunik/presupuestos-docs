# Reconciliación de diseño — mockup × Sistema DM v1.7

> Qué del método DM no está contemplado en el mockup hi-fi, y cómo se integra.
> Complementa `docs/mockup/design_handoff_plataforma_presupuestos/README.md`, que sigue siendo la
> autoridad de tokens, medidas y copy de las pantallas que ya cubre.
> **2026-08-06** · issue #80

> **Actualización del 11-ago (#85).** Este documento especificó las pantallas nuevas pero no las
> insertó en la secuencia del flujo. Resuelto en
> `docs/00-Fundamentos/FLUJO_v1.md`: el orden lo impone la cadena de dependencias del motor, y
> **P9 va al final**, después del bloque de planta, porque el APU necesita el pico de S11.

## Punto de partida

El mockup cubre **P3, P4, P5·A/B/C, P6, P8, P9** y es sólido en lo que cubre: el gate de BOM, la
edición conversacional con delta y rechazo por guardarraíl, y la comparativa de opciones con flujo de
caja están bien resueltos y no se tocan en lo esencial.

El problema es de **cobertura**, no de calidad: se diseñó antes del paquete v1.7, de modo que la
mitad del valor del Sistema DM no tiene superficie, y el recorrido diseñado **no llega a emitir una
oferta válida**.

## Regla de diseño que gobierna esta reconciliación

Las piezas nuevas no son «pantallas de datos»: casi todas son **momentos de decisión humana**. El
mockup ya acertó con el gate de BOM al tratarlo como *«un momento de diseño deliberado, no un botón
perdido»*. Ese mismo criterio se aplica a las cuatro compuertas restantes.

Corolario de color, con el rojo reservado a gates y alertas críticas: **hay dos gates bloqueantes, no
uno.** El de precios de MP (P13) merece el mismo peso visual que el del BOM. Los otros tres
(programa, interferencia, recalibración) **exigen decisión registrada pero no bloquean el cálculo** →
van en ámbar, no en rojo. Esa distinción visual es la mitigación de la fatiga de alertas que el
consultor pide explícitamente.

---

# Parte 1 · Cambios en pantallas existentes

## P3 · Wizard de intake — de 3 a 4 pasos

**El cambio más consecuente de toda la reconciliación.** `plan_montaje_t_sem` es campo *required* de
E6 y sin él el motor no llega a `precio_licitacion` (inv. 11). El wizard no lo pide.

### Nuevo paso 4 · «Plan de montaje y programa»

Stepper pasa a 4 filas. Título del paso: **«Plan de montaje»**, subtítulo *«Secuencia y ritmo de
obra»*.

Contenido, en dos bloques separados por `1px --line`:

**Bloque A — Plan de montaje (t/semana).** Tabla editable densa: columna `Semana` (mono 12,
`--text-4`) y columna `t/semana` (input mono 13, 34 px de alto, borde `--input-border`, radio 8).
Botón secundario «Importar del cliente» (acepta xlsx/csv) y «Añadir semana» al pie. A la derecha,
tarjeta `--surface-sunken` con los derivados en vivo, en mono: **Total del plan** · **Pico t/sem**
(en `--amber` si supera la velocidad con OT) · **Semanas** · **Ritmo medio**.

**Bloque B — Estado de la ingeniería (fase RSS).** Tres tarjetas seleccionables en fila
(`repeat(3,1fr)`, gap 12, radio 12, padding 14). Cada una: punto de 10 px del color del semáforo,
título 13.5/600, subtítulo 11.5 `--text-3`, y el % de crecimiento por defecto en mono 13:

| Fase | Color | Subtítulo | Default |
|---|---|---|---|
| Verde | `--teal` | Ingeniería para construcción o cercana | 6% |
| Ámbar | `--amber` | Diseño en desarrollo | 10% |
| Rojo | `--red` | Anteproyecto o alta incertidumbre | 15% |

Seleccionada: borde del color, fondo del lavado correspondiente. Debajo, campo «% de crecimiento
aplicado» con el default precargado y —**solo si el usuario lo cambia**— un textarea obligatorio
«Justificación del cambio» (inv. 6: todo valor distinto del defecto exige justificación registrada).

> ⚠️ **Sin resolver (#70):** qué pasa cuando **no hay** plan de montaje. Es un caso real y frecuente
> — CNARCCS cotiza con montaje excluido. El mock representa el caso con plan; el comportamiento
> degradado (¿derivar del plazo? ¿bloquear? ¿modo sin S11/S12?) es decisión pendiente.

### Añadidos al paso 1 (Datos generales)

- **Clase de complejidad (HH/ton)**: selector único de 4 opciones —24 / 40 / 60 / 90— con la
  tipología como texto de apoyo (inv. 2). **Un selector, nunca cuatro campos**: la clase mueve
  desperdicio, soldadura, conexiones y horas de forma coherente.
- Enlace secundario **«Partir en subproyectos por familia»**, visible siempre pero solo necesario
  cuando las familias difieren ≥1 salto (inv. 3). Lleva a P16.

### Añadido al paso 3 (Consideraciones)

**Tabla de desviaciones de concurso** (inv. 10), 5 columnas: `Concepto | Norma | Concurso | Rige |
Impacto`. La columna `Rige` es un segmentado de dos valores (Norma / Concurso) con **Concurso como
default** —el concurso rige donde difiera—. `Impacto` en mono, y un pie con el **total sumado** que
entra al presupuesto. Una desviación con impacto vacío se marca en ámbar: *«una desviación sin costo
asignado es una desviación sin registrar»*.

## P4 · Progreso de generación

Una corrección: la etapa 4 dice **«22 reglas de validación»**. La cifra no tiene fuente (#68).
Sustituir por **«Invariantes y validaciones de percepción»** hasta que el catálogo esté cerrado.

## P5 · Workspace — tres indicadores nuevos en la barra superior

El principio *«el estado del sistema siempre visible»* obliga a exponer tres estados que hoy no se
ven. Van en la barra superior de 56 px, después del chip de versión del modelo, como chips de la
misma familia (mono 11, radio 6):

| Chip | Contenido | Colores |
|---|---|---|
| **Semáforo RSS** | Punto de 6 px + `RSS 9.3% · ámbar` | Punto del color de la fase; fondo `rgba(255,255,255,.09)` |
| **Doble chequeo** | `Peso ✓` o `Peso ✗ detenido` | Teal si ok; **rojo si `detener_emision`** (inv. 4) |
| **Estado de emisión** | `Emisión bloqueada` con motivo en tooltip | Ámbar cuando falta una compuerta; oculto cuando está habilitada |

**Barra inferior — consecuencia de F0.** Hoy muestra «Costo estimado $49.5M» desde el primer output.
En F0 el motor no existe: el campo va con estado **«pendiente de motor»** (chip `--surface-alt`,
`--text-3`), no con una cifra. Añadir ese estado a la matriz de la barra.

## P6 · Grilla BOM — sección de exclusiones

Bloque nuevo bajo el pie de subtotal, **siempre visible, nunca colapsado por defecto** (inv. 5: el
cero silencioso está prohibido por diseño). Cabecera «Exclusiones declaradas» (12/700 uppercase
`--text-4`) y filas con pill `Excluido` (`--surface-alt`, `--text-3`) + concepto + leyenda:

- **Escaleras** — no se cobran como acero estructural (AISC 303-22 §2.2); partida separada.
- **Pernos de anclaje F1554** — suministro del contratista de cimentación.
- **Nelson studs** — excluidos del alcance de fabricación.
- **Steel deck y escaleras de concreto** — fuera del alcance de estructura metálica.

Redacción tomada del caso CNARCCS, que resuelve bien este punto.

## P9 · Opciones de oferta — cuatro añadidos

1. **Renombrar «Escenarios» → «Opciones de oferta»** en toda la pantalla. Son las 3 opciones
   comerciales; los *escenarios de programa* son otra cosa (P15). Ver `00-Fundamentos/GLOSARIO.md`.
2. **Leyenda de alcance** al pie, en tarjeta `--surface-sunken` de ancho completo, texto 12
   `--text-3`, con el texto obligatorio literal. No es letra pequeña de condiciones: es una
   declaración de responsabilidad y va visible.
3. **Tarjeta «Composición y reconciliación»**: APU AISC · Más X% a margen comparable · **Δ de
   reconciliación** con pill `✓ <1%` en teal (o rojo si excede) · % de crecimiento aplicado y su
   fase RSS · justificación si difiere del default. Es el indicador de salud del inv. 6.
4. **Corregir E2 a `67,076` $/ton** y total `$49.27M` — el valor real del caso es 67,075.70, no
   67,500.

---

# Parte 2 · Pantallas nuevas

> **Nota sobre los datos de muestra.** P13 usa la tabla de precios del propio consultor
> (`Guia_Ejemplos` §3.2). **P14 y P15 usan las cifras del `golden_test.json`** —no datos
> inventados— porque son las únicas cifras de carga de planta, ocupación y escenarios de programa que
> existen con respaldo, y son además las que el motor debe reproducir exactas. Consecuencia a tener
> en cuenta al leer el mock: **esas dos pantallas muestran el caso de referencia de 100 t, no
> CNARCCS**, y su horizonte es de 5 semanas. Al implementar, se pueblan desde E7 real. Cifras
> completas en el anexo.

## P13 · Gate de confirmación de precios de materia prima — **Crítica**

**Segunda compuerta bloqueante.** Sin `confirmado = true` registrado en bitácora,
`estado_emision: "bloqueada"` y no hay `precio_licitacion` (inv. 8).

**Formato: modal centrado de 720 px**, no pantalla completa. Razón de diseño: es una interrupción
deliberada en el flujo de emisión, no un destino de navegación. Overlay `rgba(16,24,40,.45)`,
tarjeta blanca radio 16, sombra `--shadow-overlay`.

- **Cabecera** (padding 22/24, borde inferior `--line`): eyebrow «Compuerta de emisión» 11/700
  uppercase en `--red`; título 17/600 Space Grotesk **«Confirma los precios de materia prima»**;
  subtítulo 12.5 `--text-3` *«Sin confirmación registrada no se emite precio de licitación. La
  confirmación queda ligada a esta oferta como evidencia.»*
- **Tabla de precios**, 5 columnas `1fr 96px 132px 96px 104px` → Material · Precio · Fuente · Fecha ·
  **Vigencia**. Cifras en mono 12.5. La columna Vigencia lleva pill por estado:
  `9 días` en teal · `6 días` en **ámbar** (≤7 días) · `Vencido` en **rojo**. Una fila vencida
  colorea toda la fila con `--red-050`.

| Material | Precio | Fuente | Fecha | Vigencia |
|---|---|---|---|---|
| Perfil W (A992) | $24.80/kg | Cotización molino | 2026-07-21 | 9 días |
| Placa (A572-50) | $26.10/kg | Distribuidor | 2026-07-24 | 12 días |
| HSS (A500-C) | $28.35/kg | Distribuidor | 2026-07-18 | **6 días** |
| Soldadura FCAW E71T-1 | $118.00/kg dep. | Proveedor | 2026-07-15 | 30 días |

- **Aviso de vigencia corta** si algún material está ≤7 días: tarjeta `--amber-050`, borde
  `--amber-border`, texto 12 `--amber-800`: *«HSS (A500-C) vence en 6 días. Si la oferta se emite
  después, la compuerta se reabre automáticamente.»*
- **Pie**: casilla obligatoria de 22 px + texto 13 *«Confirmo que estos son los precios considerados
  en la oferta»*; a la derecha «Cancelar» (secundario) y **«Confirmar y habilitar emisión»**
  (`--red`, 42 px, **deshabilitado hasta marcar la casilla**). Bajo el botón, 11.5 `--text-4`:
  «Se registrará: J. Pérez · 06/08 · oferta CNARCCS-2026-014».

**Estado posterior — bitácora.** Una vez confirmado, el modal se sustituye por una tarjeta de
registro en P9/P10: pill `Confirmado` teal + «J. Pérez · 06/08 14:12 · 4 materiales» + enlace «Ver
bitácora ›». Al vencer una vigencia, la pill pasa a `Reabierto` en ámbar.

## P14 · Carga de planta y ocupación de habilitado — Alta

Pantalla completa 1440×900. Es la superficie que conecta el precio con la planta real.

- **Cabecera**: título 20/600 «Carga de planta» + pill de **frescura de E7** — `Actualizado hace 2
  días` en teal, `hace 9 días` en ámbar, `hace más de 14 días` en rojo con el texto «S12 no es
  fiable». A la derecha «Actualizar carga» (secundario).

  > Requisito de producto, no de proceso: el consultor advierte que *«E7 desactualizado convierte S12
  > en ficción bien presentada»*. Ver #71.

- **Gráfico de carga combinada**, tarjeta de ancho completo, alto 300 px. Barras apiladas por semana
  (5 semanas del caso de referencia: **67 / 83 / 60 / 70 / 45 t/sem**) con un segmento por origen: el
  proyecto nuevo en `--teal` y los contratos en curso en `--teal-200`. El eje se lee en toneladas por
  semana y cada barra lleva su valor en mono encima. Leyenda con cuadrados de 9 px.

- **Fila de ocupación de habilitado**, tarjeta de ancho completo: una columna por semana. Cada una:
  etiqueta de semana mono 10.5 `--text-4`, barra vertical de 90 px de alto máximo, y el % en mono 12
  debajo (**89 / 111 / 80 / 93 / 60 %** sobre capacidad de 600 HH/sem). **Marca de 100% como línea
  discontinua roja** cruzando la tarjeta. Barras ≥100% en `--red`, 85–99% en `--amber`, resto en
  `--teal-200`. **Alerta no suprimible** sobre la semana 2: pill `≥100%` en rojo, sin control de
  silencio (inv. 14).

- **Panel derecho 400 px — chequeos S11** (inv. 12), tres tarjetas:
  1. **Velocidad de montaje**: pico del plan vs. base / con OT. Si el pico excede, `alerta_subcontrato_hh`
     con las HH en mono y el costo de resolución. Pill ámbar.
  2. **Velocidad de planta vs. equilibrio**: volumen contratado, punto de equilibrio y equilibrio con
     Lean. Si opera bajo equilibrio → **`advertencia_bajo_equilibrio`, no suprimible**: tarjeta con
     borde `--amber-border`, déficit en toneladas con y sin Lean, y texto explícito de que la
     advertencia no puede desactivarse.
  3. **Requisición neta** (inv. 13): cascada visible bruta → inventario libre → **neta a comprar**,
     en tres filas mono con la neta en 600. No solo el neto.

## P15 · Decisión de programa — escenarios A/B/B\* /C — Alta

Pantalla completa. El diferenciador que ningún competidor integra al costeo.

- **Cabecera**: «Decisión de programa» 20/600 + pill `Requiere decisión` en **ámbar** (exige decisión
  registrada, no bloquea el cálculo) + contexto a la derecha: «Semana 2 al 111% de ocupación ·
  déficit 920 HH».

- **Cuatro tarjetas** en grid `repeat(4,1fr)`, gap 16, radio 16, padding 18. La recomendada (B\*)
  lleva borde `--teal` y `--shadow-scen`; **C lleva borde `--input-border` y aspecto atenuado**.
  Cada tarjeta: nombre display 15/600 · descripción 12 `--text-3` · **Costo del déficit** en mono 20
  · filas separadas por borde superior con `Utilidad`, `Retraso` y `Pena máx. soportable` · botón de
  selección al pie.

| | **A** | **B** | **B\* optimizado** | **C** |
|---|---|---|---|---|
| Qué hace | Cumplir ambos programas a fuerza de OT/subcontrato | Resolver el déficit con la mezcla estándar | OT + desplazar solo el excedente | No entrar al contrato |
| Costo del déficit | $145,521 | $115,025 | **$64,148** | — |
| Utilidad | $751,082 | $781,578 | **$832,455** | — |
| Retraso | 0 sem | 0 sem | **1 sem** | — |
| Pena máx. soportable | — | — | **$81,373** | — |
| Estado | Alternativa | Alternativa | **Recomendado** | **No recomendable** |

- **Detalle del escenario recomendado**, tarjeta bajo las cuatro: desglose de la optimización en tres
  cifras mono —**OT 1,000 HH** · **Desplazar 600 HH a Sem 5** · **Retraso 1 semana**— con la fórmula
  visible como texto de apoyo: `retraso mínimo = ceil(HH_a_desplazar / HH_sem)`. Cumple el principio
  de trazabilidad: todo número visible puede explicarse.

- **Compuerta de comunicación al cliente** — obligatoria (inv. 14). Tarjeta `--amber-050`, borde
  `--amber-border`, borde izquierdo 4 px `--amber`:
  > **Acuerda el retraso antes de comprometer el contrato nuevo.**
  > El escenario recomendado desplaza 1 semana la entrega de *Nave Industrial Tultitlán*. Debes
  > comunicarlo y acordarlo con ese cliente **antes** de firmar. Sin acuerdo registrado, la
  > recomendación cae al escenario A.

  Casilla obligatoria «Retraso comunicado y acordado con el cliente afectado» + campo de fecha y
  responsable. **El botón «Confirmar programa» está deshabilitado hasta marcarla.**

- **Nota sobre C**, texto 11.5 `--text-4` al pie: *«El escenario C no se recomienda con la planta
  operando bajo el punto de equilibrio»* (inv. 14).

## P16 · Interferencia de familias y subproyectos — Alta

Puede vivir como pantalla completa o como panel expandido del workspace; el mock la resuelve como
pantalla para poder mostrar el payload completo.

- **Banner de alerta** superior: `--amber-050`, borde izquierdo 4 px `--amber`, pill
  `Alerta · no suprimible` + código mono. Título 14/700: «Los misceláneos saturan habilitado y
  reducen la productividad de las armaduras». Texto de apoyo: *«Esta alerta no se puede desactivar.
  Requiere una decisión documentada.»*

- **Tabla comparativa de familias**, 2 columnas de datos con la métrica a la izquierda. La familia
  más rentable se resalta en teal; la de HH altas en ámbar:

| Métrica | Misceláneos | Armaduras |
|---|---|---|
| Clase (HH/ton) | **90** | **24** |
| Tonelaje | 80 t | 320 t |
| Rentabilidad ($/HH de margen) | $31 | **$58** |
| HH en conflicto | 210 HH/sem · semanas 3–5 | — |
| Impacto | — | **−12% productividad → 380 HH extra** |

- **Cuatro opciones para la dirección**, en filas clicables (radio 12, borde `--line`, padding 14),
  cada una con su consecuencia en mono a la derecha. La IA propone; la dirección dispone:
  1. **Repriorizar** — mover misceláneos a la ventana con holgura
  2. **Partir el lote de misceláneos** — dos entregas
  3. **Subcontratar los misceláneos** — libera habilitado, coste all-in
  4. **Aceptar el impacto documentado** — se registra y se costea en el presupuesto

  La opción 4 es una **decisión registrada, no una supresión**. Distinción explícita en el copy.

- **Tarjeta de subproyectos** (inv. 3): dado el salto ≥1 en el eje, el proyecto se **parte**. Dos
  filas, una por subproyecto, cada una con su clase, tonelaje, APU propio y carga de planta.
  Texto de apoyo: *«Promediar clases distintas está prohibido: el promedio esconde el error en ambas
  direcciones.»*

## P17 · Cierre de proyecto y recalibración — Media

- **Formulario de cierre**: 8 filas, una por rubro, tres columnas `Rubro | Presupuestado | Real` más
  una columna calculada **Δ%** que colorea ≤±5% en teal y >±5% en ámbar. Rubros: costo total · HH por
  familia · HH/ton efectivas · desperdicio · peso final · precio de acero · montaje · desviaciones de
  concurso.
- **Bandeja de propuestas de recalibración** (vista de dirección): por cada propuesta, parámetro
  afectado, valor actual → propuesto, **evidencia** (nº de proyectos, magnitud, causa probable), y
  botones **Aprobar** / **Rechazar**. Aprobar genera una **versión nueva** de parámetros y lo dice
  explícitamente: «Al aprobar se crea `v2026.2`. Los presupuestos anteriores siguen ligados a
  `v2026.1`.»
  Ejemplo: *«Elevar clase 40 → 43 HH/t. Evidencia: 3 proyectos (43.1, 42.6, 43.8 HH/t efectivas;
  +7.9% sobre tolerancia ±5%). Causa probable: subestimación de armado en conexiones a momento.»*
- **Lista de cierres pendientes**: proyectos emitidos sin cierre registrado, con aviso de que **no
  alimentan recalibraciones**.

## P18 · Desviaciones de concurso — Media

No necesita pantalla propia: es la tabla descrita en el paso 3 de P3, reutilizada como **panel del
workspace** (cuarta tab, junto a BOM · Alertas · Propiedades) para poder consultarla y editarla
durante la revisión. Mismas 5 columnas y mismo total sumado.

## P19 · Requisición y secuencia de fabricación — Media

Tres salidas *required* del esquema hoy sin superficie alguna:

- **Requisición neta** (inv. 13): cascada bruta → inventario libre → neta. Ya presente como tarjeta
  en P14; aquí en detalle por material, con los préstamos entre contratos registrados.
- **Lotes y déficit** (inv. 11): déficit de HH por semana y su costo de resolución, con la vía
  elegida — **sobretiempo con prima 50%** o **subcontrato all-in**. El déficit nunca se oculta.
- **Chequeo de grúa** (inv. 12 / §4.8): piezas críticas identificadas y el costo de partirlas, con
  nota de que la decisión la valida el EOR. El `ROL.docx` del consultor ya define los parámetros de
  grúa torre para el caso.

Formato natural: pantalla con tres tarjetas apiladas, o tabs de una pantalla «Fabricación».

---

# Parte 3 · Estado de integración en el prototipo

| Pieza | Prioridad | Especificada | En el prototipo |
|---|---|---|---|
| P3 · paso 4 plan de montaje + fase RSS | Alta | ✓ | ✓ |
| P3 · clase HH/ton en paso 1 | Alta | ✓ | ✓ |
| P3 · desviaciones de concurso en paso 3 | Media | ✓ | ✓ |
| P4 · retirar «22 reglas» | — | ✓ | ✓ |
| P5 · chips de RSS, doble chequeo y emisión | Alta | ✓ | ✓ |
| P6 · exclusiones declaradas | Alta | ✓ | ✓ |
| P9 · leyenda, reconciliación, E2 corregido | Alta | ✓ | ✓ |
| **P13 · gate de precios de MP** | **Crítica** | ✓ | ✓ |
| **P14 · carga de planta y ocupación** | Alta | ✓ | ✓ |
| **P15 · escenarios de programa** | Alta | ✓ | ✓ |
| **P16 · interferencia de familias** | Alta | ✓ | ✓ |
| P17 · cierre y recalibración | Media | ✓ | pendiente |
| P18 · desviaciones como panel del workspace | Media | ✓ | pendiente |
| P19 · requisición, lotes y grúa | Media | ✓ | pendiente |
| **P11·A–E · flujo paralelo de definición de planta** | **Alta** | parcial — ver #94 | pendiente |
| **P20 · ingesta asistida de planos** | **Alta** | parcial — ver #94 | pendiente |

**Añadidos el 12-ago (#94):** el flujo de definición de planta es **paralelo** al del proyecto —la
planta se configura una vez, se versiona y el proyecto se asocia a ella—, y la ingesta asistida es la
compuerta humana de la percepción, donde el usuario confirma sobre el plano lo que el agente
interpretó. Ambos son de prioridad Alta y ninguno está mockeado. Estructura del flujo de planta en
`docs/00-Fundamentos/FLUJO_v1.md`.

Las tres pendientes son de prioridad Media y su especificación está completa: se pueden mockear en
la siguiente ronda sin más decisiones de diseño.

---

# Anexo · Cifras del golden test

Las de P14/P15 en el prototipo usan estos valores, que son los que el motor debe reproducir
**exactos** (`golden_test.json`, caso 100 t / clase 40 / apernada):

| Concepto | Valor |
|---|---|
| Carga combinada | 67 / 83 / 60 / 70 / 45 t/sem |
| Ocupación de habilitado | 89% / **111%** / 80% / 93% / 60% (capacidad 600 HH/sem) |
| Déficit | Semana 2: **920 HH** |
| Escenario A | costo $145,521 → utilidad $751,082 |
| Escenario B | costo $115,025 → utilidad $781,578 |
| **Escenario B\*** | OT 1,000 HH + desplazar 600 HH a Sem 5 → retraso **1 sem**, costo **$64,148** → utilidad **$832,455**, pena máx. **$81,373** |
| Recomendación | `escenario_b_optimizado` + `advertencia_comunicacion_cliente: true` |
| S11 montaje | pico 38 t/sem vs. 15 base / 24 con OT → `alerta_subcontrato_hh` 560 HH |
| S11 equilibrio | volumen 318 t vs. equilibrio 490 t (368 con Lean) → déficit 172 t / 50 t con Lean |
| Requisición | 121.55 − 34.00 = **87.55 t** netas |
| RSS | **9.27%** — ámbar |
