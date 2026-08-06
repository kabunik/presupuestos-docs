# Glosario

> Vocabulario común del proyecto. Existe porque tres cuerpos documentales —el paquete del consultor,
> nuestros documentos de alcance y el mockup— usan términos distintos para lo mismo, y el mismo
> término para cosas distintas.
>
> **Usa la columna «Término canónico».** Los sinónimos se listan para poder rastrearlos en material
> antiguo, no para seguir usándolos.

---

## Ambigüedades resueltas

### «Escenarios» — son dos cosas distintas

Es la confusión más costosa del proyecto: hizo que la mitad de S12 quedara invisible en el diseño.

| Término canónico | Qué es | Campo del esquema | Valores | Decisión que soporta |
|---|---|---|---|---|
| **Opciones de oferta** | Las 3 alternativas comerciales que se presentan al cliente. Difieren **en supuestos, nunca en la fórmula** (inv. 6) | `opciones` (`minItems: 3, maxItems: 3`) | E1 Conservador / E2 Optimizado / E3 Agresivo — o A Base / B Optimizado / C Mínimo viable en los casos reales | Qué se ofrece al cliente |
| **Escenarios de programa** | Cómo resolver el déficit de capacidad de planta cuando el contrato nuevo compite con los contratos en curso (inv. 14) | `s12_programa.escenarios` + `recomendacion` | A / B / B\* optimizado / C | Si se entra al contrato y cómo se programa |

Nunca decir «escenarios» a secas. En el contrato de integración: `OfferOption` y `ProgramScenario`,
no `Scenario`.

### «Alertas» — tres familias con dueños distintos

| Familia | Origen | Dueño | Efecto |
|---|---|---|---|
| **Alertas del motor** | Invariantes, `output_schema.json` | dcode / tools | 8 nombradas; varias **no supresibles**; algunas bloquean emisión |
| **Validaciones de percepción** | Lectura de planos → modelo | dcode / D2 | Condicionan el gate de BOM |
| **Guardarraíles de edición** | Catálogo del tenant, disponibilidad | dcode / D1 + `TenantConfig` | Rechazan el delta, no la oferta |

**«Las 22 alertas» no existe.** Cifra sin fuente, originada en nuestros propios documentos. Ver T4 en
la [reconciliación](RECONCILIACION_SistemaDM_x_Plataforma.md).

### «Peso» — cuatro pesos distintos, nunca intercambiables

Confundirlos es la causa #1 de pérdidas según el consultor. Cifras del golden test entre paréntesis:

| Término canónico | Qué es | Golden test | CNARCCS |
|---|---|---|---|
| **Peso de planos** | Lo que suman los planos | 100.00 t | 667.79 t (modelo: perfiles + placas) |
| **Peso cobrable / facturable** | Peso conforme a **AISC 303-22 §9.2**: dimensiones nominales del detallado, sin descontar cortes, biseles, perforaciones ni soldadura. **Es sobre lo que se cobra** | 113.70 t | 734.57 t (planos + 10% conexiones) |
| **Peso a comprar** | Cobrable + desperdicio de fabricación. **Es lo que se procura** | 121.25 t (factor 1.2125) | 785.02 t (7% / 13%) |
| **Peso reconstruido** | Recalculado desde el listado de perfiles (Σ longitud × kg/m) **por un camino independiente**, solo para el doble chequeo del inv. 4 | — | — |

En el Sistema DM el facturable **es igual** al cobrable AISC §9.2. Un solo peso alimenta material y
mano de obra (inv. 1): nunca hay dos tonelajes.

### «Calibración», «parámetros» y «configuración»

| Término canónico | Qué es | Ciclo de vida |
|---|---|---|
| **`TenantConfig`** / calibración | Parámetros de planta del fabricante: eje HH/ton, factores de desperdicio, velocidades, $/HH, catálogo de perfiles, umbrales | **Versionada e inmutable** (inv. 16). Cada cambio → versión nueva |
| **`version_parametros`** | Identificador de la versión con la que se calculó un presupuesto (`v2026.1`) | *Required* en entrada y salida. Sella cada oferta |
| **`PriceList`** | Precios de materia prima con fuente, fecha y **vigencia** | Continua; **caduca**. Distinta de la calibración |
| **`PlantLoad`** / E7 carga | Proyectos en curso, HH libres/sem, capacidad de habilitado | **Semanal**. Estado operativo, no configuración |

Una «pantalla de ajustes» que edita valores en sitio **viola el inv. 16**.

---

## Términos del dominio

