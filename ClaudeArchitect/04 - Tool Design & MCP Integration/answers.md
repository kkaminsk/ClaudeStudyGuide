# Domain 4: Tool Design & MCP Integration — Answer Key

---

**1. Answer: B** — Model Context Protocol

**Rationale:** In the Claude ecosystem, MCP means Model Context Protocol — an open standard for connecting AI applications to external tools, data sources, and workflows. It does not mean "managed control plane." This is explicitly called out as a common exam trap.

---

**2. Answer: C** — `tool_use_id`

**Rationale:** The `tool_use_id` is the join key between Claude's intent and your execution layer. Each `tool_use` block includes an `id`, and your `tool_result` must reference it so Claude can match the result to the specific tool call it requested.

---

**3. Answer: A** — Tools, Resources, and Prompts

**Rationale:** MCP servers can expose three main categories of capability: Tools (actions Claude can invoke), Resources (read-oriented content Claude can fetch and inspect), and Prompts (reusable prompt templates or guided workflows). Understanding the distinction between these three is important for choosing the right mechanism for each integration.

---

**4. Answer: C** — HTTP

**Rationale:** Claude Code documents three transport types: HTTP for remote servers (recommended for many cloud services), SSE for remote streaming-style servers, and stdio for local processes. HTTP is the recommended transport for remote SaaS integrations like GitHub, Notion, and Stripe.

---

**5. Answer: B** — Smaller tools are easier for Claude to choose correctly and easier for you to validate safely

**Rationale:** Good tool design matters because Claude relies heavily on descriptions and schema shape when deciding what to do. The study guide recommends preferring separate `create_ticket`, `get_ticket`, `list_open_tickets` tools over one giant `ticket_api` tool with many modes. Smaller, focused tools are easier for Claude to route correctly and easier for you to secure and validate.

---

**6. Answer: C** — In `.mcp.json` at the project root

**Rationale:** Project-scoped MCP servers live in `.mcp.json`, not in `CLAUDE.md` or personal config files. This is the collaboration-friendly approach for team-shared integrations checked into git. Claude Code prompts for approval before using project-scoped servers. This is also listed as a common exam trap — MCP config belongs in `.mcp.json`, not `CLAUDE.md`.

---

**7. Answer: B** — When the capability is read-only reference material like docs or config snapshots

**Rationale:** A common mistake is turning every integration into a tool. Resources are for read-oriented reference material (architecture docs, dashboards, config snapshots). Tools are for actions Claude needs to invoke with parameters. If the capability is basically read-only reference material, a resource is safer and easier for Claude to use.

---

**8. Answer: C** — Return a `tool_result` with `is_error: true` and a structured error payload

**Rationale:** The study guide emphasizes surfacing errors explicitly: avoid silent failure. If a tool fails, say why in a structured way and mark it as an error with `is_error: true`. Claude can often recover if the failure is explicit. Silent failures or empty results make debugging impossible and degrade the agent loop.

---

**9. Answer: B** — `strict: true` constrains the model's output, but server-side validation is still needed as a defense-in-depth practice

**Rationale:** The study guide states: "Treat tool inputs as untrusted even when `strict: true` is enabled. Validate again on the server side." `strict: true` uses grammar-constrained decoding to enforce schema conformance on Claude's side, but defense-in-depth requires server-side validation as well. This is a security best practice for all tool implementations.

---

**10. Answer: C** — `list_changed` notification

**Rationale:** MCP servers can send `list_changed` notifications so hosts can refresh capabilities dynamically without reconnecting. This is mentioned in the study guide as an important reliability feature — it allows MCP integrations to remain current as server capabilities change over time.
