# Python API Client Standard

How to consume external REST APIs in Python projects (acting as an HTTP client).

---

## 1. Authentication

Always load credentials from `.env`, never hardcode them in the source.

```python
import os
from dotenv import load_dotenv

# ✅ GOOD — from .env
load_dotenv()
API_KEY = os.getenv('API_KEY')

# ❌ BAD — hardcoded
API_KEY = "sk-1234abcd..."
```

**Rules:**
- Call `load_dotenv()` before any `os.getenv()`.
- If the variable does not exist, `os.getenv()` returns `None` — validate before using it (see `error-handling.md`).
- Never log or print the API key, not even partially.

---

## 2. Client Instantiation

Create the client once in `main()` and pass it as a parameter. Never use global variables.

```python
# ✅ GOOD — instantiated in main(), passed as a parameter
def fetch_users(client: SomeAPIClient) -> list[dict]:
    return client.get('/users')

def main():
    client = SomeAPIClient(api_key=API_KEY)
    users = fetch_users(client)


# ❌ BAD — global variable
client = SomeAPIClient(api_key=API_KEY)  # at module level

def fetch_users() -> list[dict]:
    return client.get('/users')  # uses the global implicitly
```

**Rule:** the client is a dependency — create it once in `main()` and pass it explicitly.

---

## 3. Separate Fetch from Transform

The function that calls the API returns raw data. A separate function does the mapping or extraction.

```python
# ✅ GOOD — separated
def fetch_users(client: SomeAPIClient) -> list[dict]:
    """Call the API and return raw data."""
    response = client.get('/users')
    return response['data']

def transform_users(raw_users: list[dict]) -> list[dict]:
    """Extract only the needed fields."""
    return [
        {'id': u['id'], 'name': u['full_name'], 'email': u['email_address']}
        for u in raw_users
    ]


# ❌ BAD — fetch and transform mixed
def get_users(client: SomeAPIClient) -> list[dict]:
    response = client.get('/users')
    return [
        {'id': u['id'], 'name': u['full_name']}
        for u in response['data']
    ]
```

**Rules:**
- Never mix the HTTP call with data mapping in the same function.
- If the API changes its structure → you only touch the fetch function.
- If the fields you need change → you only touch the transform function.
- **Exception:** if there is no transformation (you use the data as-is), a single function is fine.

---

## 4. Pagination

Standard pattern: `while True` + stop condition + explicit cursor.

```python
def fetch_all_users(client: SomeAPIClient) -> list[dict]:
    all_users = []
    cursor = None

    while True:
        response = client.get('/users', params={'cursor': cursor, 'limit': 100})
        all_users.extend(response['data'])

        if not response['has_more']:
            break

        cursor = response['next_cursor']

    return all_users
```

**Rules:**
- `cursor = None` for the first page — most APIs ignore it when it is `None`.
- Always set an explicit `limit` — don't rely on the API default.
- Accumulate in a local list and return at the end — don't process inside the loop (that is transform).
- The exact field name (`has_more`, `next_page`, `total_pages`) depends on each API — adapt the name, not the pattern.

**Note:** some APIs use offset pagination (`page=1`, `page=2`...) instead of a cursor. The `while True` + stop-condition structure stays the same.

---

## 5. Retry and Timeout

```python
import time
import logging

def fetch_with_retry(client: SomeAPIClient, endpoint: str, max_retries: int = 3) -> dict:
    for attempt in range(1, max_retries + 1):
        try:
            response = client.get(endpoint, timeout=10)

            if response.status_code == 200:
                return response.json()

            if response.status_code >= 500:
                logging.warning(f"Retry {attempt}/{max_retries} — status {response.status_code}")
                time.sleep(2 ** attempt)  # backoff: 2s, 4s, 8s
                continue

            if response.status_code >= 400:
                raise ValueError(f"Client error {response.status_code} — no retry")

        except requests.exceptions.Timeout:
            logging.warning(f"Retry {attempt}/{max_retries} — timeout")
            time.sleep(2 ** attempt)

    raise RuntimeError(f"API failed after {max_retries} retries: {endpoint}")
```

