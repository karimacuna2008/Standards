# FastAPI — async vs def

Cuándo declarar un endpoint `async def` y cuándo `def`, para no bloquear el event loop.

---

## 1. La regla

- Si el handler hace **I/O bloqueante** (drivers síncronos: MySQL vía pymysql, Firebase Admin SDK, httpx síncrono, clientes de Google Cloud síncronos) → declara el endpoint con **`def`**. FastAPI lo ejecuta en un threadpool y el event loop queda libre.
- Si el handler es **totalmente async** (usa `await` sobre I/O async: `httpx.AsyncClient`, driver async de DB) → declara **`async def`**.
- **Nunca** mezcles: un `async def` que llama I/O bloqueante sin `await` **congela el event loop** para todas las peticiones concurrentes, no solo para la suya.

## 2. Cuando necesitas async pero el trabajo es bloqueante

Si el endpoint debe ser `async def` (p. ej. usa `await request.json()`) pero por dentro hace trabajo bloqueante:

- Mándalo a un threadpool: `await run_in_threadpool(funcion_bloqueante, args)` (`from fastapi.concurrency import run_in_threadpool`).
- O agéndalo con `background_tasks.add_task(...)` si no necesitas el resultado en la respuesta (ideal para webhooks: responde rápido y procesa después).

## 3. Decisión rápida

| El handler… | Declaración |
|---|---|
| Hace queries síncronas / usa un SDK síncrono | `def` |
| Solo `await` sobre I/O async | `async def` |
| `async def` obligado + trabajo bloqueante | `async def` + `run_in_threadpool` / `BackgroundTasks` |

## 4. Por qué no "async siempre"

Migrar toda la capa de datos a async real solo rinde a muy alto QPS sostenido, y exige drivers async en todo el stack. Algunos SDKs (Firebase Admin) no tienen API async, así que igual terminas en un threadpool. Para servicios internos de bajo/medio volumen sobre Cloud Run (que escala horizontalmente añadiendo instancias), `def` + threadpool es la opción correcta y mucho más simple.

---
**Version:** 1.0
**Last Updated:** 2026-06-29
