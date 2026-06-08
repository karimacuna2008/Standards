# Python Automation Standard

How to write CLI scripts and automation tools in Python.

---

## 1. Interactive Menu vs Argparse

```python
# Interactive menu — for scripts used manually, step by step
def show_menu():
    print("\n=== Data Tool ===")
    print("  1. Import data")
    print("  2. Process records")
    print("  3. Export results")
    print("  0. Exit")


# Argparse — for scripts run from the CLI or automatically
# python script.py --input data.csv --output results.csv --verbose
import argparse

parser = argparse.ArgumentParser(description='Process data files')
parser.add_argument('--input', required=True, help='Input CSV file')
parser.add_argument('--output', required=True, help='Output file')
parser.add_argument('--verbose', action='store_true')
```

**Decision rule:**

| Situation | Use |
|---|---|
| The user runs the script manually and picks options | Interactive menu |
| The script runs from CI, cron, or a pipeline | argparse |
| It chains with other scripts or takes variable parameters | argparse |
| Internal tool used occasionally, multiple steps | Interactive menu |

**Rule:** if the script will ever be automated or called from another process, use argparse from the start — adding a menu later is harder than the other way around.

---

## 2. `main()` + `if __name__ == '__main__'`

Mandatory in every CLI script. Executable code goes inside `main()`, never loose at module level.

```python
# ✅ GOOD — all executable code inside main()
def show_menu():
    pass

def process_data():
    pass

def main():
    validate_env()
    while True:
        show_menu()
        choice = input("\nOption: ").strip()
        if choice == '0':
            break

if __name__ == '__main__':
    main()


# ❌ BAD — code loose at module level
validate_env()       # runs on import
show_menu()          # runs on import
choice = input(...)  # runs on import
```

**Rules:**
- All executable code inside `main()`.
- `if __name__ == '__main__': main()` always at the end of the file.
- The only things at module level: imports, constants, and function/class definitions.

---

## 3. Action Dict Pattern

Dispatch menu options with a dictionary instead of `if/elif` chains.

```python
# ❌ BAD — if/elif chain, grows endlessly
def handle_choice(choice: str) -> None:
    if choice == '1':
        import_data()
    elif choice == '2':
        process_records()
    elif choice == '3':
        export_results()
    elif choice == '4':
        show_stats()


# ✅ GOOD — action dict
def main():
    actions = {
        '1': import_data,
        '2': process_records,
        '3': export_results,
        '4': show_stats,
    }

    while True:
        show_menu()
        choice = input("\nOption: ").strip()

        if choice == '0':
            break

        if choice in actions:
            actions[choice]()
        else:
            print("  ✗ Invalid option")
```

**With parameters — use lambda:**

```python
actions = {
    '1': lambda: import_data(source='csv'),
    '2': lambda: process_records(verbose=True),
}
```

**Rules:**
- Adding a new menu option = adding one line to the dict, nothing else.
- The exit option (`'0'`) always outside the dict — handled before the lookup.
- Invalid options: a single error message, no extra logic needed.
- Use `lambda` only when the function needs fixed arguments — if it has none, pass the function directly.

---

## 4. Output Conventions

Consistent status prefixes so output is easy to read at a glance.

```python
print("  ✓ Users imported successfully")   # successful operation
print("  ✗ File not found: data.csv")      # error, operation failed
print("  ⚠️ No records found — skipping")  # warning, continues
print("  → Processing 150 records...")     # operation in progress
```

**Indentation:**

```python
# Section title: no indentation, blank line before
print("\n=== Export Results ===")

# Main action: 2 spaces
print("  → Exporting to results.csv...")
print("  ✓ 342 records exported")

# Detail or sub-item: 4 spaces
print("    - Skipped: 5 duplicates")
print("    - Errors: 0")
```

**Tables with f-strings:**

```python
# Align columns with fixed width
print(f"  {'Name':<20} {'Status':<10} {'Records':>8}")
print(f"  {'-'*20} {'-'*10} {'-'*8}")
for item in results:
    print(f"  {item['name']:<20} {item['status']:<10} {item['count']:>8}")
```

**Rules:**
- `✓` success, `✗` error, `⚠️` warning, `→` in progress — consistent across the whole project.
- 2 spaces of base indentation, 4 for sub-items.
- Always a blank line before section titles (`\n===`).
- Error messages always include the data that caused the problem (`File not found: data.csv`, not just `File not found`).

---

## 5. `sys.exit()` vs `return`

When to end the whole script vs simply leave a function.

