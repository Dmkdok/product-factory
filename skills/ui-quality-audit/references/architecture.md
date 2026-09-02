# Front-end architecture fitness (UI-facing)

TOC: [Scope](#scope) · [Structure](#1-structure) · [Components](#2-components--design-system) · [State](#3-state) · [Data-UI](#4-data--ui-boundary) · [Errors](#5-errors--resilience) · [Perf UX](#6-performance-as-ux) · [Framework notes](#7-framework-notes)

Audit whether the *implementation* can sustain a high-quality UI. Cite concrete modules. Prefer principles from **alan2207/bulletproof-react** (~35k★), feature-based boundaries, SOLID at component scale — adapted to the stack in use (React/Vue/Svelte/Angular/SwiftUI/Compose/WinUI/etc.).

## Scope

Only architecture that **hurts UI quality, consistency, or change-safety**. Do not demand Clean Architecture cosplay on a 3-page brochure site.

## 1. Structure

| Expect | Smell |
|--------|-------|
| Feature/module folders for product areas; shared primitives isolated | Flat `components/` dumping ground; cross-imports between features’ internals |
| Unidirectional dependency: shared → features → app | Features import each other’s private files |
| Public API per feature (`index` exports) when scale warrants | Deep relative imports everywhere |
| Routes/screens map clearly to modules | “God” `App` / `MainWindow` owns all UI |

**Target:** colocate UI + styles + tests per feature; promote only stable primitives to shared design-system layer.

## 2. Components & design system

| Expect | Smell |
|--------|-------|
| Buttons, inputs, dialogs, toasts from one primitive set | 5 visual button styles with no shared API |
| Variants via props/tokens, not copy-paste | Duplicated CSS with tiny deltas |
| Presentational vs container/hooks separation where logic is heavy | 800-line components fetching + laying out + validating |
| Composition (slots/children/compound) over prop explosion | 20+ boolean props |
| Styles via tokens (color, space, type, radius, elevation) | Magic numbers; hard-coded hex in dozens of files |

**SOLID lens (components):**

- **S** — one reason to change per component/hook
- **O** — extend via composition/variants, not edit cores for each screen
- **I** — don’t force unused props/context
- **D** — UI depends on hooks/services abstractions, not raw fetch sprinkled in JSX/XML

## 3. State

Classify and verify boundaries (bulletproof-react model):

| Kind | Belongs in | Smell |
|------|------------|-------|
| Local UI | Component state | Global store for “isTooltipOpen” |
| Cross-screen UI | Lightweight global store | Server data mirrored manually into Redux/Zustand without need |
| Server/cache | Query library / platform equivalent | ad-hoc `useEffect` fetch forests, no cache/error policy |
| URL | Router search/params | Critical filters only in ephemeral memory |

**Target:** single policy for async status (loading/error) so UI states stay consistent.

## 4. Data ↔ UI boundary

| Expect | Smell |
|--------|-------|
| Typed API client + validation at boundary | Screens trust raw JSON shapes |
| Mapping DTO → view-model | UI binds to backend field names/jargon |
| Mutations with explicit success/error UX | Silent failures; partial updates |

## 5. Errors & resilience

| Expect | Smell |
|--------|-------|
| Error boundaries / route-level error UI | White screen on throw |
| User-recoverable messages + retry | Raw stack traces / empty toasts |
| Form errors tied to fields | Only banner at top with no focus move |

## 6. Performance as UX

| Expect | Smell |
|--------|-------|
| Route/screen-level code split for large apps | Mega-bundle blocking first interaction |
| Lists virtualized when long | DOM of 5k rows |
| Images sized/lazy; CLS avoided | Layout jumps on load |
| Stable skeletons matching layout | Random spinners shifting content |

Tie to Doherty Threshold and perceived performance — not micro-benchmark theater.

## 7. Framework notes

- **React/Next:** feature folders; avoid prop-drilling jungles; server/client component boundaries shouldn’t break a11y (labels, ids).
- **Vue/Nuxt / SvelteKit:** same feature discipline; stores don’t become kitchens.
- **Mobile (SwiftUI/Compose/RN/Flutter):** use platform navigation patterns; respect safe areas; one component kit.
- **Desktop (WPF/WinUI/Qt/Electron):** command/menu parity; keyboard first; density appropriate to desktop, not blown-up mobile.

If stack is unknown, infer from manifests and still apply structure/state/component smells generically.
