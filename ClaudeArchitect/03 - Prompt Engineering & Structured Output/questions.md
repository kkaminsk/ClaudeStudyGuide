# Domain 3: Prompt Engineering & Structured Output — Practice Questions

**Instructions:** Choose the single best answer for each question.

---

**1.** In the Claude Messages API, where is the system prompt specified?

A) As a message with `role: "system"` in the `messages` array
B) As a top-level `system` parameter
C) In the `output_config.system` field
D) Embedded in the first user message

---

**2.** What is the correct way to configure JSON structured outputs in the current Messages API?

A) Set `response_format: "json"` on the request
B) Use `output_config.format` with a `json_schema` type and schema definition
C) Add `format: "json"` to the system prompt
D) Use `structured_output: true` as a top-level parameter

---

**3.** What is the difference between JSON outputs and strict tool use?

A) They are the same feature with different names
B) JSON outputs constrain Claude's final answer text; strict tool use constrains tool argument schemas
C) JSON outputs work with tools; strict tool use works with text responses
D) Strict tool use is deprecated in favor of JSON outputs

---

**4.** Which delimiter style does Anthropic generally recommend for separating sections within a Claude prompt?

A) Markdown headers (`## Section`)
B) Triple backticks
C) XML-style tags (e.g., `<instructions>`, `<examples>`)
D) JSON objects

---

**5.** When using few-shot prompting, what happens if the model understands the task but formats the output incorrectly?

A) Increase the thinking budget
B) Add more representative examples that demonstrate the correct format
C) Switch to a larger model
D) Remove all examples and rely on instructions alone

---

**6.** What is the documented limitation of JSON structured outputs regarding message prefilling?

A) Prefilling is required for JSON outputs to work
B) JSON outputs are incompatible with message prefilling
C) Prefilling improves JSON output quality
D) There is no interaction between the two features

---

**7.** In a data extraction pipeline, when should you use Claude with a schema versus a deterministic parser (regex, native JSON/XML parser)?

A) Always use Claude — it handles all formats better
B) Use deterministic parsers for stable, structured sources; use Claude for messy documents requiring contextual understanding
C) Always use regex first, then Claude as a fallback
D) Use Claude only when the document is in PDF format

---

**8.** What is the recommended approach for reasoning on Claude Opus 4.6 and Sonnet 4.6?

A) Set `thinking=true` on the request
B) Use adaptive thinking with the `effort` parameter
C) Add "think step by step" to the prompt
D) Enable `reasoning_mode: "extended"` in settings

---

**9.** When Claude refuses a structured output request (`stop_reason: "refusal"`), what should your application expect?

A) The output will still match the schema perfectly
B) The output payload may not match the schema
C) Claude will automatically retry with a modified prompt
D) The response will be empty with no content blocks

---

**10.** What is a key property ordering behavior when using structured outputs with both required and optional fields?

A) Fields appear in random order
B) Optional fields appear first, then required fields
C) Required properties appear first, then optional properties, each group preserving schema order
D) Fields are sorted alphabetically regardless of required/optional status
