# Kit de Desarrollo — Sistema DM v1.7
Contenido:
- input_schema.json / output_schema.json — contratos de datos (campos nuevos v1.7:
  alerta_interferencia_familias, confirmar_precios_mp, leyenda_alcance,
  propuesta_recalibracion, version_parametros; en input: fase_rss→default de crecimiento,
  confirmacion_precios, registro_cierres en E7).
- motor_calculo_spec.md — especificación §4.1–4.16 + 16 invariantes.
- golden_test.json — caso de referencia (LEY; correr en CI).
- ejemplo_input.json / ejemplo_output.json — fixtures del caso.
Orden de resolución de conflictos: documento rector → este kit → Excel → narrativa.
No construir en v1: IFC, simuladores gráficos, integración contable, recalibración automática
(prohibida por inv. 16) ni estimaciones de montaje (siguiente desarrollo; interfaz reservada).
