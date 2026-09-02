---
name: iterate-product
description: >-
  Delivery pipeline for changing a product that already ships: green baseline,
  delta intake, impact and regression map, amendments to SPEC/DECISIONS/TASKS,
  an approval gate, then implementation, testing and review. Use instead of
  orchestrate-product whenever docs/SPEC.md already describes a built product —
  audits, bug reports, owner requests, refactors, performance targets.
compatibility: >-
  Claude Code, Cursor Agent. Requires a previous delivery's docs/SPEC.md,
  docs/PLAN.md and docs/TASKS.md, and a runnable test suite.
metadata:
  author: product-factory
  version: "1.0.0"
  lang_user: ru
  lang_internal: en
---

# Iterate Product

Change a product that already works, without breaking what already works.

`orchestrate-product` builds from nothing: it interrogates a brief, writes `SPEC.md` and `PLAN.md`
from scratch, and its resume branch assumes a build that was interrupted. None of that fits here.
The brief is settled, the documents exist, and the tree is the thing being modified. **The risk is
no longer "we build the wrong thing" — it is "we break the right thing."** Phases 0 and 2 below
exist for that risk and have no counterpart in the greenfield pipeline.

## Routing

| Situation | Skill |
|-----------|-------|
| Empty repo, no `docs/SPEC.md` | `orchestrate-product` |
| Build interrupted mid-milestone, tasks unticked, tree ahead of the checkboxes | `orchestrate-product` (resume branch) |
| Shipped product: audit to remediate, bug report, new owner request, refactor, performance target | **this skill** |
| One well-defined task inside a milestone that is already approved | `implement-product` directly |

## Language policy

Speak to the user in **Russian**. Keep `docs/`, task prompts and subagent handoffs in **English**.

## Skill name resolution

Every cross-reference below (`review-product`, `code-review`, `deploy-product`, ...) is a bare skill
name. Installed straight into a project's `.claude/skills/`, that resolves as-is. Loaded as a plugin
(`--plugin-dir`), Claude Code may list the same skill as `product-factory:<name>` instead — if the
bare name doesn't trigger, retry once with that prefix before concluding the skill is missing.

## Hard gate (non-negotiable)

**No implementation code until the user approves the delta.** Allowed before approval: reading,
running the existing suite, and writing under `docs/`. Forbidden: source edits, migrations,
dependency changes.

The gate is on the *delta*, not on the product. Never re-litigate settled decisions to get here —
if a change contradicts one, that contradiction is the thing to put in front of the user.

## Pipeline

```
0 Baseline → 1 Delta intake → 2 Impact map → 3 Amend docs → GATE → 4 Implement → 5 Verify → 6 Review
  → 7 Close → 8 Deploy (optional)
```

Read on demand, one level down:

- [references/impact-map.md](references/impact-map.md) — how to build Phase 2, with the blast-radius classes
- [references/amendments.md](references/amendments.md) — exact edit rules for `docs/` in Phase 3
- Delegation, model tiers and the Serena reading protocol are unchanged from the parent pipeline:
  `.claude/skills/orchestrate-product/references/delegation.md`

Copy this into `docs/STATUS.md` and tick as you go:

```text
Iteration I<n> progress:
- [ ] 0 baseline recorded (branch, suite result, timestamp)
- [ ] 1 delta intake agreed (in / out / deferred)
- [ ] 2 impact map written
- [ ] 3 docs amended (SPEC, DECISIONS, TASKS M<n>)
- [ ] GATE approved by the owner
- [ ] 4 implementation
- [ ] 5 verification green, baseline suites still green
- [ ] 6 review clean or waived
- [ ] 7 closed (STATUS rewritten, milestone ticked)
- [ ] 8 deploy (optional, only if a real deploy target exists)
```

### Phase 0 — Baseline

Do this before reading a single line of the change request. It costs one command and it is the only
thing that distinguishes a regression you caused from one you inherited.

1. Branch: `session/<YYYY-MM-DD>-<slug>` or `iteration/I<n>-<slug>`. Never work on `main`.
2. Confirm the tree is clean. Uncommitted work from a previous session is a finding, not a starting
   point — report it and stop.
3. Run every suite the project has, by its documented command, and record the counts and the
   timestamp in `docs/STATUS.md` under `## Baseline I<n>`. Never pipe a test run through
   `head`/`tail`; the exit code you get back is the pipe's.
4. A red baseline is a fact to report to the user before any new work, not something to fix quietly.

### Phase 1 — Delta intake

Replaces Phase 1 elicitation. The Definition of Ready is already met; do **not** re-interview the
brief, the personas or the scope. Ask only what the change itself leaves open.

Establish, in Russian, in one batch with recommended answers:

1. **Source** — audit document, bug, owner request, external constraint. Name the artefact.
2. **In** — the specific items being taken this round, by their own identifiers (`F-002`, a task id,
   a described symptom).
3. **Out** — items explicitly deferred, each with a reason. These go to `DECISIONS.md`, not to
   silence.
4. **Acceptance** — what the owner will look at to call it done. One line per in-scope item.
5. **Budget** — how much of the change is worth doing now, when a full list would take weeks.
6. **Non-negotiables** — behaviour that must not change even if it would be improved by changing it.

