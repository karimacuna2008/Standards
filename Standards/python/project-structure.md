# Project Structure Standard

How to lay out a medium/large Python project (FastAPI service).

## 1. Layout
- App de un solo paquete bajo `src/`. `main.py` en la raíz solo crea la app y la arranca.
- `src/config.py` (constantes + env), `src/clients.py` (clientes externos), `src/logging_utils.py`, `src/security.py` (auth).
- `src/models/` (Pydantic por área), `src/services/` (lógica/IO por dominio, sin FastAPI), `src/routers/` (un `APIRouter` por área; endpoints delgados).
- `tests/` espeja la estructura.

## 2. Reglas
- Una responsabilidad por módulo; los que cambian juntos viven juntos.
- Los routers solo: validar → llamar a un service → devolver. Sin lógica de negocio.
- Los services no importan FastAPI ni routers (testeables solos). Reciben los clientes por parámetro.
- Imports absolutos desde `src.` Paquetes con `__init__.py`.

## 3. main.py
Crea `FastAPI`, middleware, valida entorno al arranque, monta routers (`include_router`), y arranca uvicorn bajo `if __name__ == "__main__":`.

---
**Version:** 1.0
**Last Updated:** 2026-06-04
