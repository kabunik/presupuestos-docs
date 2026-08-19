# motor_calculo_spec.md — Sistema DM v1.7
Contrato del motor de cálculo. Rige junto con input_schema.json y output_schema.json.
Orden de resolución de conflictos: documento rector → este kit → Excel → narrativa.

## Invariantes (LEY — ninguna implementación puede violarlos)
1. Peso único AISC 303-22 §9.2 para material y MO.
2. Eje de complejidad único HH/ton (24/40/60/90): mueve desperdicio, soldadura, conexiones y horas.
3. Una clase por proyecto o subproyecto; subproyectos por familia si difieren ≥1 salto del eje;
   alerta_interferencia_familias cuando una familia de HH altas satura habilitado y reduce la
   productividad de familias más rentables — nunca se suprime; decisión humana. (v1.7)
4. Doble chequeo del peso (IFC vs reconstruido) antes de emitir.
5. Exclusiones explícitas (escaleras, F1554, Nelson studs) — nunca cero silencioso.
6. Precio como composición única; "Más 6%" con % de crecimiento por defecto atado al semáforo
   RSS: verde 6% / ámbar 10% / rojo 15%; ajustable por perfil, SIEMPRE registrado; reconciliación
   APU vs Más X% < 1% a margen comparable. (v1.7)
7. RSS coherente con contingencia por fase.
8. Precios con vigencia + compuerta confirmar_precios_mp: sin confirmación explícita registrada
   NO se emite precio de licitación. (v1.7)
9. Métrica Lean única (ahorro HH/ton) para motor y gainsharing.
10. El concurso RIGE sobre la norma; desviaciones registradas y costeadas.
11. El plan de montaje es ENTRADA; déficit explícito (OT prima 50% / subcontrato all-in).
12. Doble chequeo de velocidad; advertencia_bajo_equilibrio nunca se suprime.
13. Inventario primero: requisición neta; préstamos registrados.
14. S12 optimizado: carga en HH y t/sem + % ocupación habilitado (alerta ≥100%); minimizar
    retraso (OT primero, desplazar sólo excedente, retraso = ceil(HH/HH_sem)); advertencia
    obligatoria de comunicar el retraso al cliente afectado ANTES de comprometer; C nunca se
    recomienda bajo equilibrio.
15. Costo empresa real ($/HH, tolerancia 2%); overhead cubre cargas; todo ANTES de impuestos.
16. Cierre de proyecto obligatorio; desviación > tolerancia (±5% default) → propuesta de
    recalibración con evidencia; parámetros NUNCA se recalibran solos (aprobación dirección +
    versionado); cada presupuesto ligado a su versión de parámetros; sin cierre no alimenta
    recalibraciones. (v1.7 NUEVO)

## Secciones del motor
§4.1 Peso y factores (planos → facturable → comprar; doble chequeo; exclusiones).
§4.2 Boom por familias.
§4.3 APU AISC (material E3×peso; MO propio; MO subcontrato del pico S11; indirectos; margen).
§4.4 Más X% (peso propuesto = planos×(1+crecimiento); crecimiento default por fase RSS,
     registrado; doble MO prefab+fab; reconciliación <1%).
§4.5 Costo empresa (E5; tolerancia 2%; chequeo cargas<overhead).
§4.6 Desviaciones de concurso (rigen; costo al presupuesto).
§4.7 Lotes y secuencia por plan de montaje contra planta CARGADA; déficit explícito.
§4.8 Chequeo de grúa (piezas críticas, costo de partir; valida EOR).
§4.9 S11 doble velocidad (alerta_subcontrato_hh = max(0,pico−v_ot)×HH/ton;
     advertencia_bajo_equilibrio nunca suprimible).
§4.10 Inventario primero (neta = bruta − libre; préstamos registrados).
§4.11 S12 programa optimizado (carga combinada t/sem y HH/sem; ocupación; escenarios A/B/B*/C;
      retraso mínimo; pena máxima; advertencia de comunicación).
§4.12 Interferencia de familias (activa cuando familia de HH altas satura habilitado y reduce
      productividad de familias más rentables; payload completo del output_schema). (v1.7)
§4.13 Flujo de caja y capítulo financiero; precio de licitación = APU + financiero.
§4.14 RSS (sqrt(sum(c_i^2))) y semáforo; gobierna contingencia y default de crecimiento. (v1.7)
§4.15 Compuerta confirmar_precios_mp y leyenda_alcance en la emisión. (v1.7)
§4.16 Cierre y recalibración (rubros del cierre; tolerancia; propuesta con evidencia;
      versionado; presupuesto↔versión). (v1.7 NUEVO)

## Criterio de aceptación permanente
golden_test.json debe reproducirse EXACTO en CI en cada commit. Un número distinto sin cambio
aprobado de parámetros = build roto.

## Interfaz reservada
Módulo de estimaciones de montaje (siguiente desarrollo): consumirá plan_montaje (E6) y
S8/S9/S11. No romper esos contratos.
