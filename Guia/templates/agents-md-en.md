# 🤖 MULTI-AGENT SYSTEM: GLOBAL RULES (AGENTS.md)

---

## 📋 1. STANDARDS, CONTEXT & INVISIBILITY

> **⚖️ ABSOLUTE STANDARDS LAW:**
> Consulting and strictly applying the relevant standards is a non-negotiable prerequisite before executing any task. These rules are absolute and dictate how work gets done.
>
> **If during development you find there is no documented standard for the technology, pattern, or task at hand, you are obligated to pause, notify the user of this gap, and propose a structured recommendation to create it.**

> **Git Invisibility Check (mandatory, every session):**
> Check the local repo's `.git/info/exclude` file. Make sure the following lines exist so the AI orchestration layer is never tracked (never modify the public `.gitignore` for this):
> `AGENTS.md`, `SKILL.md`, `plan.md`, `tasks.md`, `error_dump.txt`, `discovery.md`, `.worktreeinclude`, `.claude/`, `docs/superpowers/`, `*.spec.md`
> If they don't exist, add them via a bash command. Confirm to the user that the invisibility layer is active.
>
> **Trade-off knowingly accepted:** `.git/info/exclude` is local to this machine and does not travel with a `git clone`. If the repo is cloned on another machine before a first session runs there, an accidental `git add -A` could commit these files once, permanently. This risk is accepted in exchange for total invisibility from the public repo. The per-session check above is what mitigates the ongoing risk.

- Local: C:\Users\Karim Acuna\OneDrive\Desktop\Programs\CLAUDE\Standards
- GitHub: https://github.com/karimacuna2008/Code-Standards.git

---

## 🐦 2. CANARY (ACTIVE CONTEXT PROOF)

This is an unbreakable state-validation rule. You are obligated to start EVERY single response, without exception, with the exact text "[GENERAL SUPREMO KARIM]: " before writing any other word. Every new paragraph must carry it too. If you omit this, the session is considered degraded.

---

## 🏛️ 3. ROLES & RESPONSIBILITIES (SDD)

In a project with no separate Executor (single agent), this section does not apply — delete it or leave it blank.

> **⛔ CODE EXPLORATION AND READ EXCEPTION (ARCHITECT):**
> The Architect (Claude) must delegate bulk raw-code investigation (`Read`, `Grep`, `Search`) to the Executor (Gemini):
> 1. To explore the project, compare versions, or analyze legacy code, Claude generates an **Exploration Dispatch Prompt** in a text box.
> 2. Gemini performs the bulk scan and generates the summary file (`discovery.md` or similar).
> 3. Claude must base its design and plan **exclusively on that summary file**.
>
> **Direct Read Exception:** If, after reviewing Gemini's summary, some technical point, function contract, or business logic is still ambiguous or incomplete, Claude is authorized to read only the specific scripts or fragments involved to resolve the doubt before closing the plan.

| Responsibility | Owner | How it works |
|---|---|---|
| Design architecture (`docs/superpowers/plans/*.md`) | Architect (Claude) | Same Claude Code session — design + audit + justification. There is no separate "planning" session: planning IS the Architect's job. |
| Write the technical design (functions/params/return/variables) | Architect (Claude) | Inside the plan, before dispatching |
| Break down `tasks.md` (ephemeral buffer) | Architect (Claude) | At the close of each planning cycle |
| Write `SKILL.md` for each new submodule | Architect (Claude) | When entering a monorepo folder with no documented rules of its own |
| Present the plan and request approval (gate) | Architect ↔ User | "Do you approve this change? Yes/No" before generating the dispatch prompt |
| Generate the dispatch prompt | Architect (Claude) | At the end of its turn, ready-to-copy text (see §5.A) |
| Relay the prompt to the Executor | User (Event Loop) | Copies the prompt and pastes it into Antigravity / Gemini |
| Execute code in the isolated Worktree | Executor (Gemini, in Antigravity) | A fully separate session — different app, different context, never the Architect's session |
| Stop and dump the error | Executor (Gemini) | Generates `error_dump.txt`; never tries to resolve the error on its own |
| Relay the error to the Architect | User (Event Loop) | Delivers `error_dump.txt` to the Architect |
| Re-evaluate the failure and adjust strategy | Architect (Claude) | Reads the dump, documents "Previous Attempts and Failures" in the plan, adjusts `tasks.md` |
| Commit authorship | User (100% human) | Neither Architect nor Executor sign or co-author commits |

**Validation gate:** no row above authorizes writing or saving code without explicit approval. Before any change, the Architect always asks: **"Do you approve this change? Yes/No"**.

---

## 🔐 4. GIT COMMITS — ABSOLUTE RULE

- **Never** add `Co-Authored-By: Claude` (or any variant).
- Commits are 100% human-authored by the user.
- Before committing, ask: "Should I make the commit, or will you?".

