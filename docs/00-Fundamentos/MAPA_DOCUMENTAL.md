# Mapa documental

> Inventario de todas las fuentes del repositorio, con su naturaleza, su autoridad y su uso.
> Revisado el **2026-08-19** contra el contenido real de cada archivo.

> **Reorganización del 19-ago.** Todo el material del consultor vive ahora bajo
> `docs/doc inicial/Contexto completo Michelena/`, repartido en seis carpetas temáticas. Las rutas de
> las secciones 1 y 2 cambiaron; si un documento cita la ruta vieja, está roto.

## Cómo leer la columna «Autoridad»

- **Normativa** — manda. Contradecirla es un defecto.
- **Referencia** — dato verdadero de dominio; se usa como fixture, oráculo o insumo de calibración.
- **Comercial** — material de venta. Útil para entender la propuesta de valor; **no** es spec.
- **Trabajo** — documento nuestro, vivo, revisable.
- **Obsoleto/derivado** — no citar; existe un original mejor.

---

## 1 · `docs/doc inicial/Contexto completo Michelena/Sistema completo/` — paquete Sistema DM v1.7

Entrega del consultor del **2026-08-01**. Es la fuente normativa de toda la aritmética.

| Archivo | Naturaleza | Autoridad | Uso |
|---|---|---|---|
| `Kit_Desarrollo_Sistema_DM.zip` + `Kit_Desarrollo_Sistema_DM/` | 7 archivos: `motor_calculo_spec.md`, `input_schema.json`, `output_schema.json`, `golden_test.json`, `ejemplo_input.json`, `ejemplo_output.json`, `README.md`. **Se conservan también sueltos** (11-ago) para que los esquemas y el golden test sean diffeables y la CI pueda leerlos sin descomprimir | **Normativa — máxima** | Contrato técnico del motor. Base de C1.1 y de la CI de D3 |
| `Sistema_DM_Documento_Maestro.pdf` | Narrativa 18 secciones + anexo Lean, 9 pág. | Normativa (narrativa) | Explica el *por qué* de cada invariante. Última autoridad en conflictos |
| `Guia_Ejemplos_y_Evaluacion_Experta.pdf` | Recorrido del golden test, casos límite v1.7, **rúbrica de 13 criterios** | Normativa | La rúbrica es el checklist de auditoría de toda oferta emitida |
| `Ejemplo_Presupuesto_Sistema_DM.xlsx` | 28 hojas (E1–E7, S1–S12, LEAN, Rentabilidad, VE, Bitacora_Precios, Interferencia_Familias, Cierre_Proyecto, Parametros_Version) | Normativa (oráculo) | Oráculo hoja por hoja para implementar y verificar las tools |
| `Evaluacion_Honesta_Pros_Contras.pdf` | Debilidades v1.6 y su estado, riesgos residuales, veredicto de adopción | Referencia | Riesgos residuales a diseñar: disciplina de datos, fatiga de alertas, calibración inicial |
| `Brief_Arranque_Desarrollo.pdf` | Plan de 5 fases / 18 semanas con criterios de aceptación por cifra | Referencia | **No es nuestro plan de proyecto.** Se absorbe como plan interno de D3 (ver decisión D1) |
| `Memoria_Descriptiva_Fabricacion_Ejemplo.pdf` | Ejemplo de memoria descriptiva generada | Referencia | Formato de salida de `memoria_descriptiva` |
| `claude_documento-maestro-notas.md` | Registro de entregables y changelog v1.6→v1.7 | Trabajo (del consultor) | Índice del paquete; declara qué existe y en qué versión |

**Falta en el repo:** el documento rector `claude/instrucciones-sistema-dm.md` (v1.7), que el propio
paquete declara como máxima autoridad. Vive en el proyecto de Claude del consultor. Pedirlo.

### Contenido del kit — detalle

| Archivo | Qué contiene |
|---|---|
| `motor_calculo_spec.md` | 16 invariantes + secciones del motor §4.1–4.16 + criterio de aceptación permanente + interfaz reservada del módulo de montaje |
| `input_schema.json` | `SistemaDM.Input` v1.7 — `version_parametros` + E1 ficha, E2 take-off, E3 precios, E4 planta/VSM, E5 financieros, E6 especificaciones, E7 bases |
| `output_schema.json` | `SistemaDM.Output` v1.7 — 18 campos *required*: `boom`, `apu_aisc`, `mas6`, `opciones`, `flujo`, `capitulo_financiero`, `precio_licitacion`, `leyenda_alcance`, `requisicion_neta`, `secuencia_grua`, `lotes`, `s11_velocidad`, `s12_programa`, `rss`, `lean`, `confirmar_precios_mp`, `alerta_interferencia_familias`, `version_parametros` |
| `golden_test.json` | Caso 100 t / 40 HH·t / apernada. **Ley con tolerancia cero.** Ver cifras en [ALCANCE_v1.md](ALCANCE_v1.md) |

