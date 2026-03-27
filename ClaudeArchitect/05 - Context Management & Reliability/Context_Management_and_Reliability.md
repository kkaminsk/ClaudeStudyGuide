# Domain 5: Context Management & Reliability (15%)

## Executive Summary

Context management is the discipline of deciding what Claude should see now, what should be summarized, what should be cached, and what should be moved out of the active window entirely. Even with 1M-context variants of Claude Opus 4.6 and Sonnet 4.6, context is still a constrained resource: more tokens increase cost and latency, and overly long conversations can reduce focus ("context rot").

Claude gives you different levers at different layers:

- **Claude Code**: `CLAUDE.md`, auto memory, rules, skills, subagents, `/context`, `/memory`, `/compact`, session resume, and model selection such as `opus[1m]`
- **Messages API**: prompt caching, token counting, server-side compaction, context editing, adaptive / extended thinking
- **Reliability controls**: retries, budgets, max turns, permission modes, auto mode, human escalation, and observability

The exam-relevant idea is not "keep as much history as possible." It is "keep the most relevant history available at the lowest safe cost."

## Context as a Budget, Not a Dumping Ground

Long context is powerful, but it is not free.

- Larger prompts cost more and take longer.
- Irrelevant history competes with the information Claude actually needs.
- Tool-heavy sessions can bloat quickly because files, search results, and command output all consume context.

Current Anthropic docs show:

- **Claude Opus 4.6**: up to 1M tokens of context
- **Claude Sonnet 4.6**: up to 1M tokens of context
- **Claude Haiku 4.5**: 200k tokens of context

In Claude Code, 1M context is exposed through model options such as `opus[1m]` and `sonnet[1m]` when your plan or provider supports them. Use long context when needed, but still curate aggressively.

## Claude Code Tactics for Context Management

Claude Code loads more than just the visible conversation. According to the product docs, startup context can include:

- `CLAUDE.md`
- auto memory
- MCP tool names
- skill descriptions
- appended system prompt content

As Claude works, file reads, matching rules, tool results, and summaries also enter context.

### Commands and features that matter most

| Tool or command | What it helps with | Why it matters |
|---|---|---|
| `/context` | Live breakdown of current context usage | Helps you see where tokens are going |
| `/memory` | Shows loaded `CLAUDE.md` and memory | Debugs instruction loading |
| `/compact` | Replaces long conversation history with a summary | Keeps sessions usable over time |
| `claude -c` | Continue most recent session in current directory | Preserves prior context intentionally |
| `claude -r <session>` | Resume a specific session | Useful for targeted follow-up work |
| `/model opus[1m]` or `/model sonnet[1m]` | Switch to extended context variants | Helps on very large repos or documents |
| `--append-system-prompt` | Add targeted instructions at startup | Better than retyping operational guidance |

### Claude Code best practices

- Keep each `CLAUDE.md` focused and under roughly 200 lines.
- Move specialized guidance into `.claude/rules/` and path-scope it when possible.
- Use subagents for exploration so large reads stay out of the main conversation.
- Use `/compact` when a session is getting noisy, rather than letting it rot.
- Prefer the smallest model and context window that safely fits the task.

## API Tactics for Context Management

When you build directly on the Messages API, context control becomes part of your application design.

### Prompt caching

Prompt caching reduces latency and cost for repeated prefixes. Anthropic supports two patterns:

- **Automatic caching**: add a top-level `cache_control` field
- **Explicit caching**: place `cache_control` on specific cacheable blocks

Important points:

- Cached prefixes cover `tools`, then `system`, then `messages`.
- Default TTL is **5 minutes**.
- Anthropic also supports a **1-hour** cache duration.
- Caching works best when the cached prefix is genuinely stable across requests.

Bad pattern: place the cache breakpoint on a block that changes every request, such as one with a timestamp or current user message.

### Token counting

Use the token-counting endpoint before large requests when you need to estimate context size, cost exposure, or compaction thresholds. This is one of the safest ways to avoid accidental context blowups.

### Server-side compaction

Anthropic now documents **server-side compaction** as the recommended strategy for long-running API conversations. It is currently a beta feature controlled through `context_management.edits` with the `compact_20260112` strategy.

Core behavior:

1. The API notices that input tokens crossed a trigger threshold.
2. Claude summarizes older context.
3. The response includes a `compaction` block.
4. On the next request, you pass that block back so the API can ignore earlier content.

Compaction is currently supported on Claude Opus 4.6 and Sonnet 4.6.

### Context editing

