# Domain 3: Prompt Engineering & Structured Output (20%)

## Executive Summary

Prompt engineering on Claude is about reducing ambiguity. The most reliable pattern is: (1) give a clear task, (2) separate sections with XML tags, (3) provide representative examples, and (4) constrain the output when another system has to parse it.

For structured output, Anthropic exposes two related features:

- **JSON outputs** via `output_config.format` when you want Claude's final answer in schema-conformant JSON
- **Strict tool use** via `strict: true` when you want tool names and tool inputs to match schema exactly

These solve different problems. JSON outputs constrain what Claude says; strict tool use constrains how Claude calls functions. They can be used independently or together.

The biggest exam traps are outdated API shapes. In the Messages API, the system prompt is a top-level `system` parameter, not a `role: "system"` message. Claude 4.6 reasoning is configured with adaptive thinking / effort (or an explicit `thinking` object for older or manual mode), not `thinking=true`. And the direct response text is read from `response.content[0].text`, not from legacy `completion` fields.

## Structured Outputs and JSON Schema

Structured outputs are for cases where downstream code must parse Claude's answer safely.

### Two complementary features

| Feature | Best for | Where configured | What it constrains |
|---|---|---|---|
| JSON outputs | Final answer format | `output_config.format` | Claude's direct response |
| Strict tool use | Tool routing and tool inputs | `strict: true` on tool definitions | Tool name + tool argument schema |

JSON outputs are ideal for extraction, classification, and report generation. Strict tool use is ideal when Claude must call your functions with validated inputs.

### What current Anthropic docs say

- The Messages API uses `output_config.format` for JSON outputs.
- Python and TypeScript SDK helpers may still expose convenience wrappers such as `output_format` / `outputFormat`, then translate them into the API shape for you.
- Structured outputs use grammar-constrained decoding, so the first request for a new schema can be slower while the grammar is compiled.
- Compiled grammars are cached, which improves follow-up latency for the same schema.
- Refusals and `max_tokens` cutoffs are the main cases where you should still expect output not to match the schema perfectly.

### Practical schema guidance

Keep schemas simple and aligned to the task:

- Prefer a small object with clearly named fields over a deeply nested structure.
- Use `enum` for fixed categories.
- Use `additionalProperties: false` on objects you want to keep tight.
- Make fields optional if the source material may not contain them.
- Use descriptions to explain business meaning, not just type information.

Anthropic's SDK docs also note that some unsupported JSON Schema constraints may be stripped or translated when helpers like Pydantic or Zod are used. In those cases, the SDK validates against the original schema client-side even if the model receives a simplified version.

### Important limitations and caveats

- Structured outputs are **not** the same as tool use.
- JSON outputs are incompatible with citations and message prefilling.
- Property ordering is predictable but not arbitrary: required properties appear first, then optional properties, each preserving schema order inside its group.
- Overly complex schemas can fail compilation or cause retries.
- If the model refuses (`stop_reason: "refusal"`) or is truncated (`stop_reason: "max_tokens"`), the payload may not match the schema.

### Correct Messages API example

```python
import json
from anthropic import Anthropic

client = Anthropic()

schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "email": {"type": "string", "format": "email"},
        "phone": {
            "type": "string",
            "description": "US phone number in (555) 555-5555 format when explicitly present"
        }
    },
    "required": ["name", "email"],
    "additionalProperties": False,
}

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=512,
    system="Extract contact information from the user's message.",
    messages=[
        {
            "role": "user",
            "content": "Hi, I'm Charlie Kim. Email me at charlie.kim@domain.org."
        }
    ],
    output_config={
        "format": {
            "type": "json_schema",
            "schema": schema,
        }
    },
)

result = json.loads(response.content[0].text)
print(result)
```

Key corrections versus older examples:

- `system` is top-level.
- The final JSON comes from `response.content[0].text`.
- Optional fields should be omitted unless the schema explicitly allows `null` or empty strings.

### Strict tool use example

```json
{
  "name": "save_contact",
  "description": "Persist exactly one contact record after validation.",
  "input_schema": {
    "type": "object",
    "properties": {
      "name": {"type": "string"},
      "email": {"type": "string", "format": "email"}
    },
    "required": ["name", "email"],
    "additionalProperties": false
  },
  "strict": true
}
```

With `strict: true`, Claude's tool calls must match the declared schema exactly. This is the right mechanism when your application logic depends on reliable tool arguments, not just reliable final text.

## Few-Shot Prompting

Few-shot prompting means teaching by example inside the prompt. On Claude, examples often outperform extra prose because they show the desired structure, tone, and decision rule directly.

### Recommended prompt structure

Anthropic generally recommends delimiting prompt sections with XML-style tags so the model can separate instructions, examples, and live input.

