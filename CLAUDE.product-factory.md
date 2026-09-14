For greenfield sites/apps, use skill `orchestrate-product`.

- User-facing language: Russian
- Artifacts under `docs/`: English
- No application code until the user approves the plan (`утверждаю`)
- After approval: implement → test → review → handoff → deploy (optional)
- CI is off by default — skill `setup-ci` only on explicit request, never auto-triggered
- Prefer subagents: product-planner, architect, implementer, tester, reviewer
- Token/context hygiene: skill `context-token-optimization` — always, not on request

## Git

- Never commit to `main`. Branch as `session/<YYYY-MM-DD>-<slug>` or `iteration/I<n>-<slug>`.
- Commit headers follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/):
  `type(scope): description` (`feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `chore`; scope
  optional, names the affected area). Body explains *why* and names the defect; a task reference
  goes in a footer (`Refs: T145`), not the header.
- Split commits by intent (fixes / new tests / docs), not by directory.
- Do not push unless asked.
