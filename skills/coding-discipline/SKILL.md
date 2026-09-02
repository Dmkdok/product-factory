---
name: coding-discipline
description: >-
  Behavioral guardrails against common LLM coding mistakes: think before coding,
  simplicity first, surgical diffs, goal-driven verified loops. Use during
  implementation, refactors, bugfixes, or whenever the agent risks overbuilding.
  Inspired by widely adopted Karpathy-derived agent guidelines (~170k★).
metadata:
  author: product-factory
  version: "1.1.0"
  inspired_by: multica-ai/andrej-karpathy-skills
  cross_checked_with: DietrichGebert/ponytail (community skill; JetBrains-measured -10% cost, no quality loss)
---

# Coding Discipline

Bias toward caution over speed. For trivial one-liners, use judgment.

## 1. Think before coding

- State assumptions. If uncertain, ask — do not hide confusion.
- If multiple interpretations exist, present them; do not pick silently.
- If a simpler approach exists, say so and push back when warranted.
- Stop and name what is unclear before writing code.

## 2. Simplicity first

Before writing anything new, climb down this ladder and stop at the first rung that solves it:
reuse existing code in the repo → standard library → a native platform feature (HTML/CSS,
DB constraint) → an already-installed dependency → one line → minimal new code.

- Minimum code that solves the asked problem. Nothing speculative.
- No features, abstractions, or configurability beyond the request.
- No error handling for impossible scenarios.
- If 200 lines could be 50, rewrite.
- Never minimize away: input validation at trust boundaries, error handling that prevents data
  loss, security controls, accessibility basics. Simplicity is about code, not about safety.

Ask: would a senior engineer call this overcomplicated? If yes, simplify.

## 3. Surgical changes

- Touch only what the task requires. Match existing style.
- Do not “improve” adjacent code, comments, or formatting.
- Do not refactor unrelated areas. Mention dead code; do not delete unless asked.
- Remove only orphans **your** change created (unused imports/vars).

Test: every changed line should trace to the user request / approved task.

## 4. Goal-driven execution

Turn work into verifiable goals, then loop until verified:

- “Add validation” → tests for invalid inputs, then make them pass
- “Fix bug” → failing repro first, then fix
- “Refactor X” → tests green before and after

Multi-step plan form:

```text
1. [Step] → verify: [check]
2. [Step] → verify: [check]
```

Weak criteria (“make it work”) are not enough — define the check.

## Working if

Fewer noisy diffs, fewer overbuilds, clarifying questions **before** mistakes.
