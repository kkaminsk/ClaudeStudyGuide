# Domain 5: Context Management & Reliability — Practice Questions

**Instructions:** Choose the single best answer for each question.

---

**1.** What is "context rot" in the context of Claude sessions?

A) A bug that causes Claude to lose access to tools
B) The degradation of accuracy and focus as irrelevant context accumulates in long conversations
C) A file corruption issue with session storage
D) The expiration of cached prompts after 5 minutes

---

**2.** What is the default TTL for prompt caching in Claude's API?

A) 1 minute
B) 5 minutes
C) 30 minutes
D) 24 hours

---

**3.** What is the correct order of prefix coverage for prompt caching?

A) `messages`, then `system`, then `tools`
B) `system`, then `messages`, then `tools`
C) `tools`, then `system`, then `messages`
D) All three are cached independently with no ordering

---

**4.** How does server-side compaction work in the Messages API?

A) It deletes old messages from storage
B) It summarizes older context when tokens cross a threshold, returns a `compaction` block, and expects you to pass it back on subsequent requests
C) It compresses the entire conversation into a single system prompt
D) It automatically truncates messages from the beginning

---

**5.** What is the difference between server-side compaction and context editing?

A) They are the same feature with different API names
B) Compaction summarizes everything broadly; context editing surgically clears specific content like old tool results or thinking blocks
C) Context editing is for system prompts; compaction is for user messages
D) Compaction is client-side; context editing is server-side

---

**6.** What is the correct response when Claude Code receives an HTTP 429 error from the API?

A) Immediately retry the request
B) Respect the `retry-after` header, back off with jitter, and reduce concurrency
C) Switch to a different model
D) Restart the session from scratch

---

**7.** Which Claude Code permission mode allows Claude to research and propose changes without executing them?

A) `default`
B) `auto`
C) `plan`
D) `dontAsk`

---

**8.** In Claude Code print mode, which flag specifies a model to fall back to if the primary model is unavailable?

A) `--backup-model`
B) `--secondary-model`
C) `--fallback-model`
D) `--alt-model`

---

**9.** When should work be routed to a human reviewer in a human-in-the-loop pattern?

A) Only when the model explicitly requests human help
B) When the action has high external side effects, involves sensitive data, output is ambiguous after retries, or policy requires approval
C) After every 10th model response
D) Only when the model returns an error

---

**10.** What is the key difference between HTTP 429 and HTTP 529 errors from Claude's API?

A) They are identical in meaning and handling
B) 429 is a rate-limit signal; 529 indicates service overload — different causes requiring different handling strategies
C) 429 means authentication failure; 529 means rate limiting
D) 429 is deprecated; 529 has replaced it