When the source is a written audit or issue list with severities and a phased plan, items 1–4 are
mostly extraction, not interrogation. Extract, present the result, and ask only about the cut line.

### Phase 2 — Impact map

The phase greenfield does not have, and the reason this skill exists. For every in-scope item,
determine what it touches and what could break. Read code with Serena symbol tools, never whole
files.

Produce a table with one row per item: change · touched symbols and files · SPEC requirements
affected or preserved · existing tests that cover the current behaviour · blast-radius class ·
what proves no regression. Full method, classes and the row template:
[references/impact-map.md](references/impact-map.md).

Two outputs matter more than the table itself:

- **Ordering.** Anything touching a shared primitive — design tokens, base template, schema,
  migrations, auth, a widely-referenced helper — lands first and serially. Everything downstream
  of it is scheduled after, never in parallel with it.
- **The list of tests whose expectations change.** A test that must be edited is a behaviour change
  in disguise. It needs the owner's approval in the same breath as the feature, and an ADR if the
  behaviour it asserted came from `SPEC.md`.

### Phase 3 — Amend the documents

Amend; do not regenerate. `SPEC.md` and `PLAN.md` were paid for once and are still true except
where this iteration changes them. Exact rules, formats and templates:
[references/amendments.md](references/amendments.md). In short:

- `docs/iterations/I<n>-<slug>.md` — new. Intake, impact map, exit criteria. One page.
- `docs/SPEC.md` — edit only the requirement lines this iteration changes; add new ones with new
  identifiers. Never renumber existing ones.
- `docs/DECISIONS.md` — one ADR per decision that was not already recorded: every deferral from
  Phase 1, every behaviour change from Phase 2.
- `docs/TASKS.md` — append **one new milestone** in the file's existing format. Continue the task
  numbering from the highest existing id; never restart or reuse one.
- `docs/PLAN.md` — only if the architecture actually moves. Usually it does not.

### GATE — Owner approval

Present in Russian, short: what changes, what deliberately does not, what could break and how that
is caught, the ordering constraint from Phase 2, and anything that contradicts an existing decision.
Then ask: **«Утверждаете дельту? Напишите: утверждаю»**.

On approval: write `docs/STATUS.md` → the new milestone is in progress, commit the docs, and offer
a fresh session for Phase 4 (`/clear`, then «продолжай по docs/STATUS.md»). On change requests,
edit the delta and re-ask. Never skip the gate, and never widen scope silently after it.

### Phase 4 — Implement

Load `implement-product` and `coding-discipline`; `frontend-design` for UI work. Delegate per the
parent pipeline's delegation map. Two rules specific to iteration:

- **Surgical diffs, enforced.** Existing code that the task does not name is out of bounds. An
  improvement noticed in passing becomes a line in the next iteration's intake, not a diff.
- **Behaviour changes get their own commit**, separate from the feature, naming the test that
  changed and why.

### Phase 5 — Verify

Load `test-product`. Green means both of these, in this order:

1. Every baseline suite from Phase 0 passes at its Phase 0 count or better. A count that dropped is
   a deleted test until proven otherwise.
2. Each in-scope item has a check that fails without the change. An item whose only evidence is
   "the suite is still green" is not done.

Manual-only items from the impact map are verified manually and recorded with what was done, on
what, and when.

### Phase 6 — Review

Load `review-product`; add `secure-review` when the change touched auth, uploads, input handling or
dependencies, and `web-design-guidelines` when it touched UI. The reviewer's specific job here is
regression: judge the diff against the impact map, and check that nothing outside the owned paths
moved. Also load `code-review` and `simplify` for a diff-quality pass — a different lens from
`review-product`'s SPEC/DoD-and-regression check.

### Phase 7 — Close

Tick the milestone, rewrite `## Resume here` in `docs/STATUS.md`, and report to the user in Russian:
what changed, suite results with counts, what was deferred and where that is recorded, and the next
open item. Use the `pause` skill if the session ends here.

### Phase 8 — Deploy (optional, `deploy-product`)

Skip for a local-only change. When this iteration ships to a real environment, load
`deploy-product` after close: it refuses to run past a FAIL review verdict, requires a rollback plan
written down first, and proves the deployed artifact works with a smoke test against the live
target. Writes/updates `docs/RELEASE.md`.

## Regression contract

Applies from the gate to the close, for every agent working the iteration:

- Every existing test keeps passing. Editing one is a behaviour change and follows the Phase 2 rule.
- New behaviour ships with the check that would have caught its absence.
- "Tests green" is a command that was run in this session, never a checkbox and never chat memory.
- Nothing outside the milestone's owned paths is touched.
- A deferral is a line in `DECISIONS.md`. Silence is not a deferral.

## Anti-patterns

- Re-running discovery or rewriting `SPEC.md` because a change touched it
- Starting work without a recorded baseline, then debugging an inherited failure as if it were new
- Taking a whole audit as one milestone because the audit came in one file
- Editing a failing test into passing without deciding that the behaviour changed
- Parallelising a shared-primitive change with the work that depends on it
- Reading the whole codebase to build the impact map instead of following symbol references
- Fixing what is merely nearby
- Treating a phased plan inside the source document as if it were already approved scope
- Deploying (Phase 8) past a FAIL review verdict, or with no rollback plan written down first
