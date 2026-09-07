# Project Kickoff

Run this intake before starting a new project under this standard. Three
entry points — pick the one that matches:

- **New project** — nothing exists yet. See "New project" below.
- **Resuming an existing project already under this standard** — it
  already has `docs/Estatus Actual/`, `Deuda tecnica/`, etc. See "Resuming an
  existing project" below.
- **Adopting a project that already exists but was never built under this
  standard** (a retrofit) — see "Adopting an existing project" below.

## New project — ask before anything else

Ask the user about each of these (batched or one at a time, whichever fits
the conversation) before recommending anything:

1. **Purpose:** what problem this app solves and who uses it — a
   plain-language description of the goal, not a feature list yet.
2. **Reference material:** any existing notes, docs, mockups, or similar
   apps to use as a starting point. Ask for exact paths/links — never
   assume none exist.
3. **High-level scope:** what modules/views/capabilities the user already
   has in mind. Keep this high-level — the detailed spec per module/view
   comes later, through `spec-and-plan-gate.md`.
4. **Delivery target:** downloadable (desktop, e.g. Electron/native), web,
   mobile, or more than one of these. Always ask explicitly — this drives
   the stack recommendation below.
5. **Existing backend/data:** whether there's already an API, a database,
   or another system (including the user's other apps) this project must
   integrate with. Get exact names/repos/paths. If nothing exists yet,
   confirm whether backend is in scope for this project or belongs to a
   separate one.

## After the intake: recommend, don't just record

Using the answers above, produce a recommendation — not a transcription —
covering:

- **Stack:** language(s) and framework(s) for front end and (if in scope)
  back end, justified against the delivery target and the existing
  backend/data from questions 4-5.
- **Project axes** (see `project-structure.md`): recommend a Code author
  (`Prompts` or `Direct`) and a Project target (`Final product` or
  `Mockup/Documentation`), and get explicit confirmation before locking
  them in.
- **Initial module/view breakdown**, when the scope from question 3 is
  broad enough to need decomposing into sub-projects (same scope-check
  used by `brainstorming`).

Get explicit approval on this recommendation before creating the project's
folder structure (`project-structure.md`) or writing the first line of
`CLAUDE.md`.

## Resuming an existing project

Before making any change, read in this order: `CLAUDE.md` (rules + active
threads), `docs/Estatus Actual/INDEX.md` (current state),
`Deuda tecnica/PENDIENTE.md` (open product decisions), and
`Prompts/SESIONES.md` when Code author = `Prompts`. Only ask the user what
to work on next after this read — don't ask questions these files already
answer.

## Adopting an existing project (not built under this standard)

Use this instead of "New project" when the app already exists and works,
but was never structured under this standard — no `docs/Estatus Actual/`, no
`Deuda tecnica/`, no declared axes yet.

**Two folders, not one.** The source app being read usually lives in a
different folder than the one this session is rooted in. Read the source
from the path the user gives you, but build everything from step 6 onward
inside the current project's own folder — never write into the source
app's folder unless the user explicitly says to work in place.

1. **Quick orientation pass.** Skim the README, dependency manifest, and
   entrypoints — just enough to avoid asking what the code already answers
   (rough purpose, current stack, rough structure).
2. **Deep read of the entire codebase.** Read every script/module in
   detail — not a skim. Build a real understanding of what each part does,
   how it's organized, what's duplicated/dead/inconsistent, whether the
   current stack still fits what the app needs to do, and **which modules
   are actually active/working today versus present as code but not yet
   wired in or functional.**
3. **Write one consolidated TXT** in `Planeacion/Contexto/` (e.g.
   `Planeacion/Contexto/PREGUNTAS_RETROFIT_<Proyecto>.txt`) — never the project
   root; see the hard rule in `project-structure.md` — with everything from
   step 2 that needs the user's input before deciding anything, in this
   order:
   - Open questions/doubts about functionality or architecture found during
     the read — anything ambiguous, undocumented, or that could be
     intentional or accidental.
   - Stack evaluation: whether the current stack/framework is still the
     right choice, or whether switching (e.g. to Angular, Next.js, or
     anything else that fits better) would serve the app better. Ask this
     explicitly — never assume the existing stack stays just because it's
     already there.
   - Known pain points/wishlist the user already has for this app.
   - External consumers: anything else (including the user's other apps)
     that depends on this app/its API/its data. These are hard constraints,
     not optional scope — record them before touching anything they rely on.
   - Retrofit pace: all at once, or module by module across multiple
     sessions (default to module by module for a large app).
   - **Module scope checklist:** one line per module found during the
     read, each with a checkbox (`[ ] NombreModulo`), flagging which ones
     look active/working today versus present as code but not actually
     active/wired in yet. The user marks with an X which modules enter v1
     of the refactor. Anything left unmarked — whether it's not-yet-active
     code or simply deferred — stays out of v1 with no current priority.
4. **Wait for the user's answers** in that same file.
5. **Synthesize and present, then lock in only after approval.** Turn the
   answers into a concrete plan: final stack (kept or changed) with
   justification, the two axes (see `project-structure.md`), the retrofit
   pace, the confirmed v1 module scope, and a preview of the `App/`
   scaffold's folder tree. Present this plan and get explicit approval
   (the project's validation-gate rule already requires this for any write
   — this step exists so the requirement doesn't depend on remembering
   that rule). Nothing in step 6 happens until the user confirms.
6. **Generate the new `App/` (or equivalent) folder scaffold** in the
   current project's own folder — real folders with the correct
   organization (see `architecture.md`'s file organization rules), and the
   key files as placeholders for every module marked into v1 — not
   necessarily every single file, just the structural/entry ones. Any
   module that exists in the source but wasn't marked into v1 (inactive or
   simply deferred) gets a bare placeholder only (e.g. a stub screen/route
   marked "under construction"), and its deferral gets recorded in
   `Deuda tecnica/PENDIENTE.md` so it isn't lost. The result is ready to be
   imported into the external tool (when Code author = `Prompts`) or worked
   on directly, generating only what's actually needed next.
7. From there, populate `docs/Estatus Actual/` module by module — only
   for the modules in v1 scope — as each one gets its real implementation
   ported over, using the normal batch cycle (`workflow.md`) gated by
   `spec-and-plan-gate.md` per change.
