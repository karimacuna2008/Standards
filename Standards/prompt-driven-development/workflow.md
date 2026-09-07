# Batch Workflow

Applies once code generation starts, under either Code author mode from
`project-structure.md`.

## The cycle, per batch

1. Draft the batch: write prompts (Code author = Prompts) or implement
   directly (Code author = Direct). Check `spec-and-plan-gate.md` first to
   decide whether this batch needs a spec+plan before drafting/implementing.
2. (Prompts mode only) Wait for the user to run every prompt in the batch,
   paste the resulting code, and share the external tool's own summary of
   each prompt.
3. Verify against the REAL pasted/written code — never approve a batch from
   the external tool's summary alone. Run the stack's type-check/lint (e.g.
   `tsc --noEmit`, `ruff check`, `mypy`) and confirm it's clean.
4. If verification finds a mismatch or a failing check: judge the size of
   the fix.
   - Small and isolated (one or a few files, mechanical — a typo'd name, a
     broken import, a value never applied): propose the exact fix (which
     file(s), what change) and wait for the user's explicit approval before
     applying it — this still goes through the validation gate, it does not
     skip it. Once approved, apply it directly in the local repo, then tell
     the user precisely what changed so they can paste the same change into
     the external tool's session to keep it in sync with the repo.
   - Large, or touching several files/concerns: draft a corrective prompt
     for the external tool instead — same batch cycle (user runs it, pastes
     the result back).
   Either path: re-verify (code + checks) before moving on to the checklist.
   This branch applies only to fixes surfaced while verifying an
   already-running batch — never to original feature generation, which
   always goes through the tool under Prompts mode.
5. If the code matches what was asked and checks are clean (either from the
   first pass or after step 4): produce the manual-test checklist for what
   this batch changed, grouped by module → view → component, each item
   numbered. This is what the user tests right now — once confirmed, it
   becomes the diff applied to that module's `_<Modulo>-CHECKLIST.md` in
   step 7, not a standalone file of its own.
6. Wait for the user to test by hand and answer the checklist item by item.
7. Once the user confirms every checklist item is correct (full
   confirmation only — see "Partial confirmation" below), sync every
   affected document in the same turn, before saying anything else:
   - `docs/Estatus Actual/**` — for every module/component this batch
     touched (per `project-structure.md`'s granularity rules):
     - create/update the component `.md` and `_<Modulo>.md`;
     - update `_<Modulo>-CHECKLIST.md`: add items for anything new, edit
       items whose behavior changed, remove items for anything removed —
       this file always reflects the module's full current state, it is
       never wholesale-replaced with just this batch's items;
     - update `INDEX.md` if a file was added or removed.
   - `Deuda tecnica/**` — close entries this batch resolved, log anything
     newly discovered.
   - `Prompts/SESIONES.md` (Prompts mode) — log the session/batch.
   - `CLAUDE.md` — update "Sesiones en curso" (close the thread if this
     batch finished it) and any status note this batch makes stale. If the
     batch touched a sibling repo directly, update that repo's `CLAUDE.md`
     too.
   - Claude's memory, per "Always save to memory" below.

   Do this without being asked again — it's a standing instruction, not a
   per-batch request. `docs/superpowers/specs/` and `docs/superpowers/plans/`
   are the exception: point-in-time record, never rewritten after the fact.

   **Partial confirmation:** if any item comes back wrong, skip this sync,
   fix the flagged items (back to step 3/4), and re-run the checklist from
   the top before syncing.

8. State explicitly what's next:
   - Current phase/piece not fully done → say what's still open in it.
   - Current phase/piece done → scan `Deuda tecnica/PENDIENTE.md`,
     `docs/Estatus Actual/INDEX.md`, and the project's own phase/order
     list in `CLAUDE.md` for other planned-but-not-started work, and
     summarize it so the user picks what's next — never start it
     unprompted.

## Before validating against a module's checklist

Before using a module's `_<Modulo>-CHECKLIST.md` to sign off on something
(a deploy, closing out the module, any other gate), cross-check it against
that module's `_<Modulo>.md` and every `<Componente>.md`: confirm every
documented component, view, and configuration is actually represented as
an item. Never assume step 7's incremental updates kept it perfectly in
sync — ask the user explicitly: "¿Quieres que lea los archivos de este
módulo para validar al 100% que el checklist no tiene nada desactualizado
o faltante?" If they say yes and something's missing or stale, surface it
before they rely on the checklist as complete.

## Hard rule

Never mark a prompt or change as validated based only on the external
tool's summary. Always read the actual code, line by line, before
confirming.

## Always save to memory

- Architecture or scope decisions made during the project.
- Real bugs found (from the external tool or from your own code) plus their
  fix, when the pattern is likely to repeat in a future project under this
  same standard.
- New conventions a future project under this standard should inherit.

## New-session signal (Prompts mode)

Recommend opening a new external-tool session instead of continuing the
active one when any of these hold:
- The next prompt belongs to a different module/concept than the active
  session's.
- The accumulated context in the active session is no longer relevant to
  what comes next.
- The active session has run enough iterations that it risks losing
  coherence (it starts "forgetting" rules from earlier prompts).

Otherwise, continue the active session. When recommending a switch, say so
explicitly and name the reason — never silently change a prompt's target
session.