Context editing is a separate beta feature that clears specific content instead of summarizing everything. Current documented strategies include:

- **Tool result clearing**: remove old tool results when they no longer help reasoning
- **Thinking block clearing**: decide how much prior thinking to preserve

Anthropic explicitly recommends server-side compaction as the default choice for most long-running conversations, and context editing as the more surgical option.

### Adaptive and extended thinking

For Claude Opus 4.6 and Sonnet 4.6, Anthropic recommends **adaptive thinking** plus the `effort` parameter over the older manual thinking mode. Manual `thinking: {"type": "enabled", "budget_tokens": N}` still exists for compatibility, but it is being deprecated on those models.

This matters for reliability because reasoning depth and context usage interact: more thinking can improve hard decisions, but it also changes cost, latency, and cache behavior.

## Reliability Patterns

Reliability is about predictable completion under real-world failure modes: overloads, malformed input, over-budget loops, side effects, and unsafe actions.

### Retry strategy by error class

| Condition | Meaning | Recommended action |
|---|---|---|
| 429 | Rate limited | Respect `retry-after`, back off with jitter, reduce concurrency |
| 529 | Service overloaded | Retry with backoff and jitter; consider fallback model or delayed queueing |
| 500 / 502 / 503 | Transient server-side failure | Retry cautiously with limits |
| 400 / 401 / 403 | Request, auth, or policy problem | Fix the request; do not blindly retry |

429 and 529 are not the same. The first is a rate-limit signal; the second indicates overload.

### Budgets and stop conditions

You should define budgets outside the model whenever possible.

In Claude Code print mode, the CLI exposes:

- `--max-turns`
- `--max-budget-usd`
- `--fallback-model` (documented for print mode only)

Example:

```bash
claude -p --model opus --fallback-model sonnet --max-turns 8 --max-budget-usd 5 \
  "Review this repository for security issues and summarize the top risks."
```

In the Agent SDK, the parallel ideas are `max_turns` / `maxTurns` and `max_budget_usd` / `maxBudgetUsd`.

### Permission modes

Claude Code's documented permission modes are:

- `default`
- `acceptEdits`
- `plan`
- `auto`
- `bypassPermissions`
- `dontAsk`

`plan` is especially important for reliability because it lets Claude research and propose changes before it edits anything. `auto` trades more autonomy for a classifier-backed safety layer. `bypassPermissions` is only appropriate in isolated environments you fully control.

### Auto mode

Auto mode is available only in specific plans and only on Claude Sonnet 4.6 and Opus 4.6. Anthropic's current product docs also note that it is not generally available across third-party provider surfaces such as Bedrock, Vertex, or Foundry. The classifier evaluates actions before they run and is especially relevant for shell commands, network actions, and subagent delegation. This is a product capability, not just a prompting trick.

### Human-in-the-loop patterns

Route work to humans when:

- the action has high external side effects
- the model is about to touch production systems or sensitive data
- the output remains ambiguous after retries or validation
- policy requires approval regardless of model confidence

Practical examples include PR review before merge, approval before deployment, or human validation of extracted records that drive financial or legal workflows.

## Monitoring and Observability

### In Claude Code

Claude Code gives you several first-party observability tools:

- `--debug` for detailed logging
- `--verbose` for turn-by-turn output
- hooks for policy checks, audit logs, and custom telemetry
- session transcripts and resume metadata on disk
- OpenTelemetry-related environment variables and settings for org-level monitoring

### In the API and SDKs

On the API side, monitor:

- request IDs
- input and output tokens
- cache reads and writes
- compaction iteration usage
- structured output failures
- tool latencies and error rates

Anthropic also provides usage and cost APIs that are useful for budgeting and trend analysis.

## Common Exam Traps

- Long context helps, but it does **not** remove the need for curation.
- `plan` is a real permission mode; `delegate` is not.
- `--fallback-model` is documented for Claude Code print mode, not as a universal behavior across all surfaces.
- Prompt caching, compaction, and context editing are different tools; do not collapse them into one idea.
- For Claude 4.6, adaptive thinking is the preferred mental model, not the older `thinking=true` style found in stale examples.

## Final Takeaways

Use this mental checklist:

1. Can I remove irrelevant context before I add more?
2. Can I cache or summarize this instead of replaying it raw?
3. Do I need a larger context model, or just a cleaner prompt?
4. What are my budget, retry, and approval boundaries?
5. What telemetry will tell me when this workflow is drifting?

That mindset is the core of both context management and reliability on Claude.
