# Domain 3: Prompt Engineering & Structured Output — Answer Key

---

**1. Answer: B** — As a top-level `system` parameter

**Rationale:** In the Messages API, the system prompt is a top-level `system` parameter, not a `role: "system"` message in the messages array. This is called out explicitly as one of the biggest exam traps — confusing the Messages API format with older or third-party API conventions.

---

**2. Answer: B** — Use `output_config.format` with a `json_schema` type and schema definition

**Rationale:** The current Messages API uses `output_config.format` for JSON outputs. The SDK helpers may expose convenience wrappers (`output_format` / `outputFormat`), but these translate to the same API shape. Structured outputs use grammar-constrained decoding to enforce schema conformance.

---

**3. Answer: B** — JSON outputs constrain Claude's final answer text; strict tool use constrains tool argument schemas

**Rationale:** These solve different problems. JSON outputs (via `output_config.format`) constrain what Claude says in its response. Strict tool use (via `strict: true` on a tool definition) constrains how Claude calls functions — specifically the tool name and argument schema. They can be used independently or together in the same workflow.

---

**4. Answer: C** — XML-style tags (e.g., `<instructions>`, `<examples>`)

**Rationale:** Anthropic generally recommends delimiting prompt sections with XML-style tags so the model can clearly separate instructions, examples, and live input. This is a core prompt engineering pattern for Claude — it reduces ambiguity about where one section ends and another begins.

---

**5. Answer: B** — Add more representative examples that demonstrate the correct format

**Rationale:** The study guide draws a clear distinction: few-shot prompting teaches format, policy, or style; adaptive/extended thinking gives the model more reasoning budget. If the model understands the task but formats it incorrectly, the solution is better examples. If it formats correctly but reasons poorly, increase effort or use thinking features.

---

**6. Answer: B** — JSON outputs are incompatible with message prefilling

**Rationale:** The study guide explicitly notes that JSON outputs are incompatible with both citations and message prefilling. This is listed as both a limitation and a common exam trap — prefilling and JSON structured outputs cannot be combined in the same request.

---

**7. Answer: B** — Use deterministic parsers for stable, structured sources; use Claude for messy documents requiring contextual understanding

**Rationale:** The recommended extraction pattern is: use deterministic parsing where you can, use Claude where context or normalization matters, then validate the result. Regex is best for stable string patterns; native parsers for structured formats; Claude with schemas for messy documents needing contextual extraction.

---

**8. Answer: B** — Use adaptive thinking with the `effort` parameter

**Rationale:** For Claude Opus 4.6 and Sonnet 4.6, Anthropic recommends adaptive thinking with the `effort` parameter over the older manual thinking mode. The `thinking=true` pattern is outdated and incorrect — this is explicitly flagged as one of the biggest exam traps.

---

**9. Answer: B** — The output payload may not match the schema

**Rationale:** If the model refuses (`stop_reason: "refusal"`) or is truncated (`stop_reason: "max_tokens"`), the payload may not match the schema. Applications must check the stop reason and handle these cases gracefully rather than assuming all responses will conform to the declared schema.

---

**10. Answer: C** — Required properties appear first, then optional properties, each group preserving schema order

**Rationale:** Property ordering in structured outputs is predictable but not arbitrary: required properties appear first, then optional properties, with each group preserving the order defined in the schema. This matters when downstream processing depends on field ordering in the JSON output.
