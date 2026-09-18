# Security & Secrets Standard

> ⚠️ **STATUS: DRAFT PROPOSED BY GEMINI - PENDING VALIDATION WITH CLAUDE**
> <!-- VALIDAR CON CLAUDE: Review secret management, quarantine rules, and token sanitization before final consolidation -->

Universal operational security rules across all projects, repositories, and languages.

---

## 1. Zero Secrets in Source Code
- **Prohibited:** Never hardcode passwords, API keys, private tokens, OAuth client secrets, or Service Account private keys inside any file tracked by version control.
- **Compromise Policy:** Any credential committed to Git must be treated as instantly compromised and immediately revoked and rotated.

---

## 2. Credential Quarantine & Git Exclusions
Every project repository that requires configuration or authentication must include the following patterns in its local `.gitignore`:

```text
# Environment & Secrets
.env
.env.*
!.env.example
*.pem
*.key
*service_account*.json
*credentials*.json

# Streamlit Secrets
.streamlit/secrets.toml
!.streamlit/secrets.toml.example
```

---

## 3. Mandatory Template Files
To guarantee reproducible onboarding and deployment without exposing credentials, every project must provide committed template files with dummy/placeholder values:
- Backend / CLI services: `.env.example`
- Streamlit applications: `.streamlit/secrets.toml.example`

Format:
```ini
# .env.example
API_KEY=your_api_key_here
DATABASE_URL=mysql+pymysql://user:password@localhost:3306/dbname
CORS_ORIGINS=http://localhost:3000,http://localhost:5173
```

---

## 4. Redaction & Sanitization in Logs and Exceptions
- **Sensitive Key Redaction:** Loggers and middleware must automatically redact keys matching:
  `["x-api-key", "api_key", "apikey", "authorization", "token", "access_token", "refresh_token", "password", "secret"]`.
  Their values must be replaced with `"***REDACTED***"` before serializing to stdout or disk.
- **Exception Sanitization:** End-user error responses must return generic status messages and a correlated `request_id`. Never leak raw database connection strings, database passwords, or full server paths in HTTP 4xx/5xx responses.

---

## 5. Selective Git Invisibility vs Public Documentation
- **Orchestration / Context Invisibility:** Internal orchestration files (`AGENTS.md`, `SKILL.md`, `tasks.md`, `error_dump.txt`, `discovery.md`, `.claude/`, `docs/superpowers/`, `*.spec.md`) MUST be excluded locally via `.git/info/exclude` (not `.gitignore`).
- **Public Documentation Preservation:** Never exclude `docs/` wholesale. Public status docs (`docs/Estatus Actual/`), architecture specs (`docs/architecture.md`), and standards gap reports (`docs/STANDARDS_GAP.md`) must remain tracked and committed to Git.

---
**Version:** 1.0 (Draft)  
**Last Updated:** 2026-09-17
