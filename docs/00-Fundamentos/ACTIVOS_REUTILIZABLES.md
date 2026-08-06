# Activos reutilizables

> Qué del material entregado se usa **tal cual**, para qué, y en qué ítem del backlog.
> Revisado el **2026-08-06**.

## Hallazgo principal: J1.4 ya está entregado

El issue **J1.4 (#65) — «Insumos de Daniel: parámetros de calibración + casos de evaluación (fechas
compromiso)»** figura como pendiente de recibir insumos. **Los insumos están en el repo desde el
material inicial y el paquete v1.7.** Nadie los había catalogado como tales.

Lo mismo vale, en distinto grado, para C1.1 (esquemas), D2.1 (prompt de percepción), D4.1 (casos de
evaluación) y K6.1 (contenido de la calibración).

---

## Tabla maestra

| Activo | Qué es exactamente | Se usa para | Ítem | Cómo se usa |
|---|---|---|---|---|
| `Kit.../input_schema.json` + `output_schema.json` | JSON Schema draft-07 v1.7, entrada E1–E7 y salida de 18 campos *required* | Contrato de integración | **C1.1 #2** | **Mapear**, no rediseñar. Mapeo preliminar en [CONTRATO_INTEGRACION_v0.md](CONTRATO_INTEGRACION_v0.md) |
| `Kit.../motor_calculo_spec.md` | 16 invariantes + §4.1–4.16 + criterio de aceptación + interfaz reservada | Spec de las tools | **D3.1 #55**, **D3.2 #56** | Especificación normativa. Cada sección §4.x → una o varias tools |
| `Kit.../golden_test.json` | Caso 100 t / 40 HH·t / apernada con salida esperada completa | **Compuerta de CI** | **D4.2 #60** | Test de regresión, **tolerancia cero**. Un número distinto rompe el build |
| `Kit.../ejemplo_input.json` + `ejemplo_output.json` | Fixtures del caso de referencia | Fixtures de desarrollo | D3.1 #55 | Entrada/salida mínima para arrancar |
| `Informe_Factores_...xlsx` (12 hojas) | Tabla maestra de factores + **factores por eje HH/ton 24/40/60/90** + contingencias por fase + base normativa AISC | **Contenido de `TenantConfig`** | **K6.1 #35**, **K6.2 #36**, C1.1 #2 | Define qué campos tiene la calibración y con qué valores por defecto |
| `ROL.docx` | Rol + proceso de 8 pasos: lectura de planos → grilla de ejes → catálogo de perfiles → elementos → longitudes/pesos → IfcOpenShell IFC4 → Excel BOM → documentar. Con parámetros fijos y chequeo de grúa | **Prompt de percepción** | **D2.1 #51** | Punto de partida del system prompt de percepción, ya escrito por el consultor |
| `FVP Caracas/` CNARCCS Domo | IFC4 (1,470 elementos), BOM 1,473 filas, oferta 3 escenarios (xlsx + docx), resumen ejecutivo de cómputo con 7 advertencias | **Caso e2e con verdad de dominio** | **D4.1 #59**, K2.2a #15 | Eval end-to-end + fixture del visor + datos de UI (ya usado por el mockup) |
| `FVP Barquisimeto/` CNAR Metropolitano | IFC, BOM 1,625 filas × 15 col., oferta 3 escenarios, oferta formal PDF | Segundo caso e2e | D4.1 #59 | Segundo punto de medición de regresiones |
| `Ejemplo_Presupuesto_Sistema_DM.xlsx` (28 hojas) | E1–E7, S1–S12, LEAN, Rentabilidad, Evidencia, VE, Bitacora_Precios, Interferencia_Familias, Cierre_Proyecto, Parametros_Version. 136 fórmulas, 0 errores | **Oráculo del motor** | **D3.1 #55** | Verificar tool por tool contra la hoja correspondiente |
| `VSM Metalitec Proceso Completo.xlsx` (9 hojas) | VSM actual/futuro, VSM habilitado, planta, familia de producto, hoja de datos de proceso | **Fixture de E4** | D3.2 #56, K6.1 #35 | Datos reales de planta para calibración y pruebas |
| `PROGRAMA_FABRICACION_CHABACANO_Rev2.xlsx` (5 hojas) | Programa maestro 406 filas, calendario por día, plan de viajes, resumen por día | **Fixture de S12** | D3.2 #56 | Formato y datos reales del programa de fabricación y plan de despachos |
| `Guia_Ejemplos_...pdf` §4 | **Rúbrica de 13 criterios**, 6 bloqueantes | Checklist de auditoría | **D4.2 #60**, K4.2 #26 | Criterios de aceptación de calidad, y candidato a vista de producto |
| `Hoja_de_Ruta_del_Producto.pdf` | Regla de decisión ERP vs. software + lista explícita de lo que queda fuera | **Frontera de alcance** | Todo el backlog | Filtro para rechazar alcance: *decidir/predecir/estimar/simular* dentro; *registrar/administrar/contabilizar* fuera |
| `mockup/.../README.md` + `tokens.css` | Handoff hi-fi: paleta, tipografía, medidas, 8 pantallas, comportamiento, estado mínimo | **Spec de diseño** | K1.\*, K2.2, K4.\*, K5.\* | Medidas, colores y copy finales. `tokens.css` se pega directo |
| `Memoria_Descriptiva_...pdf` | Ejemplo de memoria descriptiva generada | Formato de salida | K3.5 #23 | Plantilla de `memoria_descriptiva` |
| `Lean_ML_METALITEC_Habilitado.pdf` | Lean aplicado al habilitado, 32 pág. | Sustento del inv. 9 | D3.2 #56 | Base del ahorro HH/ton |

