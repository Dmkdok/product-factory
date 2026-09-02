---
name: ui-quality-audit
description: >-
  OPTIONAL read-only UI/UX and front-end architecture audit against Nielsen,
  WCAG 2.2 AA and WAI-ARIA APG. Produces an evidence-based improvement plan;
  never edits code. Use only when the user explicitly asks for a UI/UX audit,
  deep usability/a11y review, design-system critique, or a polish plan after
  MVP — not during default orchestrate-product Phase 6 (that uses
  web-design-guidelines).
context: fork
background: false
metadata:
  author: product-factory
  version: "1.0.0"
  optional: true
  lang_user: ru
  lang_internal: en
---

# UI Quality Audit (optional, read-only plan)

**Not part of the default delivery pipeline.** Default Phase 6 stays on `web-design-guidelines` + `secure-review`. Load this skill only on explicit user request or when they ask for a deep UI remediation plan (existing apps, post-MVP polish).

## Language

- Chat with user: **Russian** (pack policy)
- Audit plan / `docs/UI-AUDIT.md`: **English** (token efficiency; match [report-template.md](references/report-template.md))

## Hard rules

1. **Do not change the project.** No file edits, patches, refactors, commits, or “quick fixes.”
2. **Deliverable = improvement plan only** — prioritized findings + concrete instructions another agent can execute later.
3. **Evidence over taste.** Every finding cites: location (`path` + symbol/selector), observed behavior, violated principle (from [references/sources.md](references/sources.md)), and target state.
4. **Cite authorities, not vibes.** Prefer NN/g heuristics, lawsofux.com, WCAG 2.2, WAI-ARIA APG, Apple HIG / Material 3, bulletproof-react. See sources.
5. **Match the product type.** Web / mobile / desktop: apply matching platform guidelines.
6. **Token discipline.** Load reference files only when that pass needs them. Prefer Serena symbol tools over dumping whole files.
7. **Do not replace** `frontend-design` (build) or `web-design-guidelines` (fast Phase 6 check). This skill is the deep optional pass.

## When references to read

| File | Read when |
|------|-----------|
| [references/criteria.md](references/criteria.md) | Every audit — scoring rubrics & checklists |
| [references/architecture.md](references/architecture.md) | UI code structure, state, components, routing |
| [references/report-template.md](references/report-template.md) | Before writing the final plan (mandatory shape) |
| [references/sources.md](references/sources.md) | Need exact citation names/URLs |
| [examples.md](examples.md) | Unsure how dense a finding should be |

## Serena-first exploration (when available)

Use **Serena MCP** for semantic navigation. This audit is **read-only** — never call Serena edit/refactor tools.

**Allowed:** `activate_project`, `onboarding` (if helpful), `list_dir`, `find_file`, `read_file`, `search_for_pattern`, `get_symbols_overview`, `find_symbol`, `find_referencing_symbols`, `find_declaration`, `find_implementations`, `get_diagnostics_for_file` / `get_diagnostics_for_symbol`, `query_project` (read-only), memory read/write for audit notes.

**Forbidden:** any Serena/client write that mutates the repo (`replace_symbol_body`, `rename_symbol`, `replace_content`, `create_text_file`, …).

**Fallback** (Serena missing): client `Glob` / `Grep` / `Read` only — still no edits.

### Efficient pass order

1. Tree → UI entrypoints (routes, screens, shells, tokens).
2. Symbol overview on key UI files; find Button/Modal/Form/Layout/Theme primitives.
3. Trace critical journeys (auth, create/edit/delete, checkout, settings, empty/error).
4. Sample tokens/shared controls; a11y hooks.
5. Score via [criteria.md](references/criteria.md); deepen only failures.

## Audit workflow

```
Audit progress:
- [ ] 0. Scope & product type
- [ ] 1. Structure map (IA + UI entrypoints)
- [ ] 2. Visual & interaction quality
- [ ] 3. Usability heuristics + Laws of UX
- [ ] 4. Accessibility (WCAG 2.2 AA floor)
- [ ] 5. Front-end architecture fitness
- [ ] 6. Ethics / deceptive patterns
- [ ] 7. Prioritized plan (report template)
```

Severity: **P0** blocker (incl. WCAG A/AA exclude / deceptive) · **P1** high friction · **P2** polish · **P3** nice-to-have.

### Output

1. Present the plan per [report-template.md](references/report-template.md) in chat (Russian summary + English findings OK).
2. If `docs/` exists (product-factory project), also write **`docs/UI-AUDIT.md`** (English, full template). Still **no app code changes**.

Each recommendation: Change · Why · Target · Where · Effort · Do not implement.

## Anti-patterns

- Running this automatically on every `orchestrate-product` delivery
- Rewriting the app in chat beyond ≤15-line target snippets
- Equating more animation/glass with quality
- Marking subjective taste as P0
