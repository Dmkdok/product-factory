---
name: orchestrate-product
description: >-
  End-to-end product delivery orchestrator for websites and software.
  Runs discovery interrogation, product spec, technical plan, gated approval,
  then delegates implementation, testing, and review to subagents.
  Use when the user wants to build a site/app from scratch, ship a polished
  product, run a full delivery pipeline, or invokes /orchestrate-product.
compatibility: Cursor Agent, Claude Code; Plan Mode recommended for Phase 1–3
metadata:
  author: product-factory
  version: "1.3.0"
  lang_user: ru
  lang_internal: en
---

# Orchestrate Product

Coordinate a full delivery pipeline. **Do not invent product decisions** — interrogate the user until the brief is sharp enough that a senior team could build without guessing.

## Resuming an in-flight delivery

If `docs/STATUS.md` exists, this is a resume, not a new start. Do this and nothing else:

1. Read `docs/STATUS.md` — it names the current phase and carries the running log.
2. Read only the `## M<n>` blocks in `docs/TASKS.md` that are still open.
3. **Trust the tree over the checkboxes.** A session that dies mid-milestone leaves finished code
   with unticked tasks. Before working on any task, confirm its state with
   `get_symbols_overview` on the files it owns, and run the suite before believing "tests green".
4. Skip every phase already ticked. Do not re-read `BRIEF.md`, and read `SPEC.md`/`PLAN.md` only
   in the sections the open tasks touch.

Never resume by summarising the previous chat. `docs/STATUS.md` is the handoff artifact; it is
cheaper and more accurate than any conversation summary, and keeping it current is what makes a
session disposable.

## Language policy

- Speak to the user in **Russian**.
- Keep artifacts, checklists, task prompts, and subagent handoffs in **English** (token efficiency).
- Russian only in chat UX and in `docs/BRIEF.ru.md` summary if the user wants a readable brief.

## Hard quality gate (non-negotiable)

**No implementation code until the user explicitly approves the plan.**

Allowed before approval: research, questions, docs under `docs/`, scaffolding folders empty of app code.
Forbidden before approval: app source, dependencies install for the product, UI code, migrations.

If the user says «просто сделай» / «fast» — still run a compressed discovery (minimum question set below), write a short plan, and ask for one-line approval (`утверждаю` / `approve`).

## Delegation policy

