# Alcance de la v1

> Qué entra en la primera versión del producto, derivado de la **decisión D2** (2026-08-06):
> *alcance del mockup + Sistema DM completo*.
> Las estimaciones de esfuerzo **no se fijan aquí** — las aporta cada equipo sobre su workstream.

## Definición de v1

**La v1 es la versión que puede emitir una oferta válida bajo los 16 invariantes.**

Ese es el criterio, y tiene consecuencias inmediatas: no basta con el recorrido del mockup
(intake → visor → gate de BOM → chat → opciones de oferta), porque bajo los invariantes ese recorrido
**no llega a un precio de licitación emitible** (ver el hallazgo transversal en
[RECONCILIACION](RECONCILIACION_SistemaDM_x_Plataforma.md)). Falta el plan de montaje como entrada
(inv. 11), la compuerta de precios (inv. 8) y la leyenda de alcance.

Por decisión D2, la v1 incorpora además el **núcleo de decisión de planta** — S11/S12, interferencia
de familias y cierre gobernado — que es donde el propio consultor sitúa el diferenciador frente al
software comercial.

## Efecto sobre las fases

La v1 así definida abarca **F0 + F1 + F2** del roadmap, y **adelanta de F3 a v1** el lazo
cotizado-vs-ejecutado (inv. 16).

Ese adelanto no es una expansión arbitraria: el propio plan del consultor sitúa el módulo de cierre y
recalibración en su F5, dentro del alcance del sistema base, no como desarrollo posterior. Lo que
queda en **F3 Ferrari (post-v1)** es lo que siempre fue extensión: benchmarks agregados
multicliente, RAG/memoria multicliente, optimizador de nesting, puente a Strumis y analítica.

