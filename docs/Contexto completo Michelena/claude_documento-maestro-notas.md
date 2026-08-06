# Sistema DM — estado del paquete (v1.7, 2026-08-01)

**Documento rector: `claude/instrucciones-sistema-dm.md` (v1.7)** — proyecto base, **16 invariantes**, regla de emisión (leyenda de alcance del precio), reglas editoriales, golden test completo con parámetros registrados. Este archivo es el registro de entregables.

## Cambios v1.6 → v1.7 (aprobados por dirección)
1. **Invariante 3 ampliado**: subproyectos por familia cuando la complejidad difiere ≥1 salto del eje; `alerta_interferencia_familias` obligatoria cuando una familia de HH altas satura habilitado y reduce productividad de familias de mayor rentabilidad — reporta familias, tonelajes, HH/ton, rentabilidad relativa e impacto, para decisión humana.
2. **Invariante 6 ampliado**: % de crecimiento del "Más 6%" atado al semáforo RSS (verde 6% / ámbar 10% / rojo 15%), ajustable por perfil y siempre registrado.
3. **Invariante 8 ampliado**: advertencia `confirmar_precios_mp` — antes de emitir resultado final, lista de precios de materia prima considerados (precio/fuente/fecha/vigencia) con confirmación explícita obligatoria; sin confirmación no hay precio de licitación.
4. **Regla de emisión nueva**: leyenda obligatoria — el precio es el mejor calculable bajo condiciones e información actuales de la compañía y lo que exige la oferta; no es predicción del precio ganador; margen final y firma son de dirección.
5. **Invariante 16 NUEVO**: retroalimentación real vs presupuesto — cierre de proyecto obligatorio (costo, HH por familia, desperdicio, peso final, acero, montaje, desviaciones); desviación >±5% genera propuesta de recalibración con evidencia; parámetros nunca se recalibran solos: aprobación de dirección + versionado; cada presupuesto queda ligado a su versión de parámetros; proyectos sin cierre no alimentan recalibraciones.
- **Golden test intacto numéricamente**: fija 6% como parámetro registrado (RSS ámbar, default 10%, justificación "caso de referencia"); nuevas salidas del caso: interferencia no_aplica, confirmar_precios_mp confirmada, leyenda presente, cierre pendiente.

## Entregables vigentes (7 archivos) — TODOS regenerados a v1.7 (2026-08-01)
1. **Sistema_DM_Documento_Maestro.pdf** — narrativa completa v1.7: 18 secciones + anexo Lean (16 invariantes, regla de emisión, interferencia, compuerta de precios, cierre inv. 16, golden test).
2. **Guia_Ejemplos_y_Evaluacion_Experta.pdf** — v1.7: recorrido del golden, casos límite nuevos (interferencia, compuerta de precios, % por RSS, recalibración) y rúbrica de 13 criterios (los marcados * bloquean emisión).
3. **Ejemplo_Presupuesto_Sistema_DM.xlsx** — v1.7: 28 hojas, 136 fórmulas, 0 errores (recalculado), golden test verificado valor por valor. Hojas nuevas: Parametros_Version, Bitacora_Precios (compuerta con estado de emisión), Interferencia_Familias y Cierre_Proyecto; 2 gráficas nativas en S12 (carga combinada y ocupación vs 100%).
4. **Kit_Desarrollo_Sistema_DM.zip** — v1.7: spec §4.1–4.16 + 16 invariantes, esquemas con los campos nuevos (alerta_interferencia_familias, confirmar_precios_mp, leyenda_alcance, propuesta_recalibracion, version_parametros; input: fase_rss→default, confirmacion_precios, registro_cierres), golden_test.json con parámetros registrados, fixtures de ejemplo y README con el orden de conflictos e interfaz reservada de estimaciones de montaje.
5. **Evaluacion_Honesta_Pros_Contras.pdf** — v1.7: debilidades 2–5 marcadas RESUELTAS con su mecanismo; riesgos residuales y veredicto de adopción (modo sombra 2–3 licitaciones).
6. **Memoria_Descriptiva_Fabricacion_Ejemplo.pdf** — v1.7 (regenerada; contenido sin cambio de fondo).
7. **Brief_Arranque_Desarrollo.pdf** — v1.7: 5 fases / 18 semanas con criterios de aceptación por números del golden; versionado exigido por inv. 16; qué NO construir en v1 (incluye estimaciones de montaje: siguiente desarrollo, interfaz reservada).

## Nota operativa
Paquete v1.7 completo generado el 2026-08-01. Fuentes de esta regeneración: scripts reportlab/openpyxl (pdfcommon.py, doc_maestro.py, doc_guia.py, doc_resto.py, build_xlsx.py, build_kit.py). Siguiente paso acordado: desarrollo (Brief), pruebas, modo sombra, ajuste con primeros cierres (inv. 16) y luego módulo de estimaciones de montaje.
