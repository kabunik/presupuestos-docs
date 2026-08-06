# BRIEF PARA CLAUDE DESIGN — Mockups de la Plataforma de Presupuestos
### Complemento del documento "UserFlows_y_Handoff_Diseno.md" · listo para producir mocks · v1

**Instrucción general:** produce mockups de alta fidelidad de las 5 pantallas listadas abajo, en este orden de prioridad. Usa los user flows del documento adjunto como fuente de verdad del comportamiento. Este brief resuelve las decisiones abiertas con defaults (marcados ⚙️ = supuesto revisable, no lo cambies por tu cuenta) y aporta datos de muestra reales. Desktop-first, viewport 1440px.

---

## 1. Identidad visual

- **Tono:** herramienta profesional de ingeniería. Densidad de información bien jerarquizada, no minimalismo vacío ni estética de chatbot. Referencias: Linear, herramientas CAD/BIM modernas.
- **Paleta:** fondo claro neutro (#F4F6F9) con superficie blanca; navy #1E2A3A para estructura y texto fuerte; teal #1F9E91 para cálculo/datos verificados; violeta #6D5BD0 exclusivo para acciones del agente/IA; ámbar #F5A623 para calibración y acentos; rojo #B0392F reservado a gates y alertas críticas. (Coherente con los diagramas de arquitectura ya socializados.)
- **Tipografía:** sans geométrica-humanista (Inter o similar); monoespaciada para cifras en grillas.
- **Regla dura:** ninguna pantalla debe leerse como "un chat con extras". El chat ocupa como máximo ~320px laterales y es colapsable.

## 2. Pantallas a producir (orden de prioridad)

### P5 · Workspace (CRÍTICA — producir en sus 3 modos)
**Layout de zonas (fijo):**
- **Centro (≥60% del ancho):** visor 3D con controles de navegación estándar (orbitar/zoom/pan), selección de elemento con highlight.
- **Derecha (panel contextual ~360px, con tabs):** BOM · Alertas · Propiedades del elemento seleccionado.
- **Izquierda (rail colapsable ~320px):** chat de edición + historial/timeline de cambios. ⚙️ *Default: panel lateral persistente colapsable (no overlay).*
- **Barra superior:** nombre del proyecto, versión del modelo (v1, v2…), estado de la sesión, botón primario contextual ("Aprobar BOM" / "Generar escenarios" / "Emitir oferta").
- **Barra inferior fina:** totales vivos — tonelaje, nº elementos, nº alertas activas, costo estimado. ⚙️ *Default: los costos SÍ se actualizan en vivo tras cada edición (badge "recalculando…" durante el proceso).*

**Modo A — Revisión del primer output:** visor con el modelo recién generado; tab BOM activa; banner del gate: "Revisa el BOM antes de cotizar" con CTA "Aprobar BOM" (rojo/prominente, momento deliberado). ⚙️ *Default: aprobación global del BOM, no parcial.*
**Modo B — Edición en vivo:** mensaje del usuario en el chat; tarjeta del agente con el **resumen del delta** ("Voy a cambiar 24 columnas del N3 de W12x40 a HSS 10x10x½ — impacto: −8.2 ton, −$14,300") con botones Aplicar/Cancelar; elementos afectados resaltados en violeta en el visor; alertas y totales actualizándose. Incluir un ejemplo de **rechazo por guardarraíl**: "Ese perfil no está en tu catálogo calibrado; alternativas: …".
**Modo C — Escenarios:** el panel derecho se expande mostrando la comparativa (ver P9 embebida) manteniendo el visor visible a la izquierda.

### P3 · Wizard de intake (3 pasos)
Stepper visible; paso 2 con dropzone para PDF/DXF/IFC + contrato; paso 3 con consideraciones (campos + texto libre); CTA final "Generar solución". Añadir la pantalla de progreso (streaming por etapas: Leyendo planos → Geometría → BOM → Validando) con actividad del agente visible.

### P6 · Grilla BOM + gate
Grilla densa editable: Marca | Perfil | Grado | Cant. | Long. (m) | kg/m | Peso total (t) | Origen (badge "tool ✓"). Selección de fila ⇄ highlight en 3D (mostrar la conexión visual). Cabecera con totales y botón "Aprobar BOM" + registro "Aprobado por J. Pérez · 12/07 · v1". Estado "re-abierto" cuando una edición posterior invalida la aprobación.

### P8 · Chat de edición + historial
El rail izquierdo en detalle: input, mensaje→delta→aplicado como conversación estructurada (tarjetas, no burbujas informales); timeline de cambios con revertir; estados: interpretando / aplicando / aplicado / rechazado.

### P9 · Escenarios + flujo de caja
⚙️ *Default: 3 escenarios fijos.* Tarjetas comparativas con: nombre, $/ton, total, plazo, supuestos clave, badge "vigente/desactualizado". Debajo: gráfico de flujo de caja por hitos (anticipo, avances, retención) y checklist de ingeniería de valor cuantificado. Selector de versión del modelo (v1 vs v2) para comparar. ⚙️ *Default: la comparación v1 vs v2 SÍ entra en los mocks (versión simple: dos columnas).*

## 3. Datos de muestra (usar estos, no lorem ipsum)

**Proyecto:** "CNARCCS — Domo" · Cliente: CNARCCS · Estructura: domo/cubierta especial · **734 t cobrables · 1,470 elementos**.
**BOM (extracto):** W12x40 A992 (columnas, 96 uds), W18x50 A992 (vigas principales, 210 uds), HSS 8x8x⅜ A500 (arriostres, 180 uds), PL ½" A36 (placas, 340 uds), perfiles armados según fórmula kg/m.
**Escenarios:** E1 Conservador $73,200/ton · E2 Optimizado $67,500/ton · E3 Agresivo (ing. de valor completa) $61,800/ton. Plazos 9 / 8 / 7 meses.
**Alertas de ejemplo:** "Desperdicio proyectado 8.4% supera umbral 8%" (crítica) · "Perfil W21x132 sin stock — sustituto sugerido: armado equivalente" (media) · "HH/ton habilitado fuera de rango histórico de la planta" (media).
**Flujo de caja:** anticipo 30%, avances mensuales contra entregas, retención 5%.
**Tenant:** METALITEC · capacidad 200 t/mes.

## 4. Qué NO hacer

- No layouts chat-first ni pantalla completa de conversación.
- No inventar funcionalidades fuera de los flows (nada de ERP: producción en vivo, facturación, inventario contable).
- No cambiar los defaults ⚙️ — si un flujo resulta ambiguo, elegir la opción más simple y anotarla como supuesto.
- No usar rojo fuera de gates/alertas críticas.

## 5. Entregable

Mocks hi-fi navegables (P5 en sus 3 modos + P3, P6, P8, P9), consistentes entre sí (mismo sistema visual), listos para presentar en reunión de equipo y recoger feedback sobre los supuestos ⚙️.
