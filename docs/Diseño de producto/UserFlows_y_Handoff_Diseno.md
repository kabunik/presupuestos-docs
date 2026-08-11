# User Flows y Handoff de Diseño — Mockups de la Plataforma
### Plataforma de Presupuestos de Estructuras Metálicas · Kabunik × dcode · v1 (borrador para discusión)

**Propósito del documento:** (Parte 1) definir los user flows básicos derivados de la arquitectura y visión acordadas, como base de discusión en la próxima reunión; (Parte 2) servir de brief de handoff para design (+ Claude Code) en la generación de wireframes y mockups una vez recogido el feedback.

---

# PARTE 1 · USER FLOWS

## Principio rector (recordatorio de la visión)

La plataforma **no se percibe como un chat**: es un taller de ingeniería. El **visor 3D es el protagonista**; el chat es el *volante del modelo*. El usuario ve una solución estructural, la edita conversando, y observa la reconstrucción en tiempo real. Referencia de mercado: Steel Genie by ALLPLAN llega hasta "planos → modelo 3D → takeoff editable → export"; nuestro flujo continúa donde ellos se detienen (costo calibrado, escenarios, flujo de caja, ingeniería de valor, edición conversacional en vivo).

## Personas (mínimas para los flows)

- **Estimador/Ingeniero de ofertas** (usuario principal): crea proyectos, revisa el modelo, edita, genera escenarios, emite la oferta.
- **Admin del fabricante** (tenant): configura la calibración de planta, gestiona usuarios de su organización.
- **Admin Kabunik** (interno): licencias, organizaciones, monitoreo. *(Flujo secundario, no prioritario para mockups.)*

---

## FLOW 0 · Onboarding y calibración del fabricante *(una vez por tenant · previo a operar)*

> Convierte el know-how en configuración privada del tenant. Sin calibración completa, la organización no puede generar ofertas.

1. Admin del fabricante recibe invitación → crea organización → acepta términos.
2. **Wizard de calibración** (guardable por pasos):
   a. Layout de planta y equipos.
   b. Eficiencias (HH/ton por proceso) y capacidad instalada.
   c. Precios de materiales y catálogo de perfiles disponibles.
   d. Umbrales de alertas (defaults sugeridos, ajustables).
3. Revisión y confirmación → estado del tenant: **"Calibrado — listo para operar"**.
4. Invita usuarios (estimadores) a la organización.

**Estados clave:** incompleto (bloquea creación de proyectos) · calibrado · desactualizado (sugerir revisión periódica).

---

## FLOW 1 · Alta de proyecto e intake

1. Dashboard de proyectos → **"Nuevo proyecto"**.
2. **Wizard de intake (3 pasos):**
   a. Datos generales (nombre, cliente, tipo de estructura, ubicación/destino para transporte).
   b. **Carga de archivos:** planos PDF/DXF (+ IFC opcional), contrato PDF (opcional).
   c. Opciones y consideraciones (las del sistema del consultor: alcance, condiciones particulares).
3. **"Generar solución"** → pantalla de progreso con streaming del agente (etapas visibles: leyendo planos → construyendo geometría → BOM → validando).
4. Al terminar → entra al **Workspace** con el primer output.

**Estados clave:** borrador · procesando (con progreso en vivo y posibilidad de salir/volver) · listo para revisión · error de percepción (mensaje claro + reintento).

---

## FLOW 2 · Primer output: revisión del modelo y compuerta de BOM *(human gate)*

> El momento "wow": la solución estructural aparece en 3D. Pero también la compuerta de calidad: nada se cotiza sin BOM aprobado.

1. **Workspace** con layout de tres zonas: **visor 3D central** (navegable: orbitar, zoom, seleccionar elemento) · panel derecho contextual · barra inferior/lateral de estado.
2. Panel de **BOM en grilla editable**: perfiles, grados, longitudes, pesos; selección bidireccional (clic en fila ⇄ resalta en 3D).
3. Panel de **alertas de validación**: cada alerta enlaza al elemento afectado en el visor.
4. Usuario corrige lo necesario (editar fila del BOM o pedir cambio por chat) → **"Aprobar BOM"**.
5. La aprobación dispara el cómputo de costos → habilita escenarios.

