# Pipeline details

## Definition of Ready

DoR is met when all are true:

1. **Problem** — who hurts, what job, what success looks like
2. **Users** — primary persona + at least one secondary or "none"
3. **Scope v1** — concrete features in / out
4. **Constraints** — stack, hosting, budget, deadline, language, brand, a11y, legal
5. **Content** — real copy sources or explicit permission to draft
6. **Integrations** — auth, payments, CMS, analytics listed or "none"
7. **Quality bar** — tests required? browser QA? performance budget?
8. **Done** — user can describe "I will accept the result when…"

## Phase outputs

| Phase | Artifact | Owner |
|-------|----------|-------|
| 1 | `docs/BRIEF.md` | product-planner |
| 2 | `docs/SPEC.md` | product-planner |
| 3 | `docs/PLAN.md`, `docs/TASKS.md` | architect |
| 4 | source tree | implementer(s) |
| 5 | test report in `docs/STATUS.md` | tester |
| 6 | review notes | reviewer |
| 7 | `docs/HANDOFF.md` | parent |
| 8 (optional) | `docs/RELEASE.md` | parent / implementer |

## Web-first defaults (when user did not choose)

Prefer current mainstream defaults unless constrained:

- **Site / marketing**: static or SSG (Astro / Next) + accessible HTML + distinctive design
- **SaaS web app**: TypeScript, Next.js or similar full-stack, Postgres, auth library with good docs, Playwright e2e
- **API**: OpenAPI-first, typed handlers, integration tests

Always verify library docs with a quick web check if unsure about mid-2026 APIs.

## Parallelism rules

Safe to parallelize:

- Independent pages/modules with no shared files
- Backend route groups vs frontend presentational components (if contracts frozen)
- Test authoring for finished modules

Serialize:

- Schema / migrations
- Auth foundation
- Design tokens / global styles (establish once, then parallelize pages)
