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
- **main** — stable, release-ready code only.
- **develop** — integration branch; all work merges here first.
- **feature/**, **fix/** — temporary; branch off `develop`, delete after merge.

Naming: `feature/user-login`, `fix/email-validation`.

## 4. Merging
- **feature → develop:** squash into a single commit summarizing the work (Conventional Commits format).
- **develop → main:** merge on release; squash optional (one commit per release).
- One feature/fix per branch.

## 5. Tags — Semantic Versioning
`vMAJOR.MINOR.PATCH` — `v1.0.0` first release · `v1.1.0` new features · `v1.1.1` fixes only. Tag `main` after a release merge.

## 6. Standing rules
- Small, meaningful commits — Conventional Commits always.
- Keep `develop` stable (only tested code); keep `main` for releases.
- Delete feature branches after merging.

---
*Conceptual git tutorials, first-time setup, multi-machine sync, command cheat-sheets and troubleshooting are human-facing and live in the future HTML guide, not here.*

**Version:** 2.1  
**Last Updated:** 2026-06-12
