# Planning Standard

How Claude drives project planning with the user — from the initial idea or problem to a fully defined, approved plan. Walk these phases in order when starting any new project, and confirm each with the user before moving to the next. No code is written until the plan is approved (see the validation gate in `CLAUDE.md`).

## Throughout: ask, don't assume
At every phase, ask the user whatever is needed to define things well. When anything is ambiguous, missing, or has more than one reasonable option, ask before moving on — never fill gaps with silent assumptions. Surface questions as they come up, and close a phase only when its information is complete.

## 1. Capture the idea / problem
- Get the raw idea or problem first: what it is, what pain it solves, who it is for.
- Restate it back in one or two sentences and confirm it is correct before going on.

## 2. Objective & scope
- **Objective:** one clear statement of what the project will do.
- **Scope:** list what is IN and explicitly what is OUT — set boundaries now.
- Confirm both with the user.

## 3. Constraints & non-negotiables
- Timeline / urgency.
- Available resources (tools, libraries, services, skills).
- External dependencies (systems, data sources, accounts).
- Hard requirements that must be met.
- Branching model: ask whether the project needs `main` + `develop` or just
  `main` (see `git-workflow.md` §3) — this also determines the Cloud Run
  service setup in `deployment.md` if the project deploys there.

## 4. Success criteria
- Define what "done" looks like: measurable outcomes and acceptance conditions.
- If it cannot be measured or checked, restate it until it can.

## 5. Technology & approach
- Propose the stack/libraries and **justify** each: why this choice over alternatives and how it fits the existing stack (per the domain-reasoning rule in `CLAUDE.md`).
- Sketch the high-level approach: main components, data flow, separation of concerns.

## 6. Breakdown
- Split into 3–5 phases (e.g. setup → core → testing → polish/docs).
- For each phase: concrete tasks (not vague goals), dependencies (what blocks what), rough effort (S/M/L), and priority (critical / important / nice-to-have).
- List risks, each with a mitigation.

## 7. Present & validate (gate)
- Present a structured summary of the whole plan with its justifications.
- **Stop and get explicit approval before any implementation** (`CLAUDE.md` gate).
- Work begins only once the plan is approved.

## Plan summary template
Use this to present the plan in step 7:

```
PROJECT: <name>
IDEA / PROBLEM: <what and why>
OBJECTIVE: <one statement>
SCOPE: <in> | <out>
CONSTRAINTS: <timeline, resources, dependencies, non-negotiables>
SUCCESS CRITERIA: <measurable outcomes>
TECHNOLOGY: <choice — justification — alternatives rejected>
PHASES:
  1. <phase — key tasks>
  2. <phase — key tasks>
  3. <phase — key tasks>
RISKS: <risk — mitigation>
```

## During execution
- If the plan must change, state why, assess the impact, and update the plan before proceeding.
- Track task completion and flag blockers as they appear.

---
*Planning theory and worked examples are human-facing → future HTML guide.*

**Version:** 2.1  
**Last Updated:** 2026-06-16
