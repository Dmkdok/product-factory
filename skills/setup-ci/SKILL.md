---
name: setup-ci
description: >-
  Scaffolds a minimal CI pipeline (install, lint, test, and build where
  applicable) matched to the project's real stack and test commands. Optional
  and off by default — most product-factory projects run without CI. Use only
  when the user explicitly asks for CI/CD, or answers yes to "CI expected?"
  at discovery and confirms they want it set up now.
disable-model-invocation: true
metadata:
  author: product-factory
  version: "1.0.0"
---

# Setup CI

## When this applies

Skip this skill entirely unless the user explicitly asks for it. **Most projects built with this
pack run without CI, and that's a legitimate choice, not a gap to fix silently.** A "yes" to "CI
expected?" at discovery is a note for later, not a trigger to run this now — confirm before
scaffolding anything.

## Goal

One working pipeline that runs the project's real lint + test commands on push/PR, using tooling
that already exists in the repo — not a generic template that assumes a stack the project doesn't
have.

## Preflight

- [ ] Provider confirmed — infer from hosting (a GitHub remote → GitHub Actions by default) or ask
      if genuinely ambiguous. Don't default to a provider the project shows no sign of using.
- [ ] Real lint/test commands read from `docs/CONVENTIONS.md` first, then `package.json` scripts /
      `Makefile` / `pyproject.toml` — the same sources `close-session` checks. Never invent a command.
- [ ] Checked for an existing CI config in the repo — extend it, don't stand up a second competing
      pipeline next to one that already runs.

## Workflow

1. Detect the stack from its manifest (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, …)
   and the real lint/test commands per the preflight above.
2. Generate the minimal pipeline: install deps (cached) → lint → test → build, and only include
   build if the project actually has a build step. **No deploy step here** — verification and
   shipping are different concerns; `deploy-product` already owns shipping, and mixing them makes
   this pipeline hold secrets it doesn't need.
3. Write it to the provider's conventional path (e.g. `.github/workflows/ci.yml` for GitHub
   Actions).
4. Trigger on push to non-`main` branches and on PRs into `main` — match this pack's own branch
   convention (`session/<YYYY-MM-DD>-<slug>`, `iteration/I<n>-<slug>`) rather than inventing another.
5. If the suite needs secrets or services (DB, API keys) to run, name them by variable name only —
   never a value — and point to where they must be configured (repo/CI secrets settings), per
   `secure-review`'s secrets rule.
6. Tell the user the pipeline exists but hasn't run yet — the first real signal is the next push,
   not this session's belief that the YAML is correct.
7. Note it in `docs/CONVENTIONS.md` (`CI: <provider>, runs lint+test on push/PR`) and in
   `docs/PLAN.md` → `## Deploy notes` if that file exists.

## Rules

- **Verify only, don't ship.** No deploy step, no credentials beyond what the test suite itself
  needs to run.
- **Minimal by default.** Install → lint → test → (build). No caching tricks, matrix builds, or
  multi-OS runs unless asked — unrequested configuration is exactly what `coding-discipline` warns
  against.
- **Never fabricate a command.** If no documented lint/test command exists anywhere, find one from
  the repo the way `close-session` does, or ask — don't guess a plausible-looking one.

## Anti-patterns

- Auto-running this because the user said "deploy" or mentioned tests — CI is a separate, opt-in ask
- A generic templated pipeline that doesn't match the project's actual test command
- Adding a deploy step here instead of pointing to `deploy-product`
- Treating a "yes" to "CI expected?" at discovery as consent to scaffold immediately, instead of
  confirming when the user actually wants it built