```python
# ✅ sys.exit(1) — fatal error during initialization, the script cannot continue
def main():
    validate_env()     # sys.exit(1) if variables are missing
    connect_to_db()    # sys.exit(1) if there is no connection


# ✅ return — the action failed, but the script can continue (back to the menu)
def import_data() -> None:
    if not os.path.exists('data.csv'):
        print("  ✗ data.csv not found")
        return  # back to the menu, the user can try another option


# ❌ BAD — sys.exit(1) inside a menu action
def import_data() -> None:
    if not os.path.exists('data.csv'):
        print("  ✗ data.csv not found")
        sys.exit(1)  # kills the whole script — the user loses all context
```

**Decision rule:**

| Situation | Use |
|---|---|
| Missing environment variable at startup | `sys.exit(1)` |
| DB/API connection fails at startup | `sys.exit(1)` |
| A menu action fails | `return` |
| The user chooses to exit | `break` in the menu loop |
| Unrecoverable error mid-execution | `sys.exit(1)` with a clear message |

**Rule:** `sys.exit(1)` only at startup or on truly unrecoverable failures. Inside menu actions, always `return` — the user should be able to keep using the script.

---

## 6. User Input Validation

Check that what the user types is valid before processing it.

```python
# ✅ Menu option validation
def get_menu_choice(valid_options: list[str]) -> str:
    while True:
        choice = input("\nOption: ").strip()
        if choice in valid_options:
            return choice
        print(f"  ✗ Invalid option. Choose from: {', '.join(valid_options)}")

# In main():
actions = {'1': import_data, '2': process_records, '3': export_results}
choice = get_menu_choice(list(actions.keys()) + ['0'])


# ✅ Integer validation with range
def get_int_input(prompt: str, min_val: int, max_val: int) -> int:
    while True:
        raw = input(prompt).strip()
        try:
            value = int(raw)
            if min_val <= value <= max_val:
                return value
            print(f"  ✗ Enter a number between {min_val} and {max_val}")
        except ValueError:
            print("  ✗ Invalid input — enter a number")


# ✅ Confirmation validation (yes/no)
def confirm(prompt: str) -> bool:
    while True:
        answer = input(f"{prompt} (y/n): ").strip().lower()
        if answer in ('y', 'yes'):
            return True
        if answer in ('n', 'no'):
            return False
        print("  ✗ Enter 'y' or 'n'")
```

**Rules:**
- Always a `while True` loop — repeat until the input is valid.
- Specific error message — say what is valid, not just "invalid input".
- Always `.strip()` when reading input — removes accidental spaces and newlines.
- Reusable functions (`get_menu_choice`, `get_int_input`, `confirm`) — don't validate inline everywhere.

---

## 7. Progress Feedback

How to show progress on long operations without external libraries.

```python
# ✅ Simple counter for batches
def process_records(records: list[dict]) -> None:
    total = len(records)
    print(f"  → Processing {total} records...")

    for i, record in enumerate(records, 1):
        process_single(record)

        if i % 50 == 0 or i == total:
            print(f"  → {i}/{total} processed")

    print(f"  ✓ Done — {total} records processed")


# ✅ Inline progress (overwrites the same line)
def download_files(files: list[str]) -> None:
    total = len(files)
    for i, file in enumerate(files, 1):
        download(file)
        print(f"  → Downloading... {i}/{total}", end='\r')

    print(f"  ✓ {total} files downloaded        ")  # spaces clear the previous \r


# ✅ Summary at the end
def import_data(records: list[dict]) -> None:
    success, skipped, errors = 0, 0, 0

    for record in records:
        result = process(record)
        if result == 'ok':
            success += 1
        elif result == 'skip':
            skipped += 1
        else:
            errors += 1

    print(f"  ✓ Import complete: {success} imported, {skipped} skipped, {errors} errors")
```

**Rules:**
- Always show the total before starting — the user needs to know how much to expect.
- Update every 50 items or every 10% — not every item (too noisy) nor only at the end (looks hung).
- Always a closing summary message — never end in silence.
- For very long operations (>1000 items), use `end='\r'` so you don't flood the terminal.

---

## Summary — Quick Reference

| Rule | What to do |
|---|---|
| Menu vs argparse | Menu for manual use; argparse for CI/pipelines |
| `main()` | All executable code inside, never loose |
| Action dict | `actions = {'1': func}`, exit option outside the dict |
| Output | `✓ ✗ ⚠️ →`, 2-space base, 4 for sub-items |
| `sys.exit` vs `return` | `sys.exit(1)` at startup/unrecoverable; `return` in actions |
| Input validation | `while True` + `.strip()` + specific message + reusable functions |
| Progress | Total before, update every 50 items, summary at the end |

---

**Version:** 1.1  
**Last Updated:** 2026-05-31
