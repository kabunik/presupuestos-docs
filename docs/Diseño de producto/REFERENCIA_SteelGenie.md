# Referencia de diseño — Steel Genie by ALLPLAN

> Flujo extraído de 16 capturas del webinar, en `docs/steel genie/` numeradas por su orden en el
> recorrido. **Qué tomamos, qué no, y con qué lo mapeamos.**
> **2026-08-12** · insumo de P11 y P20 · issue #97

## Por qué mirarlo

Steel Genie resuelve bien **la mitad que nosotros teníamos sin diseñar**: la interacción del humano con
la ingesta de planos. Es el competidor que la Segmentación v2 ya identificaba —*«llega hasta el visor
3D y la edición previa a exportar; nuestro flujo va más allá»*— y esa frontera se confirma en el
recorrido: **no tiene planta, ni capacidad, ni programa, ni precio.** Termina en modelo + BOM + IFC.

Así que tomamos su flujo de ingesta como referencia y conservamos íntegro lo que viene después.

## Su flujo, por pestañas

`Projects` → `Plans` · `Columns` · `Braces` · `BOM` · `Config`, con un toggle **3D** siempre presente.

### Plans — el corazón, y lo que nos falta

| Pieza | Cómo funciona |
|---|---|
| **Rail de páginas** | Cada página del PDF es una tarjeta: miniatura, código de hoja (`S2.00`), badge de tipo (`Foundation`), badge de estado, y un botón ▶ para lanzar la detección de esa página |
| **Estado por página** | `Not Set` → `Estimating` → `Requesting Build` → `✓ Built`. El estado es **por hoja**, no por proyecto |
| **Campos obligatorios por página** | **Top of Steel / Bottom of Column** y **Escala** (`1/8" = 1'-0"` + su ratio `96:1`). Con validación explícita: *«set a valid T.O.S.»* bloquea la página |
| **Modo recorte** | Se define la región a procesar arrastrando manijas; autoguarda al soltar. Aviso: *«asegúrate de que el recorte cubre todas las líneas de grilla para una detección precisa»* |
| **Filtro de capas** | Toggles de `Plan · Grids · Beams · Undefined Beams · Columns`; etiquetas de `Piecemarks / Lengths / Reactions`; y un **filtro de longitud** (de 10 a 20 pies) para aislar lo que falta revisar |
| **Color sobre el plano** | Violeta = detectado · **rojo = requiere atención** · amarillo = listo para revisión rápida · verde = conexiones y arriostres |
| **Resumen vivo** | Dos bloques, **Project Summary** y **Sheet Summary**, con los mismos 13 contadores: Column, Beam, Vertical/Horizontal Brace, Joists, Moment/Default Connection, Bolt, Camber, Anchor, Weld Studs, **Total Weight (tons)** y **Hrs/Ton** |
| **Vista partida con 3D** | Plano a la izquierda, 3D a la derecha, sincronizados. El 3D colorea **por tipo de miembro o por peso** (mapa de calor con bandas en lbs) y muestra al pie el elemento seleccionado: `B_284 \| w16×31 \| 29'-1 3/4" \| 903 lbs \| Not Set \| Seq: 1` |

Lo que la IA lee del plano, según su propio copy: **escala, T.O.S., miembros y cotas de columna**.

### Config — y el patrón más interesante de todos

Panel **Genie Insights**: la IA escanea las notas generales de todas las hojas y **reporta su
inferencia en prosa**, incluido lo que *no* pudo determinar. Textual del recorrido:

> «En todas las hojas —incluidas DESIGN A (S0.10), FOUNDATION AND GRADE LEVEL FRAMING PLAN (S2.00) y
> los distintos planos de estructura— **no aparece lenguaje de diseño LRFD ni ASD**, por lo que el
> proyecto no usa método especificado. Aunque las hojas S2.10, S6.00 y S6.10 referencian marcos
> arriostrados y de momento genéricos, **les faltan los calificadores (SMF, IMF, etc.)** que definirían
> un sistema sismorresistente, con lo que todos los grupos SFRS quedan clasificados como ninguno.»

Debajo, secciones de configuración de proyecto: método de diseño de conexiones (ASD / LRFD), cálculo
de reacción de extremo de viga (con slider de % de UDL), tipo de marco sísmico (Non-Seismic / OMF /
IMF / SMF), tipo de conexión, material, prioridades de tamaño, recubrimiento, embarque. Con
**Restore / Apply** al pie y badges `Predesign` y `Under Development`.

### Braces y BOM

**Braces** es un programador de arriostres por bahía: se elige columna inicial y final, método de
conexión, tipo de marco sísmico, tipo de arriostre y sección; se dibuja la elevación con los niveles y
sus cotas. Aparecen dos patrones que nos importan: **«Model is outdated. Rebuild required.»** y
**«Unsaved Changes» con Restore / Apply**.

