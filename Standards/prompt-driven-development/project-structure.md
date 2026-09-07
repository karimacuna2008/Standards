# Project Structure

Applies to any project following the prompt-driven-development pattern. Run
the intake in `kickoff.md` before any of this on a brand-new project — the
two axes below are its output, not something to guess ad hoc.

## Declare two axes in the project's CLAUDE.md, under "About this project"

- **Code author:** `Prompts` (an external tool generates code from prompts
  you write) or `Direct` (you write the code yourself, no prompt hand-off).
- **Project target:** `Final product` (continues to installable/deployable,
  connected to real services/APIs) or `Mockup/Documentation` (stops at mock
  data + full documentation, never connects to real services, never packaged).

## Standard folder layout

```
<project root>/
  CLAUDE.md
  App/                        # actual app code — rename to match the stack (e.g. src/)
  Planeacion/
    Contexto/                 # context + question files handed to the user
  Deuda tecnica/
    DB.md
    API.md
    PENDIENTE.md
    <OTRO>.md                 # extend freely, e.g. NEXUS.md, NEST.md
  Prompts/                    # only when Code author = Prompts
    SESIONES.md
    <NN>-ss-<sesion>-tk-<tarea>.txt
    Validated/
  docs/
    Estatus Actual/
      INDEX.md
      <Modulo>/
        _<Modulo>.md
        _<Modulo>-CHECKLIST.md
        _<Modulo>-RESPONSIVE.md
        <Componente>.md
    superpowers/
      specs/
      plans/
```

## Monorepo variant — multiple independently-deployable pieces

Use this instead of the single-`App/` layout above when the project ships more
than one thing that deploys **separately** (e.g. a desktop/web app + its own
API + its own database), all still tracked in **one** git repo:

```
<monorepo root>/              # the .git root — not inside any piece
  <Project> - Frontend/
    CLAUDE.md                 # lives once, in whichever piece is the day-to-day cwd
    cloudbuild.yaml
    App/
  <Project> - API/
    cloudbuild.yaml
    src/
  <Project> - DB/
    cloudbuild.yaml
    alembic/
```

- Each piece is a normal sibling folder under the repo root — **none of them
  has its own `.git`.** If a piece used to be its own repo, disconnect that
  old repo and fold its content in as a plain folder; don't keep two `.git`s
  nested inside each other.
- Each piece owns its own `cloudbuild.yaml` and gets its own Cloud Build
  trigger, filtered by path — see `deployment.md` §7.
- `CLAUDE.md` documents the **whole monorepo** from whichever piece's folder
  is the primary day-to-day working directory (or the repo root if none is
  primary) — not just that one piece. That piece's own planning folders
  (`Planeacion/`, `docs/`, `Prompts/`, `Deuda tecnica/`) stay local to it,
  following the single-app layout above; the other pieces don't need their
  own copies unless they grow complex enough to warrant it.
- Naming convention for piece folders: `<Project> - <Piece>` (e.g.
  `- Frontend`, `- API`, `- DB`) — consistent, sortable, and immediately
  readable in a file explorer next to its siblings.

## `Planeacion/Contexto/` — hard rule

Every context or question file written **for the user to answer or read**
goes in `Planeacion/Contexto/`. Never the project root, never `Prompts/`, never
`docs/`. This covers the retrofit intake of `kickoff.md`, any
pre-planning questionnaire, and any consolidated context handoff produced
later in the project's life. The rule holds regardless of Code author or
Project target.

Do not create a third "history log" file (e.g. a `CHECKLIST_V3.md`-style
changelog of what was done on each date). Current state lives in
`Estatus Actual/`; history lives in `git log`. Never maintain a document that
duplicates either.

## Document responsibilities

