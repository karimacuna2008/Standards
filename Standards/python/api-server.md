# FastAPI Server Architecture Standard

> ⚠️ **STATUS: DRAFT PROPOSED BY GEMINI - PENDING VALIDATION WITH CLAUDE**
> <!-- VALIDAR CON CLAUDE: Review FastAPI server structure, CORS resolver, and global handlers extracted from Nexus and CMS Whitelabel -->

Operational instructions for architecting, building, and maintaining FastAPI backend services.

---

## 1. Project Directory Layout
Extracted from production microservices (Nexus, CMS Whitelabel, App de Faltantes):

```
project/
├── main.py                     # Entry point: app instantiation, middleware, exception handlers, router mounts
├── core/
│   ├── config.py               # Settings, environment variables, CORS origins resolver
│   ├── observability.py        # Logging middleware, request context, EndpointDebugError
│   └── security.py             # Auth verification, API key/token dependencies
├── models/                     # Pydantic schemas (request/response validation)
├── routers/                    # Lean endpoints grouped by domain (e.g. sistema.py, auth.py, items.py)
├── services/                   # Business logic and external calls (zero FastAPI dependencies)
├── tests/                      # Pytest suite with TestClient
├── Dockerfile                  # Container build specification
├── requirements.txt            # Production dependencies
└── .env.example                # Mandatory environment variable template
```

---

## 2. `main.py` Standard Blueprint
`main.py` acts strictly as an assembly pipeline:

```python
import os
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from core.observability import (
    EndpointDebugError,
    endpoint_debug_error_handler,
    http_exception_handler,
    unhandled_exception_handler,
    usage_metrics_middleware,
)
from routers.sistema import router as sistema_router
# ... other domain routers ...

app = FastAPI(title="service-name", version="1.0.0")

# 1. Dynamic CORS resolution
def _resolve_cors_origins() -> list[str]:
    raw = (os.getenv("CORS_ORIGINS") or "").strip()
    if not raw:
        return ["http://localhost:3000", "http://localhost:5173"]
    return [origin.strip() for origin in raw.split(",") if origin.strip()]

app.add_middleware(
    CORSMiddleware,
    allow_origins=_resolve_cors_origins(),
    allow_credentials=False,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 2. Middleware & Exception Handlers
app.middleware("http")(usage_metrics_middleware)
app.add_exception_handler(HTTPException, http_exception_handler)
app.add_exception_handler(EndpointDebugError, endpoint_debug_error_handler)
app.add_exception_handler(Exception, unhandled_exception_handler)

# 3. Router Inclusions
app.include_router(sistema_router)
# ... include other routers ...
```

---

## 3. Router Conventions
- **Lean Routers:** Routers only receive requests, validate schemas via Pydantic, call a service function, and return the response. No direct SQL queries or heavy algorithms inside router functions.
- **Mandatory System Router (`routers/sistema.py`):** Every API must expose a lightweight health and version endpoint:
  - `GET /health`: Returns `{"status": "ok", "service": "name", "timestamp": "..."}`.
  - `GET /version`: Returns git commit SHA or release version.
- **Dependency Injection:** Auth and database sessions must be passed via `fastapi.Depends`.

---

## 4. Error Responses
Standardize error payloads:
```json
{
  "detail": {
    "step": "validar_permisos",
    "message": "Token de autenticación expirado",
    "context": {}
  }
}
```

---
**Version:** 1.0 (Draft)  
**Last Updated:** 2026-09-17