**BOM** es tabla densa con rail de filtros (categoría, piecemark, tipo de sección, sección, código de
mano de obra, principal/accesorio, estado, secuencia, pintura) y **Export → PDF o IFC**, con perfiles
predefinidos **para SDS/2 Stick Model y para Tekla Structures**, y opciones de qué incluir: copes,
conexiones, tornillos, agujeros, placas base, anclas, zapatas.

---

## Mapeo con nuestro producto

### Lo que adoptamos

| Patrón de Steel Genie | Dónde entra en lo nuestro | Por qué |
|---|---|---|
| Estado por página `Not Set → Estimating → Built` | **P20** | La ingesta es página por página, no un job monolítico. Encaja con que la percepción falle parcialmente |
| **T.O.S. y escala obligatorios por página**, con validación bloqueante | **P20** | Sin cota ni escala no hay geometría. Es el equivalente de nuestro «no derivar en silencio» |
| Modo recorte con aviso de cubrir la grilla | **P20** | Reduce ruido y errores de detección |
| Filtro de capas, incluida una capa de **«no definidos»** | **P20** | Aquí encaja nuestro **filtro de concreto** (#90) y el aislamiento de **elementos sin peso** (#91) |
| Filtro por longitud para aislar lo pendiente | **P20** | «Muéstrame solo lo que falta revisar» es la interacción de revisión más útil del recorrido |
| Resumen doble: proyecto y hoja, en vivo | **P20** | Verificar exactitud sin salir de la pantalla |
| Vista partida plano ⇄ 3D sincronizada | **P20** | Nuestra selección bidireccional ya existe entre grilla y 3D; extenderla al plano es natural |
| Colorear el 3D por atributo, con mapa de calor por peso | **P5** workspace | «Entender los cost drivers de un vistazo». Encaja con nuestra ley de color |
| **Genie Insights**: la IA reporta en prosa lo que leyó *y lo que no pudo determinar* | **P20** | Es el patrón que necesitábamos para la **simbología de soldadura** (#89): el agente declara su lectura y su incertidumbre, y el humano confirma |
| «Model is outdated. Rebuild required.» | **P5, P9, P15** | Es exactamente nuestra **invalidación en cascada** (`FLUJO_v1.md`) |
| «Unsaved Changes» con Restore / Apply | **P11** | Configuración con confirmación explícita, coherente con el versionado inmutable del inv. 16 |
| Export IFC con perfiles para SDS/2 y Tekla | **P10** | **Confirma que IFC4 es el entregable esperado**, no una preferencia nuestra. Refuerza la decisión de proyecciones |

### Lo que no adoptamos

| Suyo | Por qué no |
|---|---|
| **Su `Config` como equivalente de nuestro P11** | Su Config es **de proyecto**: método de diseño, tipo de marco sísmico. Nuestro P11 es **de planta**: capacidad, VSM, HH/ton, factores por clase — y es **versionado y compartido entre proyectos**. Son cosas distintas y necesitamos las dos |
| Tema oscuro y densidad de su UI | Nuestro sistema es claro y con la paleta de `tokens.css`. La referencia es de **flujo**, no de aspecto |
| `Hrs/Ton` como celda calculada al vuelo | Para nosotros HH/ton es **entrada calibrada de la planta** (eje 24/40/60/90, inv. 2), no un derivado del modelo. Coincide el nombre, no el significado |
| Braces como programador manual de arriostres | Nuestro alcance de v1 no incluye diseñar arriostres; la percepción los detecta y el agente los edita por lenguaje natural |

### Lo que ellos no tienen y es nuestro diferenciador

Su recorrido termina en modelo + BOM + IFC. **No aparece en ninguna pantalla:** capacidad de planta,
carga combinada, ocupación de habilitado, programa de fabricación, escenarios de programa, precios de
materia prima, opciones de oferta, flujo de caja, ni cierre contra ejecutado.

Es decir: **todo nuestro B2 y B3.** La frontera que la Segmentación v2 declaraba se confirma
observando el producto, no solo su material comercial.

---

## Consecuencias para el diseño de P20

De juntar su flujo con nuestros invariantes salen cinco requisitos que no teníamos escritos:

1. **La ingesta es por página y con estado propio.** Una página sin escala o sin cota no bloquea el
   proyecto entero: bloquea esa página y lo declara.
2. **Escala y cota de nivel son obligatorias y validadas.** Es el mismo criterio que aplicamos al
   ritmo de demanda en `PLAN_DE_MONTAJE.md`: si el sistema lo deriva, lo dice.
3. **Hace falta una capa de «no resueltos»** — elementos detectados sin sección, sin peso o sin
   clasificar. Es la superficie de `VAL-08` (#91) y del filtro de concreto (#90).
4. **El agente reporta su lectura en prosa, incluida su incertidumbre.** Es el diseño correcto para la
   simbología de soldadura (#89), y es más honesto que una lista de campos rellenos.
5. **La confirmación es por página y acumulativa**, y alimenta el gate de BOM. P20 no sustituye el
   gate: lo prepara.