**Estados clave:** en revisión · BOM aprobado (versionado: quién y cuándo) · re-abierto (si una edición posterior invalida la aprobación).

---

## FLOW 3 · Edición conversacional con reconstrucción en tiempo real *(el diferenciador)*

1. Usuario abre el **chat lateral** (siempre accesible en el workspace).
2. Escribe la petición: *"cambia las columnas del nivel 3 a HSS"*, *"usa A572 Gr.50 en vigas principales"*, *"conexiones empernadas en vez de soldadas"*.
3. El agente interpreta → muestra **qué va a cambiar** (resumen del delta) → aplica.
4. El **visor se reconstruye en vivo**; los elementos modificados se resaltan; BOM, alertas y costos se actualizan de forma incremental.
5. Cada edición queda en un **historial de cambios** (timeline) con posibilidad de revertir.
6. Si el cambio invalida el BOM aprobado → el sistema lo señala y pide re-aprobación (vuelve al gate del Flow 2).

**Estados clave:** interpretando · aplicando (elementos afectados en highlight) · aplicado · rechazado por guardarraíl (con explicación: p. ej. "ese perfil no está disponible en tu catálogo").

---

## FLOW 4 · Escenarios y validación iterativa

1. Con BOM aprobado → **"Generar escenarios"**.
2. Vista de **tarjetas comparativas** (3 escenarios): precio/ton, total, plazos, supuestos de cada uno.
3. Detalle por escenario: **flujo de caja** graficado (hitos, anticipo, retención) · **checklist de ingeniería de valor** cuantificado.
4. El usuario puede **volver a editar** (Flow 3) y regenerar escenarios para comparar variantes — comparación entre versiones del modelo (v1 vs v2: tonelaje, costo, plazo).
5. Selecciona el escenario (o escenarios) a incluir en la oferta.

**Estados clave:** escenarios vigentes · desactualizados (el modelo cambió después de generarlos — badge visible).

---

## FLOW 5 · Emisión de la oferta

1. **Preview de la oferta** (documento de 12 secciones) con los escenarios seleccionados.
2. Descarga: **oferta .docx** + **modelo IFC4** + programa de fabricación.
3. Estado del proyecto → **"Oferta emitida"** (fecha, versión del modelo asociada — trazabilidad).
4. *(Futuro F3)* Exportación del proyecto planificado a **Strumis** · registro de **cierre real** (lazo cotizado-vs-ejecutado).

---

## Inventario de pantallas (derivado de los flows)

| # | Pantalla | Flow | Prioridad mockup |
|---|---|---|---|
| P1 | Login / selección de organización | — | Baja |
| P2 | Dashboard de proyectos | 1 | Media |
| P3 | Wizard de intake (3 pasos) | 1 | **Alta** |
| P4 | Progreso de generación (streaming) | 1 | Media |
| P5 | **Workspace: visor 3D + paneles** | 2–4 | **Crítica** |
| P6 | Panel BOM (grilla editable) + gate de aprobación | 2 | **Alta** |
| P7 | Panel de alertas de validación | 2 | Alta |
| P8 | Chat de edición + historial de cambios | 3 | **Alta** |
| P9 | Escenarios comparativos + flujo de caja | 4 | **Alta** |
| P10 | Preview / emisión de oferta | 5 | Media |
| P11 | Wizard de calibración del fabricante | 0 | Media |
| P12 | Admin de organización y usuarios | 0 | Baja |

> **La pantalla P5 (Workspace) es el corazón del producto** y donde design debe invertir más: es una sola pantalla con estados múltiples (revisión, edición en vivo, escenarios), no varias pantallas separadas.

---

## Preguntas abiertas para la reunión