**Rules:**
- **Only retry 5xx errors and timeouts** — 4xx errors are the client's (invalid credentials, resource not found); retrying changes nothing.
- **Exponential backoff** — wait `2 ** attempt` seconds between retries (2s, 4s, 8s), not a fixed interval.
- **`timeout` always explicit** — never let a call wait indefinitely.
- **Raise an exception when retries are exhausted** — don't return `None` silently.

---

## 6. HTTP Error Handling

```python
def handle_response(response) -> dict:
    if response.status_code == 200:
        return response.json()

    if response.status_code == 401:
        raise PermissionError("Invalid or expired API key")

    if response.status_code == 403:
        raise PermissionError(f"Access denied: {response.url}")

    if response.status_code == 404:
        return {}  # Resource not found — return empty if it is optional

    if response.status_code == 429:
        retry_after = response.headers.get('Retry-After', 60)
        raise RuntimeError(f"Rate limit hit — retry after {retry_after}s")

    if response.status_code >= 500:
        raise RuntimeError(f"Server error {response.status_code}")
```

**Rules:**
- **401/403** → always raise — these are configuration errors the developer must see.
- **404** → depends on context: return `{}` if the resource is optional, raise if it is required.
- **429** → raise with the `Retry-After` header value if available.
- **5xx** → raise (retries were already exhausted in the previous section).
- **Never return `None` silently on critical errors** — the caller cannot tell "no data" from "the call failed".

---

## 7. Response Validation

Verify the response has the expected structure before using it.

```python
def fetch_users(client: SomeAPIClient) -> list[dict]:
    response = client.get('/users')
    data = response.json()

    # ✅ Validate before using
    if 'data' not in data:
        raise ValueError(f"Unexpected response structure: missing 'data' key. Got: {list(data.keys())}")

    if not isinstance(data['data'], list):
        raise ValueError(f"Expected list in 'data', got {type(data['data']).__name__}")

    return data['data']
```

**Rules:**
- Validate required keys before accessing — don't assume the API always returns the same thing.
- Include context in the error — show what actually arrived, not just "invalid structure".
- Validate type when it matters — if you expect a list and get a dict, the error will surface far from its origin without this check.
- Don't over-validate — only the keys and types your code uses directly.

**Especially important when:**
- The API is external and may change without notice.
- The response has multiple levels of nesting.
- It is the first integration with a new API.

---

## 8. API Call Logging

```python
import logging
import time

def fetch_users(client: SomeAPIClient) -> list[dict]:
    logging.info("GET /users")
    start = time.time()

    response = client.get('/users', timeout=10)
    elapsed = round(time.time() - start, 2)

    if response.status_code == 200:
        logging.info(f"GET /users — 200 OK ({elapsed}s)")
    else:
        logging.warning(f"GET /users — {response.status_code} ({elapsed}s)")

    return response.json()
```

**What to log and at what level:**

| Moment | Level | Example |
|---|---|---|
| Before calling | `INFO` | `"GET /users"` |
| Successful response | `INFO` | `"GET /users — 200 OK (0.43s)"` |
| Retry | `WARNING` | `"Retry 2/3 — status 503 (GET /users)"` |
| Rate limit | `WARNING` | `"Rate limit hit — retry after 60s"` |
| Final failure | `ERROR` | `"API failed after 3 retries: GET /users"` |

**Rules:**
- Never log the API key or sensitive request-body fields.
- Always include the endpoint in the message — makes it easy to search logs.
- Response time is optional but useful for spotting slowness.

---

## Summary — Quick Reference

| Rule | What to do |
|---|---|
| Authentication | `.env` + `load_dotenv()`, never hardcoded |
| Client | Create in `main()`, pass as a parameter |
| Fetch vs Transform | Separate functions when there is mapping |
| Pagination | `while True` + cursor + explicit limit |
| Retry | Only 5xx/timeout, exponential backoff, timeout always |
| HTTP errors | 401/403 raise, 404 by context, 429 with Retry-After |
| Validation | Keys + types before using, with context in the error |
| Logging | INFO success, WARNING retry/rate limit, ERROR final failure |

---

**Version:** 1.1  
**Last Updated:** 2026-05-31
