# Python Code Standards

Conventions, style, and structure for writing Python code.

---

## 1. Project Structure

**Depends on project type:**

### Small Scripts / Tools
```
project/
├─ main.py
├─ utils.py
├─ config.py
├─ requirements.txt
└─ README.md
```

### Medium / Large Projects
```
project/
├─ src/
│  ├─ module1/
│  │  ├─ __init__.py
│  │  ├─ core.py
│  │  └─ utils.py
│  ├─ module2/
│  │  ├─ __init__.py
│  │  └─ logic.py
│  └─ config.py
├─ tests/
│  ├─ test_module1.py
│  └─ test_module2.py
├─ requirements.txt
└─ README.md
```

### Data Science / Analysis
```
project/
├─ notebooks/
│  ├─ 01_exploration.ipynb
│  └─ 02_analysis.ipynb
├─ data/
│  ├─ raw/
│  └─ processed/
├─ scripts/
│  └─ processing.py
├─ requirements.txt
└─ README.md
```

**Rule:** Use what makes sense for the project. Start simple, expand if needed.

---

## 2. Imports

**Organize in 3 groups with blank lines between:**

```python
# Standard library
import os
import sys
from pathlib import Path
from datetime import datetime

# Third-party
import requests
import pandas as pd
import numpy as np

# Local
from utils import helper_function, process_data
from config import API_KEY, DEBUG_MODE
```

**Rules:**
- Group: Standard library → Third-party → Local
- Within group: Alphabetical order
- Use specific imports (NEVER `from module import *`)
- One blank line between groups

---

## 3. PEP 8 Compliance

Follow PEP 8 with **one modification:**

