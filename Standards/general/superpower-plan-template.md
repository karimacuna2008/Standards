# superpower-plan-template.md — Architecture Plan Template

Template for the plans that live in `docs/superpowers/plans/YYYY-MM-DD-name.md` (see `AGENTS.md` §7). Written by the Architect (Claude) in the same session where it designs and audits — no separate "planning" session is required (see `AGENTS.md` §3).

**When to use this template:** when starting the design of any feature/change that the multi-agent system is going to dispatch to an Executor.

**Note on language:** the fenced plan template below is written in Spanish on purpose — it's the literal payload copied into a real project's `docs/superpowers/plans/`, matching the language Karim actually writes project plans in. See `Standards/README.md` decisions log ("Excepción de idioma") for why this file is exempt from the English-only rule inside the fence.

## Template

```markdown
# Plan: [Nombre del feature/cambio]

`docs/superpowers/plans/YYYY-MM-DD-nombre.md`

## 1. Contexto
[Qué problema/objetivo motiva este plan, y por qué ahora — 3 a 5 líneas. Referencia el estándar de `Standards/` consultado por la Ley de Estándares Absolutos (`AGENTS.md` §1).]

## 2. Decisiones de diseño
[Enfoque elegido y alternativas descartadas, con la justificación de por qué — mismo espíritu que `planning.md` §5, aplicado a este ciclo puntual.]
- **Decisión:** ...
  **Alternativas consideradas:** ...
  **Por qué esta:** ...

## 3. Diseño técnico (especificación para el Ejecutor)
Para cada función/método nuevo o modificado que el Ejecutor va a construir, detalla:

| Función/Método | Funcionalidad | Parámetros | Valor de retorno | Variables nuevas |
|---|---|---|---|---|
| `nombre_funcion()` | [lógica interna y propósito] | [arg → qué controla] | [qué devuelve y su impacto en el flujo posterior] | [variables creadas/usadas relevantes] |

Esta tabla es la especificación que el Ejecutor sigue sin ambigüedad. Cumple el mismo propósito de justificación y aprobación que antes cumplía el desglose técnico del Arquitecto cuando escribía código directamente — ahora apunta a lo que el Ejecutor va a construir, no a lo que el Arquitecto tecleaba.

## 4. Intentos previos y fallos
[Se llena solo en re-evaluaciones, tras leer un `error_dump.txt` (`AGENTS.md` §5.C). Una entrada por intento fallido — no se borran, para no repetir el mismo enfoque.]

- **Intento 1** ([fecha]): [qué se intentó] → **Falló porque:** [causa raíz extraída del dump] → **Ajuste:** [qué cambia en el plan/`tasks.md`]

## 5. Desglose de tasks
[Tareas atómicas que alimentan el próximo `tasks.md` — ver `tasks-template.md`.]

**Worktree:** [uno por este plan completo (default), o uno por tarea si son explícitamente paralelas e independientes — ver `AGENTS.md` §5.A]

1. [Tarea] — archivo(s): `...`
2. [Tarea] — archivo(s): `...`

## 6. Puerta de validación
Presentado al usuario el [fecha]. **Aprobado:** Sí/No — [una línea si hubo ajustes tras la primera presentación]

---

**Plan:** `docs/superpowers/plans/YYYY-MM-DD-nombre.md`
**Estado:** [Diseño / Despachado / En ejecución / Completado]
**Última Actualización:** [fecha]
```

---

**Version:** 1.0
**Last Updated:** 2026-09-11
