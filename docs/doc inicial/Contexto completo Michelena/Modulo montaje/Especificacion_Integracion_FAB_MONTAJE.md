# ESPECIFICACIÓN DE INTEGRACIÓN — Generador IFC/Fabricación ↔ Módulo de Montaje
### METALITEC / Grupo Baysa · v1.0 · Moneda USD · Documento para AMBOS Proyectos de Claude

> Este documento define el **contrato físico de intercambio** entre el Proyecto "Generador de IFC y
> Presupuestos" (fabricación) y el Proyecto de Montaje. Copiarlo a ambos proyectos. Los esquemas JSON
> son vinculantes: cualquier cambio se versiona aquí.

---

## 0. Convención de intercambio

- **Formato:** archivos JSON UTF-8, un paquete por proyecto/oferta.
- **Paquete:** carpeta o `.zip` llamado `INTERCAMBIO_<proyecto>_<fecha>/` que se sube al otro
  Proyecto de Claude como archivos adjuntos (o se mantiene en un solo Proyecto fusionado).
- **Dirección FAB → MONTAJE:** 5 archivos (secciones 1–5). **MONTAJE → FAB:** 3 archivos (secciones 6–8).
- **Moneda:** todo importe en **USD**. Si la fuente está en otra moneda, convertir antes de exportar
  y declarar el FX usado en el campo `fx_usado`.
- **Identidad de pieza:** toda pieza se identifica por el par `{marca, guid_ifc}` (sección 9).
- **Versionado:** cada archivo lleva `{"version": "1.0", "proyecto": "...", "fecha": "YYYY-MM-DD"}`.

---

## 1. FAB → MONTAJE · `despiece_tornilleria.json`
Mata la estimación de pernos por tonelaje (alimenta `PERNOS_POR_CONEXION` / `PERNOS_SEGMENTO`).

```json
{
  "version": "1.0", "proyecto": "MAILENA_EWS", "fecha": "2026-08-12",
  "resumen": {"total_pernos_campo": 8460,
               "por_tipo": {"A307": 4650, "A325": 3200, "A490": 180, "F1554": 430},
               "conexiones_campo": 940, "pernos_prom_conexion": 9.0},
  "por_conexion": [
    {"conexion_id": "CX-0001", "marca_a": "TPP-3-01", "marca_b": "C-01",
     "tipo_junta": "apernada", "perno": "A325", "diametro_in": 0.75, "cantidad": 8,
     "metodo_apriete": "TC"}
  ]
}
```
`tipo_junta` ∈ {apernada, soldada, mixta}. `metodo_apriete` ∈ {snug, giro_tuerca, TC, DTI, torque}.
El bloque `resumen` es obligatorio; `por_conexion` es deseable (habilita HH por conexión real).

## 2. FAB → MONTAJE · `computos_soldadura.json`
Cierra la **verificación obligatoria** de soldadura de campo (`SOLD_CAMPO_KG` + `SOLD_POS_PCT`).

```json
{
  "version": "1.0", "proyecto": "MAILENA_EWS", "fecha": "2026-08-12",
  "campo": {"kg_aporte": 905, "ml_cordon": 610,
             "por_posicion_pct": {"plana": 0.30, "horizontal": 0.30, "vertical": 0.25, "sobrecabeza": 0.15},
             "por_proceso": {"FCAW": 0.7, "SMAW": 0.3},
             "cjp_juntas": 46, "pct_ut": 0.10},
  "taller": {"kg_aporte": 6120},
  "wps_referencia": ["WPS-01-FCAW-3G", "WPS-04-SMAW-4G"]
}
```
Si `campo.kg_aporte = 0`, declararlo explícitamente (0 es un dato; ausencia es bandera).

## 3. FAB → MONTAJE · `cronograma_fab.json`
Valida pull dates (chequeo O2 del cooptimizador).

```json
{
  "version": "1.0", "proyecto": "MAILENA_EWS", "fecha": "2026-08-12",
  "lotes": [
    {"lote_id": "F-01", "toneladas": 46.5, "marcas": ["PB-01", "C-01", "C-02"],
     "planta": "Lerma", "fecha_liberacion": "2026-10-05"}
  ],
  "transporte_incluido_en": "montaje"
}
```
`transporte_incluido_en` ∈ {fabricacion, montaje} — responde la pregunta esencial 3 de una vez.

## 4. FAB → MONTAJE · `tarifas_taller.json`
Habilita el **motor del lazo** (comparar `$_fab` de dos loteos).

```json
{
  "version": "1.0", "proyecto": "MAILENA_EWS", "fecha": "2026-08-12", "fx_usado": 18.5,
  "hh_ton_por_tipologia": {"pesada": 20, "mediana": 50, "liviana": 95},
  "usd_hh_taller": 14.5,
  "usd_ton_material_proceso": 260,
  "costo_cambio_lote_usd": 850,
  "penal_lote_chico_usd_ton": {"umbral_ton": 30, "recargo_usd_ton": 18},
  "capacidad_taller_ton_sem": 120
}
```
- `costo_cambio_lote_usd`: setup/reacomodo de taller por cambiar de lote (más lotes = más cambios).
- `penal_lote_chico`: recargo por lote bajo el umbral (pierde eficiencia de nido/corte).
- Estos dos parámetros son los que hacen que el loteo de montaje "le cueste" algo a fabricación —
  sin ellos el lazo siempre preferiría lotes chicos.

