# Python Error Handling Standard

How to handle errors, exceptions, and failures in Python projects.

> Note: bare `except:` and generic `except Exception` are covered in `code-standards.md` §9.

---

## 1. Response Hierarchy

Three levels of response depending on the severity of the error:

```python
# Level 1 — print and continue (minor error, the script can keep going)
for user in users:
    if not user.get('email'):
        print(f"  ⚠️ User {user['id']} has no email — skipping")
        continue

# Level 2 — return (operation failed, the caller decides what to do)
def process_file(path: str) -> dict | None:
    if not os.path.exists(path):
        print(f"  ✗ File not found: {path}")
        return None

# Level 3 — sys.exit(1) (fatal error, the script cannot continue)
def main():
    if not API_KEY:
        print("  ✗ API_KEY not set — cannot continue")
        sys.exit(1)
```

**Decision rule:**

| Situation | Response |
|---|---|
| Optional data missing, the process can continue | `print` + `continue`/`skip` |
| Operation failed, but the caller can handle it | `return None` or raise an exception |
| Invalid config, missing credential, unrecoverable failure | `sys.exit(1)` |

**Rule:** `sys.exit(1)` only at startup or on truly unrecoverable errors — never inside business-logic functions.

---

## 2. Raise vs Absorb

When to handle the error locally and when to re-raise it.

```python
# ✅ ABSORB — when you can fully handle the error here
def get_user_age(user: dict) -> int:
    try:
        return int(user['age'])
    except (KeyError, ValueError):
        return 0  # default value; the caller does not need to know

# ✅ RAISE — when the caller needs to know something failed
def load_config(path: str) -> dict:
    try:
        with open(path, encoding='utf-8') as f:
            return json.load(f)
    except FileNotFoundError:
        raise  # the caller decides what to do if there is no config

# ❌ BAD — absorbing without doing anything (silent error)
def load_config(path: str) -> dict:
    try:
        with open(path, encoding='utf-8') as f:
            return json.load(f)
    except FileNotFoundError:
        pass  # what does it return? what does the caller know? Nothing.
```

**Decision rule:**

| Situation | Response |
|---|---|
| You have a reasonable default value | Absorb and return the default |
| The error is expected and the caller need not know | Absorb and handle locally |
| The caller needs to know the operation failed | `raise` or `raise NewError from e` |
| You don't know what to do with the error | `raise` — don't absorb for the sake of it |

**Rule:** never absorb silently (`except: pass`). If you absorb, there is always an action: log, return a default, or print.

---

## 3. Required Environment Variables

Validate all required variables at startup, before anything else.

```python
import os
import sys
from dotenv import load_dotenv

def validate_env() -> None:
    """Verify all required environment variables are set."""
    load_dotenv()

    required = ['API_KEY', 'DATABASE_URL', 'SECRET_TOKEN']
    missing = [var for var in required if not os.getenv(var)]

    if missing:
        print(f"  ✗ Missing required environment variables: {', '.join(missing)}")
        sys.exit(1)

def main():
    validate_env()  # always first
    # rest of the script...
```

**Rules:**
- Always validate in a dedicated `validate_env()` function — not inline in `main()`.
- Validate **all** variables together and report every missing one in a single message.
- `sys.exit(1)` immediately if any is missing — never continue with `None` variables.
- Call `validate_env()` as the first line of `main()`, before any other initialization.

---

## 4. External Files

`FileNotFoundError` always, for any file. Add parser errors depending on the format.

```python
# Any text file (.txt, .log, .md, etc.)
try:
    with open(path, encoding='utf-8') as f:
        content = f.read()
except FileNotFoundError:
    print(f"  ✗ File not found: {path}")
    sys.exit(1)


# JSON — add JSONDecodeError (the file may exist but be corrupt or empty)
try:
    with open(path, encoding='utf-8') as f:
        data = json.load(f)
except FileNotFoundError:
    print(f"  ✗ File not found: {path}")
    sys.exit(1)
except json.JSONDecodeError as e:
    print(f"  ✗ Invalid JSON: {path} — {e}")
    sys.exit(1)


# CSV — FileNotFoundError is enough (csv.DictReader does not raise on format)
try:
    with open(path, encoding='utf-8') as f:
        data = list(csv.DictReader(f))
except FileNotFoundError:
    print(f"  ✗ File not found: {path}")
    sys.exit(1)
```

**Rules:**
- `FileNotFoundError` always, for any external file.
- For JSON: also catch `json.JSONDecodeError` — an existing file may be corrupt.
- Always include the `path` in the error message — makes it easy to know which file failed.
- Always use explicit `encoding='utf-8'` — avoids errors on systems with a different default encoding.
- Add the format-specific parser error for other formats (XML, YAML, TOML, etc.).

---

## 5. Custom Exceptions

When to create your own exception class vs use Python's built-ins.

```python
# ✅ Use built-ins for generic data or logic errors
raise ValueError("Invalid email format")
raise TypeError("Expected string, got int")
raise KeyError("Required key 'user_id' not found")


# ✅ Create your own exception when the caller needs to catch that specific error
class ConfigError(Exception):
    """Raised when configuration is missing or invalid."""
    pass

class APIAuthError(Exception):
    """Raised when API credentials are invalid or expired."""
    pass


# The caller can catch only that type
try:
    load_config('config.json')
except ConfigError as e:
    print(f"  ✗ Configuration problem: {e}")
    sys.exit(1)
```

