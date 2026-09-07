# Spec + Plan Gate

Applies regardless of Code author. Use the existing `superpowers` skills —
`brainstorming`, `writing-plans`, `executing-plans`, `systematic-debugging`
— as the mechanism. This file only states the trigger for when to use them.

| Situation | Path |
|---|---|
| Small mechanical bug (a typo'd field name, a computed style never applied, a broken import) | Fix directly, no spec/plan |
| New feature, new module, or an architecture-level refactor | `brainstorming` → spec → `writing-plans` → implementation/prompts |
| Behavior/UX change with nuance (a new rule, not just cosmetic) | Same: spec + plan before writing code or drafting the prompt |
| Visual redesign | Same: spec + plan (use the visual companion when it applies) |
| Non-obvious bug / symptom without a clear cause | `systematic-debugging` before proposing a fix |

Never skip the spec+plan path just because the code author is an external
tool or because "it's just a prompt" — the trigger is the nature of the
change, not who ends up writing the code.
