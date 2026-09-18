# Authorization & RBAC Standard (Role-Based Access Control)

> ⚠️ **STATUS: DRAFT PROPOSED BY GEMINI - PENDING VALIDATION WITH CLAUDE**
> <!-- VALIDAR CON CLAUDE: Review triplet permission syntax and dependency checks before final consolidation -->

Operational rules for role-based access control across internal platforms and multi-tenant applications.

---

## 1. Permission Naming Schema: Triplet Syntax
Permissions must be defined as granular string keys following the canonical format:
`modulo:accion:alcance`

- **`modulo`:** The business domain (e.g. `empleados`, `comisiones`, `inventario`, `sistema`).
- **`accion`:** The operation performed (`crear`, `leer`, `actualizar`, `eliminar`, `aprobar`, `exportar`).
- **`alcance`:** The target boundary (`todos`, `propios`, `equipo`, `global`).

### Examples:
- `empleados:leer:todos`: Can view any employee record.
- `ausencias:solicitar:propias`: Can request personal time off.
- `comisiones:aprobar:global`: Can approve commission payouts across the company.

---

## 2. Role Definitions
Roles are named sets of permissions stored in a configuration dictionary or database catalog:

```python
ROLES = {
    "admin": [
        "*:*:*",  # Wildcard superuser
    ],
    "manager": [
        "empleados:leer:equipo",
        "ausencias:aprobar:equipo",
        "comisiones:leer:equipo",
    ],
    "colaborador": [
        "ausencias:solicitar:propias",
        "ausencias:leer:propias",
        "reconocimientos:crear:global",
    ],
}
```

---

## 3. Enforcement in APIs (FastAPI Dependency)
Protect endpoints declaratively at the router layer using dependency injection:

```python
from fastapi import APIRouter, Depends, HTTPException, status
from core.security import get_current_user, require_permission

router = APIRouter(prefix="/empleados", tags=["Empleados"])

@router.get("/", dependencies=[Depends(require_permission("empleados:leer:todos"))])
def list_all_employees():
    return {"data": []}
```

---
**Version:** 1.0 (Draft)  
**Last Updated:** 2026-09-17
