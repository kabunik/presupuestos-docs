# Handoff: Plataforma de presupuestos de estructuras metálicas (Kabunik × dcode)

> Paquete de handoff para implementar los mockups en un codebase real con Claude Code.
> Idioma de producto: **español** (con acentos y signos de apertura). Cero emoji.

---

## 1. Overview

Plataforma multi-tenant que automatiza la **cotización y presupuestación de estructuras metálicas**: el estimador sube planos, un agente construye una solución estructural en 3D + BOM, el humano la revisa y aprueba (gate), la edita conversando, y genera escenarios de precio con flujo de caja hasta emitir la oferta.

Este paquete cubre las pantallas de **prioridad alta/crítica** de la primera etapa:

| Ref | Pantalla | Flow | Estado en el mock |
|---|---|---|---|
| P3 | Wizard de intake (**4 pasos**) | Flow 1 | 4 sub-pasos navegables · **selector de planta y versión** en el paso 1 |
| P4 | Progreso de generación (streaming) | Flow 1 | 1 estado (etapa 3 en curso) |
| P5·A | Workspace — revisión del primer output | Flow 2 | gate de BOM visible |
| P5·B | Workspace — edición conversacional en vivo | Flow 3 | delta + rechazo por guardarraíl |
| P5·C | Workspace — comparación de opciones de oferta | Flow 4 | panel derecho expandido |
| P6 | Grilla BOM + gate de aprobación | Flow 2 | estado "re-abierto" + exclusiones |
| P8 | Chat de edición + historial de cambios | Flow 3 | rail en detalle + timeline |
| P9 | Opciones de oferta + flujo de caja + v1 vs v2 | Flow 4 | pantalla completa (1440×1080) |
| **P13** | **Gate de confirmación de precios de MP** | — | modal de 720 px · compuerta bloqueante |
| **P14** | **Carga de planta y ocupación de habilitado** | — | carga combinada + chequeos S11 |
| **P15** | **Decisión de programa (A/B/B\*/C)** | — | 4 escenarios + compuerta de comunicación |
| **P16** | **Interferencia de familias y subproyectos** | — | alerta no suprimible + 4 opciones |
| **P20** | **Ingesta asistida de planos** | — | páginas con estado, capas, lectura del agente en prosa |
| **P11·A–E** | **Definición de planta** (flujo paralelo) | — | **los 5 pasos navegables** + barra de navegación propia |
| **P17** | **Cierre de proyecto y recalibración** | — | 8 rubros con Δ vs. tolerancia ±5% + bandeja de dirección |
| **P18** | **Desviaciones de concurso** | — | **cuarta tab del panel del workspace**, «Concurso» |
| **P19** | **Requisición, lotes y grúa** | — | cascada de inventario + déficit por semana + piezas críticas |

