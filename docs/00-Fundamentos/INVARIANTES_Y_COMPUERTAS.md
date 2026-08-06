# Invariantes, compuertas y alertas

> Los 16 invariantes del Sistema DM v1.7 traducidos a requisitos verificables de plataforma.
> Fuente normativa: `motor_calculo_spec.md` (kit v1.7) y `Sistema_DM_Documento_Maestro.pdf` §3.
> **Un invariante violado es un defecto, no una preferencia.**

## Cómo usar este documento

Cada invariante tiene tres lecturas:

- **Qué exige** — el enunciado normativo, resumido.
- **Qué implica para la plataforma** — lo que Kabunik debe construir o garantizar (UI, estado, datos).
- **Qué implica para el agente y las tools** — lo que dcode debe garantizar.

La columna **Bloquea** indica si su incumplimiento detiene la emisión del precio.

---

## Los 16 invariantes

### Inv. 1 · Peso único — **bloquea**

**Exige.** Un solo peso base AISC 303-22 §9.2 alimenta material **y** mano de obra. Nunca dos
tonelajes distintos.

**Plataforma.** El `Model` canónico es la única fuente de peso. La UI no puede permitir editar el
peso de material y el de MO por separado. La trazabilidad de cada cifra apunta al mismo origen — el
mockup ya lo hace bien con el badge `tool ✓` y la tarjeta de Trazabilidad.

**Agente/tools.** Un único cálculo de peso, expuesto como tool. Toda cifra derivada lo referencia.

---

### Inv. 2 · Eje de complejidad único

**Exige.** Un solo eje HH/ton con cuatro clases — **24 / 40 / 60 / 90** — que mueve de forma
coherente desperdicio, soldadura, conexiones y horas. Prohibido tomar la soldadura de una clase y el
desperdicio de otra.

**Plataforma.** En la UI de calibración (K6.1) la clase es un **selector único**, no cuatro campos
independientes. Los factores derivados se muestran como consecuencia, no como entrada editable.
Los valores por clase están en `Informe_Factores...xlsx` hoja «7. Factores por HHton».

**Agente/tools.** La clase entra por E1 `clase_hh_ton` (enum estricto: 24, 40, 60, 90).

**Tipologías de referencia:** 24 = naves simples/pre-ingeniería/racks · 40 = estándar industrial
mexicano (valor de referencia) · 60 = sismorresistente, pipe-rack, edificios >10 niveles ·
90 = offshore, refinería, monumentos.

---

### Inv. 3 · Clase por proyecto o subproyecto — **bloquea**

**Exige.** Una sola clase por proyecto, salvo que las familias difieran **≥1 salto** del eje: en ese
caso el proyecto se **parte en subproyectos** por familia, cada uno con su clase, su APU y su carga.
Promediar clases está prohibido. Cuando una familia de HH altas satura habilitado y reduce la
productividad de familias más rentables, se emite `alerta_interferencia_familias` — **nunca se
suprime**; la decisión es humana.

**Plataforma.** Requiere lo que hoy no existe:
- Modelo de datos con **subproyectos por familia** (`e1_ficha.subproyectos[]`, con `familia`,
  `peso_planos_t`, `clase_hh_ton`, `margen_por_hh`).
- UI de la alerta de interferencia con su payload completo: familias, tonelaje de cada una, HH/ton,
  HH en conflicto por semana, rentabilidad relativa ($/HH de margen), impacto estimado y las
  **opciones para la dirección** (repriorizar / partir lote / subcontratar la familia pesada /
  aceptar el impacto documentado).
- La alerta **no tiene botón de silenciar**. Sí puede tener «aceptar impacto documentado», que es
  una decisión registrada, no una supresión.

**Agente/tools.** Detectar el salto ≥1 y partir; calcular el conflicto en habilitado.

---

### Inv. 4 · Doble chequeo del peso — **bloquea**

**Exige.** El peso del IFC se confronta contra el peso **reconstruido** desde el listado de perfiles
antes de emitir. Diferencia fuera de tolerancia → `doble_chequeo: "detener_emision"`.

**Plataforma.** Estado explícito del chequeo en el workspace, junto a la versión del modelo. Cuando
está en `detener_emision`, el CTA de emisión se deshabilita con explicación.

