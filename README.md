# product-factory (plugin pack)

Portable copy of the `portfolio` project's delivery pipeline: 16 skills + 5 agents that take a
product from a one-line idea to a reviewed, tested, optionally deployed handoff. Built from
`.claude/skills/` and `.claude/agents/` in `portfolio` (`E:\Dev\_AI_\portfolio`), kept in sync by
hand — this pack lives on its own now (moved out of that repo on 2026-08-28), not a live symlink.

Lives at `E:\Dev\_AI_\skills\product-factory` and on GitHub as
[Dmkdok/product-factory](https://github.com/Dmkdok/product-factory) (moved off Desktop and put under
version control on 2026-09-02) so it can keep evolving on its own instead of being a one-off export.

## What's inside

```
.claude-plugin/plugin.json   — manifest (name: product-factory)
skills/                      — pipeline (10): orchestrate-product, iterate-product,
                                discover-requirements, draft-product-spec, draft-tech-plan,
                                implement-product, test-product, review-product, deploy-product, pause
                                bundled dependencies (6): coding-discipline, concise-mode,
                                secure-review, web-design-guidelines, frontend-design, ui-quality-audit
agents/                      — product-planner, architect, implementer, tester, reviewer
templates/                   — BRIEF/SPEC/PLAN/TASKS/STATUS/DECISIONS/REVIEW/HANDOFF/RELEASE .template.md
CLAUDE.product-factory.md    — block to paste into a new project's CLAUDE.md
SKILLS-GUIDE.md              — Russian cheat-sheet: which skill for which moment, session-management
                                advice, name pitfalls. Written for the human, not for Claude.
```

## Try it in another project

```bash
claude --plugin-dir /path/to/product-factory
```

No `marketplace.json` needed for local use — `--plugin-dir` reads `plugin.json` directly. Invoked
skills are namespaced `/product-factory:orchestrate-product` etc. to avoid colliding with anything
project-local of the same name.

**Plugin name resolution — hardened, not live-tested.** Every in-pack cross-reference (e.g.
`orchestrate-product` saying "read skill `discover-requirements`") uses the bare name.
`orchestrate-product/SKILL.md` and `iterate-product/SKILL.md` each now carry a note telling the
agent to retry with the `product-factory:` prefix if the bare name doesn't trigger, so a
plugin-loaded run degrades to one extra tool call instead of a silent skip. That note is a defensive
fix, not a live test — the first real run of this pack in an external project should still confirm
it and report back if a reference needed hand-editing.

## Bootstrapping a brand-new project

1. Install/point at this plugin (above), or just copy this folder's `skills/` and `agents/` into the
   new project's `.claude/` — nothing else to fetch, the six dependency skills below are already in
   `skills/`.
2. Paste `CLAUDE.product-factory.md`'s content into the new project's `CLAUDE.md`.
3. Copy `templates/` into the new project as `templates/product-factory/` — `orchestrate-product`
   Phase 0 reads templates from there first, and falls back to a plain `templates/` only when running
   with the pack root itself as the working directory.
4. Run `fewer-permission-prompts` once (and `update-config` for project-specific hooks/permissions)
   to cut permission-prompt friction before the pipeline starts generating tool calls in volume.
5. Start with `/orchestrate-product` (greenfield) or `/iterate-product` (repo already has `docs/SPEC.md`).

## Dependencies

**Bundled in `skills/`, kept in sync by hand with `~/.claude/skills/` on this machine** — same manual
model already used for the other 10 skills, no separate install step: `coding-discipline`,
`concise-mode`, `secure-review`, `web-design-guidelines`, `frontend-design`, `ui-quality-audit`.
`orchestrate-product` Phase 0 and `iterate-product` Phase 0 still confirm they resolve before Phase
4/6 — a cheap check for the case only part of `skills/` got copied out of this pack, not the primary
defense anymore.

**Not files at all — built into Claude Code itself, nothing to install, available on any machine
running the CLI:** `code-review`, `simplify`. These were wrongly listed as "install separately" in an
earlier version of this doc; there is no `~/.claude/skills/code-review` on disk to copy, they ship
with the `claude` binary.

## Known divergence from the portfolio project's own copy

`skills/pause/SKILL.md` here does **not** hardcode `docker compose run --rm tests` the way
`portfolio`'s own `.claude/skills/pause/SKILL.md` does — that command is correct only for that one
repo. This pack's copy asks the agent to find the real test command from the target project's own
`CLAUDE.md`/`package.json`/`Makefile` instead. Everything else is a straight copy.
