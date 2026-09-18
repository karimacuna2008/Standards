# CLAUDE.md

---

## 📋 STANDARDS & GUIDELINES REFERENCE

This project uses centralized development standards. Before starting work, check the relevant standard file:

**Standards Location (Local):** `C:\Users\Karim Acuna\OneDrive\Desktop\Programs\CLAUDE\Standards\`

**GitHub Backup:** https://github.com/karimacuna2008/Code-Standards.git

### How to use Standards:
1. If working with **Python**: Check `Standards/python/<topic>.md`
2. If working with **Planning/Git/Docs**: Check `Standards/general/<topic>.md`
3. If local Standards folder doesn't exist:
   - Clone from GitHub: `git clone https://github.com/karimacuna2008/Code-Standards.git "C:\Users\Karim Acuna\OneDrive\Desktop\Programs\CLAUDE\Standards"`
   - Or ask: "Standards folder not found locally, should I fetch from GitHub?"

---

## 🔒 1. MANDATORY VALIDATION GATE

After presenting the plan and justifications, you must stop and ask for validation for each specific change. **You are not authorized to execute, write, or save any code until the user explicitly confirms each step.**

NEVER modify anything without explicit approval. This is a critical rule to ensure alignment and prevent unauthorized changes. Always ask: "Do you approve this change? Yes/No" before proceeding.

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

## 📚 3. STANDARDS FILES TO FOLLOW

Depending on the work, read the corresponding INDEX first:

- **Python** → `Standards/python/INDEX.md`
- **Planning/Git/Docs** → `Standards/general/INDEX.md`

> When creating a new standard, always update the corresponding `INDEX.md` before closing the session.
> For roadmap and decisions (done / in progress / pending), see `Standards/README.md`.

---

## ✅ ABOUT THIS PROJECT

This workspace **authors and maintains** the centralized development standards (`Standards/`) and the usage guide (`Guia Basica/`). Its purpose is to *produce* the standards that other projects consume — it is not a normal application project.

- **This file is not the reusable template.** Two separate, parallel root templates are copied into other projects: `CLAUDE.md` (single-agent) at `Guia/templates/claude-md.html`, and `AGENTS.md` (multi-agent, Architect/Executor) at `Guia/templates/agents-md.html` (Copiar EN/ES) — neither replaces the other; pick one per project (see `Standards/README.md` decisions log) and migrate with `Standards/general/agents-migration.md` if a project moves from one to the other later.
- **Roadmap & decisions** (done / in progress / pending) live in `Standards/README.md` — the single source of truth for status.
- When working here you are usually **writing or editing standard documents** (`Standards/**/*.md`); match the existing files' style and obey the validation gate above.
- **Audience — the `Standards/**/*.md` are for Claude, not for the user.** They are written as **operational instructions** (what to do, which option to choose, in what format). They must **not** contain conceptual tutorials or human explanations (what something is, how it works, *why* X over Y). The user does not read them.
- **Human-facing guides come later as HTML.** A complete, readable guide for the user — with the *what / how / why* and teaching content — **will be authored in the future as HTML** (in `Guia Basica/`). That is where conceptual/explanatory material belongs; the `Standards/*.md` stay lean and Claude-facing.

---

**Last Updated:** 2026-05-31  
**Standards Version:** v1.0 (in progress)
