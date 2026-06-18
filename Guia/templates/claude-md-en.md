# CLAUDE.md

---

## 📋 STANDARDS

> FIRST SESSION: Check if the folder at `Local` exists.
> - If it exists: delete this block and use that path directly.
> - If it doesn't exist: ask the user: "Should I download the standards from GitHub and
>   save them locally, or always read them from GitHub?"
>   - If they choose local: clone the repo, save it to the user's Desktop, update the
>     `Local` path here and delete this block.
>   - If they choose GitHub: delete the `Local` line and delete this block. Always keep
>     the GitHub line.
>
> Once the Standards path is resolved, you MUST tell the user to run the following
> command and wait for their confirmation before proceeding with any other task:
>   git update-index --skip-worktree CLAUDE.md
> Do not continue until the user confirms it was executed.

Local: C:\Users\Karim Acuna\OneDrive\Desktop\Programs\CLAUDE\Standards
GitHub: https://github.com/karimacuna2008/Code-Standards.git

### How to use Standards:
- If working with **Python** → read `python/INDEX.md` inside the Standards folder
- If working with **Planning/Git/Docs** → read `general/INDEX.md` inside the Standards folder
- Each `INDEX.md` contains the list of available files and when to use each one

---

## 🔒 1. MANDATORY VALIDATION GATE

After presenting the plan and justifications, you must stop and ask for validation for each specific change. **You are not authorized to execute, write, or save any code until the user explicitly confirms each step.**

NEVER modify anything without explicit approval. Always ask: "Do you approve this change? Yes/No" before proceeding.

The language depends on the one used in the session.

---

## 📝 2. CHANGE MANAGEMENT & JUSTIFICATION

You are prohibited from executing changes or writing final code until the following justification process is complete:

### Plan-First Rule
Provide a structured summary of all proposed changes first.

### Domain-Specific Reasoning
- **Data Science & AI/ML:** Explain the methodology. Justify why this specific model/approach was chosen over alternatives and why it fits this specific use case.
- **Tech & Libraries:** Identify all libraries/technologies to be used. Justify their necessity and how they integrate into the existing stack.

### Deep Technical Breakdown (Scripts & Functions)
For every function or method (native or library-based), detail:
- **Functionality:** Internal logic and purpose.
- **Parameters:** Breakdown of arguments and what they control.
- **Return Values:** What it outputs and its impact on the subsequent flow.
- **Variables:** Breakdown of variables used and those that will be created.

---

## 🔄 3. CONTEXT MANAGEMENT & CONTINUITY

When accumulated context grows large, recommend the user run `/compact` or
`/clear` as appropriate:

**When to recommend `/compact`:**
- The current task or phase is still in progress (not finished)
- Upcoming work depends on recent details (decisions, code, debugging) that
  should be preserved in summarized form, not discarded
- Context space just needs to be freed up, with no change in scope or goal

**When to recommend `/clear`** (and provide a ready-to-paste starter comment
for the next session):
- The current phase or task is complete and what follows has a different scope or goal
- The accumulated context contains information no longer relevant to next steps
  (resolved bugs, prior exploration, discarded approaches)
- The user asks whether it would be convenient to clear the context

**When recommending `/clear`, always include:**
A complete, specific comment the user can paste at the start of the next session:
- What was accomplished or decided in the current session
- The exact point where work will resume
- What to do next and in what order

**Starter comment format:**
> Refer to CLAUDE.md
> Context: [brief summary of what was done / where we left off]
> Next: [what to do, starting from [file / function / phase / step]]

Keep it concise but complete enough that the new session needs no re-explanation.

---

## ✅ ABOUT THIS PROJECT

[Add project-specific context here]

---

**Last Updated:** 2026-06-08
**Standards Version:** v1.0
