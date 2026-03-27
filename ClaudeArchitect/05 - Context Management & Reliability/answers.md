# Domain 5: Context Management & Reliability — Answer Key

---

**1. Answer: B** — The degradation of accuracy and focus as irrelevant context accumulates in long conversations

**Rationale:** The study guide states that even with 1M-context variants, context is still a constrained resource. More tokens increase cost and latency, and overly long conversations can reduce focus — this is what "context rot" describes. The exam-relevant principle is: keep the most relevant history available at the lowest safe cost, not keep as much history as possible.

---

**2. Answer: B** — 5 minutes

**Rationale:** The default TTL for prompt caching is 5 minutes. Anthropic also supports a 1-hour cache duration. Caching works best when the cached prefix is genuinely stable across requests. Placing cache breakpoints on blocks that change every request (like timestamps) defeats the purpose.

---

**3. Answer: C** — `tools`, then `system`, then `messages`

**Rationale:** Cached prefixes cover `tools`, then `system`, then `messages` in that order. This means tool definitions and system prompts benefit most from caching since they appear first in the prefix. Placing stable content early maximizes cache hits.

---

**4. Answer: B** — It summarizes older context when tokens cross a threshold, returns a `compaction` block, and expects you to pass it back on subsequent requests

**Rationale:** Server-side compaction works in four steps: (1) the API notices input tokens crossed a trigger threshold, (2) Claude summarizes older context, (3) the response includes a `compaction` block, (4) you pass that block back on the next request so the API can ignore earlier content. This is currently supported on Claude Opus 4.6 and Sonnet 4.6.

---

**5. Answer: B** — Compaction summarizes everything broadly; context editing surgically clears specific content like old tool results or thinking blocks

**Rationale:** These are separate beta features. Compaction summarizes and replaces older context holistically. Context editing provides fine-grained clearing strategies (tool-result clearing, thinking-block clearing). Anthropic recommends server-side compaction as the default for most long-running conversations and context editing as the more surgical option.

---

**6. Answer: B** — Respect the `retry-after` header, back off with jitter, and reduce concurrency

**Rationale:** HTTP 429 is a rate-limit signal. The recommended action is to respect the `retry-after` header, back off with jitter, and reduce concurrency. Immediately retrying without backoff will make the problem worse and may extend the rate limiting period.

---

**7. Answer: C** — `plan`

**Rationale:** The `plan` permission mode lets Claude research and propose changes before editing anything. This is especially important for reliability because it enables review before execution. `auto` provides more autonomy with a classifier-backed safety layer. `bypassPermissions` is only appropriate in fully isolated environments.

---

**8. Answer: C** — `--fallback-model`

**Rationale:** In Claude Code print mode, `--fallback-model` specifies a model to fall back to if the primary model is unavailable. The study guide notes this is documented for print mode specifically, not as a universal behavior across all surfaces — a distinction the exam may test.

---

**9. Answer: B** — When the action has high external side effects, involves sensitive data, output is ambiguous after retries, or policy requires approval

**Rationale:** The study guide lists four conditions for routing to humans: (1) high external side effects, (2) touching production systems or sensitive data, (3) output remains ambiguous after retries or validation, (4) policy requires approval regardless of model confidence. Practical examples include PR review, deployment approval, and validation of financial/legal records.

---

**10. Answer: B** — 429 is a rate-limit signal; 529 indicates service overload — different causes requiring different handling strategies

**Rationale:** The study guide explicitly states: "429 and 529 are not the same. The first is a rate-limit signal; the second indicates overload." For 429, respect `retry-after` and reduce concurrency. For 529, retry with backoff and jitter, and consider a fallback model or delayed queueing. Conflating these two errors leads to incorrect retry strategies.
