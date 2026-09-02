# Report template (mandatory output shape)

Write the final answer in this structure. Language: match the user’s language for prose; keep principle IDs (H4, SC 2.4.7, Fitts’s) in standard form.

```markdown
# UI Quality Audit Plan — <Project name>

## Meta
- Product type: <web app | marketing site | iOS | Android | desktop | …>
- Scope audited: <paths / routes / screens>
- Assumptions: <bullets>
- Method: Serena semantic pass + criteria checklists (read-only; no code changes)
- Authorities used: <short list>

## Executive verdict
<2–4 sentences: overall craft level, top risks, whether UI would survive senior review after fixes>

## Scorecard
| Area | Score (1–5) | Notes |
|------|-------------|-------|
| Visual hierarchy & craft | | |
| Interaction & controls | | |
| Usability (Nielsen + Laws of UX) | | |
| Accessibility (WCAG 2.2 AA) | | |
| States & feedback | | |
| IA / navigation | | |
| Architecture fitness for UI | | |
| Ethics (no deceptive patterns) | | |

## Critical user journeys
For each top journey: goal → steps observed → friction → severity.

## Findings (ordered P0 → P3)
### F-001 — <short title>
- **Severity:** P0|P1|P2|P3
- **Where:** `path` · `SymbolOrSelector` · route/screen
- **Evidence:** what the user sees/does (1–3 sentences)
- **Principle:** e.g. Nielsen H5; WCAG SC 2.4.7; Fitts’s Law; bulletproof-react feature boundary
- **Change:** alter | rewrite | remove | add
- **Target state:** concrete acceptance criteria (behavior + visual)
- **Implementation notes for coding agent:** numbered steps; files to touch; patterns to reuse; what NOT to do
- **Effort:** S|M|L
- **Dependencies:** other F-IDs if any

(repeat)

## Recommended rewrite map
| Module / screen | Action | Replace with / converge to | Priority |
|-----------------|--------|----------------------------|----------|
| | keep / refactor / rewrite | | |

## Design-system / token actions
- Gaps in primitives (Button, Input, Modal, …)
- Token work (color, type, space, focus rings)
- Consistency rules to enforce

## Accessibility remediation queue
Ordered list mapped to WCAG SC; mark automated vs manual verify.

## Architecture actions (UI-enabling only)
Boundary fixes, state policy, shared primitives — only if they unblock UI quality.

## Phased execution plan
### Phase A — Stop the bleeding (P0)
- …
### Phase B — Coherent product UI (P1)
- …
### Phase C — Craft & scale (P2–P3)
- …

## Definition of done (for a later coding agent)
Checklist the implementer must satisfy before calling the UI “professional-grade,” e.g.:
- [ ] All P0/P1 findings closed
- [ ] Primary journeys have full UI states
- [ ] WCAG 2.2 AA checks listed above pass
- [ ] Shared primitives used; no rogue duplicate controls on audited screens
- [ ] Focus order + visible focus verified on modals/dialogs
- [ ] No deceptive patterns remain

## Out of scope / non-issues
What looked fine; avoid bikeshedding later.
```

### Density rules

- Prefer **many specific findings** over one vague essay.
- Cap narrative; put detail in Target state + Implementation notes.
- Short code/pseudo only when it clarifies the target (≤15 lines).
- Never apply the changes in the audited repo while this skill is active.