Las seis últimas se añadieron reconciliando con el Sistema DM v1.7 (#80), resecuenciando el flujo
(#85) y tras la revisión del 12-ago (#94, #97). Su especificación —junto con la de P17, P18 y P19,
aún sin mockear— está en `docs/Diseño de producto/RECONCILIACION_DISENO_Mockup.md`.

El **flujo de definición de planta es paralelo** al del proyecto: tiene su propia barra de navegación
en el prototipo y se entra a él desde el **selector de planta del paso 1 del intake**. La planta se
configura una vez, se versiona, y el proyecto se asocia a `(planta, versión)`.

**P20 y P11 toman como referencia de flujo el recorrido de Steel Genie**, extraído en
`docs/Diseño de producto/REFERENCIA_SteelGenie.md` con las 16 capturas en `docs/steel genie/`. Es
referencia de **flujo**, no de aspecto: la paleta y la tipografía siguen siendo las de `tokens.css`.

Fuera de alcance en esta etapa: P1 login, P2 dashboard, P7 aislado, P10 emisión de oferta, P12 admin.

---

## 2. About the design files

Los archivos de `design/` son **referencias de diseño creadas en HTML** — prototipos que muestran aspecto y comportamiento previstos, **no código de producción para copiar tal cual**.

La tarea es **recrear estos diseños en el entorno del codebase destino** (React/Next, Vue, etc.) usando sus patrones y librerías establecidos. Si todavía no existe entorno, elegir el stack más adecuado (recomendación para este producto: **React + TypeScript + Vite/Next**, tabla virtualizada tipo TanStack Table, visor 3D con **three.js + web-ifc / IFC.js**) e implementar allí.

`Plataforma Presupuestos.dc.html` es un componente de diseño con streaming: un único archivo con plantilla + clase de lógica, que depende de `support.js` (incluido) y de las hojas de tokens del sistema Kabunik. Se abre en navegador desde la carpeta `design/` **solo si** se colocan también los tokens del sistema Kabunik en `_ds/kabunik-design-system-.../`; si no están, la página funciona igual pero con las fuentes de Google Fonts (que sí se cargan por CDN) y sin las variables `--*` de Kabunik, que no se usan en la maquetación (todo el estilo del mock es inline y literal). Léelo como **fuente de verdad de medidas, colores y copy**.

**No portar la arquitectura del prototipo** (una sola clase con `state.screen`): en producción cada pantalla es una ruta y el workspace es una vista con modos derivados del estado del proyecto, no de un switcher.

---

## 3. Fidelity

**Alta fidelidad (hi-fi).** Colores, tipografías, tamaños, espaciados y copy son finales para esta ronda. Recrear pixel-perfect con las librerías del codebase.

Desktop-first, **viewport de diseño 1440 px**; alto de referencia 900 px (P9: 1080 px porque hace scroll). No hay breakpoints móviles en esta etapa: el producto es de escritorio.

⚙️ **Supuestos revisables** (marcados así en el brief; en el prototipo son *tweaks* conmutables):

| Supuesto | Default implementado | Alternativa mostrada |
|---|---|---|
| `chatMode` | `panel` — rail lateral persistente y colapsable | `overlay` — rail flotante sobre el visor |
| `bomApproval` | `global` — aprueba las 28 marcas de golpe | `parcial` — checkboxes por grupo en P6 |
| `costUpdate` | `vivo` — costos se recalculan tras cada edición (badge "recalculando…") | `tras-regenerar` — badge "pendiente de regenerar" |

No congelar estos tres en el código antes de que el equipo los cierre: dejarlos como configuración o feature flag.

---

## 4. Design tokens

### 4.1 Color

Paleta de **producto** (definida en el brief, coherente con los diagramas de arquitectura). Hereda de Kabunik la tipografía y la sobriedad, pero **no la paleta corporativa** — el producto tendrá su propio sistema de diseño en el futuro.

| Token | Hex | Uso |
|---|---|---|
| `--bg` | `#F4F6F9` | Fondo de aplicación |
| `--bg-viewer` | `#EDF1F5` | Fondo del visor 3D (claro) |
| `--surface` | `#FFFFFF` | Paneles, tarjetas, grillas |
| `--surface-sunken` | `#F8FAFC` | Cabeceras de grilla, tarjetas informativas |
| `--surface-alt` | `#EEF2F6` | Chips neutros, barras vacías |
| `--navy` | `#1E2A3A` | Barra superior, texto fuerte, botones neutros |
| `--navy-viewer` | `#16202D` | Visor sobre fondo oscuro (P4) |
| `--line` | `#E3E8EF` | Bordes 1px |
| `--line-soft` | `#F1F5F9` | Separadores de fila |
| `--input-border` | `#CBD5E1` | Bordes de campos y botones secundarios |
| `--text` | `#1E2A3A` | Texto principal |
| `--text-2` | `#475569` | Etiquetas, texto secundario fuerte |
| `--text-3` | `#64748B` | Texto de apoyo |
| `--text-4` | `#94A3B8` | Metadatos, placeholders, unidades |
| `--teal` | `#1F9E91` | Cálculo, datos verificados, acción positiva, selección |
| `--teal-700` | `#17786E` | Texto teal sobre lavado |
| `--teal-050` | `#E8F6F4` | Lavado teal (badges "tool ✓", fila seleccionada) |
| `--teal-200` | `#BFDDD9` | Barras de gráfico secundarias |
| `--teal-border` | `#CFE7E3` | Borde de tarjeta aplicada |
| `--violet` | `#6D5BD0` | **Exclusivo del agente/IA**: deltas, elementos afectados, "aplicando" |
| `--violet-700` | `#5847BD` | Texto violeta sobre lavado |
| `--violet-050` | `#EEEAFB` | Lavado violeta (badge "editado", "recalculando…") |
| `--violet-border` | `#DCD6F5` | Borde de tarjeta de delta |
| `--violet-row` | `#FAF9FE` | Fila de BOM editada por el agente |
| `--amber` | `#F5A623` | Calibración, alertas medias, retención en gráficos |
| `--amber-700` | `#9A6408` | Texto ámbar |
| `--amber-800` | `#7C5308` | Texto ámbar sobre lavado cálido |
| `--amber-050` | `#FEF7EA` / `#FEFBF5` | Lavado ámbar (banner, tarjeta de alerta media) |
| `--amber-100` | `#FBE7C2` | Pill "Re-abierto" |
| `--amber-border` | `#F5D7A6` / `#F5E4C4` | Bordes cálidos |
| `--red` | `#B0392F` | **Solo gates y alertas críticas** |
| `--red-050` | `#FDF7F6` | Fondo de tarjeta crítica |
| `--red-100` | `#FBE3E0` | Pill "Crítica" / "Sin aprobar" |
| `--red-border` | `#F0C9C4` | Borde crítico |
| `--brand-cyan` | `#6EC6D8` | Cian Kabunik — **solo** marca (cuadrado del logotipo) y detalles sobre ink |

**Reglas duras de color**
1. Rojo **nunca** fuera de gates y alertas críticas.
2. Violeta **solo** para lo que produce el agente (delta propuesto, highlight de elementos afectados, estado "aplicando", origen "editado").
3. Teal para números verificados por herramienta y para la selección bidireccional grilla ⇄ 3D.
4. Ámbar para calibración, umbrales y alertas medias.
5. Sin gradientes decorativos, sin texturas, sin patrones.

### 4.2 Tipografía

| Familia | Uso | Fuente |
|---|---|---|
| **Space Grotesk** 400/500/600/700 | Titulares y nombres de escenario (display) | Google Fonts |
| **DM Sans** 400/500/700 | Toda la UI | Google Fonts |
| **JetBrains Mono** 400/500/600 | **Todas las cifras en grillas, KPIs, tiempos, códigos, versiones** | Google Fonts |

Escala usada en los mocks:

| Rol | Tamaño / peso | Notas |
|---|---|---|
| Título de pantalla (wizard) | 26 px / 600 Space Grotesk | |
| Hero de progreso (P4) | 34 px / 600 Space Grotesk, `letter-spacing:-.4px`, `line-height:1.12` | |
| KPI grande (P9) | 34 px / 600 JetBrains Mono, `letter-spacing:-1px` | |
| KPI de cabecera (P6) | 22 px / 600 JetBrains Mono | |
| KPI de panel (P5·C) | 24 px / 600 JetBrains Mono | |
| Título de sección/tarjeta | 15–17 px / 600 Space Grotesk | |
| Cuerpo | 13–14 px / 400 DM Sans, `line-height:1.45–1.55` | |
| Celda de grilla | 12–13 px (mono para cifras) | |
| Etiqueta de campo | 12 px / 600, color `--text-2` | |
| Eyebrow / cabecera de columna | 10.5–12 px / 700, uppercase, `letter-spacing:.4–1px`, color `--text-4` | |
| Badge / pill | 10.5–11.5 px / 600–700 | |

Idioma y tono del copy: imperativo + beneficio, frases cortas, sentence case en botones y menús ("Aprobar BOM", "Generar escenarios", "Emitir oferta"). Cifras siempre concretas.

### 4.3 Espaciado, radios, sombras

- Grid base **4 px**. Paddings frecuentes: 8 / 12 / 14 / 16 / 20 / 22 / 26 px. `gap` en grillas de tarjetas: 12–16 px; en formularios: 18 px.
- Radios: **6** chips y badges pequeños · **8–9** botones e inputs de barra · **10–12** botones grandes, inputs, tarjetas internas · **14–16** tarjetas y paneles · **999px** pills.
- Bordes: 1 px `--line`. Selección de fila: `box-shadow: inset 3px 0 0 #1F9E91` (no cambiar el borde).
- Sombras: tarjetas en reposo **sin sombra**. Banner de gate `0 6px 20px rgba(16,24,40,.08)`; tarjeta de delta `0 6px 18px rgba(109,91,208,.12)`; escenario recomendado `0 6px 20px rgba(31,158,145,.13)`; rail en modo overlay `0 18px 44px rgba(16,24,40,.18)`; barra de navegación de mocks `0 12px 34px rgba(16,24,40,.3)`.
- Alturas fijas: barra superior **56 px** · barra inferior de totales **40 px** · pie del wizard **70 px** · fila de tabs **44 px** · botones de barra **34 px** · botones de formulario **42 px** · inputs **42 px**.
- Anchos fijos: rail izquierdo **320 px** · panel derecho **360 px** (**440 px** en modo escenarios) · panel lateral del wizard **300 px** · panel derecho de P6 **400 px** · columna de chat en P8 **560 px**.

---

## 5. Screens / Views

En todas las pantallas la **barra superior** es idéntica: 56 px de alto, fondo `--navy`, padding lateral 16 px, `gap:14px`; a la izquierda cuadrado `9×9 px` `--brand-cyan` con radio 2 px + wordmark **"Bid"** (Space Grotesk 14/600, blanco); breadcrumb con separador `/` en `rgba(255,255,255,.28)`; a la derecha acciones. **Decisión del 11-ago:** el producto se llama **"Bid"** en los mockups — se retira "Kabunik" del wordmark. Sigue siendo un nombre de trabajo, no la marca definitiva.

### P3 · Wizard de intake

**Propósito:** dar de alta el proyecto y sus insumos hasta lanzar "Generar solución".

**Layout:** columna izquierda 300 px (blanca, borde derecho `--line`, padding 28/24) con el stepper vertical; contenido a la derecha (padding 36/44, ancho de contenido máx. 760–820 px); pie de 70 px con "Atrás" (secundario) y CTA primario a la derecha.

- **Stepper:** cada paso es una fila `display:flex; gap:12px; padding:11px 12px; border-radius:11px`. Activo: fondo `#F1F8FA`, círculo 26 px `--teal` con número blanco mono. Completado: círculo `--teal-050` con `✓` en `--teal-700`. Pendiente: círculo `--surface-alt`, número `--text-4`. Título 13.5/500, subtítulo 11.5 `--text-4`.
- **Paso 1 — Datos generales:** grid 2 columnas, gap 18 px. Campos: Nombre del proyecto (`CNARCCS — Domo`), Cliente (`CNARCCS`), Tipo de estructura (`Domo / cubierta especial`), Destino (`Monterrey, N.L. · 312 km`). Inputs 42 px, borde `--input-border`, radio 10, texto 14. Los selects muestran `›` en `--text-4` a la derecha.
- **Paso 2 — Carga de archivos:** dropzone `border:2px dashed #A8C6D6; background:#F1F8FA; border-radius:16px; padding:34px`, centrada: cuadro 44 px blanco con `+` teal, título 15/600 "Arrastra planos aquí o selecciona archivos", subtítulo 12.5 `--text-3` "PDF · DXF · IFC · máx. 250 MB por archivo". Debajo, lista de archivos: fila blanca, borde `--line`, radio 12, padding 12/14, `gap:14px`; badge de extensión 34 px mono 10/600 sobre `--surface-alt`; nombre 13.5/500 + meta 11.5 `--text-4`; estado pill: **Leído** (`--teal-050`/`--teal-700`) o **Procesando** (`--amber-050`/`--amber-700`).
  Archivos de muestra: `CNARCCS_Domo_Planos_Arq.pdf` (48 páginas · 62 MB, Leído) · `Domo_Estructural_R3.dxf` (14.2 MB, Leído) · `CNARCCS_arq.ifc` (186 MB · IFC4, Procesando) · `Contrato_marco_CNARCCS.pdf` (22 páginas · 3.1 MB, Leído).
- **Paso 3 — Consideraciones:** grid 2 columnas (Alcance = "Suministro + montaje", Grado preferente = "A992 / A572 Gr.50", Tipo de conexión = "Empernada en obra", Acabado = "Galvanizado en caliente") + textarea de 120 px mínimo con las consideraciones libres del brief.
- **Pie:** "Borrador guardado automáticamente · 14:32" a la izquierda (12.5 `--text-4`). CTA: "Continuar" (`--navy`) en pasos 1–2, **"Generar solución"** (`--teal`) en el paso 3 → navega a P4.
- **Tarjeta informativa** al fondo del rail: "Calibración de planta activa — HH/ton, catálogo de perfiles y precios de material se toman de la calibración de METALITEC (v4, 02/07)".

### P4 · Progreso de generación

**Propósito:** hacer visible el trabajo del agente sin bloquear al usuario.

**Layout:** pantalla completa sobre `--navy`, texto blanco. Contenido centrado en grid `520px | 1fr`, `gap:80px`, máx. 1180 px.

- Eyebrow "Agente estructural · en curso" 11/700 uppercase en `--brand-cyan`; titular 34/600 Space Grotesk.
- **Lista de etapas**, cada una `padding:14px 0`, separador `rgba(255,255,255,.07)`: punto de 10 px (`--teal` completada · `--brand-cyan` con `pulseDot 1.2s` en curso · `rgba(255,255,255,.2)` pendiente), título 14.5 (600 activo/completado, 400 y `rgba(255,255,255,.4)` pendiente), detalle mono 12.5 `rgba(255,255,255,.45)`, tiempo a la derecha.
  1. Leyendo planos — 48 páginas · 312 vistas indexadas — 00:41 (completada)
  2. Construyendo geometría — 1,470 elementos · 28 marcas — 01:56 (completada)
  3. Generando BOM — Perfiles y grados asignados · 734 t — 00:22 (en curso)
  4. Validando — Invariantes y validaciones de percepción — — (pendiente)
- Barra de progreso 4 px, pista `rgba(255,255,255,.1)`, relleno 64 % `--brand-cyan`.
- **Panel derecho:** vista previa de geometría sobre `--navy-viewer`, radio 12, alto 360 px; debajo tres cifras mono: Elementos 1,470 · Tonelaje 734 t · Alertas 3 (en `--amber`).
- Copy de barra superior: "Puedes salir: te avisamos al terminar".

### P5 · Workspace (pantalla crítica — 3 modos de una misma vista)

**Zonas fijas** (de izquierda a derecha, alto total 900 px):

1. **Barra superior (56 px):** wordmark · `/` · "CNARCCS — Domo" (13.5/500 blanco) · chip de versión del modelo (mono 11, `--brand-cyan`, borde `rgba(110,198,216,.4)`, radio 6) · chip de estado de sesión (punto de 6 px con `pulseDot 1.6s`, fondo `rgba(255,255,255,.09)`) · a la derecha "Historial" (secundario, borde `rgba(255,255,255,.18)`) y **CTA primario contextual**.
2. **Rail izquierdo (320 px):** cabecera 44 px "Edición conversacional" (12/700 uppercase `--text-2`) con `‹` de colapso; cuerpo scrollable con padding 14 y `gap:12px`; composer fijo abajo (borde superior `--line`, padding 12): caja con borde `--input-border`, radio 12, mín. 66 px, placeholder "Describe el cambio…", atajo `⌘↵` mono 11 y botón de envío 30 px `--violet` con `↑`.
3. **Visor central (flex:1):** fondo `--bg-viewer`; wireframe isométrico del domo centrado; controles verticales arriba a la derecha (4 botones de 34 px, blanco 94 % con borde `--line`, radio 9: `⟳` orbitar, `＋` acercar, `−` alejar, `⤢` encuadrar); abajo a la izquierda dos chips mono 11 blancos: "Isométrica NE · 1:180" y "|——| 10 m"; etiqueta teal de elemento seleccionado sobre el modelo (mono 11, radio 6).
4. **Panel derecho (360 px; 440 px en modo C):** fila de tabs de 44 px — **BOM** (contador `1,470`) · **Alertas** (`3`) · **Propiedades**; tab activa en 600 con `box-shadow: inset 0 -2px 0 #1F9E91`, inactiva `--text-3`.
5. **Barra inferior (40 px):** totales vivos en JetBrains Mono 12, `gap:22px` — `Tonelaje 734.0 t` · `Elementos 1,470` · `Alertas 4` (en `--amber`) · `Costo estimado $49.5M` · badge de recálculo · a la derecha "Modelo v2 · sincronizado 14:52".

**Contenido de las tabs**

- **BOM (compacta):** cabecera con "BOM · 1,470 elementos" + pill de estado (`Sin aprobar` rojo en modo A, `Re-abierto` ámbar tras editar) y nota explicativa. Grilla de 5 columnas `64px 1fr 52px 44px 52px`, filas de 8 px de padding, separador `--line-soft`. Columnas: Marca (mono 11.5) · Perfil + grado (13/12.5, grado en `--text-4`) · Cant. · t (600) · Origen (badge `tool ✓` teal, o `editado` violeta si lo tocó el agente). Fila seleccionada: fondo `--teal-050` + `inset 3px 0 0 --teal`.
- **Alertas:** tarjetas con borde y lavado por severidad, pill de severidad arriba a la izquierda y código mono (`VAL-07`) a la derecha; título 13/600; cuerpo 12 `--text-3`; acciones "Ver en el modelo ›" (teal 11.5/600) e "Ignorar" (`--text-4`).
  1. **Crítica · VAL-07** — "Desperdicio proyectado 8.4% supera el umbral 8%" — concentrado en el anillo 3 por cortes de 4.9 m sobre barra de 12 m; sugiere empatar con el lote A-330.
  2. **Media · VAL-12** — "Perfil W21x132 sin stock — sustituto sugerido: armado equivalente".
  3. **Media · VAL-19** — "HH/ton habilitado fuera del rango histórico de la planta" — 17.4 HH/t vs banda 12.1–15.8.
- **Propiedades:** ficha del elemento seleccionado (Marca, Perfil, Grado, Longitud, kg/m, Peso unitario, Nivel, Conexión, Acabado) en filas `space-between` de 9/14 px con valor en mono 500; debajo, tarjeta **Trazabilidad**: "Peso calculado por `tool:weight_calc` sobre kg/m del catálogo METALITEC v4. Costo unitario de `tool:price_lookup`, lista 02/07" (los nombres de tool van en mono 11.5 sobre lavado teal).

**Modo A — Revisión del primer output** (`v1`, sesión "En revisión" ámbar, CTA "Aprobar BOM" en `--red`)
Banner de gate flotando sobre el visor (top 14, left/right 16): tarjeta blanca, borde `--red-border`, **borde izquierdo 4 px `--red`**, radio 12, sombra suave. Título 14/700 rojo "Revisa el BOM antes de cotizar"; subtítulo 12.5 `--text-3` "Ningún escenario ni oferta se calcula sobre un BOM sin aprobar. 3 alertas de validación abiertas."; botón "Aprobar BOM" 38 px `--red`. El rail muestra una tarjeta de bienvenida + 4 **sugerencias** clicables ("Cambia las columnas del nivel 3 a HSS", "Usa A572 Gr.50 en vigas principales", "Conexiones empernadas en vez de soldadas", "Reduce el desperdicio del anillo 3").

**Modo B — Edición conversacional en vivo** (`v2`, sesión "Aplicando cambios" violeta, CTA "Generar escenarios" teal, tab Alertas activa)
- Mensaje de usuario: burbuja `--navy`, blanco, radio `12 12 4 12`, alineada a la derecha, máx. 86 %.
- **Tarjeta de delta** (borde `--violet-border`, sombra violeta): eyebrow "Delta propuesto" + hora; título "24 columnas del N3: W12x40 → HSS 10x10x½"; tres métricas en grid (Tonelaje **−8.2 t**, Costo **−$14,300** en teal, Alertas **+1** en ámbar) sobre `--surface-sunken` con radio 9; nota "Requiere recálculo de 48 placas base y revisión de la alerta de esbeltez en el anillo 3."; acciones **Aplicar** (violeta) / **Cancelar** (secundario) y estado "Aplicando…" con punto pulsante.
- **Rechazo por guardarraíl** (borde `--red-border`, fondo `--red-050`): eyebrow "Rechazado por guardarraíl"; título "W21x132 no está en tu catálogo calibrado"; explicación; 3 alternativas en filas clicables con acción "Usar": *Armado 900x14 A572* (Ix equivalente 98% · +$2,140/t) · *W21x122 A992* (En catálogo · −4.8% capacidad) · *W24x117 A992* (En catálogo · +6 cm de peralte).
- **Visor:** las columnas afectadas se pintan en `--violet` con grosor 4 y animación `glowV 1.4s` (parpadeo de opacidad de trazo, sin recarga ni flash); nodos violeta de 4.5 px en su base. Chip inferior derecho violeta: "24 elementos afectados · reconstruyendo".
- **Barra inferior:** costo en violeta + badge "recalculando…" (violeta) o "pendiente de regenerar" (ámbar) según ⚙️ `costUpdate`.

**Modo C — Escenarios** (`v2`, sesión "BOM aprobado", CTA "Emitir oferta")
El panel derecho crece a 440 px y sustituye las tabs por: cabecera "Escenarios" + pill "Vigentes · v2" + selector `v1 | v2` (segmentado, activo `--navy`); tres tarjetas compactas de escenario (nombre, pill, plazo, `$/ton` en mono 24, total en teal, barra de progreso de 6 px, supuesto en 12); y una tarjeta "Flujo de caja · E2" con 9 barras (primera teal = anticipo, última ámbar = retención) y leyenda mono de 10 px. El visor permanece visible a la izquierda.

### P6 · Grilla BOM + gate

**Propósito:** revisar el takeoff a fondo y ejercer la compuerta de aprobación.

- Barra superior con breadcrumb `… / BOM`, chip `v2`, "Exportar CSV" (secundario) y **"Re-aprobar BOM"** en `--red`.
- **Cabecera de KPIs** (blanca, 16/20 px, separadores verticales de 34 px): Peso cobrable **734.0 t** · Elementos **1,470** · Marcas **28**; a la derecha, tarjeta ámbar (máx. 520 px, radio 12) con pill **Re-abierto** y el registro: "Aprobado por **J. Pérez · 12/07 14:05 · v1**. La edición «columnas N3 → HSS 10x10x½» invalidó la aprobación."
- **Grilla densa** de 8 columnas: `92px 168px 84px 70px 84px 82px 96px 108px` → Marca · Perfil · Grado · Cant. · Long. (m) · kg/m · Peso (t) · Origen. Cabecera pegajosa sobre `--surface-sunken`, 10.5/700 uppercase `--text-4`, padding 10/20. Filas de 11/20 px, separador `--line-soft`; cifras en mono 12, peso en 12.5/600; fila editada por el agente con fondo `--violet-row` y badge `editado`; fila seleccionada teal con `inset 3px 0 0`. Pie de subtotal: "Subtotal mostrado · 8 de 28 marcas" + 1,318 uds + 522.1 t.
- **Panel derecho (400 px):** miniatura del visor de 280 px con el elemento seleccionado resaltado y chip teal "C-115 · HSS 10x10x½ resaltado"; texto explicando la **selección bidireccional**; ficha de propiedades; y —solo con ⚙️ `bomApproval: parcial`— tarjeta "Aprobación por grupos" con checkboxes: Columnas (96) ✓ · Vigas principales (330) ✓ · Arriostres (440) · Placas y conexiones (604).

**Datos de la grilla (usar estos):**

| Marca | Perfil | Grado | Cant. | Long. (m) | kg/m | Peso (t) | Origen |
|---|---|---|---|---|---|---|---|
| C-101 | W12x40 | A992 | 96 | 5.80 | 59.5 | 33.1 | tool ✓ |
| C-115 | HSS 10x10x½ | A500 | 24 | 4.90 | 92.3 | 10.9 | editado |
| V-201 | W18x50 | A992 | 210 | 9.20 | 74.4 | 143.7 | tool ✓ |
| V-230 | W21x62 | A992 | 120 | 7.60 | 92.3 | 84.2 | tool ✓ |
| A-310 | HSS 8x8x⅜ | A500 | 180 | 4.40 | 45.0 | 35.6 | tool ✓ |
| A-330 | HSS 6x6x¼ | A500 | 260 | 3.20 | 27.7 | 23.1 | tool ✓ |
| AR-410 | Armado 900x14 | A572 | 64 | 12.00 | 168.0 | 129.0 | tool ✓ |
| PL-05 | PL ½" | A36 | 340 | — | — | 12.4 | tool ✓ |

### P8 · Chat de edición + historial

**Propósito:** detalle del rail izquierdo y trazabilidad completa de cambios.

- Split fijo: **columna de conversación 560 px** (blanca, borde derecho) + **historial** a la derecha.
- Conversación: mismas tarjetas del modo B más una tercera entrada **aplicada** (borde `--teal-border`): "186 conexiones del anillo perimetral pasan a empernadas — −214 HH de taller, +$3,900 en tornillería. El BOM aprobado quedó re-abierto: requiere nueva aprobación antes de cotizar." con acciones "Ver diff ›" y "Revertir".
- Composer ampliado con chips de contexto: `@marca`, `#nivel`, `catálogo`.
- **Timeline:** grid `104px 24px 1fr`; hora mono 11.5 a la izquierda; línea vertical de 1 px `--line` con punto de 11 px (borde 2 px del color de fondo) coloreado por estado; tarjeta a la derecha (máx. 640 px) con pill de estado (blanco sobre color), versión mono, autor, título 13.5/600, delta 12.5 `--text-3` y acciones "Ver diff ›" / "Revertir" (deshabilitada en gris `#CBD5E1` cuando no aplica).

| Hora | Estado | Ver. | Autor | Entrada | Delta |
|---|---|---|---|---|---|
| 14:58 | Aplicado (teal) | v2 | J. Pérez | Conexiones empernadas en el anillo perimetral | 186 conexiones · −214 HH · +$3,900 tornillería |
| 14:54 | Rechazado (rojo) | — | Guardarraíl | W21x132 en vigas de borde | Perfil fuera del catálogo calibrado METALITEC v4 |
| 14:51 | Aplicando (violeta) | v2 | J. Pérez | Columnas N3 → HSS 10x10x½ | 24 elementos · −8.2 t · −$14,300 |
| 14:22 | Aplicado (teal) | v1 | J. Pérez | Grado A572 Gr.50 en vigas principales | 330 elementos · +$6,100 · −0 t |
| 14:05 | Gate (navy) | v1 | J. Pérez | BOM aprobado | Aprobación global · 1,470 elementos · 734 t |
| 13:48 | Generado (teal) | v1 | Agente | Primer output del modelo | 1,470 elementos · 28 marcas · 3 alertas |

Nota de cabecera: "Cada entrada es revertible; revertir crea una nueva versión".

### P9 · Escenarios + flujo de caja

**Propósito:** comparar escenarios, entender el impacto financiero y decidir qué entra en la oferta.

Alto 1080 px con scroll; padding 24/26; `gap:20px`.

- Cabecera: "Comparación de escenarios" (20/600 Space Grotesk) + pill "Vigentes · generados sobre v2 · 14:58" + a la derecha "Base: 734 t cobrables · calibración METALITEC v4". En la barra superior, selector `Modelo v1 | v2` y CTA teal "Emitir oferta".
- **Tres tarjetas** en grid `repeat(3,1fr)`, gap 16, radio 16, padding 20. La recomendada (E2) lleva borde teal y sombra teal. Cada una: nombre (17/600) + pill de estado + botón de selección a la derecha ("En la oferta" teal sólido en E2, "Incluir" secundario en las otras); `$/ton` en mono 34; fila de Total / Plazo / Margen separada por borde superior; 3 bullets con guion `—`.

| | E1 Conservador | E2 Optimizado (recomendado) | E3 Agresivo (requiere aprobación) |
|---|---|---|---|
| $/ton | 73,200 | 67,500 | 61,800 |
| Total | $53.7M | $49.5M | $45.4M |
| Plazo | 9 meses | 8 meses | 7 meses |
| Margen | 22% | 19% | 15% |
| Supuestos | Sin ingeniería de valor · compra spot · subcontrato de montaje completo | Empates y retal (−2.1% desperdicio) · conexiones empernadas en perímetro · compra consolidada en 2 órdenes | Armados equivalentes en vigas de borde · montaje en dos frentes · retención negociada a 3% |

- **Flujo de caja por hitos · E2** (tarjeta izquierda del grid `1fr 460px`): 9 barras de 190 px de alto máximo con etiqueta mono encima — M0 **14.9M** (teal, anticipo 30%), M1 3.2M, M2 5.1M, M3 6.4M, M4 7.8M, M5 6.9M, M6 4.6M, M7 2.5M (todas `--teal-200`, avance contra entrega), M8+ **2.5M** (ámbar, liberación de retención 5%). Leyenda con cuadrados de 9 px.
- **Ingeniería de valor cuantificada** (tarjeta derecha): filas con casilla de 22 px (marcada teal con `✓` / vacía con borde), título 13, nota 11.5 y ahorro en mono teal —
  Armados equivalentes en vigas de borde (18 elementos · Ix 98%) **$3.10M** ✓ · Empates y aprovechamiento de retal (8.4% → 6.3%) **$2.41M** ✓ · Conexiones empernadas en perímetro (−214 HH) **$1.68M** ✓ · Reducción de grado en arriostres secundarios (pendiente de validación estructural) **$1.18M**. Pie: "Ahorro aplicado en E3 — **$8.37M**".
- **Comparación entre versiones** (tarjeta a lo ancho, grid `1.4fr 1fr 1fr 1fr`): Métrica | v1 · 12/07 | v2 · hoy | Δ (verde teal si mejora, rojo si empeora, gris si igual).

| Métrica | v1 | v2 | Δ |
|---|---|---|---|
| Tonelaje cobrable | 742.2 t | 734.0 t | −8.2 t (mejora) |
| Elementos | 1,470 | 1,470 | 0 |
| Costo E2 | $50.9M | $49.5M | −$1.4M (mejora) |
| Plazo E2 | 8.5 meses | 8 meses | −0.5 m (mejora) |
| Alertas abiertas | 2 | 3 | +1 (empeora, rojo) |

---

## 6. Interactions & behavior

- **Navegación entre pantallas del prototipo:** barra flotante inferior (`position:fixed`, `bottom:18px`, centrada, fondo `rgba(30,42,58,.96)`, blur 8 px, radio 12, padding 6). **Es andamiaje del mock: no se implementa en producción.**
- **Wizard:** "Continuar" avanza de paso; en el paso 3 el CTA cambia a "Generar solución" y lanza P4. "Atrás" retrocede sin perder datos (borrador autoguardado).
- **Generación (P4):** etapas en streaming; el usuario puede salir y volver (el job sigue). Al terminar → Workspace con el primer output.
- **Selección bidireccional:** clic en fila de BOM → aísla y resalta en 3D (teal) y actualiza la tab Propiedades; clic en elemento del visor → desplaza la grilla a esa marca. Es el comportamiento central de P6 y del panel derecho de P5.
- **Edición conversacional:** `interpretando → delta propuesto (Aplicar/Cancelar) → aplicando → aplicado`, o `rechazado por guardarraíl` con alternativas del catálogo. Durante "aplicando" el visor **no** se recarga: solo cambian los elementos afectados (highlight violeta con pulso) y el resto sigue navegable.
- **Orden del flujo — no es preferencia, lo impone el motor.** Las pantallas se recorren por bloques:
  **B1** modelo y peso (P5·A, P6, P5·B, P8) → **B2** planta y programa (P14, P16, P15) → **B3** costo y
  precio (P5·C, P9) → **B4** emisión (P13, P10) → **B5** cierre (P17). La razón es que §4.3 del motor
  calcula el APU con el **pico de S11**, y S11 se calcula sobre la planta cargada: **no hay precio
  antes de resolver la planta.** Detalle en `docs/00-Fundamentos/FLUJO_v1.md`.
- **Gate de BOM:** sin aprobación no hay bloque de planta, ni costeo, ni opciones. Aprobar registra autor + fecha + versión. Cualquier edición posterior que altere el takeoff pasa el BOM a **re-abierto**, marca las opciones como **desactualizadas** y **también invalida la decisión de programa** — lotes, S11, S12 y APU se calcularon sobre un peso que ya no es el vigente.
- **Costo antes de B3:** mientras el bloque de costo y precio no haya corrido, el total de la barra inferior va en `—` con badge «pendiente de planta y costeo». No se muestra una cifra que no existe.
- **Versionado:** cada aplicación de delta incrementa la versión del modelo (v1 → v2…). Revertir crea una versión nueva, no borra historial.
- **Animaciones** (mínimas y funcionales): `pulseDot` 1.2–1.6 s (opacidad .35 → 1) en puntos de estado; `glowV` 1.4 s en los trazos de elementos afectados; `transition .15s ease` en color de fondo para hover. Sin bounce, sin parallax, sin animaciones de entrada.
- **Hover:** superficies neutras → `#F1F5F9`; botón rojo → `#8F2E26`; teal → `#17786E`; filas de grilla → fondo `--surface-sunken` (sin cambiar el borde).
- **Foco:** anillo de 2 px en el color de acción correspondiente (teal por defecto).
- **Vacíos y errores** (no dibujados, especificar en implementación): error de percepción de planos → mensaje claro + reintento, manteniendo los archivos ya subidos.

## 7. State management

Estado mínimo por proyecto:

```
project: { id, name, client, structureType, destination, status }
   status: 'borrador' | 'procesando' | 'listo_para_revision' | 'oferta_emitida' | 'error_percepcion'
model:   { version, elements, tonnage, marks, geometryUrl }
bom:     { rows[], approval: { state: 'sin_aprobar'|'aprobado'|'reabierto', by, at, modelVersion } }
alerts:  [{ code, severity: 'critica'|'media'|'baja', title, body, elementIds[] }]
edits:   [{ id, at, author, state: 'interpretando'|'aplicando'|'aplicado'|'rechazado', title, delta, modelVersion }]
scenarios: { generatedFromVersion, state: 'vigentes'|'desactualizados', items[] }
viewer:  { selectedElementIds[], highlightedElementIds[], camera }
ui:      { rightTab: 'bom'|'alertas'|'propiedades', railCollapsed, mode: 'revision'|'edicion'|'escenarios' }
```

- `ui.mode` **se deriva** del estado del proyecto (BOM sin aprobar → revisión; edición en curso → edición; escenarios vigentes → escenarios), no es un selector manual.
- Los deltas llegan por streaming (SSE/WebSocket) y actualizan `model`, `bom`, `alerts` y totales de forma incremental.
- Los costos se recalculan según ⚙️ `costUpdate`.

## 8. Assets

- **Sin imágenes ni iconos de librería.** Los controles del visor y los indicadores usan caracteres unicode: `⟳ ＋ − ⤢ › — ✓ ↑ ‹ +`. Si el codebase necesita un set de iconos, usar **Lucide** con `stroke-width:1.6` (criterio Kabunik) y sustituir los unicode uno a uno.
- **Logotipo:** en los mocks es un cuadrado cian de 9 px + wordmark de texto, como **marcador**. Sustituir por el logotipo real de Kabunik (`assets/logo-kabunik-dark.png` sobre la barra navy) o por la marca definitiva del producto cuando exista. Nunca redibujar el logo.
- **Visor 3D:** el domo del mock es un wireframe SVG generado por código (`dome()` en la clase de lógica) — **placeholder**. En producción se sustituye por el visor real (three.js + web-ifc). Lo que sí debe conservarse: proyección isométrica, malla en gris azulado, elemento seleccionado en teal grueso, elementos afectados por el agente en violeta con pulso, y los chips de encuadre/escala.
- **Fuentes:** Google Fonts (Space Grotesk, DM Sans, JetBrains Mono). Si se requieren binarios autohospedados, pedir los `.woff2`.

## 9. Relación con el sistema Kabunik

Este es un **producto nuevo** que hereda de Kabunik la tipografía (Space Grotesk + DM Sans), el registro de copy en español, la sobriedad (fondos planos, sin gradientes, sin emoji) y el criterio de bordes/radios/sombras — pero **usa su propia paleta funcional** (navy/teal/violeta/ámbar/rojo), porque el cian corporativo no funciona como color interactivo ni cubre los estados semánticos que este producto necesita. El cian `#6EC6D8` queda reservado a la marca.

Cuando el producto tenga su propio sistema de diseño, este documento es el punto de partida de sus tokens.

## 10. Files

```
design_handoff_plataforma_presupuestos/
├─ README.md                              ← este documento (autosuficiente)
├─ tokens.css                             ← tokens listos para pegar en el codebase
├─ design/
│  ├─ Plataforma Presupuestos.dc.html     ← prototipo hi-fi (todas las pantallas)
│  └─ support.js                          ← runtime del prototipo (no portar)
└─ insumos/
   ├─ BRIEF_Claude_Design_Mockups.md      ← brief original con los defaults ⚙️
   └─ UserFlows_y_Handoff_Diseno.md       ← user flows, personas, inventario de pantallas
```

En el prototipo: la plantilla HTML contiene la maquetación de cada pantalla; la clase de lógica (`class Component`) contiene los datos de muestra, los estilos calculados por estado y el generador del wireframe del domo.

## 11. Qué NO hacer

- No layouts chat-first ni conversación a pantalla completa: el rail nunca supera 320 px y es colapsable.
- No inventar funcionalidades fuera de los flows (nada de ERP: producción en vivo, facturación, inventario contable).
- No cambiar los defaults ⚙️ sin decisión del equipo.
- No usar rojo fuera de gates y alertas críticas, ni violeta fuera de acciones del agente.
- No portar el switcher de pantallas ni la clase única del prototipo a producción.
- **No dar control de silencio a las alertas no suprimibles.** `advertencia_bajo_equilibrio`,
  `alerta_ocupacion` (≥100%), `alerta_interferencia_familias` y `advertencia_comunicacion_cliente`
  no llevan «Ignorar» ni «Ocultar». «Aceptar el impacto documentado» sí es válido: es una decisión
  registrada, no una supresión.
- **No tratar el gate de precios de MP (P13) como un aviso.** Bloquea la emisión igual que el gate
  de BOM y lleva el mismo peso visual (rojo). Los otros tres momentos de decisión —programa,
  interferencia, recalibración— exigen decisión registrada pero **no** bloquean el cálculo: van en
  ámbar. Esa distinción es la mitigación de la fatiga de alertas.
- **No mostrar cifras de costo antes de que exista el motor.** En F0 el visor entrega geometría, BOM
  y alertas de percepción; el total de la barra inferior va en estado «pendiente de motor», no con un
  número.
