# Domain 4: Tool Design & MCP Integration — Practice Questions

**Instructions:** Choose the single best answer for each question.

---

**1.** In the Claude ecosystem, what does MCP stand for?

A) Managed Control Plane
B) Model Context Protocol
C) Multi-Channel Pipeline
D) Message Communication Platform

---

**2.** When Claude returns a `tool_use` block, what unique identifier must be referenced in the corresponding `tool_result`?

A) `request_id`
B) `tool_name`
C) `tool_use_id`
D) `session_id`

---

**3.** What are the three core capability types that MCP servers can expose?

A) Tools, Resources, and Prompts
B) Tools, Schemas, and Endpoints
C) Actions, Events, and Logs
D) Commands, Queries, and Subscriptions

---

**4.** Which transport type does Claude Code recommend for remote MCP servers like cloud SaaS integrations?

A) stdio
B) WebSocket
C) HTTP
D) gRPC

---

**5.** Why does Anthropic recommend designing single-purpose tools rather than one large multi-mode tool?

A) Claude can only call one tool per turn
B) Smaller tools are easier for Claude to choose correctly and easier for you to validate safely
C) Multi-mode tools are not supported by the Messages API
D) Single-purpose tools use less context

---

**6.** Where should team-shared MCP server configurations be stored so they are checked into version control?

A) In `CLAUDE.md`
B) In `~/.claude.json`
C) In `.mcp.json` at the project root
D) In `.claude/settings.local.json`

---

**7.** When should you use an MCP Resource instead of an MCP Tool?

A) When Claude needs to execute a write operation
B) When the capability is read-only reference material like docs or config snapshots
C) When authentication is required
D) When the external system uses REST APIs

---

**8.** What should you do when a tool execution fails?

A) Return an empty `tool_result` and let Claude infer the failure
B) Retry the tool call automatically three times before reporting
C) Return a `tool_result` with `is_error: true` and a structured error payload
D) Silently skip the failed tool and continue the loop

---

**9.** Why should tool inputs be validated even when `strict: true` is enabled on the tool definition?

A) `strict: true` only validates output schemas, not inputs
B) `strict: true` constrains the model's output, but server-side validation is still needed as a defense-in-depth practice
C) `strict: true` is not actually enforced by the API
D) Tool inputs are always trusted when sent by Claude

---

**10.** What feature allows MCP servers to notify the host (like Claude Code) that their available capabilities have changed?

A) `tool_refresh` event
B) `capability_update` webhook
C) `list_changed` notification
D) Automatic polling every 60 seconds