```xml
<instructions>
Classify the sentiment and explain the label in one sentence.
</instructions>

<examples>
  <example>
    <input>The checkout flow is smooth and fast.</input>
    <output>{"label":"positive","reason":"The text praises the product experience."}</output>
  </example>
  <example>
    <input>I keep getting logged out and losing my work.</input>
    <output>{"label":"negative","reason":"The text describes a frustrating failure."}</output>
  </example>
</examples>

<input>
The dashboard is okay, but exporting reports still fails sometimes.
</input>
```

### Best practices

- Start with a few representative examples, then add more only when they improve a failure mode you can observe.
- Match the examples to the real task as closely as possible.
- Cover edge cases deliberately instead of hoping the model generalizes.
- Label negative or counterexamples clearly if you use them.
- Keep instructions short and direct; let examples carry most of the behavioral load.

There is no universal rule that "3-5 examples" is always optimal. A few good examples are a strong starting point, but more examples can help if they remain relevant and fit within your context budget.

### Negative examples are allowed

Negative examples can be useful when you label them clearly.

```xml
<example>
  <input>Translate to French: I am happy.</input>
  <good>Je suis heureux.</good>
  <bad>I'm happy.</bad>
</example>
```

The mistake to avoid is ambiguity: if the model cannot tell whether an example is desirable or undesirable, performance usually gets worse.

### Few-shot prompting vs extended thinking

These are different levers:

- **Few-shot prompting** teaches format, policy, or style.
- **Adaptive / extended thinking** gives the model more reasoning budget.

If the model understands the task but formats it incorrectly, add better examples. If the model formats correctly but reasons poorly, increase effort or use thinking features.

For Claude Opus 4.6 and Sonnet 4.6, Anthropic recommends **adaptive thinking** with the `effort` parameter instead of the older pattern of manual `thinking` budgets.

## Data Extraction Patterns

Data extraction is where prompt engineering and structured outputs meet operational reality. The best pattern is usually: use deterministic parsing where you can, use Claude where context or normalization matters, then validate the result.

### Method selection

| Method | Best for | Strength | Weakness |
|---|---|---|---|
| Regex | Stable string patterns like IDs, emails, dates | Fast and precise | Brittle for messy text |
| Native parser | JSON, XML, CSV, HTML tables | Deterministic | Requires structured source |
| NER / classical NLP | Entity-heavy text | Good for common entity types | Limited domain nuance |
| Claude + schema | Messy documents, contextual extraction, normalization | Flexible and expressive | Needs prompt/schema discipline |

### Recommended pipeline

1. Decide whether the source is deterministic enough for a normal parser.
2. If not, ask Claude for structured output with a schema.
3. Validate the result in application code.
4. Normalize types, units, and formatting after extraction.
5. Log ambiguous or low-confidence cases for human review.

### Example extraction workflow

Suppose you need to extract one support ticket summary from a long email thread:

- Use Claude to identify the customer, product area, issue summary, and severity.
- Return the result as JSON outputs.
- Validate severity against an enum in code.
- If severity is missing or the model refuses, route to a human reviewer.

This pattern is more reliable than trying to parse the same email thread with regex alone.

## A Correct End-to-End Example

```python
import json
from anthropic import Anthropic

client = Anthropic()

schema = {
    "type": "object",
    "properties": {
        "customer": {"type": "string"},
        "product_area": {"type": "string"},
        "severity": {
            "type": "string",
            "enum": ["low", "medium", "high", "critical"]
        },
        "summary": {"type": "string"}
    },
    "required": ["customer", "product_area", "severity", "summary"],
    "additionalProperties": False,
}

prompt = """
<instructions>
Extract one support-ticket record from the email thread.
</instructions>

<examples>
  <example>
    <input>Acme reports that checkout fails for some EU customers.</input>
    <output>{"customer":"Acme","product_area":"checkout","severity":"high","summary":"Checkout intermittently fails for EU customers."}</output>
  </example>
</examples>

<input>
Customer: Northwind
Product: analytics dashboard
Problem: exports time out during month-end reporting
Impact: CFO cannot deliver report today
</input>
"""

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=512,
    system="You are a precise data-extraction assistant.",
    messages=[{"role": "user", "content": prompt}],
    output_config={
        "format": {
            "type": "json_schema",
            "schema": schema,
        }
    },
)

ticket = json.loads(response.content[0].text)
print(ticket)
```

## Common Exam Traps

- `system` is top-level, not a `messages[]` role.
- `thinking=true` is outdated and incorrect.
- JSON outputs and strict tool use are related, but not interchangeable.
- Optional fields should be optional or nullable in the schema; do not rely on invalid placeholders like empty strings unless the schema explicitly permits them.
- Prefilling is not the same as structured outputs, and JSON outputs are incompatible with message prefilling.
- If you need typed agent results from the Agent SDK, look for `structured_output` on the final result message.

## Final Takeaways

For exam prep, remember this decision order:

- Need a reliably parseable **final answer** -> use JSON outputs.
- Need reliably structured **tool arguments** -> use strict tool use.
- Need better formatting or policy adherence -> add examples.
- Need deeper reasoning -> raise effort / use thinking features.
- Need production reliability -> validate everything in application code anyway.
