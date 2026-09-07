# Database Migrations

Cómo evolucionar el esquema SQL de forma segura.

---

## 1. Idempotencia, no parseo de errores

Una migración debe poder correrse varias veces sin romper. **Nunca** decidas si una migración "ya se aplicó" parseando el **texto** del error del motor (ej. `"1060"`, `"1054"` de MySQL): es frágil y depende de la versión/idioma del servidor.

En su lugar, **pregúntale al esquema** con el inspector de SQLAlchemy antes de aplicar:

```python
from sqlalchemy import inspect

inspector = inspect(engine)
existe_tabla = inspector.has_table("webhook_events")
columnas = {col["name"] for col in inspector.get_columns("merchants")}
```

- `ADD COLUMN` → aplica solo si la columna **no** existe.
- `RENAME COLUMN a → b` → aplica solo si la columna **destino** aún no existe.
- `CREATE TABLE` → usa `CREATE TABLE IF NOT EXISTS` y/o verifica `has_table`.

## 2. Cuándo basta esto y cuándo Alembic

- **Esquema estable, pocos cambios, un solo entorno:** una lista de migraciones idempotentes verificadas con el inspector es suficiente, sin setup extra.
- **Esquema que cambia seguido, varios entornos, o necesitas rollback/historial:** migra a **Alembic** — versionado real (tabla `alembic_version`, `upgrade`/`downgrade`). Es el destino recomendado cuando el método ligero se queda corto.

## 3. Reglas

- Cada migración lleva metadata de **qué verifica** (tabla/columna), para que el chequeo de "ya aplicada" no dependa de adivinar.
- Una migración que falla se **loguea** pero no detiene a las demás ni tumba el arranque/endpoint.
- El naming de columnas y tablas sigue `database.md` (PK/FK descriptivas, palabras completas, nunca `id` a secas).

---
**Version:** 1.0
**Last Updated:** 2026-06-29
