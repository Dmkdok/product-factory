---
name: context-token-optimization
description: >-
  Applies a lean, source-verified set of context/token-saving habits to any
  task: bounded reads instead of full-file dumps, batched independent tool
  calls, scratchpad persistence instead of a growing transcript, a disciplined
  tool-schema/subagent footprint, and an English-internal / user-language-chat
  split. Load by default for any real task, however small — skipping a
  trivial one-off exchange is itself the correct application of the rule, not
  an exception to it. For product-delivery pipeline work, subagent role/model
  tiering lives in orchestrate-product's delegation.md; this skill covers
  everything else, in or out of that pipeline.
metadata:
  author: product-factory
  version: "1.0.0"
  sources: See references/deep-dive.md for the full citation list.
---

# Context & Token Optimization

**Golden rule (Anthropic's context-engineering guidance, verbatim intent): write as little context
as required, and as much as necessary.** Cutting tokens is never the goal by itself — the goal is
the smallest set of high-signal tokens that reliably produces the correct result. A shorter context
that drops a decision-relevant fact is a regression, not an optimization. When unsure, keep the
information and trim the packaging instead.

## Core habits (apply by default, every task)

- **Bounded reads, not full dumps.** `Read` takes `offset`/`limit`. `Grep`/`Glob` take
  `output_mode`, `head_limit`, `-A`/`-B`. Reach for these before a whole-file or whole-log read.
  For source code with Serena active, `get_symbols_overview`/`find_symbol` supersede `Read` on a
  module — see `delegation.md`'s reading protocol, don't re-derive it here.
- **Filter shell output before it lands in context.** Pipe through `grep`/`head`/`wc -l` etc.
  rather than printing a raw log and reasoning over it inline.
- **Batch independent tool calls in one turn.** Already a hard rule in this harness — it also
  avoids paying for extra round-trips of restated context.
- **Persist multi-step state to the scratchpad, not the transcript.** Collecting N items or
  iterating a long list: append to a scratchpad file and re-read it, instead of keeping a growing
  list alive in conversation history.
- **Don't `ToolSearch` a tool you won't call this turn.** A deferred tool's schema, once fetched,
  stays part of the session's context going forward — every fetch is a one-way addition, not a
  free look. Same principle when authoring a subagent: restrict its `Tools:` list to what its job
  needs.
- **Internal work in English, chat in the user's language.** Reasoning, scratchpad notes, and
  subagent-to-subagent handoffs in English tokenize denser and draw on deeper training coverage;
  the reply the user reads stays in their language. This is already codified per-skill here
  (`orchestrate-product`, `iterate-product`, `concise-mode`'s language policies) — this bullet names
  it as a token/quality lever in its own right, it doesn't replace those policies.

## Subagents and model tier — cross-reference, don't restate

`orchestrate-product/references/delegation.md` owns the role→model table, the context-budget-per-
subagent math, and "reporting back = summary only." `iterate-product` points at that file instead
of copying it — do the same rather than re-deriving the framework here.

- **Inside the product-delivery pipeline:** use `delegation.md`.
- **Outside it, the same two principles still apply:** subagent isolation costs **3–15x** the
  tokens of doing the work inline (Anthropic's multi-agent research architecture) — worth it only
  when a subtask pulls in >1000 tokens irrelevant to the parent once done, needs genuine
  parallelism, or needs a different model tier. Route mechanical/fan-out work to a cheap model,
  judgment calls to the strong one.
- **`Explore` does not default to a cheap model on its own** — pass `model: "haiku"` explicitly on
  the `Agent` call when the fan-out is mechanical; don't assume the agent type implies the tier.
- **Cap fan-out width and watch for oversized returns.** An uncapped subagent (one that spawns more
  subagents, or a tool call that returns an oversized result) can multiply cost by another order of
  magnitude on top of the 3–15x baseline — there's no automatic circuit breaker. A subagent
  reporting back far more than its own token budget is a signal to re-scope it, not to summarize
  after the fact.
- **This harness's `Agent` tool is stricter than the general heuristic either way:** don't spawn
  subagents proactively — only when the user explicitly asks, names an agent type, or a loaded
  skill's own instructions call for it.
- `/fast` mode is Opus with faster output, not a cheaper tier — don't route mechanical work to it
  expecting a cost win.

## When to start a fresh session

- The same correction has been given twice in this dialogue without it sticking — the context past
  that point is failed attempts, not useful history; a new session with a sharper prompt usually
  resolves it faster than a third try.
- The task has drifted to something unrelated to what the dialogue was originally about.
- A plan was just approved, or a milestone just closed — the deliberation that produced it is dead
  weight for the execution phase; the written plan/`docs/STATUS.md` carries what's needed forward.
- Reviewing code the same session just wrote — a fresh session has no stake in the code being fine.

Say so plainly and suggest `/clear` or a new session; don't silently push forward in a polluted
context on the assumption that raising it would itself cost tokens.

## Guardrails — never trade correctness for tokens

- Never summarize away a fact the final answer depends on. Over-inclusion is recoverable; silent
  omission usually isn't caught until it's wrong.
- Keep citations/sources attached to claims when compressing research findings.
- A compaction pass preserves the plan and open todos, not just "what happened so far" — losing the
  plan is the most common way long agentic runs go off the rails after a context trim.
- Verify after compressing: spot-check that the specific numbers/names/decisions the user will act
  on survived a rewrite or a subagent's summary intact.
- If a token-saving technique and correctness ever conflict, correctness wins.

## Further reading

Citations, verified tool-schema/multi-agent-cost figures, TOON/LLMLingua with their accuracy
caveats, and provider/infra techniques (batch APIs, semantic caching, output-token budgets) for
when you're building your own Claude-backed system: [references/deep-dive.md](references/deep-dive.md).