---

## 2 · `docs/doc inicial/` — documentación original del consultor

Material previo (ene–jun 2026), anterior al paquete v1.7. Contiene los **casos reales** y los
**parámetros de calibración**: es el activo más subestimado del repo.

| Archivo | Naturaleza | Autoridad | Uso |
|---|---|---|---|
| `Informe_Factores_Presupuesto_Estructuras_Metalicas (1).xlsx` | 12 hojas: tabla maestra de factores, desperdicio placas/perfiles/ángulos, tornillería, soldadura, conexiones, **factores por eje HH/ton 24/40/60/90**, ancho a molino, calculadora, notas técnicas, referencias AISC | **Referencia — alta** | **Es el contenido de `TenantConfig`.** Insumo directo de K6.1 y C1.1 |
| `ROL.docx` | Rol y proceso de 8 pasos para percepción de planos PDF → IFC4 + BOM, con parámetros fijos | **Referencia — alta** | **Es el prompt de percepción de dcode** (D2.1), ya escrito |
| `FVP Caracas/` — CNARCCS Domo | IFC4 (1,470 elementos), BOM (1,473 filas × 7), oferta 3 escenarios, oferta .docx, resumen ejecutivo de cómputo | **Referencia — alta** | Caso de evaluación end-to-end con verdad de dominio. **Es el caso que usa el mockup** |
| `FVP Barquisimeto/` — CNAR Metropolitano | IFC, BOM (1,625 filas × 15), oferta 3 escenarios, oferta formal PDF | Referencia | Segundo caso e2e |
| `VSM Metalitec Proceso Completo.xlsx` | 9 hojas: VSM actual/futuro, VSM habilitado, planta, familia de producto, hoja de datos de proceso | Referencia | Fixture de E4 (planta/VSM) |
| `VSM Metalitec.xlsx` | 3 hojas — subconjunto del anterior | Obsoleto/derivado | Usar el «Proceso Completo» |
| `PROGRAMA_FABRICACION_CHABACANO_Rev2.xlsx` | 5 hojas: programa maestro (406 filas), calendario por día, plan de viajes, resumen | Referencia | Fixture de S12 y del plan de despachos |
| `Hoja_de_Ruta_del_Producto.pdf` | Frontera del producto y regla de decisión ERP vs. software | **Normativa (frontera)** | Regla: si lleva a *decidir/predecir/estimar/simular* → dentro; si lleva a *registrar/administrar/contabilizar* → ERP, fuera |
| `Software_Estructuras_Caso_Metalitec.pdf` | Caso San Antonio Abad vs. Chabacano; capacidades; convivencia con Strumis | Comercial | Propuesta de valor y evidencia. Cifras de venta, no spec |
| `Como_Funciona_el_Software (3).pptx` | Guía en 4 capas para equipos de Ofertas y Planta | Comercial | Modelo mental del usuario final |
| `Lean_ML_METALITEC_Habilitado (1).pdf` | Lean aplicado al habilitado (32 pág.) | Referencia | Sustento del ahorro HH/ton (inv. 9) |
| `files (5).zip` | Sin inventariar | — | Revisar y clasificar |

---

## 2b · Las cinco carpetas nuevas del 19-ago

Entrega del **14–19 de agosto**, reorganizada por el equipo en carpetas temáticas. Es el material que
más mueve la aguja desde el paquete original: aquí aparecen la calibración con la planta **real** y el
primer par IFC + planos.

### `Modulo presupuestos/` — Sistema de Presupuesto Automático v1.3

