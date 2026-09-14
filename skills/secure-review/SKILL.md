---
name: secure-review
description: >-
  Security review for app code: secrets, authz, injection, XSS, CSRF, risky
  defaults, dependency hygiene. Optionally runs Semgrep when installed.
  Use during review-product Phase 6, before handoff, or when the user asks for
  a security audit. Lean checklist inspired by Trail of Bits static-analysis
  practice — not a full Semgrep/CodeQL plugin clone.
context: fork
agent: reviewer
background: false
metadata:
  author: product-factory
  version: "1.2.0"
  inspired_by: trailofbits/skills static-analysis
---

# Secure Review

Skeptical security pass. Prefer evidence over vibes. Write findings into `docs/REVIEW.md` under `## Security` (or a dedicated `docs/SECURITY.md` if long).

## Hard rules

1. Never print secret values; redact.
2. If Semgrep/CodeQL unavailable, still complete the **manual checklist** — tools are optional amplifiers.
3. Critical findings block handoff (FAIL) unless waived in `docs/DECISIONS.md`.

## Workflow

```text
1. Detect stack (web, API, mobile, desktop) and trust boundaries
2. Manual checklist (below)
3. If `semgrep` on PATH → optional scan (ask user if large/paid rulesets)
4. Dependency audit (below) — run the stack's real audit tool, don't skip to "lockfile exists"
5. Triage: true positive vs noise; cite file paths
6. Merge into review verdict
```

### Dependency audit

Run whichever applies to the detected stack — don't invent a tool that isn't installed, and say so
if none is available rather than silently skipping to the lockfile-only check:

```bash
npm audit --omit=dev          # Node, or: pnpm audit / yarn npm audit
pip-audit                     # Python, if installed; pip list --outdated as a weaker fallback
cargo audit                   # Rust
govulncheck ./...             # Go
```

Triage output the same way as Semgrep findings — Critical/High block unless waived. For any **new**
or major-version-bumped dependency in the diff, also spot-check its license (`npx license-checker`,
the package registry page, or the repo's own `LICENSE`) against this project's own license — a
copyleft dependency (GPL/AGPL) pulled into a project distributed under a permissive license is a
finding, not a footnote.

### Optional Semgrep (when installed)

```bash
semgrep --metrics=off --config p/secrets --config p/security-audit .
```

Language packs if relevant (`p/python`, `p/javascript`, `p/typescript`, `p/golang`, …). Prefer `--metrics=off`. Do **not** invent Trail of Bits subagent plugins; if the user has `trailofbits/skills` installed separately, defer to that for deep scans.

## Manual checklist (high-yield)

### Secrets & supply chain
- [ ] No API keys/tokens in repo, logs, or client bundles
- [ ] `.env` gitignored; `.env.example` has placeholders only
- [ ] Lockfile present
- [ ] Dependency audit tool run for the detected stack (see Dependency audit above); Critical/High
      vulnerabilities triaged, not left unexamined
- [ ] New/major-bumped dependencies license-checked against the project's own license

### Authn / authz
- [ ] Auth required routes actually enforced server-side
- [ ] IDs in URLs/body checked against caller permissions (IDOR)
- [ ] Sessions/JWT: secure flags, reasonable expiry, no sensitive data in client storage without need

### Injection & XSS
- [ ] SQL/NoSQL via parameterized queries / ORM binds
- [ ] Shell/exec: no unsanitized user input
- [ ] HTML rendering: escape by default; sanitize if `dangerouslySetInnerHTML` / equivalents
- [ ] SSRF: user URLs validated/allowlisted if fetched server-side

### Web misuse
- [ ] CSRF strategy for cookie-based mutating requests
- [ ] CORS not `*` with credentials
- [ ] File upload: type/size/path traversal controls if applicable
- [ ] Rate limit or equivalent on auth and costly endpoints (note if missing)

### Data & privacy
- [ ] PII minimized in logs
- [ ] Error messages don’t leak stacks to end users in production builds

## Output format

```markdown
### Security
**Tooling:** manual | semgrep (<configs>) | skipped (<reason>)

| ID | Severity | Issue | Where | Fix |
|----|----------|-------|-------|-----|
| S-001 | Critical/High/Medium/Low | … | `path` | … |
```

No issues → explicitly write `No critical/high findings in scoped review.`
