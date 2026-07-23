# <Nombre del proyecto>

> **Núcleo común.** Las secciones *Reglas inviolables (universales)*, *Flujo de trabajo*,
> *Definición de hecho*, *Antes de tocar código* y *Validación automática* son idénticas en
> todos los proyectos: no las edites por proyecto, edítalas en la plantilla compartida.
> El flujo completo (modelo, plan mode, sesiones paralelas) vive en `docs/modelo-trabajo-agentico.md`.

## Qué es
<1–2 frases: qué hace el sistema y qué piezas integra (BD, colas, servicios externos, agente…).>

## Arquitectura — repos en este workspace

| Repo | Lenguaje | Rol |
|---|---|---|
| `<repo>` | `<lenguaje>` | `<rol>` |

## Reglas inviolables (universales)

- **No commits a `main`/`master` directos.** Branch + PR, siempre.
- **Nunca commitear secrets ni `.env`.** Sensibles vía `direnv`/entorno, jamás en el repo.
- **Tests pasan antes de pedir review.** Si no pasan, dilo explícitamente.

## Reglas inviolables (de este proyecto)

- <p.ej. BD de dev SIEMPRE local: Postgres `localhost:<port>`, Mongo `localhost:<port>`… Jamás servicios remotos desde aquí.>
- <p.ej. Migraciones nunca se editan retroactivamente — siempre nueva migración.>
- <p.ej. Dumps de BD no van a git (configurado en `.gitignore`).>
- <p.ej. El contrato entre `<servicio A>` y `<servicio B>` se documenta en `<ruta>`.>

## Flujo de trabajo

- **Una tarea = una rama = un PR.** No mezcles tareas en una rama.
- **Nomenclatura:** `feature/<issue>-desc`, `fix/<issue>-desc`, `chore/<issue>-desc`.
  El `<issue>` enlaza la rama con el GitHub Project.
- **Toda tarea parte de un issue.** Si no existe, créalo antes de empezar.
- **Cada rama sale de un `main` actualizado y el PR va contra `main`.** Antes de crear la rama: `git checkout main && git pull`. La `base` del PR es **siempre `main`**.
- **Un PR a la vez (secuencial).** Mergea el PR → `git checkout main && git pull` → recién entonces arranca el siguiente. **No** empieces una tarea brancheando desde otra rama de feature aún sin mergear.
- **No apilar PRs.** Si una tarea depende de otra sin mergear, espera a que se integre. Si apilar es inevitable: la `base` del PR dependiente debe ser **`main`** (no la rama padre); mergea primero el padre, **rebasa el hijo sobre el `main` actualizado** y verifica que el diff sea **solo tu cambio** antes de mergear.
- **⛔ Nunca mergees un PR cuya `base` sea otra rama de feature** — se integra en esa rama, no en `main`. (Pasó con `validar-desktop` #19/#20: quedaron dentro de la rama del backbone y hubo que hacer un PR de consolidación a `main`.)
- **Tras mergear:** borra la rama integrada (local y remota) y haz `git pull` de `main` antes de la siguiente tarea.
- **Tareas en paralelo sobre el mismo repo → git worktree**, nunca dos sesiones sobre el mismo working tree:
  ```bash
  git worktree add ../<repo>-<feature> -b feature/<issue>-<desc>
  # al terminar:
  git worktree remove ../<repo>-<feature>
  ```
- **Ante ambigüedad o bloqueo, detente y márcalo** en el issue/PR. No improvises una solución que se aleje de la spec.

## Definición de "hecho" (DoD)

Un PR está listo para review cuando:
- [ ] Tests verdes y lint limpio (ver hooks).
- [ ] Pasa la revisión de calidad (code-reviewer / skill `senior-qa`).
- [ ] La descripción enlaza el issue (`Closes #<issue>`) y la spec en docs.
- [ ] Si tocaste un contrato entre servicios, su spec se actualizó en este mismo PR.
- [ ] PR pequeño y enfocado (una sola tarea).

## Stack y tests

- `<repo>` → `<comando de test>`

## Cómo levantar entorno local

```bash
<comandos para levantar BD, servicios y apps>
```

## Skills configuradas
Las skills en `.claude/skills/` se aplican automáticamente cuando corresponde: `<lista>`.

## Convenciones cross-repo
- <reglas de coordinación entre repos: qué cambios requieren justificación, versiones, dependencias de plataforma…>

## Antes de tocar código
Lee siempre `<repo-de-docs>/` para entender el feature. Si una spec contradice el código, marca el conflicto en lugar de asumir cuál tiene razón.

## Validación automática
Hooks que corren al editar: linter por archivo (PostToolUse) y tests al cerrar turno (Stop). Si recibes feedback de lint/test tras una edición, corrígelo antes de continuar.