| Archivo | Naturaleza | Autoridad | Uso |
|---|---|---|---|
| `Especificacion_Tecnica_Sistema_Presupuestos_v1.3.docx` | Motores M1–M7, modos A/B/C, reglas R1–R26, pipeline de verificación V1–V7, fórmulas normativas | **Normativa** | Es la especificación de la aritmética *comercial y de planta*. Complementa al kit, no lo sustituye — ver [SISTEMA_PRESUPUESTOS_v1.3.md](SISTEMA_PRESUPUESTOS_v1.3.md) |
| `Documento_Maestro_Conceptos_Presupuesto_v1.3.docx` | 27 conceptos + 18 correcciones de coherencia + glosario | Normativa (narrativa) | El *por qué*. Su §26 es una lista de contradicciones que el consultor mismo detectó y corrigió |
| `Paquete_Base_Desarrollo_Sistema_Presupuestos_v1.3.zip` | 6 semillas JSON + 5 archivos fuente + LEEME | **Normativa (semillas)** | `parametros_modelo_y_verificador.json` **cierra el mapeo de tipologías**; `factores_desperdicio.json` y `rendimientos_budget_engine_v41.json` son las tablas paramétricas |
| `Motor_Pintura_Desperdicios (1).xlsx` | 15 hojas: catálogo TDS, factores F1–F7, ambiente A1–A6, perfil de anclaje R0–R4, cálculo por capa, abrasivo, ignífugo por Hp/A, APU por familia, **contrato de datos** | **Normativa** | No eran campos que faltaban: es un **motor entero** con su interfaz. Ver [MOTOR_PINTURA.md](MOTOR_PINTURA.md) |
| `Grafico_Rentabilidad_Presupuesto.html` | Tablero interactivo de referencia, 2 pestañas | Referencia de UI | Insumo de diseño de la curva de rentabilidad |
| `Documento_Maestro_Conceptos_Presupuesto.docx` | Versión anterior sin numerar | Obsoleto/derivado | No citar: existe la v1.3 |

### `Proyectos ejemplo/` — el par IFC + planos que faltaba

| Archivo | Naturaleza | Autoridad | Uso |
|---|---|---|---|
| `Ampliacion_Bodega_50.ifc` | IFC4, mm, **409 elementos estructurales**. Generado con IfcOpenShell 0.8.5 el 13-ago | Referencia | **Desbloquea la percepción**: primer caso con modelo y planos del mismo proyecto. Ojo: es un IFC *sintetizado*, no nativo de Tekla |
| `Ampliación bodega 50 4.pdf` | Planos, 21 MB | Referencia | El otro lado del par. Insumo directo de D2 y de P20 |
| `Resumen_HHton_Soldadura_IFC_...pdf` | 596.2 t · **28.2 HH/ton ponderado** · 16,808 HH · 9 familias con su HH/ton · cómputo de soldadura en metros lineales | **Referencia (oráculo)** | Segundo caso real después de CNARCCS, y el único con desglose por familia. Declara su propia clase de estimación: **Clase 4/5, banda ±15–20%** |
| `IFC Bodega-1/2/3.jpeg` | Capturas del modelo | Referencia | Verificación visual rápida |

### `Analisis planta/` — de layout a carga de planta