**Agente/tools.** La percepción debe producir **dos pesos por caminos independientes**, no uno
(ver T10 en la reconciliación). `e2_take_off.peso_reconstruido_t` existe precisamente para esto.

---

### Inv. 5 · Exclusiones explícitas — **bloquea**

**Exige.** Escaleras, pernos de anclaje F1554 y Nelson studs aparecen como partidas **excluidas con
leyenda**, nunca como cero silencioso. *«El cero silencioso es el origen de la mayoría de las
disputas de alcance; el sistema lo prohíbe por diseño.»*

**Plataforma.** El BOM y la oferta tienen una sección de **exclusiones visible**, no un pie de nota.
El caso CNARCCS lo hace bien: excluye cimentación, steel deck, escaleras de concreto y Nelson studs
de forma nominal.

**Agente/tools.** `e2_take_off.exclusiones[]` siempre poblado; nunca vacío por omisión.

---

### Inv. 6 · Precio como composición única — **bloquea** (el % registrado)

**Exige.** El precio es una composición única: material + MO a costo empresa + indirectos y overhead
+ capítulo financiero + margen. Las **3 opciones de oferta difieren en supuestos, nunca en la
fórmula**. El método por partidas se llama **«Más 6%»** (nunca «método Baysa») y su % de crecimiento
por defecto está atado al semáforo RSS:

| Fase RSS | Default | Cuándo |
|---|---|---|
| Verde | 6% | Ingeniería para construcción o cercana |
| Ámbar | 10% | Diseño en desarrollo |
| Rojo | 15% | Anteproyecto / alta incertidumbre |

Ajustable por perfil, pero **siempre registrado**; si difiere del defecto exige justificación.
Ambos métodos (APU AISC y Más X%) reconcilian **<1% a margen comparable**.

**Plataforma.** El % de crecimiento es un campo con **default derivado** del semáforo y con captura
obligatoria de justificación al desviarse. Mostrar la reconciliación APU vs. Más X% como indicador
de salud, no oculta. El nombre comercial se conserva aunque el porcentaje varíe.

**Agente/tools.** `mas6.crecimiento_aplicado`, `crecimiento_default_por_rss`,
`justificacion_si_difiere`, `reconciliacion_vs_apu`.

---

### Inv. 7 · Incertidumbre RSS coherente con contingencia

**Exige.** La incertidumbre se agrega por raíz de suma de cuadrados sobre las partidas con rango, y
debe ser coherente con la contingencia por fase (semáforo).

**Plataforma.** Semáforo visible en el workspace: gobierna el default del inv. 6 y la contingencia.
Contingencias por fase de ingeniería en `Informe_Factores...xlsx` hoja 10 (anteproyecto +15%,
básica +8–10%, detalle 30% +5–7%, detalle 70% +3–5%, detalle 100% 0–2%).

**Agente/tools.** `rss: { valor, fase, componentes[] }`.

---

### Inv. 8 · Acero con vigencia y compuerta de confirmación — **bloquea**

**Exige.** Bloquear o alertar con precios vencidos. **Antes de emitir**, presentar la lista completa
de precios de MP considerados —material, precio, fuente, fecha, vigencia restante— y exigir
**confirmación explícita registrada en bitácora**. Sin confirmación no hay `precio_licitacion`.

**Plataforma.** **Segunda compuerta humana de la v1**, de rango igual a la del BOM. Ver T5 en la
reconciliación para la comparativa. Requiere:
- Pantalla/modal de confirmación con la tabla de precios y su vigencia restante.
- **Bitácora de precios** persistente, ligada a la oferta como evidencia, auditable en el cierre.
- Estado `estado_emision: habilitada | bloqueada` visible en el workspace.
- Reapertura automática al vencer la vigencia de cualquier precio.

**Agente/tools.** `e3_precios.confirmacion_precios { confirmado, por, fecha, referencia_oferta }` en
entrada; `confirmar_precios_mp` en salida.

---

### Inv. 9 · Métrica Lean única

