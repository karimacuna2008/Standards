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
│   ├── project-structure.md     📝 TODO — folder organization
│   ├── security.md              📝 TODO — secrets, input validation
│   └── auth-rbac.md             📝 TODO — authn/authz, API keys, RBAC/scopes
├── python/
│   ├── code-standards.md        ✅ Style, structure, conventions
│   ├── functions.md             ✅ How to write functions
│   ├── api-client.md            ✅ Consume external REST APIs (HTTP client)
│   ├── automation.md            ✅ CLI scripts, bots
│   ├── error-handling.md        ✅ Exception control flow
│   ├── data-analysis.md         🚧 IN PROGRESS (3/10 sections)
│   ├── api-server.md            📝 TODO — FastAPI as a server
│   ├── observability.md         📝 TODO — structured logs, metrics, tracing
│   ├── cloud-services-gcp.md    ✅ GCP integration (provider-specific)
│   ├── testing.md               📝 TODO
│   ├── data-science.md          📝 TODO
│   ├── databases.md             📝 TODO
│   └── async.md                 📝 TODO
├── javascript/                  📝 TODO (planning only)
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
| `python/code-standards.md` | PEP 8 (120 chars), f-strings, imports, naming, logging |
| `python/functions.md` | Naming, type hints, docstrings, when to extract |
| `python/api-client.md` | Consume external REST APIs (HTTP client): auth, pagination, retry, client-as-dependency |
| `python/automation.md` | CLI scripts, menu vs argparse, output conventions |
| `python/error-handling.md` | Exception control flow: raise vs absorb, chaining, error responses (logging mechanics live in `observability.md`) |
| `python/cloud-services-gcp.md` | GCP integration: Firestore, Cloud Storage, Firebase Auth, Cloud Logging, .gcloudignore, Cloud Build deploy |
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
| `javascript/*` | decidir cuáles aplican según el stack (solo planear) |

---

## 🧭 Decisions made

- **`data-analysis.md` y `data-science.md` son dos archivos separados:**
  - `data-analysis.md` → análisis puro: EDA, limpieza, exploración, visualización
  - `data-science.md` → ML/AI: feature engineering, entrenamiento, evaluación de modelos
  - Un proyecto de data science **siempre empieza con `data-analysis.md`** antes de modelar.
- **`data-analysis.md` (scope/estilo):** stack pandas + matplotlib + seaborn (plotly cuando se necesita); cubre workflow + convenciones; soporta notebook y script; al iniciar se pregunta "¿notebook o script?".
- **JavaScript:** solo planear; decidir qué standards aplican según el stack usado.
- **`apis.md` → renombrado a `api-client.md`:** el nombre era ambiguo. `api-client.md` cubre **consumir** APIs externas (somos el cliente HTTP: auth, paginación, retry). Exponer APIs propias es otro estándar: `api-server.md` (FastAPI como servidor).
- **`error-handling.md` vs `observability.md`:** `error-handling.md` = control de flujo ante fallos (lanzar vs absorber, chaining, respuestas de error). `observability.md` = capa de instrumentación (logs estructurados, niveles, redacción de sensibles, métricas, trazas, middleware). La **mecánica de logging vive en `observability.md`**; error-handling solo referencia el formato. **Métricas no es un archivo** — es un pilar dentro de observability.
- **Generalidad vs especificidad:** los archivos de estándar son **generales/agnósticos de proveedor** (p.ej. `observability.md` describe el enfoque, **no** menciona GCP). Lo específico de un proveedor/stack lleva el proveedor en el nombre (p.ej. `cloud-services-gcp.md`) y ahí se adaptan los principios generales.
- **Audiencia (2026-05-31):** los `Standards/**/*.md` se escriben **para Claude** (solo instrucciones imperativas). Las explicaciones conceptuales y el "por qué" van a las guías HTML futuras; los `.md` se mantienen escuetos. La excepción es este `README.md`, que es el índice de estado legible.
- **Idioma (2026-05-31):** todos los `.md` de estándar se escriben en **inglés**, sin importar el idioma de la sesión de trabajo.

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

**Last Updated:** 2026-08-02  
**Status:** In Progress (v1.0)
