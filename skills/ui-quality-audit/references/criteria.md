# Audit criteria (checklists)

TOC: [Visual](#1-visual--layout) · [Interaction](#2-interaction--controls) · [Nielsen](#3-nielsen-10-heuristics) · [Laws of UX](#4-laws-of-ux-high-signal) · [WCAG](#5-accessibility-wcag-22-aa-floor) · [States](#6-ui-states-completeness) · [Content](#7-content--ia) · [Ethics](#8-ethics--deceptive-patterns)

Score each item: **Pass / Partial / Fail / N/A**. Failures become findings.

## 1. Visual & layout

| Check | Fail if |
|-------|---------|
| Hierarchy | Multiple competing “primary” foci; brand/product signal weaker than random chrome |
| Spacing rhythm | Random gaps; cramped clusters next to empty voids; no consistent scale (4/8-pt or design tokens) |
| Typography | Default system stack with no scale; body unreadable; headings skip levels visually or in DOM |
| Color | Decorative color only; status by color alone; low contrast; no semantic tokens |
| Alignment | Ragged columns; inconsistent card/row insets; optical imbalance |
| Density | Hero/dashboard clutter; secondary metadata in first viewport without need |
| Imagery | Abstract filler as the only visual idea when product UI could anchor |
| Motion | Pure noise; >essential motion without reduced-motion respect |
| Platform fit | Ignores HIG / Material / Fluent conventions for that OS |

Target craft: laconic, aligned, token-driven. Prefer fewer unique sizes/colors.

## 2. Interaction & controls

| Check | Fail if |
|-------|---------|
| Affordances | Clickable things look static; static look clickable (Norman: affordance/signifier) |
| Targets | Touch <24×24 CSS px (WCAG 2.5.8); comfort prefer ~44×44 on mobile |
| Labels | Icon-only without accessible name; placeholder-as-label |
| Forms | No inline validation strategy; destructive submit without confirm; unclear required fields |
| Feedback | Actions with no pending/success/error within ~400ms perceived response (Doherty) |
| Focus | Invisible focus; focus lost in modals; keyboard trap |
| Defaults | Risky defaults; empty required with no guidance |
| Patterns | Custom widgets reinventing tabs/menus/dialogs without APG keyboard model |

## 3. Nielsen 10 heuristics

Map findings to H1–H10 (NN/g):

1. **Visibility of system status** — progress, sync, save, selection
2. **Match real world** — user language, natural order, not internal jargon
3. **User control & freedom** — cancel, undo, exit nested flows
4. **Consistency & standards** — internal + platform conventions (Jakob’s Law)
5. **Error prevention** — constraints, confirms for irreversible
6. **Recognition rather than recall** — visible options; don’t force memory across steps
7. **Flexibility & efficiency** — shortcuts for experts without harming novices
8. **Aesthetic & minimalist design** — remove competing noise
9. **Recognize, diagnose, recover from errors** — plain language + fix path
10. **Help & documentation** — contextual, task-focused when needed

## 4. Laws of UX (high-signal)

Use when explaining *why* friction exists ([lawsofux.com](https://lawsofux.com)):

| Law | Audit lens |
|-----|------------|
| Fitts’s | Primary actions large enough and near related content |
| Hick’s | Choice overload on menus, CTAs, settings; progressive disclosure |
| Miller’s / Chunking | Long undifferentiated lists; break into groups |
| Jakob’s | Novel navigation/controls without benefit |
| Doherty | Spinners forever; no optimistic/skeleton feedback |
| Aesthetic-Usability | Ugly/inconsistent chrome undermines trust even if “functional” |
| Postel’s | Forms reject valid input harshly; poor parsing of user input |
| Tesler’s | Complexity dumped on user that belongs in the system |
| Peak-End | Error-only endings; no clear success completion |
| Von Restorff | Nothing stands out as primary CTA — or everything does |
| Proximity / Similarity / Common region | Related controls not grouped; unrelated grouped |

## 5. Accessibility (WCAG 2.2 AA floor)

Treat AA failures as **P0/P1**. Fast high-yield checks (~80% of real fails):

- Text contrast ≥4.5:1 (large text ≥3:1); UI components/graphics ≥3:1 (1.4.3, 1.4.11)
- Semantics / headings / labels (1.3.1, 2.4.6, 3.3.2)
- Keyboard operable + no trap (2.1.1, 2.1.2)
- Visible focus; focus not obscured (2.4.7, 2.4.11, 2.4.12)
- Target size ≥24×24 (2.5.8); dragging has alternative (2.5.7)
- Errors in text, not color alone (3.3.1, 1.4.1)
- Name, Role, Value for custom controls (4.1.2); status messages (4.1.3)
- Consistent help/nav where applicable (3.2.6); redundant entry (3.3.7); accessible auth (3.3.8)
- Custom widgets follow [WAI-ARIA APG](https://www.w3.org/WAI/ARIA/apg/) patterns

Native apps: also check platform a11y (VoiceOver/TalkBack/UI Automation) equivalents.

## 6. UI states completeness

For each critical screen/component, require:

`default · hover/focus · active · disabled · loading · empty · error · success · partial/permission-denied`

Missing empty/error/loading on primary flows → at least **P1**.

## 7. Content & IA

- One job per view/section; clear page title
- Primary action obvious; destructive actions separated
- Navigation depth matches mental model; wayfinding (“you are here”)
- Microcopy: verbs for actions, nouns for objects; no blameful errors

## 8. Ethics / deceptive patterns

Flag as **P0** if present (deceptivepatterns / darkpatterns literature):

- Forced continuity, sneak-into-basket, hidden costs
- Confirmshaming, obstruction, roach motel (easy in / hard out)
- Fake urgency/scarcity, misdirection, disguised ads
- Privacy zuckering / default sharing overreach

Legitimate persuasion ≠ deception; cite the specific pattern name.
