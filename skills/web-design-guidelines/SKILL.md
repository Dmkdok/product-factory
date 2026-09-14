---
name: web-design-guidelines
description: Review UI code for Web Interface Guidelines compliance — mechanical accessibility/best-practice checks, not a deep UX audit. Use when asked to "review my UI", "check accessibility", "audit design", or "check my site against best practices". For a deeper usability/architecture pass, use ui-quality-audit instead.
metadata:
  author: vercel
  version: "1.1.0"
  argument-hint: <file-or-pattern>
---

# Web Interface Guidelines

Review files for compliance with Web Interface Guidelines.

## How It Works

1. Fetch the latest guidelines from the source URL below (fall back to the bundled copy if the fetch fails)
2. Read the specified files (or prompt user for files/pattern)
3. Check against all rules in the fetched (or fallback) guidelines
4. Output findings in the terse `file:line` format

## Guidelines source

Try the live source first, for the freshest rules:

```
https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md
```

Use WebFetch to retrieve it. **If the fetch fails or errors** (URL moved, offline, rate-limited),
use the bundled [fallback.md](fallback.md) instead — say once that live rules were unreachable and
the bundled snapshot is in use, don't fail the review silently or leave it half-done.

## Usage

When a user provides a file or pattern argument:
1. Fetch guidelines from the source URL above (or `fallback.md` per the rule above)
2. Read the specified files
3. Apply all rules from the fetched/fallback guidelines
4. Output findings using the format specified in the guidelines (or, on fallback.md, the `file:line` format from this SKILL.md)

If no files specified, ask the user which files to review.
