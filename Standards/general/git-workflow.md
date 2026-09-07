# Git Workflow Standard

Rules for Claude when helping with version control. Claude **may** run git commands itself (via shell), but only after asking, and commits must read as fully human-authored by the user.

## 1. Execution & authorship
- You **may** run `git commit`, `git push`, `git merge`, etc. yourself, through the
  Bash/cmd/PowerShell tool — as if the user had typed the command in their terminal.
- **Ask first, every time:** "¿Hago el commit yo o lo haces tú?" Run the command
  only if the user says you should. If the user will do it, output the commands instead.
- Commits must read as **fully human-authored by the user**: rely on the repo's
  configured git identity (`user.name` / `user.email`). Do **not** pass `--author`,
  do **not** override the identity, do **not** sign as the assistant.
- **Never** add `Co-Authored-By` trailers or "Generated with…" lines.

## 2. Commit messages — Conventional Commits
`type(scope): description`
- **type:** feat · fix · docs · style · refactor · test · chore
- **scope:** affected area (auth, api, database, …)
- **description:** en **español**, minúscula, imperativo ("agrega", no "agregado"). El `type` y el `scope` se mantienen en inglés (son keywords de Conventional Commits).

Examples:
- `feat(auth): agrega validación de login de usuario`
- `fix(api): maneja la respuesta nula en el fetch`
- `docs(readme): actualiza las instrucciones de instalación`

## 3. Branch model
At the start of every project, ask: **"¿Este proyecto necesita ramas `master` y `develop`, o solo `master`?"** Not every project needs the two-branch model — pick one of the following and stick to it for the life of the project.

**Two-branch model** (when approved):
- **master** — stable, release-ready code only.
- **develop** — integration branch; all work merges here first.
- **feature/**, **fix/** — temporary; branch off `develop`, delete after merge.

**Single-branch model** (when the project doesn't need staged releases):
- **master** — the only long-lived branch.
- **feature/**, **fix/** — temporary; branch off `master`, delete after merge.

Naming: `feature/user-login`, `fix/email-validation`.

**Auditoría pre-entrega (`release-prep/`):** al cerrar un proyecto, antes de entregarlo, se hace una pasada de auditoría contra estos estándares y mejores prácticas (seguridad, rendimiento, naming, etc.) para corregir los huecos del código existente. Todo ese trabajo —todas sus fases— vive en una rama dedicada con prefijo **`release-prep/`** (ej. `release-prep/auditoria`), no en `feature/`/`fix/`. Sus commits siguen Conventional Commits en español como cualquier otro.

This decision also drives the Cloud Run service setup in `deployment.md`.

**Migrating an existing repo from `main` to `master`:** this convention applies to new projects going forward. A repo that already has `main` with CI/CD deployed against it (Cloud Build trigger, GitHub Actions, etc.) keeps `main` — never rename it on your own initiative. Only migrate a specific existing repo when the user explicitly asks for that repo by name, since renaming the default branch requires reconfiguring the deploy trigger too.

## 4. Merging
- **Two-branch model:**
  - **feature → develop:** squash into a single commit summarizing the work (Conventional Commits format).
  - **develop → master:** merge on release; squash optional (one commit per release).
- **Single-branch model:**
  - **feature → master:** squash into a single commit summarizing the work (Conventional Commits format).
- One feature/fix per branch.

## 5. Tags — Semantic Versioning
`vMAJOR.MINOR.PATCH` — `v1.0.0` first release · `v1.1.0` new features · `v1.1.1` fixes only. Tag `master` after a release merge.

## 6. Standing rules
- Small, meaningful commits — Conventional Commits always.
- Keep `develop` stable (only tested code); keep `master` for releases.
- Delete feature branches after merging.

## 7. Files that never get committed
Add these to `.gitignore` at project start, before the first commit — never reference them from a README or other public-facing doc. Applies to every branch of the repo (each branch carries its own `.gitignore`; don't assume one branch's copy covers another).

**Claude-internal / local-only — never commit, ever, on any branch:**
- `CLAUDE.md` — internal instructions for Claude, not for the repo's audience.
- `.claude/` — Claude Code's own local session/tool state (permissions, scheduled tasks, etc.). Usually already kept out via `.git/info/exclude` by the tool itself, but add it to the tracked `.gitignore` too so a fresh clone on another machine is covered from the start.
- `.superpowers/` — the Superpowers plugin's own local working state (`subagent-driven-development` progress/task briefs/review diffs, etc.). Same reasoning as `.claude/`: it has its own nested `.gitignore` in some versions, but that only protects paths *inside* it — the directory itself still needs an entry in the project's real `.gitignore`.
- `docs/superpowers/` — design specs and plans written by the `brainstorming`/`writing-plans` skills. **Absolute rule, no exceptions:** even when a skill's default flow says to commit the spec to git, skip that step — leave it untracked here instead.
- `*_REFACTOR_PLAN.md` (or any other local planning doc that isn't `docs/superpowers/`) — internal planning artifacts from Claude's brainstorming/writing-plans/subagent-driven-development skills. Use a **wildcard**, not a literal filename per branch/topic (`BITACORA_REFACTOR_PLAN.md`, `DEVOLUCIONES_REFACTOR_PLAN.md`, …) — a hardcoded name only ignores that one file on the branch that added it; an untracked plan doc from another branch (carried over on disk after a `git checkout`, since untracked files aren't removed by switching branches) stays unignored and can slip into a `git add -A` on a branch that never listed it.

**Apply this identical block to every branch of a repo, not just the one being actively worked on** — a project with multiple long-lived branches (e.g. one branch per component/script) needs the same `.gitignore` entries everywhere, added at the point each branch is created. A rule added to one branch's `.gitignore` mid-project doesn't retroactively protect branches that already exist; go back and add it to each of them.

**Python:**
- `__pycache__/`, `*.pyc`
- `.pytest_cache/` (pytest self-ignores this via its own nested `.gitignore`, but list it explicitly too — don't rely on a tool-generated file being present)

**PyInstaller (or equivalent build tooling):**
- `build/`, `dist/` — PyInstaller's work/output dirs. Never commit a compiled binary through these; a release `.exe`/binary is copied out to wherever it's actually distributed from (a release branch, a separate `master` worktree, GitHub Releases, …), not left in `dist/`.
- `*.spec` — PyInstaller writes this on every run; regenerate it, don't hand-edit and commit it. If a build script deletes it after a successful compile, it can still survive an interrupted/crashed build — the `.gitignore` entry is the actual guarantee, not the cleanup step in the build script.

`CLAUDE.md` always goes in `.gitignore`, **even when `git update-index --skip-worktree CLAUDE.md` is also in use.** `--skip-worktree` only hides local edits from `git status`/`git diff` for a file already tracked — it does not stop the file from being committed or pushed, and does not apply to a fresh clone. `.gitignore` is the actual guarantee that the file never reaches GitHub; treat the two as separate, both-required steps, never one in place of the other.

If any of these was already committed before this rule was applied, untrack it with `git rm -r --cached <path>` (keeps the local file, stops tracking it) rather than leaving it in history going forward.

---
*Conceptual git tutorials, first-time setup, multi-machine sync, command cheat-sheets and troubleshooting are human-facing and live in the future HTML guide, not here.*

**Version:** 2.6  
**Last Updated:** 2026-07-21
