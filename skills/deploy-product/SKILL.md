---
name: deploy-product
description: >-
  Ships a reviewed build to its real target and proves it is actually live —
  not just that a deploy command exited 0. Optional Phase 8 after
  orchestrate-product/iterate-product's handoff or close. Use only when a real
  deploy target exists and the user wants it shipped, not just handed off.
disable-model-invocation: true
metadata:
  author: product-factory
  version: "1.2.0"
---

# Deploy Product

## When this applies

Skip this skill entirely for a local-only product with no real deploy target — nothing here is
mandatory scaffolding. Load it when `orchestrate-product` Phase 7 or `iterate-product` Phase 7 has
already produced a PASS (or PASS WITH WAIVERS, no open Critical/High) in `docs/REVIEW.md`, and the
user wants the result running somewhere real: staging, production, a NAS, whatever the project calls
its live target.

## Goal

Prove the *deployed* thing works, not that a deploy command returned 0. Same skeptical stance as
`test-product`: no "should work," here applied to the shipped artifact instead of the local tree. A
green CI pipeline can still ship a broken migration or a misconfigured env var — only a smoke test
against the real target proves otherwise.

## Preflight (before touching anything live)

- [ ] Review verdict is PASS or PASS WITH WAIVERS with every Critical/High resolved — read
      `docs/REVIEW.md` yourself, do not take "it was reviewed" on faith.
- [ ] Target environment named explicitly — which one (staging vs production), and why this one.
- [ ] Env vars / secrets for that target exist and are checked by **name**, never by value — diff the
      names in `.env.example` (or equivalent) against what the target actually has configured. Never
      paste actual secret values into chat, `docs/`, or a commit.
- [ ] Migrations, if any, reviewed for reversibility before they touch real data.
- [ ] Rollback plan written down **before** the deploy starts — the exact command or steps that undo
      this deploy — not improvised after something breaks.

## Workflow

1. Confirm the target and the trigger method from the project itself (CI pipeline, a deploy script,
   `docker compose up` on a host, a push to a deploy branch, ...) — read it, don't assume one exists
   or guess from a different project's memory.
2. Run the project's own deploy path and capture what it actually printed — not "should have worked."
3. Smoke test against the live URL/host: re-run the same critical-path checks `test-product` ran
   locally, now against the real target. A local-only green suite proves nothing about what's live.
4. Confirm the rollback path is real — state the exact command/step, don't just assert one exists.
5. Record everything in `docs/RELEASE.md` ([templates/RELEASE.template.md](../../templates/RELEASE.template.md)
   or `templates/product-factory/RELEASE.template.md` in an installed project): what shipped, the
   exact commit/version deployed (`git rev-parse --short HEAD`, or a tag if the project cuts them),
   where, when, by what method, smoke-test evidence, the rollback command, and what will surface a
   failure after this point (platform logs, error tracker, uptime check, or "none — check manually" —
   pull this from `docs/PLAN.md` → Observability rather than inventing it here).
6. Append one entry to `docs/CHANGELOG.md` ([templates/CHANGELOG.template.md](../../templates/CHANGELOG.template.md)):
   what changed, user-visible first, with the same commit/version as step 5. If the file doesn't
   exist yet (a product built before this template existed), create it from the template rather than
   skipping the entry.
7. Update `docs/STATUS.md` and `docs/HANDOFF.md` so a resuming session knows the current *live*
   state, not just what the tree contains.

## Rules

- Never deploy from a FAIL review verdict.
- A green deploy pipeline is not proof the app is live and correct — the post-deploy smoke test is
  what proves it. Do not skip step 3 because step 2 exited cleanly.
- Secrets: name what each one is for, never write actual values into `docs/` or the conversation.
- If no real deploy target exists yet, say so and stop — this phase is optional, not something to
  invent scaffolding for.

## Anti-patterns

- Deploying straight after Phase 6 review without reading the actual verdict
- "It deployed" treated as the definition of done, with no smoke test against the live target
- Rollback plan invented after something breaks, instead of written down before the deploy starts
- Secret values copy-pasted into `docs/RELEASE.md`, `docs/STATUS.md`, or chat
- Running this phase silently, or skipping it silently when the user expected the result shipped