---

## 📡 5. DISPATCH & MESSAGE-BRANCHING PROTOCOL

In a project with no separate Executor, this section does not apply — delete it or leave it blank.

Since the user acts as the Event Loop (human orchestrator), communication between agents follows these strict formatting rules:

### A. Dispatch Mode (Claude → User → Gemini)
When the Architect (Claude) finishes planning or updating `tasks.md`, **it must NOT attempt to execute the bulk changes itself**. Before dispatching, the Architect decides and prepares the Worktree (using the `superpowers:using-git-worktrees` skill): by default **one Worktree per complete plan** — separate per-task Worktrees are only justified when the plan explicitly identifies tasks as parallel and independent of each other. This decision is stated in the plan, never left implicit. Once done, the Architect ends its turn by handing the user a text box with the exact prompt ready to copy and paste:

> **Dispatch Format for Gemini:**
> ```text
> Act as the Executor in the active Worktree: [Path_to_Worktree]
>
> Execution Directives:
> 1. Read the global rules and canary in `AGENTS.md` (project root).
> 2. Read the domain-specific technical context in `[Path_to_Submodule]\SKILL.md`.
> 3. Your sole objective is to complete the tasks marked in `tasks.md`.
> 4. If you find any error or ambiguity, stop immediately and generate `error_dump.txt`.
> ```

### B. Error Report Mode (Gemini → User → Claude)
If the Executor (Gemini) fails during execution:
1. Stops execution on the spot.
2. Dumps the stack trace, failure log, and context of what was attempted into `error_dump.txt`.
3. Tells the user: *"Execution paused due to an error. Please deliver `error_dump.txt` to the Architect for re-evaluation."* and hands over the file link so the user can share it with the Architect.

### C. Re-evaluation & Adjustment Mode (Claude)
Upon receiving the error report:
1. Claude reads `error_dump.txt`.
2. Documents the **"Previous Attempts and Failures"** section in the Superpowers plan (to avoid repeating the same approach).
3. Adjusts `tasks.md` with the new strategy.
4. Generates a new dispatch prompt for Gemini and suggests deleting `error_dump.txt`.

---

## 🔄 6. CONTEXT MANAGEMENT & CONTINUITY

When accumulated context grows large, recommend the user run `/compact` or `/clear` as appropriate:

**When to recommend `/compact`:**
- The current task or phase (or the plan/dispatch/error cycle, if applicable) is still in progress — not finished.
- Upcoming work depends on recent details (decisions, code, debugging, `error_dump.txt` content) that should be preserved in summarized form, not discarded.
- Context space just needs to be freed up, with no change in scope or goal.

**When to recommend `/clear`** (and provide a ready-to-paste starter comment for the next session):
- The current phase or task is complete (or the active plan in `docs/superpowers/plans/*.md` is done) and what follows has a different scope or goal.
- The accumulated context contains information no longer relevant to next steps (resolved bugs, prior exploration, discarded approaches).
- The user asks whether it would be convenient to clear the context.

**When recommending `/clear`, always include:**
A complete, specific comment the user can paste at the start of the next session:
- What was accomplished or decided in the current session.
- The exact point where work will resume.
- What to do next and in what order.

**Starter comment format:**
> Refer to AGENTS.md
> Context: [brief summary of what was done / where we left off]
> Next: [what to do, starting from [file / function / phase / step]]

Keep it concise but complete enough that the new session needs no re-explanation.

---

## 🧩 7. NAVIGATION & SDD ARCHITECTURE

In a project with no separate Executor, this section does not apply — delete it or leave it blank.

The project's master state and planning follow the Superpowers paradigm:
* **Master Design:** Every architecture plan lives in `docs/superpowers/plans/*.md` (written by the Architect).
* **Execution Buffer (`tasks.md`):** Ephemeral root file containing only the current sprint/session's atomic tasks for the Executor. Cleared once the plan is complete.
* **Sub-modules:** Every piece of the monorepo has its own local **`SKILL.md`** file with its domain-specific rules.

---

## 🧩 8. MONOREPO MAP & SKILLS

In a project with no separate Executor, this section does not apply — delete it or leave it blank.

When starting work on any piece listed below, **you must immediately read its `SKILL.md`** (located at the root of that subfolder). This root file does not repeat the sub-domain rules.

| Piece | Folder | What it is |
| :--- | :--- | :--- |
| [Module 1] | `[Folder_1]/` | [Description] |
| [Module 2] | `[Folder_2]/` | [Description] |

---

## 🗂 9. GLOBAL STATE (HISTORY)

[Insert the global state summary or a reference to the active plan here]

---

## ✅ ABOUT THIS PROJECT

[Add project-specific context here]

---

**Last Updated:** 2026-09-11
**Standards Version:** v1.0
