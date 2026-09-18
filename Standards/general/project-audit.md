# Project Discovery, Audit & Standardization Standard

> ⚠️ **STATUS: DRAFT PROPOSED BY GEMINI - PENDING VALIDATION WITH CLAUDE**
> <!-- VALIDAR CON CLAUDE: Review multi-agent audit phases, ignored folders, and model capability matrix before final consolidation -->

Operational rules for exploring, auditing, standardizing, and documenting existing or legacy repositories using autonomous AI agents and subagents.

---

## 1. Core Principles
- **No Refactoring Before Alignment:** Never touch, format, or alter source code until discovery is complete and the user has answered critical alignment questions (`preguntas_proyecto.txt`).
- **Context Isolation:** Subagents must only receive the slice of context strictly necessary for their audit domain (logic, style, or documentation).
- **Safe Non-Invasive Edits:** Code improvements must be purely cosmetic (naming, PEP 8 120 chars, alphabetical import blocks, docstrings). Deep functional architectural gaps are documented in `docs/STANDARDS_GAP.md` instead of being refactored blindly.
- **Surgical Git Invisibility:** Local AI orchestration files are quarantined in `.git/info/exclude`; public documentation (`docs/Estatus Actual/`, `docs/STANDARDS_GAP.md`, `README.md`) must remain tracked by Git.

---

## 2. Multi-Agent Audit Workflow (5 Phases)

### Phase 1: Objective Discovery (Subagent Discovery)
- **Directory Pre-Filtering:** Automatically ignore non-source and virtual folders:
  `{".venv", "venv", "env", "node_modules", "__pycache__", ".git", "dist", "build", ".pytest_cache", ".ruff_cache"}`.
- **Output:** Generate `discovery.md` in repository root containing:
  1. Complete file catalog and inferred role of each file.
  2. Identified external dependencies (libraries, databases, external APIs).
  3. Identified UI components (Streamlit, Tkinter, React) vs backend logic.
- **Tone:** 100% factual and descriptive. Zero quality judgements or refactoring proposals in this phase.

### Phase 2: Paired Subagent Auditing & Alignment Gate
Distribute auditing across 3 specialized pairs:
- **Pair 1 (Logic & Security):**
  - Subagent 1A: Business logic flow, error handling, and dependency sanity.
  - Subagent 1B: Secret quarantine check vs `general/security.md` (no hardcoded keys, `.env` quarantine).
- **Pair 2 (Code & Style):**
  - Subagent 2A: PEP 8 formatting, line length (max 120 chars), and 3 alphabetical import blocks.
  - Subagent 2B: Type annotations, function signatures, and docstrings vs `python/functions.md`.
- **Pair 3 (Documentation & Git):**
  - Subagent 3A: `README.md` completeness and breadcrumb navigation links.
  - Subagent 3B: Public documentation integrity in `docs/` and Git tracking.

#### Alignment Gate (`preguntas_proyecto.txt`):
If critical ambiguities exist (repo visibility, dead/legacy scripts, hardcoded keys), pause execution and generate `preguntas_proyecto.txt`:
- **No fixed question limit:** Formulate as many questions as strictly necessary to resolve critical ambiguities and align architectural decisions. However, exercise sound judgment and restraint to avoid overwhelming the user—focus strictly on essential blockers and high-impact choices.
- Structured multiple choice: `[A]`, `[B]`, `[C]`.
- Always mark `(Recomendado)` on the best choice with a 1-line justification.
- **Hard Stop:** Wait for user response before proceeding to Phase 3.

### Phase 3: Safe Cosmetic Refactor
Executed only after alignment answers are confirmed:
1. Apply cosmetic fixes (meaningful variable names, PEP 8 120 chars, 3-group imports, docstrings).
2. Validate syntax immediately using `python -m py_compile <file>`. Zero functional logic changes.

### Phase 4: Master Documentation & Standards Gap Report
- **Subagent 4A (`README.md`):** Write standard README (Title, Problem it solves, Features, Setup, Usage, Configuration).
- **Subagent 4B (`docs/STANDARDS_GAP.md`):** Document technical debt items. Each entry must list:
  1. Standard name, version, and reference date.
  2. Concrete code comparison: *"Cómo está hoy"* vs *"Cómo debe ser según el estándar"*.
  3. Recommendation to verify if the standard has been updated.
  - Link `docs/STANDARDS_GAP.md` prominently inside the root `README.md`.

### Phase 5: Closure & Git Ownership
Ask the user: *"¿Hago el commit yo o lo haces tú?"* per `AGENTS.md` §4. Never assume Git authorship.

---

## 3. AI Model Specialization & Capability Matrix (Benchmarks & Roles)

Based on industry benchmarks (SWE-bench Verified/Pro, Chatbot Arena Coding) and real-world token economics:

| Role in Workflow | Recommended Models | Why / Strengths | Trade-offs |
|---|---|---|---|
| **Orchestrator / Architect / Planner** | **Claude (3.5 / 3.7 Sonnet / Opus)** | Highest SWE-bench resolve rates, superior nuanced reasoning, architectural rigor, and strict instruction-following without deviation. | Token caps / rate limits; costly for raw codebase dumps. |
| **Heavy Scanner / Mass Executor ("Talacha")** | **Gemini (3.8 Flash / 3.1 Pro / 2.0)** | Colossal context window (1M–2M tokens), near-instant throughput, abundant token limits. Perfect for scanning whole repositories, batch audits, and volume formatting. | Needs structured prompt guardrails to avoid unsolicited logic changes. |
| **Sandboxed CI/CD & Cloud PRs** | **OpenAI Codex / GPT-5.x Codex** | Sandboxed execution in OpenAI cloud, native GitHub integration for PR reviews and isolated test suites. | Redundant if already using Claude for architecture and Gemini for local execution. |
| **Algorithmic Debugger / Deep Logic** | **OpenAI o-series (o1 / o3-mini)**, **DeepSeek R1** | Deep chain-of-thought verification for complex mathematical, concurrency, or algorithmic edge cases. | Higher latency; not suited for broad directory operations. |

> **User Fleet Decision Rule:**
> - **Primary Pair:** Use **Claude** for high-level architecture, superpower planning, and validating drafts; use **Gemini (3.8 Flash / 3.1 Pro)** for mass discovery, paired subagent auditing, and documentation volume.
> - **Codex Evaluation:** Purchasing OpenAI Codex is **optional/secondary**; only recommended if migrating execution to cloud-based GitHub PR sandboxes. For local workspace auditing, the Claude + Gemini combination provides maximum efficiency.

---
**Version:** 1.0 (Draft)  
**Last Updated:** 2026-09-17
