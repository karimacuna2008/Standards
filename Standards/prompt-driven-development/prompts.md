# Writing Prompts (external code-generation tool)

Applies only when the project's Code author axis = `Prompts`
(see `project-structure.md`).

## File naming

`{NN}-ss-{sesion}-tk-{tarea}.txt`

- `NN`: two-digit sequence number, global to the project, sequential, never
  resets across sessions.
- `sesion`: the exact session name as it appears in `Prompts/SESIONES.md`.
- `tarea`: short slug for the task/change this specific prompt makes.

Example: `107-ss-VALIDACION-tk-fix-motivo-ticket.txt`

## Folder lifecycle

- `Prompts/` holds pending prompts.
- `Prompts/Validated/` holds prompts already run and confirmed via the
  workflow cycle (see `workflow.md`). The user moves files there manually —
  never move files into `Validated/` yourself.

## Sizing

Prefer several small, focused prompts over one large prompt. Small prompts
are easier for the external tool to execute correctly and easier to
verify/debug afterward. Hand the user a full batch — as many small prompts
as make sense — so they can run all of them before you review.

## `Prompts/SESIONES.md`

Keep one table, one row per external-tool session:

| Session (exact name) | Covers | Prompts |
|---|---|---|

Update the "Prompts" column as prompts run. This table lives here, not in
`CLAUDE.md`, so `CLAUDE.md` stays short.

## First line of every prompt file

`Sesión: <session name>`
