---
name: test-product
description: >-
  Verifies product quality with automated tests and Playwright browser checks.
  Use after implementation milestones, before claiming delivery complete, or
  when debugging UI behavior. Adapts Anthropic webapp-testing patterns
  (reconnaissance-then-action, networkidle, role selectors).
---

# Test Product

## Goal

Prove the product works against SPEC acceptance criteria. Skeptical by default. No “should work.”

## Preferred toolkit

1. Project test runner (vitest/jest/pytest/…) — unit/integration
2. **Playwright** for e2e (Chromium; headless in CI/scripts)
3. Browser MCP (Cursor/Claude) for exploratory QA / screenshots when useful

Upstream patterns: `vendor/anthropics/webapp-testing` ([anthropics/skills](https://github.com/anthropics/skills)). Prefer matching the repo’s language (TS `@playwright/test` vs Python `playwright`).

## Decision tree

```text
Static HTML only?
  yes → read file / file:// ; script against known selectors
  no  → server running?
          no  → start app (compose/dev script); wait for port
          yes → reconnaissance-then-action (below)
```

## Workflow

```text
1. Read docs/SPEC.md acceptance → checklist in docs/STATUS.md
2. Boot app; note base URL
3. Unit/integration suite
4. Playwright: critical user journeys only (auth, create, pay, core CTA)
5. On failure: screenshot + console + trace; fix or DECISIONS waiver
6. Re-run until green or explicit waiver
```

## Reconnaissance-then-action

1. `goto` → `wait_for_load_state("networkidle")` (or equivalent ready signal)
2. Screenshot / inspect DOM
3. Prefer **role/name/text** selectors (`getByRole`, `get_by_role`) over brittle CSS chains
4. Then act (click/fill/assert)

## Playwright baseline

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("http://localhost:3000")
    page.wait_for_load_state("networkidle")
    page.get_by_role("button", name="Sign in").click()
    page.screenshot(path="docs/qa/home.png", full_page=True)
    browser.close()
```

Or TypeScript `@playwright/test` if the repo is JS/TS-first — match the stack.

## Rules

- Never assert on a dynamic page before it is ready
- Prefer role/text selectors over brittle CSS chains
- Critical SPEC items without evidence ⇒ not done
- Record results in `docs/STATUS.md` under `## Test report`
- Keep scripts/fixtures under `docs/qa/` or the project’s test dirs — not one-off `/tmp` only

## Reading the run correctly

- **Never pipe a test command through `head`/`tail`.** The shell reports the *last* command's exit
  code, so a red suite behind `| tail -60` returns 0 and reads as green. Redirect to a file and
  check the runner's own exit code, or let the runner print its summary.
- Start compact (`pytest -q --tb=no`, `vitest --reporter=dot`); pull a full traceback only for the
  case you are diagnosing. A full-traceback run of a broadly failing suite can cost tens of
  thousands of tokens and tells you the same thing as the summary.
- When many tests fail at once, look for one shared cause before reading each failure. Process-wide
  state torn down by the first test — a closed pool, a shut-down executor, a cached singleton — is
  the usual culprit, and fixing it turns the whole block green at once.
- Screenshots are expensive. One per visual claim, scoped to the element, one theme unless the
  behaviour is theme-specific.

## Reading code under test

Use Serena rather than opening source files whole: `get_symbols_overview` for a module's shape,
`find_symbol include_body=true` for the function under test, `find_referencing_symbols` to find
everything that touches a suspect symbol.
