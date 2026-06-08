# Testing Standard

How to test Python projects. When refactoring existing code, start with **characterization tests** as a safety net; add unit tests for pure logic.

## 1. Tooling & layout
- `pytest` as the runner; `httpx` + FastAPI `TestClient` for endpoints.
- Tests live in `tests/`, one module per unit/endpoint: `test_<thing>.py`.
- Shared fixtures in a root `conftest.py`. Ensure the project root is importable (a root `conftest.py` that does `sys.path.insert(0, ...)`, or `pythonpath = .` in `pytest.ini`).

## 2. Test types
- **Characterization** — pin the **current** behavior of existing code before refactoring. They assert what the code does **today**, not what it "should" do. Required before restructuring legacy logic; they must stay green before and after each step.
- **Unit** — verify pure functions/helpers in isolation.
- **Integration** — exercise a flow across units, with external services mocked.

## 3. Mocking external services
- Never hit real Google/Drive/Sheets/Firestore/HTTP in tests. Mock at a clear **seam**: patch the project's own accessor functions (`get_drive_service`, `count_videos_in_folder`, …) with `monkeypatch.setattr`.
- Let pure helpers (string parsing, mapping) run for real.

## 4. Naming & structure
- `test_<unit>_<case>()` — e.g. `test_total_videos_happy`, `test_total_videos_blank_campana`.
- Arrange / act / assert; one behavior per test.
- For characterization, assert the **full** response shape (status + body) so any drift is caught.

## 5. Running
- `pytest` (all) · `pytest tests/test_total_videos.py -v` (one module).
- Green before and after every refactor step — no exceptions.

---
**Version:** 1.0
**Last Updated:** 2026-06-04