| Archivo | Naturaleza | Autoridad | Uso |
|---|---|---|---|
| `Carga de Planta Zapotlan.xlsx` | 4 proyectos reales × semana con ton y HH; 65 → 235 t/sem; ~8,850 HH/sem | **Referencia (oráculo)** | Es exactamente la estructura de P14. Valida la superficie contra un caso real |
| `LAYOUT METALITEC 2026.01.19-Model (6).pdf` | Layout de la planta | Referencia | Uno de los entregables que Daniel debía enviar. Insumo del paso de procesos y estaciones (#115) |
| `metalitec-baysa-v7_2.html` | «El acueducto»: pieza narrada sobre flujo, capacidad, HH y los dos cuellos de botella | Comercial / didáctica | Explica el modelo mental de capacidad. **No es spec**, pero es la mejor explicación de por qué HH es la unidad |

### `Elementos estructurales/` — el motor de piezas complejas (etapa 2)

**Reclasificado el 20-ago.** Daniel aclaró en la reunión qué es esta carpeta, y no era lo que supusimos.
No son dos oráculos de HH: son **dos tipologías de elemento complejo**, de las que aparecen
habitualmente en proyectos **mineros** —tolvas, cintas transportadoras—, y el material detalla su forma
de cálculo y presupuesto. Es un **motor propio** y entra en la **etapa 2**, no en la v1.

| Archivo | Naturaleza | Autoridad | Uso |
|---|---|---|---|
| `Analisis_habilitado_armado_soldadura_AG22_Rev1.html` | Trabe armada de alma esbelta, 7 ud, 2,031.56 kg/ud, 33 piezas/ud, con verificación AISC/AWS y RFIs | Referencia — **etapa 2** | Especificación del motor de piezas complejas. Y declara una frontera que **sí aplica ya**: «la pintura no entra en el total de fabricación bajo ninguna circunstancia» |
| `Analisis_habilitado_armado_soldadura_AC6_Rev0_1.html` | Mismo formato, otra marca | Referencia — etapa 2 | AG22 cierra un RFI heredado de este |

Lo que el motor analiza va más allá del costo: **armado, soldadura y accesibilidad de antorcha y de
llaves**. Cubre los casos en los que la geometría obliga a *cambiar de proceso* —electrodo doblado en
lugar de Fluxor porque la antorcha no entra—. Eso no es un factor: es una decisión de método que cambia
el rendimiento.

Hoy trabaja sobre **planos de taller**, y Daniel tiene en curso un **convertidor de PDF a planos de
taller** para alimentarlo.

Ambos llevan en la cabecera **«Instrucciones Rev. 3.8»**, mientras nuestros documentos citan el rector
en **v1.7**. La numeración del consultor avanzó y no sabemos qué cambió en el camino.

### `Modulo montaje/` y `Sistema completo/`

Ya inventariadas: la primera en [MOTOR_MONTAJE.md](MOTOR_MONTAJE.md), la segunda es la sección 1 de
este mapa con su ruta nueva.

---

## 3 · `docs/Diseño de producto/` — nuestros documentos de alcance

| Archivo | Naturaleza | Autoridad | Estado tras esta revisión |
|---|---|---|---|
| `Segmentacion_Tareas_Kabunik_dcode_v2.md` | Reparto Kabunik/dcode, arquitectura agéntica, contrato de integración §7, fases | **Normativa** (reparto y arquitectura) | Vigente, con **3 correcciones pendientes**: «22 alertas» sin fuente, tool `motor_alertas_22` a renombrar, y el contrato §7 debe absorber los esquemas del kit. Ver [RECONCILIACION](RECONCILIACION_SistemaDM_x_Plataforma.md) |
| `HANDOFF_GitHub_Projects_Plan.md` | Plan materializado en el GitHub Project | Trabajo (ejecutado) | Ya materializado en [projects/2](https://github.com/orgs/kabunik/projects/2) con 14 epics y 65 issues. Mantener como registro histórico |
| `UserFlows_y_Handoff_Diseno.md` | 6 user flows, personas, inventario de 12 pantallas, preguntas abiertas | Trabajo | Vigente. Sus 6 preguntas abiertas quedan resueltas o reasignadas en [ALCANCE_v1.md](ALCANCE_v1.md) |
| `BRIEF_Claude_Design_Mockups.md` | Brief de mockups con defaults ⚙️ y datos de muestra | Trabajo | Vigente; el alcance de pantallas se amplía por decisión D2 |
| `arquitectura_motor_presupuestos.svg` | Diagrama de arquitectura | Trabajo | Vigente |

---

## 4 · `docs/mockup/design_handoff_plataforma_presupuestos/` — handoff de diseño

| Archivo | Naturaleza | Autoridad | Uso |
|---|---|---|---|
| `README.md` | Handoff hi-fi autosuficiente: tokens, 8 pantallas, comportamiento, estado, assets | **Normativa** (diseño visual y comportamiento de UI) | Medidas, colores y copy **finales** para las pantallas que cubre |
| `tokens.css` | Tokens listos para pegar | Normativa (diseño) | Paleta funcional, tipografía, espaciado, medidas fijas de layout |
| `design/Plataforma Presupuestos.dc.html` | Prototipo hi-fi navegable | Referencia | Fuente de verdad de medidas. **No portar su arquitectura** (clase única con `state.screen`) |
| `design/support.js` | Runtime del prototipo | Referencia | No portar |
| `insumos/BRIEF_Claude_Design_Mockups.md` | Copia byte a byte de `docs/Diseño de producto/` | Obsoleto/derivado | Duplicado. Canónico: el de `Diseño de producto/` |
| `insumos/UserFlows_y_Handoff_Diseno.md` | Copia byte a byte de `docs/Diseño de producto/` | Obsoleto/derivado | Ídem. Se conservan porque el paquete de handoff es autosuficiente por diseño |

**Cobertura del mockup:** P3 intake, P4 progreso, P5·A/B/C workspace, P6 BOM+gate, P8 chat+historial,
P9 escenarios+flujo de caja. **Fuera:** P1 login, P2 dashboard, P7 aislado, P10 emisión, P11
calibración, P12 admin — más las pantallas nuevas que introduce la decisión D2.

---

## 5 · Raíz del repositorio

| Archivo | Uso |
|---|---|
| `CLAUDE.md` | Instrucciones de proyecto para agentes. Actualizado en esta revisión: rutas, jerarquía de fuentes, orden de resolución de conflictos |
| `CLAUDE.base.md` | Núcleo común compartido entre proyectos. **No editar por proyecto** |
Nota: `Planificacion_Fechas_v1.md` se reubicó bajo `docs/Diseño de producto/` y está en **v2**
(2026-08-06), revisada tras las decisiones D1/D2. Es **propuesta**, pendiente de validar en J1.3.

---

## Convención de carpetas

`00-Fundamentos/` lleva prefijo numérico para ordenarse primero: es la puerta de entrada al repo.
El resto de carpetas conserva el nombre con el que se cargó el material, para no romper referencias
externas ya compartidas con el consultor y con dcode.
