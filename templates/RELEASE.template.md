# Release

## Shipped
- What:
- Version / commit: <git tag or `git rev-parse --short HEAD` at deploy time — the only way a later
  session can answer "what exactly is running right now" without guessing>
- Where (environment / URL):
- When:
- Method (CI pipeline / manual command):

## Preflight
- Review verdict: (see `docs/REVIEW.md`)
- Env vars / secrets confirmed present (names only, never values):
- Migrations run:

## Smoke test
- Evidence:

## Observability
- What will surface a failure after this deploy (platform logs, error tracker, uptime check, or
  "none — check manually"): per `docs/PLAN.md` → Observability.

## Rollback
- Command / steps:
- Verified before deploy: yes/no

## Notes
-