| Document | Answers | Update trigger |
|---|---|---|
| `CLAUDE.md` § Sesiones en curso | What is being worked on right now / what's next | On opening/closing a thread; delete the entry once it closes — do not accumulate history here |
| `docs/Estatus Actual/<Modulo>/_<Modulo>.md` + `<Componente>.md` | What exists today for that module/component and its current state | Synced the moment the user gives full checklist confirmation — see `workflow.md` step 7, not left for later |
| `docs/Estatus Actual/<Modulo>/_<Modulo>-CHECKLIST.md` | The module's complete, always-current manual-test checklist — every component/view/config that exists in it today | Updated (never wholesale-replaced) the moment the user gives full checklist confirmation — see `workflow.md` step 7 |
| `docs/Estatus Actual/<Modulo>/_<Modulo>-RESPONSIVE.md` | How the module holds up across screen widths/font scaling, and what's still broken | Run whenever the module gets a responsive audit — see `responsive-audit.md`; not tied to a batch's checklist confirmation like the two rows above |
| `Deuda tecnica/**` | What's missing in backend/DB/external apps | When a new gap surfaces |
| `Prompts/SESIONES.md` | Which external-tool session covers what, and which prompts it ran | On opening/closing a session, or after running a batch |
| `Planeacion/Contexto/**` | Context and question files written for the user to answer or read | Whenever one is produced — see the hard rule below |

## `Estatus Actual/` granularity rules

- One `.md` per component when the component has its own source file
  (mirror the codebase 1:1). Fold trivial components (one-line wrappers)
  into the parent module's `.md` as a subsection instead of creating a
  near-empty file.
- Each component `.md` contains: what it does; current state (done /
  partial / placeholder); the last batch that touched it; open items linked
  to `Deuda tecnica/` when relevant; and an "Interacciones" section with:
  - its own trigger/appearance condition, stated concretely (e.g. "opens
    when the X button in `ComponentY` is pressed; closes on
    confirm/cancel") — this belongs in the component's own file, not only
    as a link to somewhere else;
  - links to the components it interacts with, plus a link to
    `_<Modulo>.md` for the full sequence when more than two pieces are
    involved.
- Each `_<Modulo>.md` contains the module's full overview: what it is, how
  it works end to end, which role/user type uses it, and useful reference
  info (paths of every relevant file, key config, related external
  systems). It is also the **only** place that tells the full
  cross-component orchestration/flow (order of events, shared state). Never
  repeat that full narrative inside a component file — component files
  state only their own local trigger.
- Each `_<Modulo>-CHECKLIST.md` contains the module's complete manual-test
  checklist: every component, view, and configuration/edge case that
  exists in the module today, each item numbered. Unlike a batch log, this
  file is never wholesale-replaced — a batch updates only the items its
  changes affect (add items for what's new, edit items whose behavior
  changed, remove items for what was removed), so at any point it reflects
  the module's actual current state, never just the latest batch. Before
  using it to gate something (a deploy, closing out the module), cross-check
  it against `_<Modulo>.md` and every `<Componente>.md` to confirm nothing
  existing is missing from it — see `workflow.md`.
- When the module's code splits into step/sub-module folders (see
  `architecture.md`'s file organization rules), mirror that nesting here:
  `Estatus Actual/<Modulo>/<Step>/<Step>.md` (plus its own
  `_<Step>-CHECKLIST.md`) plays the same role for that step/sub-module that
  `_<Modulo>.md`/`_<Modulo>-CHECKLIST.md` play for the whole module, with
  one `.md` per its own component alongside it.
- `Estatus Actual/INDEX.md` is a lookup table: module/view/component → file
  to open. Update it whenever a file is added or removed.
- If a change touches two modules at once, add a note to both
  `_<Modulo>.md` files (and check both `_<Modulo>-CHECKLIST.md` files),
  cross-referencing each other.
- `_<Modulo>-RESPONSIVE.md` follows the same per-module pattern as the checklist, but for
  cross-screen-size behavior instead of functional correctness — see `responsive-audit.md` for
  what widths to test, testing method, and what a finding records.