1. ¿El chat de edición es panel lateral persistente o overlay invocable? (afecta el layout del workspace)
2. ¿La aprobación del BOM es global o puede ser parcial (por grupos de elementos)?
3. ¿Cuántos escenarios simultáneos comparamos en pantalla — fijo en 3 o configurable?
4. ¿La comparación entre versiones del modelo (v1 vs v2) entra en primeros mockups o se difiere?
5. ¿El estimador ve costos en tiempo real durante la edición (Flow 3) o solo tras regenerar escenarios? (implicación directa de latencia para dcode)
6. ¿Onboarding de calibración: lo hace el fabricante solo o guiado por Daniel/Kabunik (modo asistido)?

---

# PARTE 2 · BRIEF DE HANDOFF PARA DESIGN (+ Claude Code para mockups)

## Objetivo del encargo

Producir **wireframes de baja fidelidad** de las pantallas P3, P5, P6, P8 y P9 (las de prioridad alta/crítica) para validar los flows con el equipo; después, **mockups de alta fidelidad** del Workspace (P5) con sus estados. Los mockups pueden prototiparse como HTML/React estático con Claude Code para hacerlos navegables.

## Principios de diseño (no negociables)

1. **Plataforma, no chat.** El visor 3D es el protagonista visual; el chat es una superficie secundaria de apoyo. Ninguna pantalla debe leerse como "un chatbot con extras".
2. **El estado del sistema siempre visible.** Jobs en curso, streaming del agente, elementos afectados por una edición, alertas activas, versión del modelo: el usuario nunca debe preguntarse "¿qué está pasando?".
3. **Trazabilidad como estética.** Todo número visible puede explicarse (viene de una tool); las alertas enlazan a elementos; las ofertas referencian versión del modelo. El diseño debe hacer visible esa seriedad de ingeniería.
4. **Compuertas explícitas.** El gate de aprobación de BOM es un momento de diseño deliberado (no un botón perdido): comunica que el humano manda.
5. **Densidad profesional.** Usuarios ingenieros: prefieren densidad de información bien jerarquizada (grillas, paneles) sobre minimalismo vacío. Referencias: herramientas CAD/BIM modernas, Linear, y el patrón de Steel Genie (subir → escanear → revisar en 3D/tabla → exportar) como base a superar.

## Restricciones técnicas a respetar en el diseño

- El visor 3D renderiza IFC/geometría vía librería web (IFC.js/Three.js); prever controles estándar de navegación 3D y selección de elementos con panel contextual.
- La reconstrucción en vivo llega como **deltas** (streaming): diseñar la transición visual (highlight de lo cambiado, no "flash" de recarga completa).
- La edición conversacional puede tardar segundos: diseñar los estados intermedios (interpretando → aplicando → aplicado) sin bloquear la navegación del visor.
- Multitenancy: el theming debe soportar identidad neutra de plataforma (no personalización por tenant en esta fase).

## Entregables solicitados a design

1. **Wireframes low-fi** de P3, P5, P6, P8, P9 (con los estados clave de cada flow).
2. **Mockup hi-fi del Workspace (P5)** en sus tres modos: revisión de primer output · edición conversacional en vivo · comparación de escenarios.
3. **Mini design system inicial:** tipografía, paleta, componentes base (paneles, grillas, tarjetas de escenario, badges de alerta/estado).
4. *(Con Claude Code)* **Prototipo navegable** HTML/React de los wireframes validados, para la siguiente ronda de feedback.

## Insumos disponibles

- Documento de segmentación de tareas v2 (arquitectura y responsabilidades).
- Diagramas de arquitectura (3 capas y agéntica) en SVG.
- Diagrama de user flows (SVG adjunto a este documento).
- Caso real de referencia: proyecto CNARCCS Domo (oferta de 3 escenarios) como contenido realista para poblar mockups.

---

*Documento de trabajo · v1 para discusión en reunión de equipo · Kabunik 2026. Tras el feedback, actualizar la Parte 1 (flows y preguntas resueltas) y activar la Parte 2 como encargo formal a design.*
