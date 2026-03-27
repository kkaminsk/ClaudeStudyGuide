# Domain 1: Agentic Architecture & Orchestration — Answer Key

---

**1. Answer: B** — The `tool_use_id` from the original `tool_use` block

**Rationale:** Claude's tool-use contract requires that every `tool_result` block references the original `tool_use_id`. This is the stable join key between Claude's intent and your execution layer. Without it, Claude cannot match the result to the request it made. The study guide explicitly states: "tool results must reference it via `tool_use_id`."

---

**2. Answer: B** — The Agent SDK manages the loop, sessions, and permissions; the Messages API is stateless and requires you to build those yourself

**Rationale:** The Messages API is stateless — every call must include the full conversation history, and session management is an application-level concern. The Agent SDK packages a production-grade agent loop with built-in tooling, permissions, hooks, subagents, and on-disk sessions. The core design choice is where the loop and session live: in your application (Messages API) or inside the SDK.

---

**3. Answer: B** — The agent returns a `ResultMessage` with an `error_max_turns` subtype

**Rationale:** The Agent SDK provides explicit guardrails: `max_turns` caps tool-use round trips and `max_budget_usd` caps spend. Hitting either yields a `ResultMessage` with error subtypes like `error_max_turns` or `error_max_budget_usd`. Whichever limit is hit first triggers the stop — they do not both need to be exhausted.

---

**4. Answer: C** — Adding extra text blocks immediately after `tool_result` blocks in the same user message

**Rationale:** Anthropic documents this as a specific pitfall: adding text blocks immediately after tool results can train Claude to prematurely end turns, producing empty responses with `stop_reason: "end_turn"`. The study guide recommends sending tool results cleanly without extra chatter.

---

**5. Answer: B** — They receive a fresh context with only the Agent tool's prompt string; only the subagent's final message returns to the parent

**Rationale:** Subagents are context-isolated agent instances. A subagent's conversation is fresh (no parent history); only the Agent tool's prompt string bridges parent to subagent, and only the final subagent message returns to the parent. This is specifically valuable because intermediate tool calls and results stay within the subagent, controlling context growth.

---

**6. Answer: C** — Master-worker pattern where a central agent decomposes tasks and delegates to specialized workers

**Rationale:** Anthropic's cookbooks demonstrate the orchestrator-workers workflow where a central LLM decomposes tasks and delegates subtasks to specialized workers. This maps directly to the master-worker pattern in the comparison table and is directly supported by Agent SDK subagents.

---

**7. Answer: A** — Continue finds the most recent session; resume targets a specific session by ID; fork copies history and diverges

**Rationale:** The study guide defines these precisely: Continue finds the most recent session in the current directory. Resume resumes a specific session by ID, enabling multi-user apps where each user maps to a session ID. Fork creates a new session with a copy of history but diverges from that point. Importantly, forking history does not branch filesystem changes.

---

**8. Answer: B** — Tool calls can have side effects, and retries or replays should not re-execute operations that have already completed

**Rationale:** Because tool calls can have side effects (DB writes, ticket creation), you should deduplicate by `tool_use.id` and record the executed result so retries can safely replay without re-executing. The Claude tool-use contract provides the stable identifier (`tool_use.id`) needed for this deduplication.

---

**9. Answer: C** — Omit the `Agent` tool from the subagent's tool list to prevent unintended recursion

**Rationale:** The study guide states: "In practice, subagents do not recurse unless you explicitly give them the `Agent` tool. The safest default is to omit `Agent` from a subagent's tool list." This prevents uncontrolled recursion while still allowing the subagent to use other tools it needs.

---

**10. Answer: B** — Stable prefixes like system prompts, tool definitions, and CLAUDE.md content are cached, reducing cost and latency for repeated prefixes across turns

**Rationale:** The Agent SDK relies on prompt caching for stable prefixes (system prompt, tool definitions, CLAUDE.md), reducing cost and latency for repeated prefixes. The study guide notes this as a key performance optimization that preserves loop semantics, and recommends placing stable content early to maximize cache hits.
