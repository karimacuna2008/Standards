# Documentation Standards

Claude-facing rules for project documentation: README, CHANGELOG, and `/docs`. Docstrings and code comments are covered in `python/code-standards.md` and `python/functions.md` — cross-reference, do not duplicate here.

## 1. README.md — required
Every project has a root `README.md`. Include these sections in order; omit any that don't apply:

1. **Title + description** — one line on what it does, then a short paragraph on the problem it solves.
2. **Features** — bullet list.
3. **Installation** — prerequisites, then steps (clone, venv, `pip install -r requirements.txt`, config).
4. **Usage** — minimal runnable example; common use cases; CLI invocation if applicable.
5. **Configuration** — environment variables and config-file keys.
6. **Architecture** — medium/large projects only: components, data flow, key decisions.
7. **Testing** — how to run the tests, if any.
8. **Troubleshooting** — common errors as an Issue → Solution table.

Skeleton:

```markdown
# Project Name
One-line description.

## Features
- ...

## Installation
### Prerequisites
- Python 3.x
### Steps
<commands>

## Usage
<minimal example>

## Configuration
- `ENV_VAR` — purpose

## Testing
<command>
```

## 2. CHANGELOG.md
Required once the project has releases. Root-level, newest entry on top.

- One entry per release: `## [MAJOR.MINOR.PATCH] - YYYY-MM-DD`.
- Group under **Added · Changed · Fixed · Removed · Deprecated** (omit empty groups).
- User-focused wording, not implementation detail.
- Link each version to its release tag.

## 3. When to create `/docs`
Create `/docs` for: APIs, libraries, and large/multi-module apps. Skip it for simple scripts, small utilities, and internal tools.

When it exists, typical files: `installation.md`, `architecture.md`, `api.md` (endpoint reference: method, path, request, response, error codes), `configuration.md`, `deployment.md`, `troubleshooting.md`.

## 4. Summary
| Item | Required | Where |
|---|---|---|
| README | Yes | Root |
| CHANGELOG | Yes, if releases | Root |
| `/docs` | Depends (§3) | Root |
| Docstrings / comments | Yes | `code-standards.md` · `functions.md` |

---
*Full README examples, writing do's/don'ts, and reference links are human-facing → future HTML guide.*

**Version:** 2.0  
**Last Updated:** 2026-05-31