---

## Cómo se reparten los dos casos de referencia

Son casos **distintos** y no compiten: cumplen funciones complementarias.

| | `golden_test.json` | CNARCCS Domo |
|---|---|---|
| Naturaleza | Sintético, construido para fijar el cálculo | Real, ejecutado, con oferta emitida |
| Escala | 100 t | 734.57 t cobrables, 1,470 elementos |
| Clase | 40 HH/ton | ~58 HH/ton reales (54 optimizado) → clase 60 del eje |
| Alcance | Fabricación con plan de montaje | Fabricación + transporte; **montaje excluido** |
| Autoridad | **LEY.** Tolerancia cero en CI | Referencia de dominio |
| Rol | Compuerta de regresión de las tools | Eval end-to-end, fixtures de UI, validación con cliente |
| Tiene IFC | No | **Sí**, IFC4 |
| Tiene modelo 3D | No | **Sí** — es el único fixture posible para el visor de F0 |

**Implicación para F0:** el visor necesita geometría, y el golden test no la tiene. El fixture de
K2.2a (#15) debe salir de CNARCCS Domo o de CNAR Metropolitano, que sí traen IFC.

**Implicación para la clase:** el caso real corre a 58 HH/ton, que en el eje del inv. 2 cae en la
clase 60, no en 40. No usar el golden test para validar la percepción ni las cifras de UI, ni el caso
real para validar el motor.

---

## Cifras del golden test (referencia rápida)

Ley del sistema. Deben reproducirse **exactas**. Entrada: 100 t de planos, clase 40 HH/ton, conexión
apernada, fase RSS ámbar, plan de montaje `[22, 38, 24, 21, 10]` t/sem, velocidad base 15 / OT 24
t/sem, $122.79/HH propio y $164.85/HH subcontrato, 600 HH libres/sem, capacidad de habilitado 600
HH/sem, inventario libre 34 t. Parámetros: `v2026.1`, crecimiento fijado en 6% con justificación
«caso de referencia» (la fase ámbar habría puesto el defecto en 10%).

| Concepto | Valor esperado |
|---|---|
| Peso facturable / a comprar / factor | 113.70 t / 121.25 t / **1.2125** |
| APU AISC | **$5,977,353** ($52.57/kg) |
| Más 6% total / a margen comparable 21% | $7,914,933 / $6,011,342 |
| Más 6% por perfil / peso propuesto | $7,793,628 / 106.0 t |
| Reconciliación vs. APU | 0.57% (< 1% ✓) |
| Lean | $429,263 (8.4%) |
| Costo empresa $/HH propio / subcontrato | 122.79 / 164.85 · cargas 99.61 < overhead 115.45 ✓ |
| Grúa | 2 piezas críticas · partir $76,000 |
| Lotes | Déficit sem. 2: 920 HH · extra $145,521 |
| Capítulo financiero → precio de licitación | $206,054 → **$6,183,408** ($54.38/kg) |
| S11 velocidad | Pico 38 t/sem vs. 15/24 → `alerta_subcontrato_hh` 560 HH · volumen 318 t vs. equilibrio 490 / 368 con Lean → `advertencia_bajo_equilibrio` (déficit 172 / 50 t) |
| Requisición | 121.55 − 34.00 = **87.55 t** netas |
| S12 carga combinada | 67 / 83 / 60 / 70 / 45 t/sem |
| S12 ocupación de habilitado | 89% / **111%** / 80% / 93% / 60% → alerta en sem. 2 |
| S12 escenarios | A $145,521 → utilidad $751,082 · B $115,025 → $781,578 · **B\* OT 1,000 HH + desplazar 600 HH a sem. 5 → retraso 1 sem, $64,148 → $832,455** (pena máx. $81,373) |
| S12 recomendación | `escenario_b_optimizado` + `advertencia_comunicacion_cliente: true` |
| RSS | **9.27%** — ámbar · componentes [.0624, .046, .036, .026, .02, .0145] |
| Interferencia de familias | `no_aplica` (una sola clase) |
| Confirmación de precios | `estado_emision: habilitada` |

---

## Qué falta pedir

| Activo | Por qué importa | A quién |
|---|---|---|
| `claude/instrucciones-sistema-dm.md` **v1.7** | Es el **documento rector** — máxima autoridad declarada por el propio paquete. No está en el repo | Consultor |
| Scripts de generación del paquete (`build_kit.py`, `build_xlsx.py`, `doc_maestro.py`…) | Permitirían regenerar y versionar el paquete sin depender de una sesión de Claude | Consultor |
| Peso de referencia del ingeniero de cálculo (Ing. Ángel Delgado) para CNARCCS | El propio resumen lo marca como faltante: sin él no se puede calibrar el cómputo (tolerancia objetivo ≤5%) | Consultor |
| Catálogo de perfiles con kg/m de METALITEC | El mockup lo referencia como «catálogo METALITEC v4». Está parcialmente en las hojas «Catálogo Perfiles» de los BOM | Consultor |
| Contenido de `files (5).zip` | Sin inventariar | Revisar |

---

## Acción sobre el backlog

Comentar en estos issues que su insumo **ya existe**, con la ruta concreta:

| Issue | Comentario |
|---|---|
| #65 J1.4 | Insumos entregados: `Informe_Factores...xlsx` (calibración) + CNARCCS y CNAR Metropolitano (casos). Falta solo el rector y el peso de referencia del ingeniero |
| #2 C1.1 | Replantear como mapeo de los esquemas del kit. Ver [CONTRATO_INTEGRACION_v0.md](CONTRATO_INTEGRACION_v0.md) |
| #59 D4.1 | Desbloquear: los dos casos e2e están en `docs/doc inicial/FVP *` |
| #60 D4.2 | `golden_test.json` disponible como compuerta de CI |
| #51 D2.1 | `ROL.docx` es el punto de partida del prompt. Añadir el requisito de **dos pesos independientes** (inv. 4) |
| #35 K6.1 | El contenido de la calibración está en `Informe_Factores...xlsx` |
| #15 K2.2a | El fixture debe salir de un IFC real (CNARCCS o CNAR Metropolitano); el golden test no tiene geometría |
| #55 D3.1 | El oráculo es `Ejemplo_Presupuesto_Sistema_DM.xlsx`, hoja por hoja |
