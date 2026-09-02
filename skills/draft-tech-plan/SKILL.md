---
name: draft-tech-plan
description: >-
  Produces PLAN.md and TASKS.md with architecture, stack choices, file map,
  milestones, parallel workstreams, and test strategy. Use after SPEC.md exists
  and before any implementation. Prefer researching current official docs.
---

# Draft Tech Plan

## Input

`docs/BRIEF.md`, `docs/SPEC.md`

## Output

- `docs/PLAN.md` — architecture & decisions
- `docs/TASKS.md` — ordered, assignable tasks with owners/paths
- Append major choices to `docs/DECISIONS.md` (ADR-lite)

## Rules

1. Prefer **proven defaults** over novelty unless SPEC demands otherwise.
2. If stack unspecified, choose one coherent stack and justify in 5 bullets.
3. Web-search official docs when API surface may have changed (assume mid-2026).
4. Define **module boundaries** so subagents can work in parallel safely.
5. Include test strategy: unit, integration, e2e/browser.
6. Include rollout: how to run locally, env vars, deploy notes.
7. No application code yet.

## PLAN.md sections

- Goals & constraints recap
- Recommended stack (with alternatives considered)
- Architecture diagram (mermaid)
- Data model / content model
- Auth & security notes
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
