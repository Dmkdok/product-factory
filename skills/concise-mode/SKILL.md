---
name: concise-mode
description: >-
  Token-efficient communication mode: strip filler and narration while keeping
  every technical fact, path, command, and error intact. Use when the user asks
  for кратко, caveman, concise, экономь токены, or less chatter. Adapts the
  popular Caveman idea without broken English that fights Russian UX policy.
metadata:
  author: product-factory
  version: "1.0.0"
  inspired_by: JuliusBrussee/caveman
---

# Concise Mode

Activate when the user wants less talk, more signal. Stay on until they say «обычный режим» / «normal mode» or the task ends.

## Rules

1. **Keep facts byte-exact:** paths, symbols, commands, diffs, error text, numbers — never paraphrase away precision.
2. **Cut filler:** no preambles (“Sure!”, “I'll now…”, “Great question”), no restating the task, no motivational fluff.
3. **Structure over prose:** bullets, tables, short headings. One idea per line when listing.
4. **Language policy still applies:** user-facing chat stays **clear Russian** (grammatically correct — not pidgin). English only for code, commands, and `docs/` artifacts.
5. **Show outcomes, not process theater:** prefer “Changed X → verified Y” over step-by-step narration of tool use.
6. **Ask only blocking questions.** Otherwise proceed.
7. **Code blocks:** minimal; no duplicate explanations under the same snippet.

## Response shape (default)

```text
## Result
- …

## Verify
- <command or check>

## Open
- <only real blockers>
```

Omit empty sections.

## Do not

- Sacrifice correctness for brevity
- Skip required skill gates (plan approval, tests, security) to “save tokens”
- Use mock-caveman broken speech with the user
