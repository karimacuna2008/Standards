# Observability Standard (Structured Logging, Metrics & Tracing)

> ⚠️ **STATUS: DRAFT PROPOSED BY GEMINI - PENDING VALIDATION WITH CLAUDE**
> <!-- VALIDAR CON CLAUDE: Review prefix tags, structured JSON payload, and EndpointDebugError contract before final consolidation -->

Operational instructions for instrumenting Python backends, APIs, and automated scripts with structured logging, execution metrics, and request tracing.

---

## 1. Core Principles
- **No Raw `print()` in Production:** Unformatted `print()` statements are prohibited in production services and background jobs. (CLI interactive tools may use prints only for the console UI, per `python/automation.md`).
- **Structured JSON with Searchable Prefix Tags:** Production logs must output JSON payloads preceded by human-readable prefix tags (e.g. `[INPUT]`, `[OUTPUT]`, `[ERROR]`, `[METRICS]`) or structured keys, enabling instant terminal scanning and automated log ingestion.
- **Request Context & Correlation:** Every incoming request or background batch must be assigned a unique `request_id` (via `ContextVar`) and correlated with cloud trace headers (`x-cloud-trace-context` for GCP / Cloud Run).
- **Zero Sensitive Data:** All logged dictionaries (headers, bodies, query params) must pass through automatic sanitization to redact credentials, tokens, and passwords.

---

## 2. Standard Log Taxonomy & Prefix Tags

| Tag / Type | When to Emit | Key Fields |
|---|---|---|
| `[INPUT]` | At request arrival / task start | `endpoint`, `method`, `request_id`, sanitized `query_params`, sanitized `body_preview` |
| `[OUTPUT]` | At request completion / task finish | `endpoint`, `method`, `status`, `latency_ms`, `request_id`, sanitized `response_body` |
| `[ERROR]` | Handled or unhandled exception | `step`, `error_type`, `message`, `context`, `request_id`, `traceback` |
| `[METRICS]` | Resource usage / batch statistics | `db_reads`, `db_writes`, `items_processed`, `latency_ms` |

---

## 3. Visual Separators & Context Tracking
For high-traffic or debug logs, wrap request lifecycles with clear demarcation lines to facilitate human reading in local dev or Cloud Logging text viewers:

```python
START_SEPARATOR = "=" * 120
END_SEPARATOR   = "_" * 120

# Usage in middleware:
print(START_SEPARATOR, flush=True)
# ... process request and emit [INPUT], [OUTPUT] ...
print(END_SEPARATOR, flush=True)
```

---

## 4. Structured Error Contract (`EndpointDebugError`)
For granular error tracking across services and routers, define a custom domain debug exception:

```python
class EndpointDebugError(Exception):
    """Raised when an internal process step fails with precise diagnostic context."""
    def __init__(
        self,
        step: str,
        message: str,
        context: dict | None = None,
        cause: Exception | None = None,
        status_code: int = 500,
    ):
        super().__init__(message)
        self.step = step
        self.message = message
        self.context = context or {}
        self.cause = cause
        self.status_code = status_code
```

---

## 5. Middleware Integration (FastAPI Pattern)
Every FastAPI service must attach an observability middleware that:
1. Generates or captures `request_id` (`uuid.uuid4().hex[:12]`).
2. Measures duration `latency_ms = int((time.time() - t0) * 1000)`.
3. Injects `x-request-id` into the response headers.
4. Emits `[INPUT]` and `[OUTPUT]` events and handles uncaught exceptions gracefully.

---
**Version:** 1.0 (Draft)  
**Last Updated:** 2026-09-17
