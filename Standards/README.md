# Code & Development Standards

Personal development standards for all projects. Guidelines for how to approach coding, planning, testing, and architecture across different project types.

**Local Path:** `C:\Users\Karim Acuna\OneDrive\Desktop\Programs\CLAUDE\Standards\`

**GitHub:** https://github.com/karimacuna2008/Code-Standards

> **Single source of truth for standards status.** The workspace `CLAUDE.md` and the guide (`Guia Basica/02_CLAUDE_md.html`) point here for the roadmap and decisions.

> **Audience.** Every file under `Standards/**` (except this README) is a **Claude-facing operational instruction** — imperative rules, decision tables, and conventions, written in English. They contain no conceptual tutorials or "why it matters" prose; that material is reserved for the future human-facing HTML guides (`Guia Basica/`). This README is the human-readable status index.

---

## 📁 Structure

```
Standards/
├── general/
│   ├── planning.md              ✅ How to plan projects
│   ├── git-workflow.md          ✅ Branches, commits, workflow
│   ├── documentation.md         ✅ READMEs, CHANGELOG, docstrings
│   ├── deployment.md            ✅ Cloud Build + GitHub CI/CD (Cloud Run, desktop apps, gated jobs)
│   ├── database.md              ✅ SQL schema design & naming (PKs, FKs, NOT NULL)
│   ├── database-migrations.md   ✅ Idempotent migrations, SQLAlchemy inspector, when to use Alembic
│   ├── database-remote-access.md ✅ Bastion VM in GCP + IAP tunneling for firewalled DBs
│   ├── agents-template.md       ✅ Root `AGENTS.md` template — multi-agent projects (Architect/Executor)
│   ├── agents-migration.md      ✅ Checklist to migrate a project from `CLAUDE.md` to `AGENTS.md`
│   ├── skill-template.md        ✅ `SKILL.md` template for each monorepo submodule
│   ├── tasks-template.md        ✅ `tasks.md` template — ephemeral Executor task buffer
│   ├── superpower-plan-template.md ✅ Architecture plan template for `docs/superpowers/plans/`
│   ├── project-structure.md     📝 TODO — folder organization
│   ├── security.md              🚧 DRAFT (Pending validation with Claude)
│   ├── auth-rbac.md             🚧 DRAFT (Pending validation with Claude)
│   └── project-audit.md         🚧 DRAFT (Pending validation with Claude)
├── python/
│   ├── code-standards.md        ✅ Style, structure, conventions
│   ├── functions.md             ✅ How to write functions
│   ├── api-client.md            ✅ Consume external REST APIs (HTTP client)
│   ├── automation.md            ✅ CLI scripts, bots
│   ├── error-handling.md        ✅ Exception control flow
│   ├── data-analysis.md         🚧 DRAFT COMPLETED (Sections 4-10 pending validation with Claude)
│   ├── quick-interfaces.md     🚧 DRAFT (Streamlit & CustomTkinter, pending validation with Claude)
│   ├── api-server.md            🚧 DRAFT (FastAPI server architecture, pending validation with Claude)
│   ├── observability.md         🚧 DRAFT (Structured logs, tags, metrics, pending validation with Claude)
│   ├── cloud-services-gcp.md    ✅ GCP integration (provider-specific)
│   ├── testing.md               ✅ Pytest, characterization tests, UI exclusions
│   ├── databases.md             🚧 DRAFT (SQLAlchemy 2.0 & Alembic, pending validation with Claude)
│   ├── async.md                 🚧 DRAFT (TaskGroup & to_thread, pending validation with Claude)
│   └── data-science.md          📝 TODO — ML/AI feature engineering and training
├── javascript/
│   ├── INDEX.md                 ✅ Which JS/TS standard to use
│   ├── react-typescript.md      ✅ React+TS: layout, tokens, data layer, gates
│   └── testing.md               📝 TODO — write only after real code exercises it
├── prompt-driven-development/
│   ├── kickoff.md                ✅ Project intake: purpose, scope, delivery target, existing backend → stack recommendation
│   ├── project-structure.md      ✅ Folder layout, doc responsibilities, Estatus Actual/ granularity
│   ├── workflow.md                ✅ Per-batch cycle, hard validation rule, memory triggers
│   ├── prompts.md                 ✅ Prompt naming, sizing, Prompts/Validated/ lifecycle
│   ├── architecture.md            ✅ Thin orchestrator, design tokens, Deuda tecnica/
│   └── spec-and-plan-gate.md      ✅ When to require brainstorming+writing-plans vs. direct fix
└── README.md                    (this file)
```

---

## ✅ Created (done)

| Standard | Covers |
|---|---|
| `general/planning.md` | Planning projects, scope, technology choice, progress |
| `general/git-workflow.md` | Feature branches, squash merge, conventional commits, tags |
| `general/documentation.md` | READMEs, CHANGELOG, docstrings, /docs |
| `general/deployment.md` | Cloud Build + GitHub CI/CD: Cloud Run APIs, desktop apps with auto-update, gated jobs/migrations, monorepo trigger-per-folder pattern |
| `general/database.md` | SQL schema design and naming: PKs, FKs, naming, NOT NULL |
| `general/database-migrations.md` | Evolving the schema: idempotent migrations, SQLAlchemy inspector, when to use Alembic |
| `general/database-remote-access.md` | Connecting to an IP-firewalled DB from an unauthorized network — GCP bastion VM + IAP tunneling |
| `general/agents-template.md` | Root `AGENTS.md` template for multi-agent projects (Architect/Executor) — parallel to, not derived from, `general/claude-template` equivalent (`Guia/templates/claude-md.html`) |
| `general/agents-migration.md` | Checklist to migrate a project from `CLAUDE.md` to `AGENTS.md` when it adds a separate Executor |
| `general/skill-template.md` | `SKILL.md` template for each monorepo submodule listed in `AGENTS.md` §8 — local domain rules and test commands |
| `general/tasks-template.md` | `tasks.md` template — ephemeral atomic task buffer for the Executor, with `error_dump.txt` dump protocol |
| `general/superpower-plan-template.md` | Architecture plan template (`docs/superpowers/plans/YYYY-MM-DD-name.md`) — context, design decisions, technical design, previous attempts and failures, task breakdown |
| `javascript/INDEX.md` | Which JS/TS standard to use for the task |
| `python/code-standards.md` | PEP 8 (120 chars), f-strings, imports, naming, logging |
| `python/functions.md` | Naming, type hints, docstrings, when to extract |
| `python/api-client.md` | Consume external REST APIs (HTTP client): auth, pagination, retry, client-as-dependency |
| `python/automation.md` | CLI scripts, menu vs argparse, output conventions |
| `python/error-handling.md` | Exception control flow: raise vs absorb, chaining, error responses (logging mechanics live in `observability.md`) |
| `python/cloud-services-gcp.md` | GCP integration: Firestore, Cloud Storage, Firebase Auth, Cloud Logging, .gcloudignore, Cloud Build deploy |
| `javascript/react-typescript.md` | React+TS: folder layout, Tailwind 4 tokens, file-size gate, `api/` shape, response validation, loading/error contract, forms, tsconfig flags, lint-as-gate, dependency policy, prompt header |
| `prompt-driven-development/*` | Building a full app (or a documented mockup of one) through Claude-directed batches — either an external tool generates the code from prompts, or Claude writes it directly; same folder layout, validation cycle, and architecture rules either way |

## 🚧 In progress

| Standard | Status |
|---|---|
| `python/data-analysis.md` | 3/10 sections done (Notebook vs Script · Estructura · Carga de datos). **Next: §4 Inspección inicial** |

## 📝 Pending (roadmap)

| Standard | Covers |
|---|---|
| `python/api-server.md` | FastAPI como servidor: routers, dependency injection, status codes, envelope de error, validación de entrada |
| `general/auth-rbac.md` | autenticación (token, API key constant-time), autorización, modelo `modulo:accion:alcance`, scopes por prioridad |
| `python/observability.md` | logs estructurados, niveles, redacción de sensibles, métricas y trazas (pilares, no archivos), middleware de request — general, sin proveedor |
| `python/testing.md` | pytest, qué testear vs qué no, fixtures, naming |
| `python/data-science.md` | ML/AI, feature engineering, entrenamiento, evaluación |
| `python/databases.md` | queries, conexiones, ORM vs SQL, migraciones |
| `python/async.md` | async/await, asyncio, cuándo usarlo |
| `general/security.md` | secretos, validación de inputs, qué nunca va en código |
| `general/project-structure.md` | organización de carpetas |
| `javascript/testing.md` | qué testear en front más allá de funciones puras, fixtures, naming. **Solo después de que exista código real que lo pruebe** |

---

## 🧭 Decisions made

- **`data-analysis.md` y `data-science.md` son dos archivos separados:**
  - `data-analysis.md` → análisis puro: EDA, limpieza, exploración, visualización
  - `data-science.md` → ML/AI: feature engineering, entrenamiento, evaluación de modelos
  - Un proyecto de data science **siempre empieza con `data-analysis.md`** antes de modelar.
- **`data-analysis.md` (scope/estilo):** stack pandas + matplotlib + seaborn (plotly cuando se necesita); cubre workflow + convenciones; soporta notebook y script; al iniciar se pregunta "¿notebook o script?".
- **JavaScript (2026-09-08):** dejó de ser "solo planear". Se escribió
  `javascript/react-typescript.md` en versión **mínima**, con las reglas que
  salieron de la auditoría del frontend de CMS Whitelabel — cada regla tiene una
  falla observada detrás, no una preferencia. `testing.md` y las convenciones de
  internals de componentes se difieren a propósito: se escriben cuando haya
  código real que las haya validado, nunca desde primeros principios.
- **`CLAUDE.md` vs `AGENTS.md` (2026-09-13):** son dos templates raíz **paralelos e independientes** — ninguno deriva del otro, no se comparte contenido entre ambos. Criterio de selección para un proyecto nuevo: si nace con un Ejecutor separado (Antigravity/Gemini) planeado desde el día 1 → `AGENTS.md` (`Guia/templates/agents-md.html`, maestro en `agents-template.md`); si es una sesión solo-Claude sin Ejecutor planeado → `CLAUDE.md` (`Guia/templates/claude-md.html`). Si un proyecto que empezó con `CLAUDE.md` suma un Ejecutor más adelante, se migra siguiendo `general/agents-migration.md` — nunca se mantienen ambos archivos activos a la vez en el mismo proyecto.
- **Doble validación en despacho (2026-09-13):** todo ciclo de despacho a Gemini lleva dos capas de validación. (1) Auto-revalidación: se agrega siempre como último ítem de `tasks.md` — mismo flujo, mismo contexto, cero costo extra de sesión. (2) Auditoría aislada (`AGENTS.md` §5.D): un segundo prompt, siempre entregado junto al de despacho, para una sesión nueva sin contexto que solo compara `tasks.md` (qué se pidió) contra el estado real de archivos (qué se hizo) y reporta veredicto — se omite solo si el ciclo es de un archivo con cambio cosmético, y esa omisión se declara explícitamente. Los gaps que encuentre reusan `error_dump.txt` y el Modo C existente — no hay artefacto ni rama de lógica nueva.
- **`apis.md` → renombrado a `api-client.md`:** el nombre era ambiguo. `api-client.md` cubre **consumir** APIs externas (somos el cliente HTTP: auth, paginación, retry). Exponer APIs propias es otro estándar: `api-server.md` (FastAPI como servidor).
- **`error-handling.md` vs `observability.md`:** `error-handling.md` = control de flujo ante fallos (lanzar vs absorber, chaining, respuestas de error). `observability.md` = capa de instrumentación (logs estructurados, niveles, redacción de sensibles, métricas, trazas, middleware). La **mecánica de logging vive en `observability.md`**; error-handling solo referencia el formato. **Métricas no es un archivo** — es un pilar dentro de observability.
- **Generalidad vs especificidad:** los archivos de estándar son **generales/agnósticos de proveedor** (p.ej. `observability.md` describe el enfoque, **no** menciona GCP). Lo específico de un proveedor/stack lleva el proveedor en el nombre (p.ej. `cloud-services-gcp.md`) y ahí se adaptan los principios generales.
- **Audiencia (2026-05-31):** los `Standards/**/*.md` se escriben **para Claude** (solo instrucciones imperativas). Las explicaciones conceptuales y el "por qué" van a las guías HTML futuras; los `.md` se mantienen escuetos. La excepción es este `README.md`, que es el índice de estado legible.
- **Idioma (2026-05-31):** todos los `.md` de estándar se escriben en **inglés**, sin importar el idioma de la sesión de trabajo.
- **Excepción de idioma — templates con payload copiable (2026-09-13):** `agents-template.md`, `skill-template.md`, `tasks-template.md` y `superpower-plan-template.md` tienen un bloque `` ```markdown `` interno que es el **payload literal** que se copia a un proyecto real (espejo del `Guia/templates/agents-md-es.md` en español). Ese bloque se queda en español porque es el idioma real en que Karim escribe sus `AGENTS.md`/`SKILL.md` de proyecto — traducirlo lo desalinearía de la copia que de verdad se usa. La regla de "todo en inglés" sigue aplicando estrictamente a todo el texto **fuera** del bloque (la prosa explicativa de "cuándo usar esta plantilla"). `agents-migration.md` no tiene payload — va 100% en inglés como cualquier estándar normal.

---

## 🚀 How to Use

1. **When starting a project:**
   - Read `general/planning.md` to structure your approach
   - Check if your project type has a specific standard
2. **When writing code:**
   - Check the relevant file in your language folder
   - Follow the naming, structure, and testing guidelines
3. **When you encounter something not covered:**
   - Create a new standard file for it, building it as you work

---

## 🔄 Workflow

1. Check if a standard exists for that topic
2. If yes → read and follow it
3. If no → create it during/after the work
4. Update **this README** to track what's been created

---

**Last Updated:** 2026-09-13  
**Status:** In Progress (v1.0)
