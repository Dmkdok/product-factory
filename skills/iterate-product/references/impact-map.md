# Impact map (Phase 2)

## Contents

- Why this phase exists
- Method: five questions per item
- Finding the answers with Serena
- Blast-radius classes and what each one forces
- Row template and worked example
- Ordering rules that fall out of the map
- Tests whose expectations change
- When the map is done

## Why this phase exists

In a greenfield build there is nothing to break, so the plan only has to be *right*. In an
iteration the plan also has to be *contained*. The impact map is the containment: it converts "fix
F-002" into a named set of symbols, a named set of tests, and a named way of proving the rest of
the product still behaves.

It is also the cheapest place to discover that two items in the same round fight each other — one
splits a CSS class the other one restyles, one renames a symbol the other one calls.

## Method: five questions per item

Answer these for every in-scope item from Phase 1. Nothing else.

1. **What does it touch?** Symbols and files, by name. Not directories.
2. **Which requirement does it change, and which does it have to preserve?** Cite `SPEC.md`
   identifiers. Most items preserve far more than they change, and the preserved list is what the
   verification is written against.
3. **What already tests this behaviour?** Name the test functions. Two answers are possible and
   both are useful: existing coverage (which must keep passing, or be deliberately changed), or
   none (which means the item ships with a new test, and explains how the defect survived).
4. **What is the blast radius?** One of the classes below.
5. **What proves no regression?** An automated check, or an explicit manual step with the exact
   thing to look at. "Careful review" is not an answer.

## Finding the answers with Serena

Reading files whole is what makes this phase expensive; symbol navigation is what makes it cheap.

- `get_symbols_overview <file>` — the shape of a module for roughly 200 tokens.
- `find_symbol <name> include_body=true` — only the function actually under change.
- `find_referencing_symbols <name>` — **the core tool of this phase.** Every caller of the thing
  you are about to change, without grep-then-open. If the reference list is long, that is the
  blast-radius answer.
- `search_for_pattern` for what has no symbol: a CSS class, a template block, an HTTP path, a
  translation key.
- `get_diagnostics_for_file` before and after, when the change is structural.

Activate the project by absolute path — project names collide across a machine.

## Blast-radius classes and what each one forces

| Class | What it means | What it forces |
|-------|---------------|----------------|
| **Local** | One symbol or one template; the reference list is short and inside the same module | Nothing special. Parallel-safe. |
| **Shared primitive** | Design tokens, base layout, a common partial, a helper with many callers | Lands **first and serially**. Everything depending on it is scheduled after. Verify the callers, not only the primitive. |
| **Contract** | A route, a response shape, a template context key, a JS/HTML data attribute | Both sides change in one task, never in two parallel ones. The test asserting the contract is named in the row. |
| **Data** | Schema, migration, stored format, anything already on disk in production | Serial, alone in its task, reversible. Requires a stated migration and rollback path, and a check against real data volume, not a fixture of three rows. |
| **Cross-cutting policy** | Auth, CSP, rate limiting, upload limits, error handling | Requires the security review in Phase 6 regardless of size, and a check that the policy still holds where it was *not* changed. |

An item that lands in two classes takes the stricter one.

## Row template and worked example

One row per in-scope item, in `docs/iterations/I<n>-<slug>.md`:

```markdown
| Item | Touches | SPEC: changes / preserves | Existing coverage | Class | Regression proof |
|------|---------|---------------------------|-------------------|-------|------------------|
```

Worked example, from a UI audit finding:

```markdown
| F-002 focus dropped on disable | `app/static/js/board.js` → `moveRow`, `_setButtonState`; `app/templates/partials/_board_row.html` | changes none; preserves F31 (keyboard reorder), F12 (target size) | `e2e/test_a11y.py::test_focus_visible_public` covers public pages only — no coverage of the admin board | Local | New e2e case: reorder to the end as admin, assert `document.activeElement` is not `<body>` |
```

Keep rows this dense. A row that does not name a symbol and a test is not finished.

## Ordering rules that fall out of the map

- Shared-primitive and data rows go first, one at a time, and the suite runs between them.
- Contract rows are single tasks owning both sides.
- Local rows may be parallelised, but only across disjoint file sets — the same ownership rule the
  parent pipeline uses, applied to a smaller surface.
- When two rows touch the same symbol, they are one task. Merge them in the plan rather than
  discovering the conflict in a merge.

## Tests whose expectations change

Collect these into their own short list, separate from the table. For each: the test, the
assertion that will no longer hold, and why the new behaviour is correct.

This list is a gate input. The owner approves it with the rest of the delta, because "we changed
what the product does" is a product decision wearing a test's clothes. If the old assertion came
from a `SPEC.md` requirement, it also gets an ADR in `DECISIONS.md`.

## When the map is done

- Every in-scope item has a row, and every row answers all five questions.
- Every row's regression proof is either an automated check or a manual step with a named target.
- The ordering constraints are written down, not implied.
- The changed-expectations list exists, even if it is empty — an empty one is a meaningful claim.
