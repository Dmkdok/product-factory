# Fallback checklist (use only when the live fetch fails)

This is a compact, load-bearing subset of universal web-interface best practice — not a mirror of
the vercel-labs guidelines. It exists so this skill never returns an empty review when the live
source is unreachable. Prefer the live fetch whenever it succeeds; reach for this file only as a
degraded-mode fallback, and say so in the output.

## Contents
- Accessibility (highest-yield checks)
- Forms & interaction
- Layout & responsiveness
- Performance-affecting UI
- States

## Accessibility

- Every interactive element reachable and operable by keyboard alone; visible focus indicator.
- Text contrast ≥4.5:1 (large text ≥3:1); non-text UI elements ≥3:1.
- Icon-only controls have an accessible name (`aria-label` or equivalent); no placeholder-as-label.
- Semantic HTML first (`button`, `nav`, `main`, headings in order) before ARIA patches it.
- Images convey meaning via `alt`; decorative images have empty `alt`.
- Touch targets ≥24×24 CSS px.

## Forms & interaction

- Every input has a programmatically associated label.
- Errors are shown in text near the field, not by color alone; the message says what to fix.
- Destructive actions require confirmation; the confirming control names the consequence.
- Disabled state is visually distinct and not the only way a field communicates "not yet available."

## Layout & responsiveness

- No horizontal scroll at common breakpoints (≥320px wide).
- Content reflows rather than truncating unpredictably at narrow widths.
- Consistent spacing scale; no ad hoc pixel values scattered through the same component.

## Performance-affecting UI

- Images sized/lazy-loaded where below the fold; no unbounded layout shift on load (CLS).
- Long lists virtualized or paginated rather than rendering thousands of DOM nodes at once.

## States

Every primary flow should have a real answer for: loading, empty, error, and success — not just the
happy path. A control with no pending/error state is a finding, not an assumption to skip.