| Término | Significado |
|---|---|
| **Eje de complejidad / clase HH/ton** | Eje único de 4 clases —**24 / 40 / 60 / 90** HH por tonelada— que mueve de forma coherente desperdicio, soldadura, conexiones y horas (inv. 2). Prohibido mezclar clases o promediarlas |
| **Subproyecto por familia** | Partición obligatoria del proyecto cuando las familias difieren **≥1 salto** del eje (inv. 3). Cada uno con su clase, su APU y su carga de planta |
| **«Más 6%» / Más X%** | Método de costeo por partidas: peso propuesto = planos × (1 + crecimiento), con doble MO prefabricación + fabricación y margen sobre precio. El % por defecto está atado al semáforo RSS (verde 6 / ámbar 10 / rojo 15). **Nunca llamarlo «método Baysa».** El nombre comercial se conserva aunque el % varíe |
| **APU AISC** | Análisis de precios unitarios por partidas AISC: material (E3 × peso) + MO propio + MO de subcontrato del pico S11 + indirectos + margen. Debe reconciliar con Más X% **<1% a margen comparable** |
| **RSS** | Incertidumbre agregada por raíz de suma de cuadrados sobre las partidas con rango: `sqrt(Σ cᵢ²)`. Su **semáforo** (verde/ámbar/rojo) comunica la madurez de la ingeniería, gobierna la contingencia y fija el % por defecto del Más X% |
| **Costo empresa ($/HH)** | Costo por hora-hombre desde datos financieros reales: nómina + prestaciones + obligaciones sindicales/CCT + mezcla propio/subcontrato. Tolerancia **2%** contra contabilidad. **Todo el motor trabaja antes de impuestos** |
| **Punto de equilibrio** | Volumen mínimo de planta bajo el cual se emite `advertencia_bajo_equilibrio`, **nunca suprimible** (inv. 12). Golden test: 490 t/mes, 368 con Lean |
| **Habilitado** | Estación de preparación de material (corte, perforado, biselado) previa a armado y soldadura. Es el **cuello de botella** típico y donde se mide la ocupación con alerta ≥100% |
| **Ahorro Lean** | Métrica **única** de eficiencia: reducción de HH/ton contra la línea base de la clase, sustentada en el VSM. La misma cifra alimenta el motor y el gainsharing (inv. 9). El personal participa del ahorro **verificado**, nunca del estimado |
| **Pena máxima soportable** | Penalización contractual máxima que un retraso puede costar sin destruir la utilidad del escenario de programa recomendado |
| **Déficit explícito** | Faltante de capacidad resuelto de forma visible: sobretiempo con **prima 50%** o subcontrato **all-in**, con su costo en el presupuesto. **Ocultarlo está prohibido** (inv. 11) |
| **Cero silencioso** | Partida que aparece en cero sin declararse como excluida. Origen de la mayoría de las disputas de alcance; **prohibido por diseño** (inv. 5). Exclusiones típicas: escaleras, pernos de anclaje F1554, Nelson studs |
| **Leyenda de alcance** | Texto obligatorio en todo precio emitido: el precio es el mejor calculable bajo las condiciones actuales, **no es predicción del precio ganador**, y el margen final y la firma son de dirección |
| **Interferencia de familias** | Situación en la que una familia de HH altas satura habilitado y reduce la productividad de familias más rentables. Genera alerta no suprimible con rentabilidad relativa en **$/HH de margen** (inv. 3) |
| **Cierre de proyecto** | Registro obligatorio de real vs. presupuestado en 8 rubros al concluir un proyecto. Desviación >±5% → propuesta de recalibración. Sin cierre, el proyecto **no alimenta** recalibraciones (inv. 16) |
| **Modo sombra** | Operar el sistema en paralelo al método actual durante 2–3 licitaciones reales, reconciliando resultados antes de migrar. Recomendación de adopción del consultor |
| **Golden test** | Caso de referencia de 100 t / 40 HH·t / apernada que debe reproducirse **exacto**. Un número distinto sin cambio aprobado de parámetros = **build roto** |

---

## Términos de la plataforma

| Término | Significado |
|---|---|
| **Modelo canónico** | El `Model` que vive en la plataforma (Kabunik) y es la única fuente de verdad de la geometría. El agente **no** lo posee: lo muta vía `model_delta` |
| **`model_delta`** | Mutación incremental del modelo canónico que el agente devuelve y la plataforma aplica. Opera sobre `Model`; E2 take-off se **deriva** de él. Si su formato no se acuerda, visor y agente se desincronizan (riesgo #1) |
| **Recálculo incremental** | Recomputar **solo lo afectado** por una edición, sin re-percibir los planos. Condición de viabilidad de F2 |
| **Compuerta humana / human gate** | Punto donde el sistema se detiene y exige decisión registrada de una persona. La v1 tiene **5**; 2 bloquean la emisión |
| **«Ninguna cifra sin tool»** | Guardarraíl: el agente nunca calcula aritmética por sí mismo; toda cifra proviene de una herramienta determinista y es trazable a ella |
| **Percepción Opción A** | Percepción propia de dcode: planos PDF/DXF → geometría → BOM → modelo, con visión. La contingencia híbrida (ingerir takeoff externo) es puerta abierta en el contrato, **no alcance** |
| **Tenant** | Organización fabricante. Su calibración es privada; la metodología es compartida (acuerdo de IP, J1.2) |
| **F0 Toyota → F3 Ferrari** | Fases del roadmap: F0 intake + percepción + visor · F1 motor y opciones de oferta · F2 edición conversacional en vivo · F3 aprendizaje y efecto de red |

---

## Nombres del producto y del sistema

| Nombre | Qué designa | Estado |
|---|---|---|
| **Sistema DM** | El sistema de costeo y decisión del consultor (v1.7). «DM» = Daniel Michelena | Nombre del consultor, estable |
| **Plataforma de Presupuestos de Estructuras Metálicas** | Nuestro producto | Descriptivo, no comercial |
| **«Kabunik Bid»** | Wordmark que aparece en los mockups | **Provisional** — así lo declara el handoff |
| **METALITEC / Grupo Baysa** | Cliente ancla y planta de referencia | — |
| **`baysa_presupuesto_v4.jsx`** | Motor original citado en la Segmentación v2 | Antecedente. La especificación vigente es `motor_calculo_spec.md` v1.7 |