**Exige.** Una sola métrica de eficiencia Lean —el **ahorro de HH/ton**— para el motor de costeo
**y** para el gainsharing con el personal. Promesa comercial conservadora: 8–13%; el 30% de San
Antonio Abad es un caso documentado, no una garantía.

**Plataforma.** No mostrar dos métricas de eficiencia distintas. El copy no promete 30%.

**Agente/tools.** `lean: { ahorro, pct }`. El personal participa del ahorro **verificado**, nunca del
estimado — se audita en el cierre (inv. 16).

---

### Inv. 10 · Jerarquía documental: el concurso rige

**Exige.** Donde las especificaciones del concurso difieran de la norma, **rigen las del concurso**.
Cada desviación se registra con cinco columnas —concepto, norma, concurso, cuál rige, impacto— y su
costo **entra al presupuesto**. *«Una desviación sin costo asignado es una desviación sin registrar.»*

**Plataforma.** Tabla de desviaciones de concurso como superficie de primera clase, con el impacto
económico visible y sumado. Es la evidencia con la que se defienden alcances en disputa.

**Agente/tools.** `e6_especificaciones.desviaciones_concurso[]` con `rige: norma | concurso` e
`impacto` numérico.

---

### Inv. 11 · El plan de montaje es ENTRADA — **bloquea**

**Exige.** El plan de montaje del cliente entra por E6 (`plan_montaje_t_sem`, *required*) y el
sistema calcula **hacia atrás**: lotes, secuencia y memoria se determinan para cumplirlo **contra la
planta cargada**, no contra una planta vacía teórica. Si la capacidad no alcanza, el déficit se
resuelve explícito —sobretiempo con prima 50% o subcontrato all-in— con su costo visible.
**Ocultar el déficit está prohibido.**

**Plataforma.** El intake debe capturarlo (ver T6). Hoy **no lo pide**: es el hueco de alcance más
consecuente detectado en esta revisión, porque sin él el motor no llega a precio.

**Agente/tools.** `lotes: { deficit_hh_por_semana[], costo_resolucion }`.

---

### Inv. 12 · Doble chequeo de velocidad (S11) — **bloquea la supresión**

**Exige.** Dos velocidades. **Montaje:** el pico de t/semana que exige el plan se cubre
explícitamente con base, sobretiempo o subcontrato. **Planta completa:** el volumen contratado se
confronta **siempre** contra el punto de equilibrio; si la planta opera bajo equilibrio, la
`advertencia_bajo_equilibrio` se emite y **no puede suprimirse**.

**Plataforma.** La advertencia no tiene control de silencio. Debe ser visible en la oferta, no solo
en una pantalla interna.

**Agente/tools.** `s11_velocidad: { alerta_subcontrato_hh, advertencia_bajo_equilibrio: { activa, deficit_t, deficit_con_lean_t } }`.

---

### Inv. 13 · Inventario primero

**Exige.** La requisición aplica el inventario **libre** de E7 antes de procurar: solo el neto va a
compras. Los préstamos entre contratos se registran siempre.

**Plataforma.** Mostrar la cascada bruta → inventario libre → neta, no solo el neto.

**Agente/tools.** `requisicion_neta: { bruta_t, inventario_libre_t, neta_t }`.

---

### Inv. 14 · Decisión de programa optimizada (S12) — **bloquea la supresión**

**Exige.** Carga combinada en **HH y t/semana** con % de ocupación de habilitado (**alerta ≥100%**).
Antes de recomendar, el sistema **minimiza tiempo y costo del retraso**: sobretiempo primero, y solo
el excedente se desplaza a la semana con holgura
(`retraso mínimo = ceil(HH_a_desplazar / HH_sem)`). **Advertencia obligatoria** de comunicar y
acordar el retraso con el cliente afectado **ANTES** de comprometer el contrato nuevo; sin acuerdo,
la recomendación cae al escenario A. El escenario C (no entrar) **nunca se recomienda** con la planta
bajo equilibrio.

**Plataforma.** Superficies nuevas (ver T7): carga combinada, ocupación semanal con umbral 100%, los
cuatro escenarios de programa con su costo/utilidad/retraso/pena máxima, y la advertencia de
comunicación como paso obligado antes de comprometer.

