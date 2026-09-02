---
name: discover-requirements
description: >-
  Relentless product discovery interview until shared understanding is reached.
  Builds docs/BRIEF.md before any coding. Use during discovery, requirements
  gathering, or orchestrate-product Phase 1. Speaks Russian to the user; writes
  English brief artifacts. Grill-me style: recommended answers, design-tree walk.
---

# Discover Requirements

## Goal

Turn a vague request into `docs/BRIEF.md` that meets Definition of Ready — **before** architecture theater or code.

## Language

- Chat with user: **Russian**
- `docs/BRIEF.md`: **English** (optional short `docs/BRIEF.ru.md`)

## Method (grill until shared understanding)

1. Skim existing `docs/` and repo; **do not ask what the codebase already answers**.
2. Walk the design tree branch-by-branch (problem → users → scope → constraints → quality). Resolve dependencies between decisions one-by-one.
3. Ask **one batch** of 5–8 questions (see orchestrate-product `references/question-banks.md`).
4. For each question: give your **recommended answer** (marked) so the user can accept/edit fast — do not strand them with open-ended void.
5. Reflect a short Russian summary of decisions; confirm misunderstandings.
6. Next batch only for gaps / contradictions.
7. When DoR is met, write `docs/BRIEF.md` from the template and stop for spec phase.

### Soft stop test

You may stop eliciting only when a senior engineer could build v1 **without guessing** product intent. If you are still inventing brand, scope, or users — keep grilling.

## Rules

- Do not propose full architecture yet — only capture constraints that affect elicitation.
- Do not write application code.
- Challenge contradictions politely (e.g. "MVP in 2 days" + "custom auth + payments + admin").
- Prefer links and examples over abstract adjectives ("modern", "красивый").
- Prefer structured choices (A/B/C) when the space is known.

## BRIEF.md must include

- One-sentence pitch
- Problem / job-to-be-done
- Primary user
- Goals & success metrics
- In scope / out of scope
- Constraints & integrations
- Brand & content notes
- Acceptance bar (user's words)
- Open questions (if any remain — ideally empty)
