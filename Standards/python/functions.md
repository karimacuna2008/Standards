# Python Functions Standard

How I (Claude) will write and detail functions in Python projects.

---

## 1. Naming Conventions

**Function names** should be:
- **Verb + noun** (`get_user`, `calculate_total`, `process_data`)
- **snake_case** (lowercase with underscores)
- **Descriptive** (what it does, not how)
- **Avoid:** Generic names (`do_stuff`, `process`, `handle`)

**Parameter names:**
- **snake_case**
- **Descriptive** (`user_id`, not `uid`)

### Recommended Verbs by Use Case:

| Operation | Recommended | Alternative | When to use |
|-----------|------------|-------------|------------|
| Get from API/DB | `fetch_` | `load_` (files), `get_` (simple access) | APIs, databases |
| Transform data | `transform_` | `parse_` (string→structure) | Change shape/format |
| Validate | `validate_` | `is_valid_` (returns bool) | Validation logic |
| Create | `create_` | `build_` (complex creation) | Instantiate objects |
| Delete from DB | `delete_` | `remove_` (collections), `clean_` (cleanup) | Persistence operations |

---

## 2. Type Hints (Required - with exceptions)

Every function must have type hints **except in trivial cases**.

**Required:**
```python
def fetch_user(user_id: int) -> dict:
    """Retrieve user data."""

def validate_email(email: str) -> bool:
    """Check if email format is valid."""
```

**Exceptions (no type hints needed):**
```python
# Simple loops
for attempt_number in range(10):
    next_value = attempt_number + 1

# List comprehensions when source is obvious
doubled_amounts = [amount * 2 for amount in amounts]  # 'amounts' already has type hint above
```

**Guidelines:**
- Use specific types (`list[str]`, not `list`)
- Use `Union` or `|` for multiple types: `str | int`
- Use `Optional[T]` only if None is expected: `Optional[str]`

---

## 3. Docstrings (Required - Structured Format)

**Always use structured format:**

```python
def process_orders(orders: list[dict], min_amount: float = 0) -> list[dict]:
    """Filter orders by minimum amount and format prices.
    
    Args:
        orders: List of order dicts with 'amount' and 'status' keys
        min_amount: Minimum order amount to include (default: 0)
        
    Returns:
        List of filtered orders with 'formatted_price' field added
        
    Note:
        Returns empty list if no orders meet criteria.
    """
```

**Format:**
- One-line summary (what it does)
- Args section (all parameters)
- Returns section (what comes back)
- Note section (special cases, if needed)

---

## 4. Deep Technical Breakdown (Explained in Chat)

When I present a function, I will explain it in detail here **in the chat** (not in docstring):

```
This function filters orders and formats prices for display.

Parameters:
- orders: Contains list of order dicts with 'amount' (float) and 'status' (str)
- min_amount: Acts as filter threshold, includes all if 0

Returns: List of dicts with original fields + new 'formatted_price' (str)

Internal logic:
- Iterates through orders, filters by amount >= min_amount
- Creates filtered_orders list as intermediate storage
- Applies currency formatting to each order's amount
- Returns empty list if no matches (handled gracefully)

Variables created:
- filtered_orders: Holds orders passing threshold
- result: Final output list with formatted prices
```

This breakdown covers:
- What each parameter controls
- What the function returns and its impact
- Key variables and their purpose
- Non-obvious logic

---

## 5. Function Size & Parameters

**No fixed limits.** Depends on complexity:

- **Size:** As long as needed for clarity. If it's getting hard to follow, consider splitting.
- **Parameters:** As many as needed. If logic becomes unclear, consider grouping in dataclass.

**General guidance:**
- Each function should do **one main job** (single responsibility)
- If function body is scattered (jumping between 3+ different tasks), consider splitting

---

## 6. No Helper Functions Without Asking

I will **ask before creating helper functions**. You decide if it's necessary.

**Ask when:**
- Code could be extracted to a separate function
- Something seems complex enough for its own function

**You decide based on:**
- Is it reusable (used more than once)?
- Does it improve clarity?
- Is it a testable unit?

---

**Version:** 1.0  
**Last Updated:** 2026-05-18
