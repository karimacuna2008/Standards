# Python Asynchronous Programming Standard

> ⚠️ **STATUS: DRAFT PROPOSED BY GEMINI - PENDING VALIDATION WITH CLAUDE**
> <!-- VALIDAR CON CLAUDE: Review TaskGroup and asyncio.to_thread concurrency patterns before final consolidation -->

Operational rules for writing asynchronous Python code (Python 3.11+).

---

## 1. When to Use `async` vs `def`
- **Use `async def`:** Only when performing concurrent I/O operations with native non-blocking async libraries (`httpx.AsyncClient`, `asyncpg`, `aiofiles`, FastAPI handlers).
- **Use standard `def`:** For CPU-bound computations, local file manipulation, synchronous client libraries (requests, pandas), or when no concurrent I/O benefit exists.

---

## 2. Structured Concurrency (`asyncio.TaskGroup`)
- **Prohibited:** Unmanaged `asyncio.gather()` or bare `asyncio.create_task()` calls that can leave background tasks orphaned when an exception occurs.
- **Mandatory (Python 3.11+):** Use `asyncio.TaskGroup()` context managers:

```python
import asyncio

async def fetch_item(item_id: str) -> dict:
    ...

async def process_batch(item_ids: list[str]) -> list[dict]:
    results = []
    async with asyncio.TaskGroup() as tg:
        for item_id in item_ids:
            tg.create_task(fetch_item(item_id))
    # All tasks are guaranteed to complete or cleanly cancel upon exception
    return results
```

---

## 3. Offloading Blocking Operations (`asyncio.to_thread`)
Never run blocking synchronous calls (e.g. `time.sleep()`, synchronous `requests.get()`, heavy Pandas calculations, or blocking file I/O) directly inside an `async def` function. Always offload them to an OS thread:

```python
import asyncio
import time

def blocking_heavy_computation(df_raw):
    # Synchronous CPU-bound processing
    time.sleep(2)
    return df_raw.describe()

async def api_handler(payload):
    # Offload cleanly without freezing the event loop
    summary = await asyncio.to_thread(blocking_heavy_computation, payload)
    return {"summary": summary}
```

---
**Version:** 1.0 (Draft)  
**Last Updated:** 2026-09-17