## 5. FAB → MONTAJE · `bom_contractual.json`
Cierra el gate **G1** automáticamente.

```json
{"version": "1.0", "proyecto": "MAILENA_EWS", "fecha": "2026-08-12",
 "toneladas_contractuales": 361.2, "fuente": "Catalogo SC-XXXX partida 4.1",
 "firmado_por": "Daniel", "fecha_firma": "2026-08-12"}
```
Si `toneladas_contractuales` difiere del IFC en más de ±3%, el módulo de montaje se detiene (G1).

---

## 6. MONTAJE → FAB · `lotes_montaje.json`
La secuencia de mínimo recurso convertida en lotes para producción.

```json
{
  "version": "1.0", "proyecto": "MAILENA_EWS", "fecha": "2026-08-12",
  "LOTE_TON": 50,
  "lotes": [
    {"lote_id": "M-01", "orden": 1, "toneladas": 44.8, "frente": "F1",
     "categorias": ["PLACA_BASE", "COLUMNA"], "marcas": ["PB-01", "C-01"],
     "pick_max_kg": 15079, "pull_date": "2026-10-19",
     "hh_montaje_lote": 410, "grua_h_lote": 96}
  ],
  "secuencia_resumen": "placas+anclas -> columnas ext->int -> cinturon -> principales -> radiales -> domo(torres) -> anillo -> secundarias -> postes"
}
```
`pull_date = fecha requerida en obra = liberacion_fab + transporte + pulmon`.

## 7. MONTAJE → FAB · `senales_ingenieria_valor.json`
```json
{"version": "1.0", "proyecto": "MAILENA_EWS",
 "senales": [
   {"id": "IV-01", "tipo": "asiento_temporal", "detalle": "Angulos de asiento en trabes radiales evitan x2 del ciclo soldado", "ahorro_estimado_usd": 9000},
   {"id": "IV-02", "tipo": "pre_armado", "detalle": "Pre-armar domo en piso reduce 8 picks de grua 90-110t", "ahorro_estimado_usd": 14000}
 ]}
```
Tipos: {asiento_temporal, pre_armado, junta_soldada_a_apernada, reagrupar_despiece, subir_peso_assembly}.

## 8. MONTAJE → FAB · `calibracion_hist.json` (compartido)
El histórico de la RETRO. Fabricación lee de aquí el **NCR detectado en campo** (métrica `ncr_hh`
y causas) para calibrar su calidad; montaje lee sus bandas. Un solo archivo, dos consumidores.

---

## 9. Identidad de pieza (obligatorio en ambos lados)
Todo export que liste piezas usa: `{"marca": "TPP-3-01", "guid_ifc": "1xCerVH81EWh..."}`.
- La **marca** viene del despiece de fabricación (posición de assembly/cast unit).
- El **guid_ifc** es el GlobalId del elemento o assembly en el IFC.
- El generador de IFC debe exportar la marca en un pset legible (Tekla: `Assembly position`;
  Baysa: `Pset_BAYSA_Cubicacion.Marca` — ya existe en MAILENA ✓).

## 10. Checklist del TEMPLATE ÚNICO de export IFC (lado generador)
Configurar en Tekla y guardar como plantilla de empresa:

| Opción | Valor | Por qué |
|---|---|---|
| Esquema | **IFC4** | Compatibilidad total con el extractor |
| ViewDefinition | Structural / DesignTransfer (NO ReferenceView solo) | ReferenceView pierde cantidades |
| **Assemblies** | **On** | Unidad de izaje real (sin esto, cada placa cuenta) |
| **Bolts** | **On** | Tornillería con `Bolt count` + `Location` |
| **Welds** | **On** | Cómputo de soldadura de campo (verificación obligatoria) |
| Property sets | On, incluir Tekla Quantity (Weight) | Peso sin recurrir a geometría |
| Pset_BAYSA_Cubicacion | Incluir (Marca, Categoria, Peso_kg, Material) | Categorías de izaje + identidad |
| Location by | Global | Alturas y placement correctos |

**Regla:** ningún IFC entra al pipeline de presupuesto sin este template. El extractor v4 tolera
exports pobres, pero cada opción apagada convierte un dato en un supuesto amarillo.

## 11. Flujo operativo del lazo (resumen)
1. FAB exporta IFC (template §10) + los 5 JSON (§1–5) → paquete al proyecto de montaje.
2. MONTAJE corre `presupuesto-montaje` (ingesta → driver → motores) sin supuestos amarillos.
3. MONTAJE corre el **motor del lazo** (`motor_lazo.py`) → compara loteo fab vs loteo montaje →
   **frontera $–T** → Daniel elige el punto (Zona Humana).
4. MONTAJE devuelve `lotes_montaje.json` + `senales_ingenieria_valor.json` → FAB reprograma.
5. Al cierre de obra: RETRO → `calibracion_hist.json` compartido → ambos lados calibran.
