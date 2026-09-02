# Amending the documents (Phase 3)

## Contents

- The rule
- `docs/iterations/I<n>-<slug>.md` — the new artefact, with template
- `docs/SPEC.md` — edit in place, never renumber
- `docs/DECISIONS.md` — one ADR per decision
- `docs/TASKS.md` — one appended milestone
- `docs/PLAN.md` — usually untouched
- `docs/STATUS.md` — the handoff
- Commit shape

## The rule

Amend, do not regenerate. Every document below was written once against a real interrogation and is
still true except where this iteration changes it. Rewriting one loses the reasoning behind
decisions nobody is revisiting, and quietly changes requirements that were never discussed.

This phase is deliberately **low freedom**: follow the formats exactly. A document that stops
matching its own conventions stops being greppable, and the next session pays for it.

## `docs/iterations/I<n>-<slug>.md` — the new artefact

One page per iteration. It holds what does not belong in the long-lived documents: the intake
decisions, the impact map, and the exit criteria for this round only.

```markdown
# Iteration I<n> — <title>

- **Source:** <artefact or request, with a path or a date>
- **Opened:** <YYYY-MM-DD>
- **Milestone:** `M<n>` in `docs/TASKS.md`
- **Baseline:** <suite name> <counts> at <YYYY-MM-DD HH:MM>, branch `<branch>`

## In scope

| Item | Why now |
|------|---------|

## Out of scope this round

| Item | Reason | Recorded as |
|------|--------|-------------|

## Impact map

<the table from references/impact-map.md>

## Expectations that change

<tests whose assertions change, or "none">

## Exit criteria

- [ ] <one line per in-scope item, phrased as something observable>
- [ ] Baseline suites green at their Phase 0 counts or better
```

Keep it to one page. It is a working document for one round, not a second SPEC.

## `docs/SPEC.md` — edit in place, never renumber

- Changing a requirement: edit **that line**, keep its identifier. The identifier is referenced from
  tasks, tests, ADRs and audits; renumbering breaks all of them silently.
- Adding a requirement: append to the relevant `## Functional requirements` block with the next
  free identifier. Never reuse a retired one.
- Removing a requirement: do not delete it. Mark it withdrawn on the line itself, with the ADR that
  withdrew it. A deleted requirement looks like one that never existed.
- Moving something to `## Non-goals` is a decision, so it needs an ADR too.
- If the change adds a user-visible flow, `## User flows` and `## Launch checklist` are the two
  sections most often forgotten. Check both.

## `docs/DECISIONS.md` — one ADR per decision

The file is ADR-lite and append-only. Follow its existing shape exactly:

```markdown
## ADR-0NN — <the decision as a sentence, not a topic>

- Date: <YYYY-MM-DD>
- Status: accepted
- Context: <what forced a choice; name the constraint or the finding>
- Decision: <what was chosen>
- Consequences: <what this buys, what it costs, and what the rejected alternative was>
```

Write one for each of:

- every item deferred at intake — the reason belongs here, not in the chat that scrolls away;
- every behaviour change that contradicts a `SPEC.md` requirement;
- every accepted risk the owner signed off at the gate.

Never edit a past ADR to reflect a new decision. Supersede it with a new one and say which it
replaces. The history of a reversal is worth more than a tidy file.

## `docs/TASKS.md` — one appended milestone

Append one milestone at the end, in the file's own format:

```markdown
## M<n> — <iteration title> *(<serial|parallel>, <parent or agent name>)*

> <one or two lines: what this milestone is for, and any constraint that binds
> several tasks together — an ordering rule from the impact map belongs here.>

- [ ] **T<id>** — <title> — paths: `<owned globs>` — deps: <task ids or none> — DoD: <observable condition>
```

Rules:

- **Continue the numbering.** Take the highest existing `T` id in the file and go on from there.
  Never restart per milestone, never reuse a retired id.
- **One task per impact-map row**, unless the map said two rows must be one task.
- **`paths:` is ownership**, and it is what keeps parallel agents from colliding. If two tasks in
  this milestone share a path, they are serial or they are one task.
- **DoD is observable.** "Focus stays on a control after the swap" passes; "improve focus handling"
  does not.
- Deferred items do not appear as unticked tasks. An unticked task means "not done yet", and the
  next session will try to do it.

## `docs/PLAN.md` — usually untouched

Touch it only when the architecture actually moves: a new dependency, a new service, a changed
boundary, a new directory in the repository map. A styling fix or a new endpoint inside an existing
router is not an architecture change. When it does move, edit `## Architecture` and
`## Repository map` and nothing else.

## `docs/STATUS.md` — the handoff

At the gate: `## Resume here` says the iteration is approved, names the milestone, and points at
`docs/iterations/I<n>-<slug>.md`. Add the `## Baseline I<n>` block from Phase 0 if it is not
already there.

At the close: rewrite `## Resume here` — not append — with the branch, the state of the tree, the
next three actions, and anything waiting on the owner. Update `## Test report` with each suite's
own last-run timestamp and command.

## Commit shape

Commit the amendments before any code, as one commit: `docs: iteration I<n> — <title>`. That commit
is the approved delta. A later question about what was agreed is then answered by `git show`, not
by memory.
