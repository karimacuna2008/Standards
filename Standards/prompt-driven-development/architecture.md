# Architecture & Generated-Code Quality

Applies regardless of Code author (Prompts or Direct).

## Thin orchestrator and file organization

Each module gets its own folder. Inside it:

- Exactly one file carries a leading underscore: the module's thin
  canvas/orchestrator (`_<Modulo>.tsx`) — state and wiring only, nothing
  else. This is the only underscore anywhere in the module, no matter how
  deep the nesting below it goes.
- Anything the canvas delegates to — a step of a flow, or an independent
  sub-module/tab — gets its own folder named after it (no underscore), but
  only when that step/sub-module actually has components of its own to
  colocate. A trivial step/sub-module with nothing else to colocate stays a
  flat file directly in the module folder, same tier as a regular component.
- Inside a step/sub-module folder: a file with the same name as the folder
  (its own local view/orchestrator, still no underscore) plus the
  components that belong specifically to it, flat alongside it.
- If a sub-module is itself complex enough to split further, reapply the
  same rule one level deeper (folder + same-named file + its own
  components) — the leading underscore never reappears below the module's
  own canvas.

This is a general separation-of-concerns rule, not framework-specific: in a
backend or script context it translates to a thin entrypoint with logic
factored into its own modules/functions. In a component-based frontend it
also avoids the editor-tab/quick-open ambiguity of every folder being named
`index.tsx`, and mirrors the precedent Next.js sets with
`_app.tsx`/`_document.tsx` for framework-level entry files.

`docs/Estatus Actual/**` mirrors this same nesting 1:1 (see
`project-structure.md`).

## Design tokens / centralized config

Centralize repeated values (colors, spacing, etc.) as named tokens instead
of scattering literals through the code.

- If a full retrofit of existing code to tokens is feasible in one pass, do
  it.
- Otherwise: the first time you touch a piece of old code for any reason,
  align it to the current token/config standard before doing anything else
  requested for that visit. If nothing else was requested, the visit is
  alignment-only.

## `Deuda tecnica/` (technical debt folder)

Applies to any project with a backend/DB/external service still under
construction, or a dependency on another of the user's own apps.

- No code in this folder — only a description of what's missing and where.
- Base files: `DB.md` (schema/constraint gaps), `API.md` (endpoint/model
  gaps), `PENDIENTE.md` (product/scope decisions not yet made — check
  before assuming something is already decided).
- Extend freely with topic-specific files as needed (e.g. `NEXUS.md` for
  what this project needs from another of the user's apps called Nexus,
  `NEST.md` for a third-party system).
- When a gap surfaces while drafting a prompt or writing code, record it in
  the matching file immediately — don't leave it only in the conversation.
