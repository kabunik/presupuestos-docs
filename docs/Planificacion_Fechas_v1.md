# Planificación de fechas — v1 (propuesta para validar con dcode)

**Fecha de elaboración:** 2026-07-23 · **Estado:** propuesta — validar en la reunión de sync (J1.3)
**Materializada en:** [GitHub Project 2](https://github.com/orgs/kabunik/projects/2), campos `Inicio`/`Fin` (vista Roadmap) y due dates de milestones.

## Resumen ejecutivo

| Hito | Fecha | Contenido |
|---|---|---|
| Contrato v0 congelado | **14 ago 2026** | C1 cerrado — desbloquea el paralelismo real |
| **M0 Toyota** | **4 sep 2026** (6 semanas) | Intake + percepción one-shot + visor 3D del primer output |
| **M1 Motor** — *app funcional* | **16 oct 2026** (12 semanas) | Cotización calibrada, alertas, escenarios, flujo de caja, BOM editable |
| **M2 Edición viva** — *el diferenciador* | **27 nov 2026** (18 semanas) | Chat de edición + reconstrucción del visor en tiempo real |
| M3 Ferrari | sin fecha (arranque ~30 nov) | Solo K7.4 con fechas indicativas; el resto se planifica al cerrar M2 |

**Lectura frente a la estimación de 2–3 meses:** la *app funcional* (M1) cae en la banda alta de esa estimación (~12 semanas desde el 27-jul). El diferenciador completo (M2, edición conversacional en vivo) requiere ~4 meses. Empujar M2 dentro de los 3 meses no es realista: es el mayor riesgo técnico declarado (latencia, recálculo incremental) y depende de que K3.2 (modelo canónico) y C1.2 (`model_delta`) estén maduros.

## Dónde ayuda la IA (y dónde no)

Desarrollar con Claude Code + harness de testing + CI/CD comprime sobre todo el workstream Kabunik: superficies UI (K1, K4, K5), API/backend (K3), infra (K8). Por eso las ventanas de esas tareas son cortas (2–3 semanas con solape).

No comprime:
- **C1 (contrato):** es negociación y acuerdo entre dos equipos, no tirar código. 3 semanas con congelación el 14-ago.
- **D2.1 (percepción):** calidad iterada contra evals; es el "long pole" de F0 (6 semanas completas).
- **J1.4 (insumos de Daniel):** dependencia externa; sin sus casos, D4 no arranca (por eso J1.4 cierra el 21-ago, antes del inicio de D4 el 7-sep).

## Camino crítico

```
C1 (congela 14-ago) → K3.2 (modelo canónico, 4-sep) → K2.3 (render canónico, 25-sep)
                                                    → D1.4 + K2.4 + K5 (F2, oct–nov)
D2.1 (percepción, 4-sep) → K2.2 usa fixture K2.2a para no esperar
J1.4 (21-ago) → D4.* (eval harness, desde 7-sep)
```

Si el contrato se desliza, **todo lo posterior se desliza semana a semana**. Es la fecha a proteger.

## Supuestos

1. Arranque formal de la planificación: **lunes 27-jul-2026** (dcode ya avanza en propuesta tecnológica del agente; Kabunik en prototipo/mockups con user flows — ambos encajan en las dos primeras semanas de F0).
2. Fases de 6 semanas: F0 (27-jul→4-sep), F1 (7-sep→16-oct), F2 (19-oct→27-nov). F2 incluye margen de hardening en su última semana.
3. Las fechas de dcode (D*) son **tracking**: dcode confirma o corrige en la reunión de sync; aquí solo marcan la ventana de integración esperada.
4. `Estimación` sigue vacía: fechas ≠ esfuerzo; cada equipo estima lo suyo.
5. K7.4 (F3) lleva fechas indicativas (30-nov→23-dic) solo para que la vista Roadmap muestre el horizonte.

## Riesgos que moverían fechas

- **Latencia de reconstrucción en vivo (F2):** si el spike K2.1 revela que la tecnología del visor no da la latencia, F2 puede necesitar re-arquitectura → +2-4 semanas.
- **Percepción (D2.1) con calidad insuficiente al 4-sep:** M0 puede salir con la compuerta híbrida (D2.3) adelantada.
- **Insumos de Daniel tarde (J1.4):** desplaza D4 y la confianza en calidad de M1.
- **Concurrencia de ediciones + human gate (K3.3):** punto de integración delicado; diseñarlo dentro de C1, no después.
