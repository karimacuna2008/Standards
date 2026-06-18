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
- **description:** lowercase, imperative ("add", not "added")

Examples:
- `feat(auth): add user login validation`
- `fix(api): handle null response in fetch`
- `docs(readme): update installation instructions`

## 3. Branch model
At the start of every project, ask: **"¿Este proyecto necesita ramas `main` y `develop`, o solo `main`?"** Not every project needs the two-branch model — pick one of the following and stick to it for the life of the project.

**Two-branch model** (when approved):
- **main** — stable, release-ready code only.
- **develop** — integration branch; all work merges here first.
- **feature/**, **fix/** — temporary; branch off `develop`, delete after merge.

**Single-branch model** (when the project doesn't need staged releases):
- **main** — the only long-lived branch.
- **feature/**, **fix/** — temporary; branch off `main`, delete after merge.

Naming: `feature/user-login`, `fix/email-validation`.

This decision also drives the Cloud Run service setup in `deployment.md`.

## 4. Merging
- **Two-branch model:**
  - **feature → develop:** squash into a single commit summarizing the work (Conventional Commits format).
  - **develop → main:** merge on release; squash optional (one commit per release).
- **Single-branch model:**
  - **feature → main:** squash into a single commit summarizing the work (Conventional Commits format).
- One feature/fix per branch.

## 5. Tags — Semantic Versioning
`vMAJOR.MINOR.PATCH` — `v1.0.0` first release · `v1.1.0` new features · `v1.1.1` fixes only. Tag `main` after a release merge.

## 6. Standing rules
- Small, meaningful commits — Conventional Commits always.
- Keep `develop` stable (only tested code); keep `main` for releases.
- Delete feature branches after merging.

## 7. Files that never get committed
Add these to `.gitignore` at project start, before the first commit — never reference them from a README or other public-facing doc:
- `CLAUDE.md` — internal instructions for Claude, not for the repo's audience.
- `docs/superpowers/` — internal planning artifacts (specs, plans) from Claude's brainstorming/writing-plans skills.

`CLAUDE.md` always goes in `.gitignore`, **even when `git update-index --skip-worktree CLAUDE.md` is also in use.** `--skip-worktree` only hides local edits from `git status`/`git diff` for a file already tracked — it does not stop the file from being committed or pushed, and does not apply to a fresh clone. `.gitignore` is the actual guarantee that the file never reaches GitHub; treat the two as separate, both-required steps, never one in place of the other.

If either file was already committed before this rule was applied, untrack it with `git rm -r --cached <path>` (keeps the local file, stops tracking it) rather than leaving it in history going forward.

---
*Conceptual git tutorials, first-time setup, multi-machine sync, command cheat-sheets and troubleshooting are human-facing and live in the future HTML guide, not here.*

**Version:** 2.3  
**Last Updated:** 2026-06-17