**Agente/tools.** `s12_programa` completo, incluida `advertencia_comunicacion_cliente` y
`recomendacion` con enum de cuatro valores.

---

### Inv. 15 · Costo empresa real, antes de impuestos — **bloquea**

**Exige.** $/HH desde datos financieros reales —nómina, prestaciones, obligaciones sindicales/CCT,
mezcla propio/subcontrato— con **tolerancia 2%** contra contabilidad. El overhead debe cubrir las
cargas de planta. **Todo costo y precio del motor son ANTES de impuestos:** IVA e ISR quedan fuera.

**Plataforma.** La UI de calibración financiera valida la tolerancia del 2% y el chequeo
`cargas < overhead`. **Ninguna cifra del motor lleva IVA.** Atención: las ofertas reales del caso
CNARCCS sí muestran IVA 16% — eso es capa de presentación de la oferta, fuera del motor. Distinguirlo
explícitamente en la UI para no contaminar los cálculos.

**Agente/tools.** `e5_financieros` con `tolerancia_vs_contabilidad` (default 0.02).

---

### Inv. 16 · Retroalimentación real vs. presupuesto — **bloquea la automatización**

**Exige.** Cierre de proyecto **obligatorio**, comparando real vs. presupuestado en al menos: costo
total, HH por familia, HH/ton efectivas, desperdicio, peso final, precio de acero, montaje y
desviaciones de concurso. Desviación **>±5%** (parametrizable) genera **propuesta** de recalibración
con evidencia —proyectos, magnitud, causa probable—. **Los parámetros nunca se recalibran solos:**
aprobación de dirección + nueva versión. Cada presupuesto queda **ligado a su versión de
parámetros**. Un proyecto sin cierre aparece en pendientes y **no alimenta recalibraciones**.

**Plataforma.** Lo más estructural del invariante para nosotros:
- `TenantConfig` **versionada e inmutable**; `version_parametros` viaja en cada presupuesto emitido.
- Módulo de **cierre de proyecto** con los 8 rubros.
- **Bandeja de propuestas de recalibración** con evidencia, para dirección: aprobar / rechazar.
- **Lista de cierres pendientes** visible en dirección.
- Prohibido cualquier autoajuste. El copy no puede decir «se actualiza solo» (ver T9).

**Agente/tools.** `propuesta_recalibracion { parametros_afectados[], evidencia[], estado, nueva_version_propuesta }`;
`e7_bases.registro_cierres[]`.

---

## Regla de emisión — leyenda de alcance del precio

Campo `leyenda_alcance`, ***required*** en la salida. Texto obligatorio en todo precio emitido:

> «Este es el mejor precio calculable bajo las condiciones e información actuales de la compañía
> (planta, inventario, costos y precios confirmados a la fecha) y lo que exige la oferta. No es
> predicción del precio ganador del mercado; la decisión final de margen y la firma de la oferta son
> de la dirección.»

El sistema entrega el **costo verdadero** y la composición del precio. La inteligencia competitiva y
el margen final quedan **fuera del motor**. Esto delimita la responsabilidad del producto y debe ser
visible en la oferta, no enterrado en condiciones generales.

---

## Compuertas humanas de la v1

| # | Compuerta | Invariante | Momento | Efecto si no se pasa |
|---|---|---|---|---|
| 1 | **Aprobación del BOM** | práctica de plataforma + inv. 4/5 | Antes de calcular costos | No hay opciones de oferta |
| 2 | **Confirmación de precios de MP** | inv. 8 | Antes de emitir | `estado_emision: bloqueada` — no hay `precio_licitacion` |
| 3 | **Acuerdo de retraso con el cliente afectado** | inv. 14 | Antes de comprometer el contrato nuevo | La recomendación cae al escenario A |
| 4 | **Decisión sobre interferencia de familias** | inv. 3 | Antes de programar | La alerta permanece abierta; decisión documentada obligatoria |
| 5 | **Aprobación de recalibración** | inv. 16 | Al cerrar proyecto con desviación >±5% | Los parámetros no cambian; la propuesta queda pendiente |

Las compuertas 1 y 2 **bloquean la emisión**. Las 3, 4 y 5 exigen **decisión registrada**, no
bloquean el cálculo pero sí condicionan la recomendación o el aprendizaje.