- Use the strongest available reasoning model for discovery, product framing, spec, planning, and the approval gate.
- Match the model tier to the phase: strongest for the decisive phases above and for review, Sonnet
  for the executing ones (implement, test), Haiku for read-only fan-out. Details and per-call
  overrides in [references/delegation.md](references/delegation.md#model-tier).
- After approval, decompose the work into small, mostly independent tasks and delegate them to subagents.
- Give each subagent a narrow scope, explicit owned paths, and a clear Definition of Done.
- Keep the parent agent responsible for integration, conflict resolution, testing, and final coherence.
- Avoid assigning overlapping file ownership to multiple subagents in the same milestone.

## Context management

Two practices from Claude Code's own best-practices guide, which apply directly to this pipeline:

- **Start a fresh session once the plan is approved.** The guide's spec workflow ends "start a fresh
  session to execute it: the new session has clean context focused entirely on implementation, and
  you have a written spec to reference." Here that is the GATE — `docs/` holds everything the build
  needs, and the discovery dialogue behind you is dead weight. `/clear` between milestones for the
  same reason: performance degrades as the window fills.
- **Send investigation to subagents.** "Scope investigations narrowly or use subagents so the
  exploration doesn't consume your main context" — the documented fix for reading many files while
  researching. A subagent works in its own window and returns only the summary.

Beyond these two, general context/token hygiene (bounded reads, batched tool calls, scratchpad
persistence, tool-schema footprint) is skill `context-token-optimization` — it applies from Phase 0
onward, not only here.

## Skill name resolution

Every cross-reference below (`discover-requirements`, `draft-product-spec`, `code-review`, ...) is a
bare skill name. Installed straight into a project's `.claude/skills/`, that resolves as-is. Loaded
as a plugin (`--plugin-dir`), Claude Code may list the same skill as `product-factory:<name>`
instead — if the bare name doesn't trigger, retry once with that prefix before concluding the skill
is missing.

## Pipeline

```
0 Intake → 1 Elicit → 2 Spec → 3 Tech plan → GATE → 4 Implement → 5 Test → 6 Review → 7 Handoff
  → 8 Deploy (optional)
```

Read phase details on demand:

- [references/pipeline.md](references/pipeline.md)
- [references/question-banks.md](references/question-banks.md)
- [references/delegation.md](references/delegation.md)
- Templates in `templates/product-factory/` (installed projects) or `templates/` (inside the product-factory pack root itself)

### Phase 0 — Intake

Classify product type:

| Type | Default path |
|------|----------------|
| Marketing / landing site | Web-first, design-heavy |
| Web app / SaaS | Full stack + auth + tests |
| API / backend | Spec + contract tests |
| Other (CLI, mobile, desktop) | Adapt phases; keep gate |

Create workspace docs (English filenames):

```text
docs/
  BRIEF.md
  SPEC.md
  PLAN.md
  TASKS.md
  CONVENTIONS.md
  DECISIONS.md
  STATUS.md
  CHANGELOG.md
```

`CONVENTIONS.md` is written in Phase 3 (`draft-tech-plan`), not here — listed now so Phase 0 scaffolding
and Phase 4 delegation both expect the same file set. `CHANGELOG.md` **is** scaffolded here, empty,
from `templates/CHANGELOG.template.md` — it only gets real entries starting at Phase 8 (`deploy-product`)
or at an `iterate-product` close for a local-only product, but the empty file exists from Phase 0 so
no later phase has to remember to create it.

Copy structure from `templates/product-factory/` when present, else `templates/` (pack root case).

**Dependency preflight.** Seven skills this pipeline calls by name — `coding-discipline`,
`concise-mode`, `secure-review`, `web-design-guidelines`, `frontend-design`, `ui-quality-audit`,
`context-token-optimization` — are bundled in this pack's own `skills/` (see README "Dependencies"),
so this only matters if the copy you're running from is partial. `code-review` and `simplify` need
no check at all — they ship with Claude Code itself, not as files. Confirm the seven resolve before
the phase that needs them (Phase 4 and 6 mostly, `context-token-optimization` from Phase 0 on); a
missing one is not fatal — that phase runs without it — but say so to the user once, in Russian,
rather than silently skipping a step they expect run.

### Phase 1 — Elicit (read skill `discover-requirements`)

Grill until shared understanding: batches of 5–8, **recommended answers** on each question, skip
what the repo already answers. Prefer structured choices when possible.

Stop eliciting only when [Definition of Ready](references/pipeline.md#definition-of-ready) is met.

Use the strongest available reasoning model for this phase (Plan Mode in Cursor when available).

### Phase 2 — Spec (`draft-product-spec`)

Produce `docs/SPEC.md`: problem, users, scope, non-goals, UX flows, acceptance criteria, risks.

### Phase 3 — Tech plan (`draft-tech-plan`)

Produce `docs/PLAN.md` + `docs/TASKS.md` + `docs/CONVENTIONS.md`: stack, architecture, file map, milestones, parallel workstreams, test strategy, Definition of Done, and the naming/structure/testing rules every Phase 4 subagent reads.

Web research is allowed for current docs/libraries — check today's date rather than assuming a fixed year. Prefer boring, proven defaults unless the user constrained otherwise.

### GATE — User approval

Present in Russian:

1. Short summary of what will be built
2. Stack and major trade-offs
3. Out of scope
4. Risks / assumptions
5. Ask: **«Утверждаете план? Напишите: утверждаю»**

On approval, write `docs/STATUS.md` → `phase: implement`, commit the docs, and offer the user a
fresh session for Phase 4 (`/clear`, then «продолжай по docs/STATUS.md»).
On change requests, update docs and re-ask. Never skip the gate.

### Phase 4 — Implement (`implement-product` + `coding-discipline` + subagents)

Delegate via Task / subagents per [references/delegation.md](references/delegation.md).

Parent agent:

- Keeps `docs/STATUS.md` updated **as each milestone lands, not at the end** — an unrecorded
  milestone is lost work when the session runs out of context
- Merges results; resolves conflicts
- Verifies each milestone before starting the next
- Does not dump entire codebase into chat — summarize (or activate `concise-mode` if the user asked)
- Reads code through Serena's symbol tools, not whole files (see
  [references/delegation.md](references/delegation.md#reading-code-serena))

For UI work, apply `frontend-design`. Prefer distinctive, brief-specific design; avoid generic AI aesthetics.

### Phase 5 — Test (`test-product`)

Run automated checks + Playwright / browser verification. Fix failures before claiming done.

### Phase 6 — Review (`review-product` + `web-design-guidelines` + `secure-review`)

Independent verification (verifier mindset). Critical functional **or** security issues must be fixed.

Also run `code-review` and `simplify` on the diff — a different lens from `review-product`'s
SPEC/DoD check: correctness bugs, reuse, and simplification/efficiency cleanups in the code itself.

### Phase 7 — Handoff

Russian summary for the user:

- What was built and how to run it
- Test status
- Known limitations
- Suggested next iterations

English `docs/HANDOFF.md` **and** an up-to-date `docs/STATUS.md` for continuity across
tools/sessions. A later agent must be able to resume from STATUS alone without the prior chat.

### Phase 8 — Deploy (optional, `deploy-product`)

Skip entirely for a local-only product. When a real deploy target exists and the user wants it
shipped, not just handed off: load `deploy-product`. It refuses to run past a FAIL review verdict,
requires a rollback plan written down before the deploy starts, and proves the deployed artifact
works with a smoke test against the live target — a green deploy pipeline is not that proof by
itself. Writes `docs/RELEASE.md` and appends to `docs/CHANGELOG.md`.

### Optional: CI (`setup-ci`)

Not a numbered phase and not default — **most projects here run without CI.** `setup-ci` is
`disable-model-invocation`, so it never triggers on its own; load it only when the user explicitly
asks for CI/CD, or said yes to "CI expected?" at discovery and now confirms they want it scaffolded.
It has no fixed place in the pipeline — commonly after Phase 4 once there's something to lint/test,
sometimes requested up front. It never adds a deploy step; that stays `deploy-product`'s job.

## Progress checklist

Copy into `docs/STATUS.md` and tick:

```text
- [ ] Phase 0 intake
- [ ] Phase 1 elicit (DoR met)
- [ ] Phase 2 SPEC.md
- [ ] Phase 3 PLAN.md + TASKS.md
- [ ] GATE user approved
- [ ] Phase 4 implementation
- [ ] Phase 5 tests green
- [ ] Phase 6 review clean (or accepted waivers)
- [ ] Phase 7 handoff
- [ ] Phase 8 deploy (optional, only if a real deploy target exists)
```

## Anti-patterns

- Coding from a vague one-liner without interrogation
- Skipping tests to “finish faster”
- Parallel agents editing the same files without ownership
- Inventing brand/voice when the user already stated them
- Closing with “should work” without running verification
- Drive-by refactors and speculative architecture (violates `coding-discipline`)
- Auto-running `ui-quality-audit` on every delivery (optional skill — only on explicit request)
- Auto-running `setup-ci` because the user mentioned tests or deploy — it's optional, ask first
- Claiming tests green from TASKS checkboxes or chat memory without running the suite
- Dumping all of `docs/` into every subagent prompt
- Deploying (Phase 8) past a FAIL review verdict, or with no rollback plan written down first
