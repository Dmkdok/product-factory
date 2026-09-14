# Conventions

Read by every subagent that touches code (see `docs/PLAN.md` → Repository map). Keep this file
short and load-bearing — rules that keep parallel work coherent, not a style-guide essay. When a
convention here conflicts with a documented project default (linter config, framework convention),
the tool config wins; write down only what isn't already enforced by tooling.

## Naming
- Files:
- Components / modules:
- Variables / functions:

## Code style
- Formatter / linter: <command, config file>
- Language-specific idioms this project follows:

## Structure
- Where new features live:
- Shared/primitive code lives in:
- Import direction (e.g. shared → features → app):

## Testing
- Test file location / naming:
- What must have a test (per `docs/PLAN.md` → Test strategy):

## Commits & branches
- Branch naming: `session/<YYYY-MM-DD>-<slug>` or `iteration/I<n>-<slug>`
- Commit header: Conventional Commits (`type(scope): description`)
- Task references: footer only (`Refs: T145`), never in the header

## Env & secrets
- `.env.example` is the source of truth for required vars; never commit real values.

## Notes
- <anything a subagent would otherwise have to rediscover by reading the whole tree>
