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

## 7. Grouped Config vs. Micro-Getters (No Single-Variable Functions)
- **Prohibición de micro-getters por variable:** Nunca crear funciones getter diminutas de una sola línea por cada variable o clave de configuración (`get_api_url()`, `get_api_key()`, `get_port()`). Este antipatrón infla el código con funciones triviales y dispersa el acceso.
- **Configuraciones agrupadas en estructuras coherentes:** Las variables y secretos afines deben agruparse en estructuras fuertemente tipadas (`@dataclass(frozen=True)`, `NamedTuple` o `BaseSettings`). El servicio o módulo consumidor solicita el bloque agrupado completo que necesita (ej. `get_api_config()`, `get_database_config()` o `load_config()`).
- **Prohibido crear funciones sueltas si el dato ya está en la configuración:** Si un valor o diccionario ya reside en memoria dentro del objeto de configuración (ej. `drive_roots`), nunca crees una función suelta a nivel de módulo (`def get_drive_root_id()`). El consumidor debe acceder directamente a la propiedad (`app_cfg.drive_roots.get(cliente)`) o a un método de la propia estructura (`app_cfg.get_drive_root_id(cliente)`).
- **Criterio estricto para separar funciones a nivel de módulo:** Únicamente se justifica crear una función independiente fuera de la estructura cuando:
  1. **Lectura diferida o costosa (Lazy Loading real):** Si obtener el valor implica una llamada HTTP externa, consulta a base de datos o descifrado pesado que no debe ejecutarse en el arranque general si no se va a utilizar.
  2. **Resolución dinámica con dependencias externas no configurables al inicio.**

---
**Version:** 1.1  
**Last Updated:** 2026-09-17
