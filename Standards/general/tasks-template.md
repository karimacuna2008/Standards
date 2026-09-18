# tasks.md — Execution Buffer Template

Template for `tasks.md`, the ephemeral file at the project root that holds only the current cycle's atomic tasks for the Executor (see `AGENTS.md` §7). It's listed in `.git/info/exclude` (never committed) and gets rewritten at the start of each new dispatch cycle.

**When to use this template:** every time the Architect closes a planning cycle and is about to generate a dispatch prompt (`AGENTS.md` §5.A).

**Note on language:** the fenced `tasks.md` template below is written in Spanish on purpose — it's the literal payload copied into a real project, matching the language Karim actually writes project files in. See `Standards/README.md` decisions log ("Excepción de idioma") for why this file is exempt from the English-only rule inside the fence.

## Template

```markdown
# tasks.md — Buffer de Ejecución

## Plan de origen
`docs/superpowers/plans/<archivo>.md` — [una línea de contexto de qué fase de ese plan cubre este ciclo]

## Worktree
[Nombre/ruta del Worktree activo asignado por el Arquitecto para este ciclo — ver `AGENTS.md` §5.A]

## Tareas
- [ ] [Tarea atómica 1 — verbo + qué archivo/función]
- [ ] [Tarea atómica 2]
- [ ] [Tarea atómica 3]
- [ ] **Validación final** — antes de cerrar el ciclo, relee cada tarea ya marcada `[x]` contra el estado real de los archivos en "Archivos objetivo". No marques esta casilla hasta confirmar que lo hecho corresponde 1:1 a lo pedido.

## Archivos objetivo
- `ruta/al/archivo1.ext` — [qué cambia]
- `ruta/al/archivo2.ext` — [qué cambia]

## Comandos de test
Correr después de cada tarea marcada `[x]`, y obligatoriamente antes de cerrar el ciclo (ver `SKILL.md` del módulo correspondiente):
```bash
[comando(s) de test]
```

## Si algo falla
1. Detente de inmediato — no sigas a la siguiente tarea.
2. Vuelca en `error_dump.txt` (raíz del proyecto): stack trace completo, comando que lo produjo, y qué tareas de esta lista ya estaban en `[x]`.
3. Dile al usuario: *"Ejecución pausada por error. Por favor entrega `error_dump.txt` al Arquitecto para re-evaluación."*
4. No borres este `tasks.md` ni marques más tareas hasta recibir un nuevo prompt de despacho.

---

**Ciclo iniciado:** [fecha]
```

---

**Version:** 1.0
**Last Updated:** 2026-09-11
