# Configuration Standard

How Python projects manage configuration, environment variables and resource IDs. Keep config out of business logic and out of source literals.

## 1. Central module
- All config lives in a single `config.py` (small/medium projects) or `src/config.py` (modular layout). No resource IDs, scopes, ports or tunables hardcoded inside business logic.
- Import config **by name** where needed (`from config import PROJECT_ID`); never redefine a config constant locally.

## 2. Environment first, sensible defaults
- Read every environment-specific value (project/resource IDs, DB names, ports, tunables) from the environment: `os.getenv("NAME", default)`.
- Provide a default **only** for non-secret values, so the app boots in dev. Coerce types explicitly: `int(os.getenv("PORT", "8080"))`.

## 3. Naming & casing
- Config constants are `UPPER_SNAKE_CASE` (`PROJECT_ID`, `PHOTOS_MAX_CONCURRENT`).
- Group with comment headers by area (GCP, OAuth, server, feature flags).

## 4. Required values & startup validation
- A `validate_env()` runs at startup and **fails fast** (clear message naming the missing var) when a required value is absent. Misconfiguration surfaces at boot, never mid-request.
- **Secrets** (API keys, tokens) have **no default** — they are required.

## 5. Secrets
- Secrets come from the environment / a secret manager, never from source or git. `.env` is git-ignored; never log secret values.

## 6. Evolution — `pydantic-settings` (optional)
- For larger configs, many secrets, `.env` files, or multiple environments, a `pydantic-settings` `BaseSettings` class is the recommended upgrade: it reads env, coerces types and validates (covering §4) at instantiation. A plain `os.getenv` module is sufficient for small configs.

---
**Version:** 1.0
**Last Updated:** 2026-06-04