**Decision rule:**

| Situation | Use |
|---|---|
| Invalid value, type, or key error | Built-in (`ValueError`, `TypeError`, `KeyError`) |
| The caller needs to catch only this specific error | Custom exception |
| The error belongs to a specific domain of your app | Custom exception |
| Generic error any code could raise | Built-in |

**Conventions:**
- Name always ends in `Error` — `ConfigError`, `APIAuthError`, `ValidationError`.
- Inherit from `Exception`, not `BaseException`.
- Docstring on the class explaining when it is raised.
- Define them at the top of the module, or in an `exceptions.py` file if there are several.

---

## 6. Exception Chaining

Use `from e` when re-raising inside an `except` to preserve the original error.

```python
# ✅ GOOD — preserves the original error in the traceback
def load_config(path: str) -> dict:
    try:
        with open(path, encoding='utf-8') as f:
            return json.load(f)
    except json.JSONDecodeError as e:
        raise ConfigError(f"Config file is corrupted: {path}") from e


# ❌ BAD — loses the original error
def load_config(path: str) -> dict:
    try:
        with open(path, encoding='utf-8') as f:
            return json.load(f)
    except json.JSONDecodeError:
        raise ConfigError(f"Config file is corrupted: {path}")
        # The traceback does not show where or why the JSON failed
```

**What each one shows:**

```
# With "from e":
json.JSONDecodeError: Expecting value: line 3 column 5 (char 42)

The above exception was the direct cause of the following exception:

ConfigError: Config file is corrupted: config.json

# Without "from e":
ConfigError: Config file is corrupted: config.json
# (the JSONDecodeError disappears — you don't know which line failed)
```

**Rules:**
- Always use `raise NewError(...) from e` when re-raising inside an `except`.
- To explicitly suppress the original: `raise NewError(...) from None`.
- Never do `raise NewError(...)` without `from` inside an `except`.

---

## 7. `finally` vs Context Managers

How to make sure resources are always cleaned up, even if there is an error.

```python
# ❌ NO handling — the file may stay open if there is an error
f = open('data.json')
data = json.load(f)
f.close()  # if json.load() fails, this never runs

# ✅ With finally — always runs, with or without error
f = open('data.json', encoding='utf-8')
try:
    data = json.load(f)
finally:
    f.close()

# ✅ With context manager — equivalent, cleaner (preferred)
with open('data.json', encoding='utf-8') as f:
    data = json.load(f)  # f.close() is automatic on leaving the block
```

**When to use each:**

| Situation | Use |
|---|---|
| The object supports `with` (files, connections, sessions) | Context manager (`with`) |
| Cleanup with no context manager (e.g. deleting a temp file) | `finally` |
| Multiple resources | Context manager with multiple `with` on one line |

```python
# ✅ Multiple resources
with open('input.csv', encoding='utf-8') as fin, open('output.csv', 'w') as fout:
    pass  # both close automatically

# ✅ finally for cleanup without a context manager
temp_path = create_temp_file()
try:
    process(temp_path)
finally:
    os.remove(temp_path)  # always deleted, even if process() fails
```

**Rule:** prefer `with` whenever the object supports it. Use `finally` only when no context manager is available.

---

## 8. `logging.exception()` vs `logging.error()`

The difference is whether the full traceback appears in the logs.

```python
import logging

try:
    result = int("abc")
except ValueError as e:

    # logging.error() — message only
    logging.error(f"Conversion failed: {e}")
    # Output: ERROR - Conversion failed: invalid literal for int() with base 10: 'abc'

    # logging.exception() — message + full traceback
    logging.exception(f"Conversion failed: {e}")
    # Output:
    # ERROR - Conversion failed: invalid literal for int() with base 10: 'abc'
    # Traceback (most recent call last):
    #   File "script.py", line 4, in <module>
    #     result = int("abc")
    # ValueError: invalid literal for int() with base 10: 'abc'
```

**Decision rule:**

| Situation | Use |
|---|---|
| Inside an `except` block and you want the traceback | `logging.exception()` |
| Outside an `except`, or the traceback adds no context | `logging.error()` |
| Expected, controlled errors (e.g. invalid user input) | `logging.warning()` |

**Rule:** inside an `except` block, always prefer `logging.exception()` — the traceback costs nothing and helps debugging. Use `logging.error()` only outside an `except` or when the traceback is noise.

---

## Summary — Quick Reference

| Rule | What to do |
|---|---|
| Response hierarchy | print+continue → return None → sys.exit(1) |
| Raise vs absorb | Absorb if you have a default; raise if the caller needs to know |
| Env vars | `validate_env()` first, all together, `sys.exit(1)` if missing |
| Files | `FileNotFoundError` always + parser error per format |
| Custom exceptions | Built-ins for generic errors; custom for specific catching |
| Exception chaining | `raise X from e` always inside `except` |
| Cleanup | `with` if the object supports it; `finally` if not |
| Logging | `logging.exception()` inside `except`; `logging.error()` outside |

---

**Version:** 1.1  
**Last Updated:** 2026-05-31
