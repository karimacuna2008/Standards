# Python Relational Databases Standard (SQLAlchemy 2.0 & Alembic)

> ⚠️ **STATUS: DRAFT PROPOSED BY GEMINI - PENDING VALIDATION WITH CLAUDE**
> <!-- VALIDAR CON CLAUDE: Review SQLAlchemy 2.0 ORM patterns, mixins, and engine builder extracted from Nexus and App de Faltantes DB -->

Operational instructions for database connections, ORM models, session management, and migrations using SQLAlchemy 2.0 in Python.

---

## 1. Engine & Session Factory Pattern
Extract and centralize database connection logic in `src/db.py`:

```python
from sqlalchemy import create_engine
from sqlalchemy.engine import Engine
from sqlalchemy.orm import sessionmaker
from src import config

def build_engine(url: str | None = None) -> Engine:
    """Builds an engine with connection health validation."""
    return create_engine(
        url or config.get_database_url(),
        connect_args=config.get_connect_args(),
        pool_pre_ping=True,  # Verifies connection health before checkout
        future=True,
    )

# Session factory for scoped transactions
SessionLocal = sessionmaker(autoflush=False, expire_on_commit=False, future=True)
```

---

## 2. Base Model & Standard Mixins (`src/models/base.py`)
All ORM entities must inherit from `DeclarativeBase` with strict type annotations (`Mapped[]` and `mapped_column()`):

```python
from datetime import datetime
from sqlalchemy import Boolean, DateTime, text
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column

# Standard MySQL table arguments: InnoDB engine, utf8mb4 charset, case-insensitive collation
TABLE_ARGS = {
    "mysql_engine": "InnoDB",
    "mysql_charset": "utf8mb4",
    "mysql_collate": "utf8mb4_0900_ai_ci",
}

class Base(DeclarativeBase):
    pass

class TimestampMixin:
    """Reusable audit timestamps for creation and update tracking."""
    created_at: Mapped[datetime] = mapped_column(
        DateTime, nullable=False, server_default=text("CURRENT_TIMESTAMP")
    )
    updated_at: Mapped[datetime] = mapped_column(
        DateTime,
        nullable=False,
        server_default=text("CURRENT_TIMESTAMP"),
        server_onupdate=text("CURRENT_TIMESTAMP"),
    )

def activo_column() -> Mapped[bool]:
    """Standard soft-delete boolean flag (default 1). Return a new instance per call."""
    return mapped_column(Boolean, nullable=False, server_default=text("1"))
```

---

## 3. Querying & Transactions (SQLAlchemy 2.0 Syntax)
- **Prohibited:** Legacy 1.x syntax `session.query(Model).filter(...)` is banned.
- **Mandatory 2.0 Syntax:** Use `select()` and `session.execute()`:
  ```python
  from sqlalchemy import select

  with SessionLocal() as session:
      stmt = select(User).where(User.activo == True).order_by(User.created_at.desc())
      users = session.scalars(stmt).all()
  ```
- **Context Managers for Mutations:**
  ```python
  with SessionLocal() as session:
      with session.begin():
          session.add(new_entity)
      # Commits automatically on block exit or rollbacks on exception
  ```

---

## 4. Schema Migrations (Alembic)
- Every project with a database must configure Alembic (`alembic init alembic`).
- `alembic/env.py` must point to `Base.metadata` to support `--autogenerate`.
- Never modify applied migration files in production; generate a new revision via `alembic revision --autogenerate -m "description"`.

---
**Version:** 1.0 (Draft)  
**Last Updated:** 2026-09-17