---

## Rúbrica de auditoría — 13 criterios

De `Guia_Ejemplos_y_Evaluacion_Experta.pdf` §4. Los marcados `*` **invalidan la emisión**. Es el
checklist con el que un evaluador experto audita cualquier oferta del sistema, y debería
materializarse como una vista de producto.

| # | Criterio | Verifica |
|---|---|---|
| 1 `*` | Peso único y doble chequeo | Un solo peso AISC alimenta material y MO; IFC vs. reconstruido en tolerancia |
| 2 `*` | Clase correcta o subproyectos | Sin promedios de clases; salto ≥1 → subproyectos por familia |
| 3 `*` | Exclusiones explícitas | Escaleras, F1554, Nelson studs con leyenda; ningún cero silencioso |
| 4 | Reconciliación de métodos | APU vs. Más X% <1% a margen comparable |
| 5 `*` | % de crecimiento registrado | Coherente con fase RSS o justificado |
| 6 `*` | Precios confirmados | Bitácora firmada; vigencias no vencidas |
| 7 | Desviaciones de concurso | Tabla completa con costo dentro del presupuesto |
| 8 `*` | Déficit explícito | Sobretiempo/subcontrato con costo visible; plan de montaje cumplido contra planta cargada |
| 9 `*` | S11/S12 con advertencias vivas | Bajo equilibrio y ocupación ≥100% visibles; retraso mínimo calculado; advertencia de comunicación presente |
| 10 | Interferencia de familias | Alerta emitida cuando aplica, con familias, tonelaje y rentabilidad relativa |
| 11 `*` | Costo empresa y antes de impuestos | $/HH desde financieros reales (tolerancia 2%); sin IVA/ISR en el motor |
| 12 | Leyenda de alcance | Presente en el precio final |
| 13 | Versión de parámetros | Presupuesto ligado a versión vigente; cierres pendientes listados |

---

## Catálogo de alertas — pendiente de definición

La cifra «22 alertas» que circula en nuestros documentos **no tiene fuente** (ver T4 en la
reconciliación). El catálogo real debe construirse distinguiendo tres familias:

**A · Alertas y advertencias del motor** (nombradas en `output_schema.json`, dueño: dcode/tools)

| Nombre | Invariante | Supresible |
|---|---|---|
| `doble_chequeo: detener_emision` | 4 | No — detiene emisión |
| `confirmar_precios_mp: bloqueada` | 8 | No — bloquea emisión |
| `alerta_interferencia_familias` | 3 | **No** |
| `advertencia_bajo_equilibrio` | 12 | **No** |
| `alerta_ocupacion` (≥100% por semana) | 14 | **No** |
| `alerta_subcontrato_hh` | 12 | No |
| `advertencia_comunicacion_cliente` | 14 | **No** |
| `propuesta_recalibracion` | 16 | No — requiere decisión |

**B · Validaciones de percepción** (dueño: dcode/D2). Sin catálogo cerrado. El caso CNARCCS lista 7
advertencias de ingeniería reales (cantidades estimadas por conteo típico, graderías modeladas de
forma agregada, falta peso de referencia del ingeniero de cálculo, correas cold-formed que conviene
separar como sub-partida, etc.) que son el mejor punto de partida. El .pptx del consultor las
clasifica en **Crítica / Alta / Media / Baja**, taxonomía que el mockup reduce a Crítica/Media/Baja.

**C · Guardarraíles de edición** (dueño: dcode/D1 + `TenantConfig`). El mockup ya modela el caso
canónico: perfil fuera del catálogo calibrado → rechazo con alternativas equivalentes. Rechazan el
delta, no la oferta.

**Riesgo a diseñar, no solo a documentar.** La `Evaluacion_Honesta` marca la **fatiga de alertas**
como riesgo residual: *«advertencias que nunca se apagan pueden normalizarse y dejar de leerse»*. La
mitigación que propone es **dueño por alerta** y que las bloqueantes bloqueen de verdad, no solo
adviertan. Eso es diseño de producto: jerarquía visual entre «bloquea» y «advierte», y asignación de
responsable.
