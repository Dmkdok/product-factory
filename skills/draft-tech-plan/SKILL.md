---
name: draft-tech-plan
description: >-
  Produces PLAN.md, TASKS.md and CONVENTIONS.md with architecture, stack
  choices, file map, milestones, parallel workstreams, and test strategy. Use
  after SPEC.md exists and before any implementation. Prefer researching
  current official docs.
metadata:
  author: product-factory
  version: "1.2.0"
---

# Draft Tech Plan

## Input

`docs/BRIEF.md`, `docs/SPEC.md`

## Output

- `docs/PLAN.md` — architecture & decisions
- `docs/TASKS.md` — ordered, assignable tasks with owners/paths
- `docs/CONVENTIONS.md` — from `templates/CONVENTIONS.template.md` (or `templates/product-factory/CONVENTIONS.template.md`
  in an installed project). Fill naming, structure, and test-location rules that follow from the
  stack chosen above — every implementer subagent reads this file on every task, so it must exist
  before Phase 4, not get invented ad hoc by the first one that needs it.
- Append major choices to `docs/DECISIONS.md` (ADR-lite)

## Rules

1. Prefer **proven defaults** over novelty unless SPEC demands otherwise.
2. If stack unspecified, choose one coherent stack and justify in 5 bullets.
3. Web-search official docs when the API surface may have changed since training — check the
   current date rather than assuming a fixed year.
4. Define **module boundaries** so subagents can work in parallel safely.
5. Include test strategy: unit, integration, e2e/browser.
6. Include rollout: how to run locally, env vars, deploy notes.
7. **Decide observability explicitly.** Logging/error-tracking/alerting in scope for v1, or a
   written "none for v1 — failures noticed via X" — either is a valid answer, an unfilled section
   is not. This is a design-time decision, not something to invent during `deploy-product`.
8. No application code yet.

## PLAN.md sections

- Goals & constraints recap
- Recommended stack (with alternatives considered)
- Architecture diagram (mermaid)
- Data model / content model
- Auth & security notes
- Observability (logging/error-tracking/alerting, or an explicit "none for v1")
- Folder / package map
- Milestones (M0 scaffold → M1 vertical slice → M2 polish → M3 harden)
- Risks
- Definition of Done

## TASKS.md format

```markdown
## M0 — Scaffold
- [ ] T001 — init project — paths: `/` — deps: none
- [ ] T002 — design tokens — paths: `src/styles/**` — deps: T001

## M1 — Vertical slice
...
```

Each task: id, title, paths, deps, DoD one-liner.
