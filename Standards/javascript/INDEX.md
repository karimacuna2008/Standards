# JavaScript / TypeScript Standards — INDEX

Read this file first to know which standard to use for the task.

| File | When to use it |
|---|---|
| `react-typescript.md` | Any React + TypeScript frontend: folder layout, design tokens, data layer, error and loading contract, forms, tsconfig, lint gates, dependency policy |

> When creating a new standard file here, add it to this table before closing
> the session.

## Status

`react-typescript.md` is **minimal on purpose** (written 2026-09-08). It carries
only rules that were settled by a real audit against a real codebase, so every
rule in it has a reason that can be stated in one line.

Not written yet, and deliberately so — write them only once real code has
exercised the conventions, never from first principles:

- `testing.md` — what to test beyond pure functions, fixtures, naming.
- Component internals — prop shape, composition, when to split.
- Responsive conventions — until `responsive-audit.md` has produced findings on
  a real module.

## Origin

These rules were extracted from the audit of the CMS Whitelabel frontend
foundation (2026-09-08). Where a rule exists to prevent a specific observed
failure, the failure is named. Do not delete those references: a rule whose
reason is forgotten gets dropped by the next person who finds it inconvenient.