> **A validar con dcode y con Daniel:** este adelanto mueve la frontera de F3 y afecta el orden de
> trabajo de dcode. Debe pasar por J1.3 (#64) antes de reflejarse en los milestones del Project.

---

## Inventario de pantallas

### Pantallas ya diseñadas — mockup hi-fi disponible

| Ref | Pantalla | Estado del diseño | Cambios que exigen los invariantes |
|---|---|---|---|
| **P3** | Wizard de intake | hi-fi, 3 pasos | **+ plan de montaje** t/sem (inv. 11, *required*) · **+ fase RSS** (gobierna el % del inv. 6) · **+ clase HH/ton o subproyectos por familia** (inv. 2/3) · **+ desviaciones de concurso** (inv. 10) |
| **P4** | Progreso de generación | hi-fi, 1 estado | Retirar «22 reglas de validación» (cifra sin fuente, ver T4) |
| **P5·A** | Workspace — revisión del primer output | hi-fi | **+ semáforo RSS** · **+ estado del doble chequeo de peso** (inv. 4) · **+ estado de emisión** (habilitada/bloqueada con motivo) · en F0 el costo va vacío o «pendiente de motor» (ver T1) |
| **P5·B** | Workspace — edición en vivo | hi-fi | Reapertura del gate 1 ya modelada ✓ |
| **P5·C** | Workspace — opciones de oferta | hi-fi | Renombrar «escenarios» → «opciones de oferta» (ver [GLOSARIO](GLOSARIO.md)) |
| **P6** | Grilla BOM + gate de aprobación | hi-fi | **+ sección de exclusiones visible** (inv. 5: escaleras, F1554, Nelson studs — nunca cero silencioso) |
| **P8** | Chat de edición + historial | hi-fi | Sin cambios de fondo ✓ |
| **P9** | Opciones de oferta + flujo de caja | hi-fi | **+ leyenda de alcance** · **+ reconciliación APU vs. Más X%** (<1%, inv. 6) · **+ % de crecimiento aplicado con su justificación** · corregir E2 a $67,076/ton |

### Pantallas del inventario original sin mockup

| Ref | Pantalla | Prioridad | Nota |
|---|---|---|---|
| **P1** | Login / selección de organización | Baja | Sin cambios respecto al inventario original |
| **P2** | Dashboard de proyectos | Media | **+ lista de cierres pendientes** (inv. 16) · **+ badge de frescura de E7** |
| **P7** | Panel de alertas (aislado) | Alta | Su contenido depende del catálogo de alertas, hoy sin definir (T4) |
| **P10** | Preview / emisión de oferta | **Alta** (sube de Media) | Es donde convergen tres invariantes bloqueantes: leyenda de alcance obligatoria, exclusiones explícitas y sello de `version_parametros`. Ya no es una pantalla de trámite |
| **P11** | Wizard de calibración del fabricante | **Alta** (sube de Media) | **+ eje único 24/40/60/90** como selector, no cuatro campos (inv. 2) · **+ validación de tolerancia 2% vs. contabilidad y `cargas < overhead`** (inv. 15) · **+ versionado inmutable**: cada cambio produce una versión nueva, no edita la vigente (inv. 16). Contenido en `Informe_Factores...xlsx` |
| **P12** | Admin de organización y usuarios | Baja | Sin cambios |

### Pantallas nuevas que introduce la decisión D2

| Ref | Pantalla | Invariante | Prioridad | Por qué |
|---|---|---|---|---|
| **P13** | **Gate de confirmación de precios de MP + bitácora** | 8 | **Crítica** | Segunda compuerta bloqueante. Tabla de precios con material/precio/fuente/fecha/vigencia restante + confirmación registrada. Sin ella no hay `precio_licitacion`. Reapertura automática al vencer vigencia |
| **P14** | **Carga de planta y ocupación de habilitado** | 12, 14 | Alta | Carga combinada en HH y t/sem contra contratos en curso, % de ocupación con **alerta ≥100%**, `advertencia_bajo_equilibrio`. Consume E7, que se actualiza **semanalmente** → necesita estado de frescura |
| **P15** | **Decisión de programa (escenarios A/B/B\*/C)** | 14 | Alta | Los cuatro escenarios con costo, utilidad, retraso en semanas y pena máxima soportable. Recomendación **condicionada** al acuse de la advertencia de comunicación al cliente afectado. C nunca se recomienda bajo equilibrio |
| **P16** | **Interferencia de familias y subproyectos** | 3 | Alta | Familias, tonelaje, HH/ton, HH en conflicto/sem, rentabilidad relativa ($/HH de margen), impacto estimado y las cuatro opciones para dirección. **Sin botón de silenciar** |
| **P17** | **Cierre de proyecto y recalibración** | 16 | Media | Captura de los 8 rubros de cierre (real vs. presupuestado) + bandeja de **propuestas de recalibración** con evidencia para aprobar/rechazar + versionado de parámetros |
| **P18** | **Desviaciones de concurso** | 10 | Media | Tabla de cinco columnas —concepto, norma, concurso, cuál rige, impacto— con el costo sumado al presupuesto. Puede vivir como panel del workspace en lugar de pantalla propia |
| **P19** | **Requisición y secuencia de fabricación** | 11, 13 | Media | Cascada bruta → inventario libre → neta; lotes con déficit por semana y su costo de resolución; piezas críticas de grúa. Hoy `requisicion_neta`, `lotes` y `secuencia_grua` son salidas *required* sin ninguna superficie |

Doce pantallas del inventario original más siete nuevas. **P13 es la de mayor urgencia** entre las
nuevas: es bloqueante de la emisión y no tiene ni diseño ni contrato.

---

## Preguntas abiertas del handoff de diseño — estado

Las seis de `UserFlows_y_Handoff_Diseno.md`, resueltas o reasignadas:

| # | Pregunta | Estado |
|---|---|---|
| 1 | ¿Chat panel lateral persistente u overlay? | **Resuelta** — panel lateral colapsable de 320 px (⚙️ `chatMode: panel`) |
| 2 | ¿Aprobación del BOM global o parcial? | **Resuelta** — global (⚙️ `bomApproval: global`); la variante parcial queda como feature flag |
| 3 | ¿Cuántos escenarios comparamos? | **Resuelta por el esquema** — `opciones` es `minItems: 3, maxItems: 3`. **Exactamente 3, no configurable** |
| 4 | ¿Comparación v1 vs. v2 en los primeros mocks? | **Resuelta** — sí, versión simple de dos columnas |
| 5 | ¿Costos en tiempo real durante la edición? | **Resuelta** — sí, con badge «recalculando…» (⚙️ `costUpdate: vivo`). Sigue siendo el supuesto de mayor riesgo de latencia para dcode |
| 6 | ¿Onboarding de calibración autónomo o asistido? | **Abierta.** Gana peso con D2: la calibración pasa a prioridad Alta e incluye validaciones financieras (inv. 15) y versionado inmutable (inv. 16). El modo asistido por Daniel/Kabunik parece necesario, pero es decisión de producto → `needs-definition` |

Los tres supuestos ⚙️ quedan **confirmados como defaults**, no congelados en código: se implementan
como configuración o feature flag, según el handoff.

---

## Fuera de alcance de la v1

Se mantiene la frontera de `Hoja_de_Ruta_del_Producto.pdf`: *decidir, predecir, estimar, simular* está
dentro; *registrar, administrar, contabilizar* está fuera.

> **Cambio de alcance del 13-ago (#112).** El módulo de montaje **entra en la v1**. Estaba fuera con
> la interfaz reservada; el consultor lo entregó funcional y nuestras vistas P15 y P19 ya muestran su
> output, así que mantenerlo fuera dejaría esas pantallas alimentadas por nada. Detalle y conflictos
> resueltos en [MOTOR_MONTAJE.md](MOTOR_MONTAJE.md).

**No entra en v1:**

| Elemento | Por qué |
|---|---|
| Alcance ERP (producción en vivo, facturación, inventario contable, nómina, órdenes de compra) | Fuera de frontera, permanentemente |
| Puente a Strumis / SAP | F3. Es el límite aguas abajo, no alcance de v1 |
| Benchmarks agregados multicliente | F3. Requiere masa crítica de tenants |
| RAG / memoria multicliente | F3 |
| Optimizador de nesting | F3 (prioridad media en la hoja de ruta del consultor) |
| **Recalibración automática de parámetros** | **Prohibida por el inv. 16**, no diferida. Nunca entra |
| ~~Módulo de estimaciones de montaje~~ | **ENTRA EN v1 (decisión de dirección, 13-ago · #112).** El consultor lo entregó funcional el 12-ago y ya teníamos vistas que muestran su output. dcode lo envuelve como tool; sus entradas viven en un wizard paralelo activado por la opción de alcance. Ver [MOTOR_MONTAJE.md](MOTOR_MONTAJE.md) |
| Simuladores gráficos y modelador dentro del motor | Módulos anexos del Sistema DM; el visor de la plataforma no es esto (ver T1) |
| Re-evaluación de ofertas vivas | Prioridad media en la hoja de ruta del consultor; no v1 |
| Biblioteca de proyectos tipo | «A evaluar» en la hoja de ruta; riesgo de cruzar a archivo administrativo |

---

## Riesgos de la v1 que hay que diseñar, no solo documentar

Los cuatro riesgos residuales que la `Evaluacion_Honesta` del consultor deja **abiertos** son
requisitos de producto encubiertos:

| Riesgo | Qué exige del producto |
|---|---|
| **Disciplina de datos de E7** — «E7 desactualizado convierte S12 en ficción bien presentada» | Estado de **frescura** de carga de planta e inventario, dueño de datos por base, cadencia semanal visible, y efecto explícito sobre la emisión cuando E7 está viejo |
| **Fatiga de alertas** — advertencias que nunca se apagan se normalizan | Jerarquía visual nítida entre **bloquea** y **advierte**; dueño por alerta; las bloqueantes deben bloquear de verdad |
| **Calibración inicial** — el golden test protege regresiones, no calibra contra la realidad | Soporte de **modo sombra**: correr 2–3 licitaciones en paralelo al método actual y reconciliar antes de migrar. Es funcionalidad, no proceso |
| **Lectura comercial** — el sistema da el costo verdadero, no el precio ganador | La leyenda de alcance lo declara. El copy del producto no puede prometer «predice el precio ganador» ni «se actualiza solo» |

Añadido por esta revisión:

| Riesgo | Qué exige |
|---|---|
| **Latencia de la reconstrucción en vivo** (riesgo técnico #1 de F2) | Recálculo incremental, caché y agente con estado. El supuesto ⚙️ `costUpdate: vivo` lo agrava: cada edición dispara recálculo de costos |
| **Desincronización visor ↔ motor** | El formato de `model_delta` debe congelarse con la restricción de que opera sobre `Model` y E2 se deriva (ver [CONTRATO_INTEGRACION_v0](CONTRATO_INTEGRACION_v0.md) §3) |

---

## Criterio de «hecho» para la v1

La v1 está terminada cuando:

- [ ] `golden_test.json` se reproduce **exacto** en CI (compuerta de D3).
- [ ] Los dos casos reales (CNARCCS Domo, CNAR Metropolitano) corren end-to-end en el eval harness.
- [ ] Los **13 criterios de la rúbrica** de auditoría se verifican sobre una oferta emitida, y los 6
      marcados `*` bloquean la emisión cuando fallan.
- [ ] Las **5 compuertas humanas** están implementadas, con las 2 bloqueantes bloqueando de verdad.
- [ ] Ninguna alerta no-supresible tiene control de silencio.
- [ ] Toda oferta emitida lleva leyenda de alcance, exclusiones explícitas y sello de
      `version_parametros`.
- [ ] Existe modo sombra utilizable para 2–3 licitaciones reales.
- [ ] Contrato de integración v0 congelado y respetado por ambos lados.
