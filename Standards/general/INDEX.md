# General Standards — INDEX

Lee este archivo primero para saber qué estándar usar según la tarea.

| Archivo | Cuándo usarlo |
|---|---|
| `planning.md` | Planear proyectos y flujos de trabajo antes de escribir código |
| `git-workflow.md` | Commits, branches, PRs, tags, convenciones de Git |
| `documentation.md` | READMEs, CHANGELOG, docstrings, carpeta /docs |
| `deployment.md` | Desplegar con Cloud Build + CI/CD desde GitHub: APIs a Cloud Run, apps de escritorio (Electron) a un bucket con auto-update, jobs/migraciones gateadas — incluye el patrón de monorepo con un trigger por carpeta |
| `database.md` | Diseñar y nombrar esquemas SQL (PKs, FKs, naming, NOT NULL) |
| `database-migrations.md` | Evolucionar el esquema: migraciones idempotentes, inspector de SQLAlchemy, cuándo Alembic |
| `database-remote-access.md` | Conectarte a una DB con firewall por IP desde una red no autorizada — bastion VM en GCP + IAP tunneling |
| `agents-template.md` | Plantilla raíz `AGENTS.md` para proyectos con Ejecutor separado (Antigravity/Gemini) — template independiente de `Guia/templates/claude-md.html` (single-agent), no lo reemplaza |
| `skill-template.md` | Plantilla `SKILL.md` para cada submódulo del monorepo listado en `AGENTS.md` §8 — reglas locales de dominio y comandos de test |
| `tasks-template.md` | Plantilla `tasks.md` — buffer efímero de tareas atómicas para el Ejecutor, con protocolo de volcado a `error_dump.txt` |
| `superpower-plan-template.md` | Plantilla de plan de arquitectura (`docs/superpowers/plans/YYYY-MM-DD-nombre.md`) — contexto, decisiones de diseño, diseño técnico, intentos previos y fallos, desglose de tasks |
| `agents-migration.md` | Checklist para migrar un proyecto ya iniciado con `CLAUDE.md` hacia `AGENTS.md` cuando suma un Ejecutor separado |
| `security.md` | Seguridad transversal: cero secretos en código, cuarentena git, templates de entorno y sanitización de logs |
| `auth-rbac.md` | Control de acceso basado en roles (RBAC): nomenclatura triplete modulo:accion:alcance y protección de routers |
| `project-audit.md` | Auditoría, descubrimiento y estandarización de proyectos legacy/existentes con subagentes y matriz de modelos |

> Cuando crees un nuevo estándar general, agrégalo a esta tabla antes de cerrar la sesión.


