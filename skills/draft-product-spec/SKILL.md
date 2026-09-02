---
name: draft-product-spec
description: >-
  Turns an approved brief into an engineering-ready SPEC.md with flows,
  acceptance criteria, and non-goals. Use after elicitation or for PRD-style
  specs before implementation planning.
---

# Draft Product Spec

## Input

`docs/BRIEF.md` (required). Chat history only as supplement.

## Output

`docs/SPEC.md` in English using `templates/product-factory/SPEC.template.md` (or `templates/SPEC.template.md` inside the product-factory pack root itself).

## Spec quality bar

- Every in-scope feature has **acceptance criteria** (Given/When/Then or bullet checks)
- Explicit **non-goals**
- UX: key user flows as numbered steps
- Data: entities and critical fields (even for static sites: content model)
- Edge cases and error states listed for interactive products
- Risks & assumptions numbered for later DECISIONS.md

## Process

1. Read BRIEF; list ambiguities — resolve with user in Russian if blocking.
2. Draft SPEC.
3. Show the user a Russian executive summary (1/2 page); ask to confirm before tech plan.
4. Do not write code.

## Inspiration (do not copy proprietary text)

Structured PRD practice similar in spirit to community PM skill packs
(e.g. deanpeters/Product-Manager-Skills — optional deeper install; CC BY-NC-SA).
This skill's wording is original to product-factory.
