---
name: review-product
description: >-
  Independent delivery review for correctness, security, UX, and completeness
  against SPEC.md. Use after tests, before handoff, or when verifying claimed
  work. Pairs with web-design-guidelines for UI audits.
context: fork
background: false
---

# Review Product

## Mindset

You are a skeptical reviewer. "Looks done" is not done. Verify.

Verify the *claims*, not the whole tree. Take each ticked task and each "tests green" assertion and
check that one thing: a ticked checkbox with no code behind it, and a green claim from a suite that
was never run, are the two failures this review exists to catch. Re-run the suite yourself.

## How to read the code

Through Serena, not whole-file reads — an end-to-end read of a mid-size project costs ~115k tokens
and surfaces nothing extra.

- `get_symbols_overview <file>` — a module's shape for ~200 tokens
- `find_symbol <name> include_body=true` — the one function you are judging
- `find_referencing_symbols` — every caller of a symbol you suspect, instead of grep-then-open
- `get_diagnostics_for_file` — compiler/linter truth before you read anything

Quote the minimum that makes a finding land; cite `file:line` and let the reader open it.

## Checklist

### Completeness
- [ ] Every SPEC in-scope item implemented or explicitly deferred in DECISIONS.md
- [ ] Acceptance criteria evidenced by tests or manual QA notes
- [ ] README / HANDOFF explains run, env, deploy

### Correctness
- [ ] Happy paths work
- [ ] Error / empty states exist for interactive flows
- [ ] No obvious broken links or placeholder lorem in production paths

### Security
- Run skill **`secure-review`** (manual checklist; Semgrep if available)
- Critical/High security findings ⇒ FAIL unless waived in DECISIONS.md

### UX / UI
- Run `web-design-guidelines` on key UI files
- Spot-check mobile width
- Distinctive design per brief (not generic AI template) — `frontend-design` bar

### Engineering
- [ ] Sensible structure matching PLAN.md
- [ ] Diffs look surgical (flag drive-by refactors)
- [ ] Dead code / TODOs called out
- [ ] CI or at least documented test commands

## Output

Write `docs/REVIEW.md`:

```markdown
# Review

## Verdict
PASS | PASS WITH WAIVERS | FAIL

## Critical
- ...

## High
- ...

## Medium / polish
- ...

## Waivers (must cite DECISIONS.md)
- ...
```

FAIL → send back to implement/test. Do not soft-pass critical issues.
