# Context & Token Optimization — Deep Dive

## Contents

- Tool-schema bloat: verified numbers
- Multi-agent cost: verified numbers and the compounding risk
- Lost in the middle: positional bias in long contexts
- TOON — real savings, real caveat
- Other OSS references
- Provider/infra techniques (building your own Claude-backed system)
- Sources

## Tool-schema bloat: verified numbers

Every non-deferred tool's name + description + schema is resent every turn. Anthropic's own
numbers (docs.claude.com, Tool Search Tool page, 2026):

- A typical 5-server setup (GitHub, Slack, Sentry, Grafana, Splunk) consumes **~55k tokens** in
  tool definitions before Claude does any work.
- Tool search (`defer_loading: true` on the tools that don't need to be present up front) **cuts
  that by over 85%**, loading only the 3–5 tools a given request actually needs.
- Tool-selection accuracy degrades once a toolset **exceeds 30–50 tools**, independent of the token
  cost — a second, separate reason to keep an agent's available tools narrow.
- Anthropic's own guidance on when it's worth setting up: 10+ tools, tool defs over ~10k tokens,
  selection accuracy already dropping, or aggregating 200+ tools across multiple MCP servers.
  Below that (fewer than 10 tools, all used every request, defs under ~100 tokens each), plain tool
  definitions are simpler and just as cheap.
- This harness already runs the deferred-tool pattern live: tools like `WebFetch`, `WebSearch`,
  `EnterPlanMode`, `mcp__serena__*` etc. are named but not loaded until `ToolSearch` fetches them;
  deferred tools are excluded from the cached prefix, so the pattern doesn't break prompt caching
  either.

## Multi-agent cost: verified numbers and the compounding risk

Anthropic's own multi-agent research architecture (the "Research" feature): a lead agent plans and
spins up 3–5 subagents in parallel, each with its own context window, then synthesizes.

- Beat single-agent Opus by 90.2% on Anthropic's internal research eval, at roughly **15x the
  token cost** of a normal single-agent turn.
- The 15x figure is a *baseline*, not a ceiling: a subagent that recursively spawns further
  subagents, or a tool call that returns an oversized result, can multiply a single run's cost by
  another order of magnitude. The published architecture has no built-in circuit breaker or
  per-run cap — that has to come from how you scope and bound the delegation yourself.
- Practical implication: cap fan-out width explicitly, and treat a subagent returning far more than
  its stated budget as a sign to re-scope its task, not as something to summarize after the fact
  once it's already in context.

## Lost in the middle: positional bias in long contexts

Liu et al., "Lost in the Middle: How Language Models Use Long Contexts" (TACL 2024) — one of the
most-cited findings in the long-context literature: recall of a fact follows a U-shape over its
position in the context, strongest at the start and end, weakest in the middle, by as much as 20+
percentage points on some benchmarks.

- Still measurably present in 2026 on large-context models, including on windows far below their
  advertised maximum — a bigger context window doesn't retire the effect.
- Practical fix isn't "make the context smaller" (that's the rest of this skill) but *governed
  placement*: fewer, higher-signal context objects, ordered deliberately, with the load-bearing
  instruction or fact placed at an edge rather than mid-document. Relevant every time you assemble a
  long tool result, a big subagent handoff, or a long synthesis for the user.

## TOON — real savings, real caveat

TOON (Token-Oriented Object Notation, `github.com/toon-format/toon`) is a compact encoding of the
JSON data model: YAML-like indentation for nested objects, CSV-style rows for uniform arrays.
Widely adopted with official and community implementations across more than a dozen languages.

- Measured savings: roughly 30–60% fewer tokens than JSON for uniform, tabular arrays of objects;
  the win shrinks fast as data gets less uniform (small or deeply irregular payloads aren't worth
  converting).
- **The caveat the "use TOON everywhere" pitch skips:** some independent benchmarks report a small
  *accuracy* cost alongside the token savings on certain retrieval/QA tasks over the encoded data —
  the format is less represented in training data than JSON, so a model's ability to parse it back
  precisely isn't automatically equal to JSON's. Benchmark on the actual task before adopting TOON
  for a pipeline where a misread field is costly; don't take the token-savings number as proof the
  output quality is unaffected.
- Use at the prompt boundary only (encoding data going *into* a prompt); keep JSON in application
  code and for anything downstream that parses the data programmatically.

## Other OSS references

- **LLMLingua** (`github.com/microsoft/LLMLingua`) — Microsoft's prompt-compression research tool,
  claims up to 20x compression via coarse-to-fine token pruning. Relevant when a fixed prompt or
  RAG context is a recurring, measurable cost in a system you operate — not a fit for one-off
  interactive work.
- **code2prompt** (`github.com/mufeedvh/code2prompt`) — turns a codebase into an LLM prompt with
  token counting; useful for scoping how much of a repo a task is about to load before doing it by
  hand.
- **awesome-context-engineering** / **awesome-llm-token-optimization** (GitHub) — curated,
  actively maintained lists of the above plus adjacent tools; check these before reaching for a new
  dependency, since the landscape moves fast and specific tools/numbers here can go stale.

## Provider/infra techniques (building your own Claude-backed system)

Mostly irrelevant to a single interactive session here, but relevant when configuring a cron job, a
scheduled routine, or advising the user on their own LLM usage. Pull current mechanics/pricing from
skill `claude-api` rather than trusting numbers memorized here — these move.

- **Prompt caching mechanics** — static content (system prompt, tool schemas, few-shot examples)
  first, volatile content (the latest user turn, the latest tool result) last. Any edit to the
  static prefix busts the cache for everything after it.
- **Batch APIs for non-real-time work** — a flat discount in exchange for async (minutes-to-hours)
  turnaround. Good fit for bulk classification, overnight report generation, large-scale
  summarization that doesn't block a user.
- **Semantic/response caching** for repeated or near-duplicate queries (FAQ-style bots, recurring
  cron checks) — cache by embedding similarity rather than exact string match.
- **Cap and shape output tokens.** Set an explicit output budget appropriate to the task — a
  classification doesn't need a long completion. Uncontrolled output length is a common silent cost
  leak, separate from everything above, which is about input-side tokens.
- **Extended-thinking budgets are a token cost too.** When calling the API directly with extended
  thinking enabled, size the thinking-token budget to the task's actual difficulty — a large fixed
  budget applied to every call (including easy ones) is the output-token version of the same
  "uncontrolled length" leak, just on the reasoning side instead of the answer side.

## Sources

- Anthropic, [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- Anthropic, [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- Claude Docs, [Tool search tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool)
- Claude Docs, [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- Liu et al., [Lost in the Middle: How Language Models Use Long Contexts](https://arxiv.org/abs/2307.03172) (TACL 2024)
- [github.com/toon-format/toon](https://github.com/toon-format/toon)
- [github.com/microsoft/LLMLingua](https://github.com/microsoft/LLMLingua)
- [github.com/mufeedvh/code2prompt](https://github.com/mufeedvh/code2prompt)
