# agents-migration.md — Migrating from CLAUDE.md to AGENTS.md

Checklist for when a project that already started with `CLAUDE.md` (single agent) adds a separate Executor (Antigravity/Gemini) and moves to the multi-agent architecture. Use it when the user decides to migrate — don't apply it preemptively to projects that will stay solo-Claude.

**When to migrate:** when the project is about to start using an Executor in Antigravity/Gemini to run code, not before. See the selection criteria in `Standards/README.md` (decisions log).

## Checklist

1. **Create `AGENTS.md` from the template.** Copy `agents-template.md` in full to the project root as `AGENTS.md`. Don't derive it by editing the existing `CLAUDE.md` — they're different structures.
2. **Carry over project context.** Copy the content of the `## ✅ ACERCA DE ESTE PROYECTO` section from the old `CLAUDE.md` into the same section of the new `AGENTS.md`.
3. **Carry over relevant history.** If the old `CLAUDE.md` kept a decisions/incidents log (even without a formal "History" section), move it into `## 🗂 9. ESTADO GLOBAL (HISTORIAL)` of the new `AGENTS.md`. Add an entry for the migration itself (date, what changed) — see the real example in an already-migrated `AGENTS.md`'s history.
4. **Fill in §7-8 (Monorepo Map).** If the project has more than one folder/module, document the map in §8. If it's a single module, leave §7-8 blank for now (fill them in once there's more than one piece).
5. **Turn on the Git invisibility layer.** Check `.git/info/exclude` and add, if missing: `AGENTS.md`, `SKILL.md`, `plan.md`, `tasks.md`, `error_dump.txt`, `discovery.md`, `.worktreeinclude`, `.claude/`, `docs/superpowers/`, `*.spec.md`. Never use the public `.gitignore` for this (see `AGENTS.md` §1 and the trade-off documented there).
6. **Retire `CLAUDE.md`.** Once `AGENTS.md` is complete and confirmed by the user, delete `CLAUDE.md` from the root — `AGENTS.md` becomes the single source of truth. Don't leave both files active in the same project.
7. **Create the first `SKILL.md`.** The first time you work under the new architecture inside any folder listed in §8, create its `SKILL.md` using `skill-template.md` — don't create all of them at once, only when entering each one.
8. **Decide the Worktree strategy for the first plan.** Before the first dispatch to Gemini, decide and document in the plan (`superpower-plan-template.md` §5) whether it's one Worktree per full plan (default) or per individual task.
9. **Confirm with the user before deleting anything.** Steps 1-5 are additive (no destructive approval needed); step 6 (deleting `CLAUDE.md`) does require explicit user confirmation before executing.

## What NOT to do

- Don't migrate a project "just in case" without the user having decided to add an Executor — migration is a real cost (Git invisibility, a `SKILL.md` per module, the dispatch protocol) that isn't justified for a session that will stay solo-Claude.
- Don't keep `CLAUDE.md` and `AGENTS.md` both active at once in the same project — it creates ambiguity about which one governs.
- Don't invent content for §7-8 (Monorepo Map) before there are real modules to document.

---

**Version:** 1.0
**Last Updated:** 2026-09-13