- ✅ 4 spaces for indentation
- ✅ 2 blank lines between top-level functions/classes
- ✅ 1 blank line between methods in a class
- ✅ Spaces around operators: `x = 1 + 2`
- ✅ No spaces inside brackets: `list[0]`, `dict[key]`
- ⚠️ **Max 120 characters per line** (instead of PEP 8's 79)

```python
# ✅ GOOD (readable, under 120)
result = very_long_function_name(arg1, arg2, arg3) if condition else default_value

# ❌ TOO LONG (over 120)
result = very_long_function_name_that_is_unnecessarily_verbose(arg1, arg2, arg3, arg4, arg5) if condition else default_value

# ✅ SPLIT LONG LINES (if needed)
result = (
    very_long_function_name(arg1, arg2, arg3)
    if condition
    else default_value
)
```

---

## 4. Naming Conventions

### Variables & Functions
```python
# snake_case - lowercase with underscores
user_name = "John"
total_amount = 100
process_user_data()
```

### Constants
```python
# UPPER_CASE - all caps with underscores
API_KEY = "secret123"
MAX_RETRIES = 3
DEBUG_MODE = True
```

### Classes
```python
# PascalCase - uppercase first letter of each word
class UserManager:
    pass

class DataProcessor:
    pass
```

### Private Methods / Attributes
```python
class UserManager:
    def public_method(self):
        pass
    
    def _internal_method(self):
        """Internal use only. Don't call from outside."""
        pass
    
    def __init__(self):
        self._internal_value = 0  # For internal use
```

**Private Convention:**
- Use `_single_underscore` for methods/attributes that are internal
- Says "this is implementation detail, don't use directly"
- Still accessible in Python, but signals intent
- Example: `_validate_email()`, `_internal_data`, `_helper_function()`

---

## 5. Strings

**Use single quotes by default, double quotes if needed:**

```python
# Default: single quotes
name = 'John'
message = 'Hello world'

# Use double quotes if string contains apostrophe/single quote
error = "Can't process this"
quote = "He said 'hello'"

# Very long strings: triple quotes
description = """
This is a longer description
that spans multiple lines
and is easier to read with triple quotes.
"""
```

---

## 6. String Formatting

**Always use f-strings (Python 3.6+):**

```python
# ✅ F-STRINGS (do this)
name = "John"
age = 30
message = f"Name: {name}, Age: {age}"

# With operations
total = f"Total: {price * quantity}"
formatted = f"Value: {number:.2f}"  # 2 decimal places

# ❌ DON'T use .format() or % unless necessary
# message = "Name: {}, Age: {}".format(name, age)
```

**Only use `.format()` for:**
- Dynamic template strings
- Compatibility with Python < 3.6 (which is rare now)

---

## 7. Blank Lines

**Follow PEP 8 spacing:**

```python
"""Module docstring."""

import os
from utils import helper


CONSTANT_VALUE = 100  # 2 blank lines before top-level code


def function1():
    """First function."""
    pass


def function2():
    """Second function."""
    pass


class MyClass:
    """Class docstring."""
    
    def __init__(self):
        """Initialize."""
        pass
    
    def method1(self):
        """First method."""
        pass
    
    def method2(self):
        """Second method."""
        pass
```

**Rules:**
- 2 blank lines before top-level functions/classes
- 1 blank line between methods in a class
- 1 blank line between logical sections within a function

---

## 8. Docstrings

**Module level:**
```python
"""
Data processing utilities.

This module handles loading, transforming, and validating data.
Functions are organized by data type (CSV, JSON, etc).
"""
```

**Class level:**
```python
class UserManager:
    """Manages user creation, authentication, and data retrieval.
    
    Attributes:
        db: Database connection instance
        logger: Logger instance for debugging
    """
    
    def __init__(self, db):
        """Initialize UserManager with database connection."""
        self.db = db
```

**Function level:** defined in `functions.md` (canonical) — same Google-style format.

**Format:** Google style docstrings (brief + details if needed)

---

## 9. Exception Handling

**Catch specific exceptions when possible:**

```python
# ✅ GOOD - Specific exceptions
try:
    result = int(user_input)
except ValueError:
    print("Please enter a valid number")
except KeyError:
    print("Required key not found")

# ⚠️ OK - Generic only when necessary
try:
    some_operation()
except Exception as e:
    logger.error(f"Operation failed: {e}")
    # Handle gracefully
```

**Rules:**
- Catch specific exceptions (ValueError, KeyError, FileNotFoundError, etc.)
- Catch generic `Exception` only if you have a good reason
- Always log or handle the error, don't silently ignore
- Never use bare `except:` (catches KeyboardInterrupt, SystemExit, etc.)

---

## 10. Logging vs print()

**Development vs Production:**

```python
import logging

# Development: use print()
print("Debug info")
print(f"User: {user_name}")

# Production/Important: use logging
logging.info("Application started")
logging.warning("High memory usage detected")
logging.error("Failed to connect to database")

# Setup logging in main
if __name__ == "__main__":
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s - %(levelname)s - %(message)s'
    )
```

**Rule:**
- `print()` for development / debugging
- `logging` for important events, errors, and production code

---

## 11. Code Comments

**When to comment:**

```python
# ✅ GOOD - Explains WHY, not WHAT
def calculate_discount(price, customer_type):
    # VIP customers get 15% discount (business rule from 2024 Q1)
    if customer_type == 'VIP':
        return price * 0.85
    return price

# ❌ BAD - Explains WHAT (code already does this)
def calculate_discount(price, customer_type):
    # If customer is VIP, multiply by 0.85
    if customer_type == 'VIP':
        return price * 0.85
    return price
```

**Rules:**
- Only comment if WHY is non-obvious
- Well-named functions/variables usually don't need comments
- Keep comments updated if code changes

---

## 12. Imports - Specific, Never Wildcard

```python
# ✅ GOOD - Explicit imports
from utils import helper_function, process_data
from config import API_KEY, DATABASE_URL

# ❌ BAD - Wildcard imports
from utils import *  # What's being imported? Unclear.
from config import *
```

**Rule:** Always import specifically. It's clear what you're using.

---

## 13. Constants Configuration

**For configuration, create a `config.py` file:**

```python
# config.py
DEBUG_MODE = True
API_KEY = "your-key-here"
DATABASE_URL = "postgresql://..."
MAX_RETRIES = 3
TIMEOUT_SECONDS = 30

# For sensitive data, use environment variables
import os
SECRET_KEY = os.getenv('SECRET_KEY', 'default-for-dev')
```

**Use in your code:**
```python
from config import DEBUG_MODE, API_KEY, MAX_RETRIES

if DEBUG_MODE:
    print("Debug mode enabled")
```

---

## 14. Summary - Quick Reference

| Item | Rule |
|------|------|
| Indentation | 4 spaces |
| Line length | Max 120 characters |
| Strings | Single quotes (double if needed) |
| String format | f-strings always |
| Variables | snake_case |
| Constants | UPPER_CASE |
| Classes | PascalCase |
| Private methods | _underscore_prefix |
| Imports | Grouped, specific (never `*`) |
| Blank lines | PEP 8 (2 before top-level, 1 in class) |
| Docstrings | Module, class, function (Google style) |
| Comments | Only WHY, not WHAT |
| Exceptions | Specific, log or handle |
| Logging | Debug: print() | Production: logging |

---

**Version:** 1.1  
**Last Updated:** 2026-05-31
