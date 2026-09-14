---
name: close-session
description: >-
  Closes out a milestone or working session properly, with the time to do it right: stops new
  work, brings the tree to a stable state, verifies the suite, commits split by intent, and
  rewrites the handoff in docs/STATUS.md. Use when a milestone lands, at the end of iterate-product
  Phase 7, or when the user is ending work for now but is not in a rush. If the user needs to leave
  immediately, use `pause` instead — that skill skips verification for speed, this one does not.
metadata:
  author: product-factory
  version: "1.1.0"
---

# Close session

## Goal

The next session must be able to start from `docs/STATUS.md` alone and be right. Not a summary of
this chat — a description of the tree as it actually is.

**Start no new work.** No refactors, no "quick fixes", no opportunistic cleanup. If something is
broken, it gets written down, not fixed. The only work allowed here is closing out what is already
open.

This skill assumes there's a real few minutes to spend — a full suite run, commits split by intent,
an accurate handoff. If that time isn't there right now, stop and use `pause` instead; don't rush
through this workflow and skip steps silently, since a half-done thorough close is worse than an
honest fast one.

## Workflow

### 1. Close what is open

Look at what is in flight — edits, background commands, subagents.

- Half-finished edit that leaves the code non-working: finish it if that is minutes, otherwise
  `git restore` it and say so in the notes. Never leave the tree in a state that does not run.
- Background commands and subagents: let them finish or stop them. Report anything still running.
- Never leave a half-applied migration or a half-written test file unmentioned.
- Run `git status`, not just `git diff` — it is the only view that surfaces untracked files and a
  merge/rebase left mid-flight. A new untracked file or directory gets a decision, never silence:
  commit it, add it to `.gitignore`, or write down in the notes why it stays untracked. The same
  look catches what should never be committed — env files, credentials, generated output — before
  it is staged, not after.
- `git stash list` — a stash from this session or an earlier one is a trap for whoever finds it
  next with no memory of what it holds. Resolve it (pop it into a real commit, or drop it if it was
  scratch) rather than leaving it anonymous.

### 2. Verify — do not trust the checkboxes

Run *this* project's real test command — do not assume one. Check `CLAUDE.md` for a documented
"Tests:" line first; if there is none, find it from the repo itself (`package.json` scripts,
`Makefile`, `pyproject.toml`/`pytest.ini`, a `docker-compose.yml` test service). A command copied
from a different project's memory is worse than asking.

In Bash never pipe a test run through `tail`/`head` — you get the pipe's exit code and a red suite
reads as green. In PowerShell a cmdlet pipe leaves `$LASTEXITCODE` alone, so
`... 2>&1 | Tee-Object $log | Select-Object -Last 30; $LASTEXITCODE` is safe there.

If the project documents a separate e2e/browser suite, run that too, on whatever host/container
split `CLAUDE.md` describes. If a suite is too slow to re-run at this hour, do not invent a result —
recover the last recorded run if the runner caches one (e.g. pytest's `.pytest_cache`), and label it
with that run's timestamp.

Run the project's lint/format gate too, whatever it is (`ruff`, `eslint`, `cargo clippy`, ...) — a
green test run with a dirty lint has verified only part of the DoD.

A red suite is a fine way to end a session. A red suite recorded as green is not.

### 3. Commit

Never commit to `main`. Branch as `session/<YYYY-MM-DD>-<slug>` if not already on one.

Split by intent, not by directory — fixes, new tests, docs/evidence are separate commits. Header
follows Conventional Commits (`type(scope): description`; task refs go in a footer, `Refs: T145`,
not the header) unless the project's own docs say otherwise — check `CLAUDE.md`/`CONTRIBUTING`
first, the same way you check it for the test command. The body still says *why*, and names the
defect. Do not push unless asked.

If the suite is red, still commit — but say so in the commit body.

### 4. Rewrite the handoff

`docs/STATUS.md`, in this order:

- **`## Resume here`** at the top, rewritten (not appended to). It carries: branch, whether the
  tree is clean, and how far it has diverged from `main`
  (`git rev-list --left-right --count main...HEAD`); where the work stands in one line; **the next
  three actions in order**, each concrete enough to start without re-reading the code; and any
  decision that is waiting on the owner, marked as theirs. A branch several commits ahead with
  `main` untouched needs nothing beyond the count; one where `main` has moved needs the
  merge/rebase called out explicitly, not discovered by surprise next session. **Keep it under 40
  lines** — a SessionStart hook, where one is configured, injects only the first ~60 lines and
  truncates the rest, so anything past that is paid for on every read and delivered to no one.
- **`## Test report`** — every suite, with its own last-run timestamp and command. Name the failing
  tests individually.
- **`## Notes`** — append dated entries for what this session learned. Root causes and traps, not a
  diary of actions. A defect worth a note is one that would cost the next session an hour.

Then `docs/TASKS.md`: tick only what its DoD actually meets, and write the open half of a partly
done task into the task line itself.

### 4b. Keep the handoff small

`docs/STATUS.md` is read at every session start and by every subagent that resumes work, so its size
is a recurring cost. Before finishing, move what is no longer live:

- Notes older than the current iteration → `docs/status-archive.md`, under a dated heading.
- A baseline whose iteration has closed → `docs/status-archive.md`.
- A milestone whose tasks are all ticked → `docs/tasks-archive.md`.

Move the text, never summarise it, and leave the pointer in `## History` accurate. **`docs/STATUS.md`
past ~20 KB means this step was skipped.** A trap that is still load-bearing does not go to the
archive — it goes to `CLAUDE.md` or `docs/CONVENTIONS.md`, where it is read at the moment it applies.

Re-read what you wrote against the tree. A "Resume here" that says the tree is dirty after you
committed it is worse than no handoff.

### 5. Report

Four or five lines to the user, in Russian: suite result, what was committed and on which branch,
the single most important thing to pick up next, and anything left running (containers, background
tasks). No wrap-up prose.

## Rules

- Read `docs/STATUS.md` before writing it. Do not summarise the chat — audit the tree.
- Prefer Serena's symbol tools (`mcp__serena__get_symbols_overview`, `mcp__serena__find_symbol`,
  ...) over whole-file reads; at close time there is no reason to burn context re-reading source.
- If the session ended mid-task, that is the single most important fact in the handoff. Lead with
  it.
- If a prior `pause` left a "killed subagents" note in `## Resume here`, resolve each one (rerun it,
  finish it by hand, or mark it abandoned with why) before writing the new handoff — don't let it
  ride forward unresolved a second time.
