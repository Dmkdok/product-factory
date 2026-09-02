# Finding density example

## Input (abbreviated)

Modal confirm delete uses a `<div onClick>` “Delete” with no keyboard handler; primary and cancel look identical; no focus trap.

## Output finding

### F-012 — Destructive confirm is not a dialog
- **Severity:** P0
- **Where:** `src/features/projects/components/DeleteProjectModal.tsx` · `DeleteProjectModal` · Projects list
- **Evidence:** Activate Delete: focus stays on the page; Enter does nothing on the control; Esc does not dismiss; Cancel and Delete share the same filled style.
- **Principle:** Nielsen H3/H5; WCAG 2.1.1, 2.4.7, 4.1.2; WAI-ARIA APG Dialog pattern
- **Change:** rewrite control to accessible modal primitive
- **Target state:** Focus moves to dialog on open and is trapped; Esc/Cancel closes; Delete is clearly destructive (danger variant); both buttons have visible focus ≥3:1; screen reader announces name/role; confirm explains irreversibility.
- **Implementation notes for coding agent:**
  1. Replace custom overlay with shared `Modal`/`AlertDialog` from the design system (or APG-compliant primitive).
  2. Use `<button>` elements; map Delete to danger variant; Cancel = secondary.
  3. On open: focus primary safe action (Cancel) or dialog label per APG; restore focus to trigger on close.
  4. Do not keep `div`+`onClick` as the final control.
- **Effort:** M
- **Dependencies:** F-003 (shared Modal primitive) if missing
