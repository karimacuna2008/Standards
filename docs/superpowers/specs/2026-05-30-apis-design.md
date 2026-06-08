# Design Spec: Python APIs Standard

**Date:** 2026-05-30  
**Target file:** `Standards/python/api-client.md`  
**Status:** Approved

---

## Scope

Standard for integrating with external REST APIs in Python projects. Covers 8 rules that apply to any HTTP client (requests, httpx, SDK wrappers).

---

## Sections

### 1. Authentication
- Always load from `.env` via `load_dotenv()` + `os.getenv()`
- Never hardcode credentials in source code
- Call `load_dotenv()` before any `os.getenv()` call
- Never log or print the API key, even partially

### 2. Client Instantiation
- Create the client once in `main()`, pass as parameter to functions
- Never use module-level global client instances
- Reason: global clients create hidden dependencies, make testing harder, and obscure what each function needs

### 3. Separate Fetch from Transform
- The function that calls the API returns raw data only
- A separate function handles mapping/extraction
- Exception: if no transformation needed, one function is fine
- Rule: never mix the HTTP call with data mapping in the same function

### 4. Pagination
- Pattern: `while True` + explicit break condition + cursor accumulation
- Always set `limit` explicitly — never rely on API defaults
- Initialize cursor as `None` for first page
- Accumulate results in local list, return at end — don't process inside loop

### 5. Retry and Timeout
- Only retry on 5xx errors and timeouts — never on 4xx
- Use exponential backoff: `time.sleep(2 ** attempt)` — 2s, 4s, 8s
- Always set `timeout` explicitly on every request
- Raise exception after exhausting retries — never return `None` silently

### 6. HTTP Error Handling
- 401/403 → always raise `PermissionError`
- 404 → return `{}` if optional resource, raise if required
- 429 → raise `RuntimeError` with `Retry-After` value from headers
- 5xx → raise `RuntimeError` (retry already exhausted)
- Never return `None` silently on critical errors

### 7. Response Validation
- Validate required keys before accessing them
- Validate type when it matters (list vs dict)
- Include actual response context in error messages
- Don't over-validate — only keys/types the code will directly use

### 8. Logging API Calls
- `INFO` before calling and on success (include endpoint + elapsed time)
- `WARNING` on retry and rate limit hits
- `ERROR` on definitive failure after all retries
- Never log API key or sensitive request body fields
- Always include endpoint in log message

---

## Decisions Made

- **Option C chosen** (Comprehensive) over B — response validation and API-specific logging don't overlap with other standards
- Logging conventions complement `code-standards.md` section 10 without duplicating it
- `404` handling left context-dependent by design — both behaviors (return empty / raise) are valid depending on use case
