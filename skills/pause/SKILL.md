---
name: pause
description: >-
  Emergency stop for the current session — halts everything in seconds so the user can walk away
  right now. Snapshots the tree as-is (no test run, no commit-splitting), writes a minimal, precise
  "Resume here" in docs/STATUS.md so the next session/agent needs almost no rereading to continue.
  Use when the user needs to leave immediately — "пауза", "срочно ухожу", "на сегодня всё", /pause.
  When there's actually a few minutes to spare for a proper wrap-up, use `close-session` instead —
  that one runs the suite and splits commits by intent; this one never does.
metadata:
  author: product-factory
  version: "1.0.0"
---

# Pause

## Goal

Stop in seconds, not minutes. The only job: nothing is lost, nothing is silently left running, and
the next agent can read a few lines and know exactly where to pick up — without rerunning tests,
rereading source, or guessing. Speed beats tidiness here: a WIP commit that says "half-done, broken,
here's what's left" is a correct pause. A polished handoff that took five minutes is not — the point
of this skill is that the user is already out the door.

**Do zero new work.** Not even a "quick fix" that's 30 seconds away. Not even running the test suite
to check it's still green. If it isn't already done, it goes in the notes as undone, not finished.

## Workflow (in order, no looping back to polish an earlier step)

### 1. Kill everything in flight — do not wait

- Any running background Bash command or subagent: stop it now — cancel it through whatever the
  harness gives you for a running background task (in Claude Code, `TaskStop`). Do not let it
  "finish real quick," and do not read its full transcript to see how far it got — that can pull a
  huge log into context for a pause that's supposed to cost nothing.
- Before stopping each one, write one checkpoint line from what you already know for free — no new
  tool call: what task/goal you gave it and, from what you've already seen it do in this
  conversation, roughly which step it was on. Put that line in `## Resume here` in step 3. Its
  actual file edits are not at risk — step 2 commits everything it already touched, so this line
  only needs to say what's *left*, not repeat what's now safely in the commit.
- If it owned one specific line in `docs/TASKS.md`, append `⏸ paused mid-work — see STATUS.md` to
  that line (one inline edit, not a pass over the rest of the file) so the next session finds it
  from either document.
- For a background Bash command specifically, its output file (the path you got back when you
  started it) is plain stdout/stderr, not a transcript — a quick `Read` of the tail is cheap and
  safe if it tells you whether the real work finished before the kill; skip it if it doesn't.
- Do not judge whether a half-finished edit is "minutes from done." There is no time budget for that
  judgment call. Leave it exactly as it is.

### 2. One snapshot commit — never split, never restore

- `git status` once — this is the only inspection step. Do not `git diff` file by file. In the same
  glance, check for anything that must never be committed — `.env`, keys, credentials, generated
  output not already gitignored. This costs nothing extra since the output is already in front of
  you; if something like that shows up untracked, leave it out of the add and say so in the
  report — a blind `git add -A` under time pressure is exactly how secrets end up in history.
- `git stash list` — if a stash exists from an earlier session, note that it exists in STATUS.md and
  move on. Do not resolve it now.
- If currently on `main`/`master`, branch to `session/<YYYY-MM-DD>-<slug>` (one command). Otherwise
  stay put.
- `git add -A` (or everything except what you just excluded above) and one commit, whatever state
  the tree is in. Message:
  `pause: session snapshot (unverified)`, body naming what's mid-edit. Never split by intent, never
  amend, never rewrite history — one commit, now, even if it doesn't build.
- **No test run. No lint run.** Do not hunt for cached results, do not reason about whether it would
  "probably still pass." STATUS.md just records the time and says unverified.

### 3. Rewrite `## Resume here` in `docs/STATUS.md` — touch nothing else

Replace only this section (leave `## Test report`, `## Notes`, everything else untouched — archiving
and trimming are a calm-session job, not this one). It must give a future agent everything needed to
act with zero rereading of code:

- Branch name, and that the tree was committed unverified.
- One line: exactly what was in progress when the pause hit.
- **The next concrete action, singular** — a file path plus what to do there, phrased so it can be
  started with no investigation. Not "continue the feature."
- **Killed subagents/commands**, one checkpoint line each from step 1 — task/goal, last known step,
  and whether it needs rerunning or just finishing by hand. This is what lets the next session skip
  redoing work that's already sitting in the commit.
- Test status: one line — "not run, paused at HH:MM."

If `## Resume here` doesn't exist yet, add it at the top of the file. If `docs/STATUS.md` doesn't
exist, create it with just this section — do not backfill the rest of the template now.

### 4. Report — one line

Russian, one line: branch, "закоммичено без проверки", what's still running if anything killed in
step 1 needs attention, and any file left out of the commit on purpose. Nothing else — the user is
leaving.

## Rules

- Every step is allowed to err toward "too blunt" (one big commit, no verification) — never toward
  "lost work" (never `git restore` a half-done edit, never leave a subagent silently still running)
  and never toward "leaked secret" (never sweep an untracked credential file into the snapshot
  commit unexamined).
- If in doubt whether something is worth an extra 10 seconds, it isn't. That budget left when this
  skill was invoked.
- One mode, no exceptions. If there's time for a proper wrap-up (full verification, commits split by
  intent, STATUS.md archiving), the user says so explicitly, or asks for it by name — use
  `close-session` for that, don't try to be thorough inside this skill.
