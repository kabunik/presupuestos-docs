# Planificación de fechas — v2 (propuesta para validar con dcode)

**v1:** 2026-07-23 · **v2:** 2026-08-06 — revisada tras las decisiones D1/D2 de #67 (método Sistema DM v1.7).
**Estado:** propuesta — validar en la reunión de sync (J1.3, #64).
**Materializada en:** [GitHub Project 2](https://github.com/orgs/kabunik/projects/2), campos `Inicio`/`Fin` (vista Roadmap) y due dates de milestones.

> **Regla que sigue vigente:** fechas ≠ esfuerzo. El campo `Estimación` sigue vacío; cada equipo estima
> lo suyo. Las fechas de dcode (`D*`) son **ventanas de integración esperadas**, no compromisos: dcode
> confirma o corrige en el sync.

## Qué cambió de v1 a v2

| # | Cambio | Efecto en fechas |
|---|---|---|
| 1 | **J1.4 entregado.** Los parámetros de calibración y los dos casos de evaluación ya estaban en el repo (ver `00-Fundamentos/ACTIVOS_REUTILIZABLES.md`) | **Gana ~4 semanas.** D4 se desbloquea ya, no el 21-ago. Es la única mejora neta de esta revisión |
| 2 | **C1.1 es mapeo, no diseño desde cero.** Los esquemas del kit existen y están validados | Reduce trabajo de C1, pero ver #4 |
| 3 | **El contrato no puede congelarse sin resolver #68 y #70.** Ambos cambian la forma del payload | **Riesgo sobre el 14-ago.** Ver «Decisión requerida» |
| 4 | **D3 cambia de naturaleza:** ya no es portar el `.jsx`, es implementar `motor_calculo_spec.md` §4.1–4.16 contra un golden test de tolerancia cero | Alcance más grande y más definido. dcode debe reestimar |
| 5 | **El inventario de tools de la Segmentación v2 §A.4 no cubre §4.1–4.16** | Alcance nuevo en D3. dcode debe reestimar |
| 6 | **Alcance v1 ampliado (D2):** S11/S12, ocupación de habilitado, escenarios de programa, interferencia de familias | Crece M1 y M2 |
| 7 | **Inv. 16 (cierre y recalibración) se adelanta de F3 a v1** | Necesita hito propio: **M2.5** |
| 8 | **D1 pasa de 1 compuerta humana a 5**, dos bloqueantes | Crece D1.2 y K3.3 |

## Resumen ejecutivo — v2

| Hito | Fecha | Estado | Contenido |
|---|---|---|---|
| Contrato v0 congelado | **14 ago 2026** | ⚠️ **en riesgo** | C1 cerrado — desbloquea el paralelismo real. Bloqueado por #68 y #70 |
| **M0 Toyota** | **4 sep 2026** | sin cambio | Intake + percepción one-shot + visor 3D del primer output |
| **M1 Motor** — *app funcional* | **16 oct 2026** → **a reestimar** | ⚠️ **crece** | Lo de v1 **+ S11/S12, ocupación de habilitado, escenarios de programa, interferencia de familias, compuerta de precios de MP** |
| **M2 Edición viva** — *el diferenciador* | **27 nov 2026** → **a reestimar** | ⚠️ **crece** | Chat de edición + reconstrucción del visor en tiempo real |
| **M2.5 Cierre y emisión** | **nuevo — sin fecha** | 🆕 | Cierre de proyecto, recalibración gobernada, versionado de parámetros, requisitos de emisión (leyenda, exclusiones, sello de versión). **Cierra la v1** |
| M3 Ferrari | sin fecha | reducido | Benchmarks agregados, RAG multicliente, nesting, puente a Strumis, analítica. **Ya no incluye el lazo cotizado-vs-ejecutado** |

**No se proponen fechas nuevas para M1, M2 y M2.5 en esta revisión.** El alcance creció por decisión de
dirección y reestimar sin dcode sería inventar. Ese es el punto principal a resolver en el sync.

## Decisión requerida en el sync: la fecha del contrato

El 14-ago está a 8 días y **hay dos huecos que cambian la forma del payload**, por lo que congelar antes
de resolverlos produciría un contrato que hay que reabrir:

- **#68 · catálogo real de alertas.** Afecta al esquema de `Alert` y al nombre de la tool
  `motor_alertas_22`. **Resuelto el 11-ago** (#68): catálogo cerrado y tool renombrada a `motor_alertas`.
- **#70 · plan de montaje en el intake.** `plan_montaje_t_sem` es *required* en E6; falta decidir el
  comportamiento cuando **no** hay plan de montaje (caso real y frecuente: CNARCCS cotiza con montaje
  excluido). Afecta a la firma de `/generate` y al ciclo de vida.

Tres salidas posibles, a elegir con dcode:

| Opción | Qué implica |
|---|---|
| **A · Resolver #68 y #70 antes del 14-ago** | Ambos son decisiones acotadas, no desarrollo. Viable si se priorizan esta semana. **Preferible:** protege la fecha |
| **B · Congelar el 14-ago un contrato v0 parcial** | Congelar sesión, modelo, deltas y streaming; dejar `Alert` y el plan de montaje como **extensiones marcadas** con política de versionado ya acordada. Permite arrancar el paralelismo sin reabrir el núcleo |
| **C · Deslizar el contrato** | Todo lo posterior se desliza semana a semana. **Última opción** |

Además hay que incorporar al contrato lo que los invariantes añaden: 2 endpoints de compuerta,
4 eventos de streaming, 2 servicios de datos y el parámetro `version` en `get_config_tenant`
(ver `00-Fundamentos/CONTRATO_INTEGRACION_v0.md` §4 y §5).

## Dónde ayuda la IA (y dónde no)

Sin cambios respecto a v1. Desarrollar con Claude Code + harness de testing + CI/CD comprime sobre todo
el workstream Kabunik: superficies UI (K1, K4, K5), API/backend (K3), infra (K8).

No comprime:
- **C1 (contrato):** es negociación y acuerdo entre dos equipos, no tirar código.
- **D2.1 (percepción):** calidad iterada contra evals; es el «long pole» de F0.
- ~~**J1.4 (insumos de Daniel)**~~ → **resuelto**: los insumos ya están en el repo.

**Nuevo en v2 — tampoco comprime:** **D3 contra el golden test**. La tolerancia es cero: no hay
«casi». Cada una de las 16 secciones §4.1–4.16 tiene su cifra exacta que reproducir, y el Excel de 28
hojas es el oráculo. Es trabajo de precisión, no de volumen, y la IA no lo acelera tanto como una
superficie de UI.

## Camino crítico — v2

```
C1 (congela 14-ago ⚠) → K3.2 (modelo canónico, 4-sep) → K2.3 (render canónico, 25-sep)
                                                      → D1.4 + K2.4 + K5 (F2, oct–nov)
D2.1 (percepción, 4-sep) → K2.2 usa fixture K2.2a  ← ahora de un IFC real (CNARCCS)
J1.4 ✔ entregado → D4.* puede arrancar YA (antes: 7-sep)
D3 (§4.1–4.16 contra golden test) → M1 · nuevo long pole del workstream dcode
inv. 16 (#73) → M2.5 · cierra la v1
```

Dos cambios de forma en el camino crítico:

1. **D4 se adelanta.** Al estar los casos disponibles, el eval harness puede construirse en paralelo a
   D3 en lugar de detrás. Eso mejora la confianza en la calidad de M1 — es donde mejor se aprovecha el
   tiempo ganado en el punto 1 del changelog.
2. **D3 se convierte en el long pole de dcode**, desplazando a D2.1. La percepción tiene un punto de
   partida escrito (`ROL.docx`) y dos casos reales contra los que medirse; el motor tiene 16 secciones
   con tolerancia cero y un inventario de tools incompleto.

Si el contrato se desliza, **todo lo posterior se desliza semana a semana**. Sigue siendo la fecha a
proteger.

## Supuestos

1. Arranque formal de la planificación: **lunes 27-jul-2026**.
2. Fases de 6 semanas: F0 (27-jul→4-sep), F1 (7-sep→16-oct), F2 (19-oct→27-nov). **F1 y F2 quedan a
   reestimar** por el crecimiento de alcance (cambios 5, 6 y 8).
3. Las fechas de dcode (`D*`) son **tracking**: ventana de integración esperada, no compromiso.
4. `Estimación` sigue vacía: fechas ≠ esfuerzo.
5. ~~K7.4 (F3) lleva fechas indicativas~~ → M3 se replanifica al cerrar M2; su alcance se redujo al
   salir de él el lazo cotizado-vs-ejecutado.
6. **Nuevo:** M2.5 no lleva fecha hasta que M1 y M2 estén reestimados.

## Riesgos que moverían fechas

Los cuatro de v1 siguen vigentes, con uno resuelto y tres nuevos:

| Riesgo | Estado |
|---|---|
| **Latencia de reconstrucción en vivo (F2)** — si el spike K2.1 revela que el visor no da la latencia → +2-4 semanas | Vigente. Agravado por el supuesto ⚙️ `costUpdate: vivo`: cada edición dispara recálculo de costos |
| **Percepción (D2.1) con calidad insuficiente al 4-sep** → M0 puede salir con la compuerta híbrida (D2.3) adelantada | Vigente, **mitigado**: hay punto de partida (`ROL.docx`) y dos casos reales |
| ~~**Insumos de Daniel tarde (J1.4)**~~ | **Resuelto** — entregados |
| **Concurrencia de ediciones + human gate (K3.3)** | **Agravado:** de 1 compuerta a 5, dos bloqueantes. Diseñarlo dentro de C1, no después |
| 🆕 **Contrato congelado sobre huecos** (#68, #70) | Reabrir el contrato después de congelarlo cuesta más que retrasarlo una semana |
| 🆕 **Golden test no reproducible** | Si D3 no reproduce las cifras exactas, M1 no cierra. El propio consultor señala los **redondeos** como riesgo: exige definir precisión decimal por campo en los esquemas |
| 🆕 **Frescura de E7** | Si la carga de planta y el inventario no se mantienen semanalmente, S11/S12 producen «ficción bien presentada» (palabras del consultor). Es requisito de producto, no de proceso — ver #71 |

## Documentos pendientes del consultor

Bloquean o degradan trabajo, y están pedidos:

| Documento | Qué bloquea |
|---|---|
| `claude/instrucciones-sistema-dm.md` **v1.7** (documento rector) | Máxima autoridad declarada del paquete. Sin él, los conflictos se resuelven un escalón más abajo |
| Peso de referencia del ing. de cálculo (CNARCCS) | Sin él no se puede medir el error de percepción contra la tolerancia objetivo ≤5% → degrada D4.1 |
| Catálogo de perfiles con kg/m de METALITEC | Necesario para `TenantConfig` real y para los guardarraíles de edición (rechazo por catálogo) |
