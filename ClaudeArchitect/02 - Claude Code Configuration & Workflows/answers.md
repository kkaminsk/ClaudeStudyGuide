# Domain 2: Claude Code Configuration & Workflows — Answer Key

---

**1. Answer: C** — ~200 lines

**Rationale:** Anthropic recommends targeting under ~200 lines per CLAUDE.md for better adherence. Larger files consume context and reduce compliance. When instructions grow beyond this, the guidance is to split them using `@imports` or modular `.claude/rules/` files.

---

**2. Answer: C** — Lazy-loaded on demand when Claude reads files in those subdirectories

**Rationale:** Subdirectory CLAUDE.md files are not loaded at launch. They are lazy-loaded when Claude reads files in those subdirectories. This is crucial for monorepos: package-specific content only appears when you work within that area, preventing irrelevant instructions from consuming context.

---

**3. Answer: C** — 5

**Rationale:** CLAUDE.md supports `@path/to/import` for importing additional files. Relative paths resolve relative to the importing file. Imports can cascade up to five recursive hops. This enables structured deep references while preventing infinite import chains.

---

**4. Answer: B** — The skill takes precedence

**Rationale:** Anthropic states that if a skill and a legacy command share the same name, the skill takes precedence. Legacy `.claude/commands/*.md` commands still work and are supported, but skills are the recommended format going forward with additional controls like frontmatter metadata and tool restrictions.

---

**5. Answer: B** — CLAUDE.md provides contextual guidance (soft); settings.json provides client-enforced rules (strong)

**Rationale:** This distinction is exam-critical. CLAUDE.md shapes behavior through contextual guidance but is not a strict enforcement mechanism. Settings rules are enforced by the client. A common pattern is: put team-shared permissions, hooks, and MCP servers in project settings; use CLAUDE.md for conventions and workflows.

---

**6. Answer: B** — `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB=1`

**Rationale:** Anthropic provides `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB=1` to strip Anthropic and cloud provider credentials from subprocess environments (Bash tool, hooks, MCP stdio servers). This explicitly reduces exposure to prompt injection attempts that could exfiltrate secrets via shell expansion.

---

**7. Answer: B** — It grants tool access without extra approval prompts while the skill is active

**Rationale:** The `allowed-tools` field can grant tool access without extra prompts while the skill is active. Baseline permissions still apply outside the skill. This allows skills to perform their intended workflows smoothly while maintaining the overall permission model.

---

**8. Answer: B** — It uses glob patterns to skip selected CLAUDE.md or rules paths, but managed policy CLAUDE.md cannot be excluded

**Rationale:** `claudeMdExcludes` uses glob patterns matched against absolute paths to skip selected CLAUDE.md or rules paths. Arrays merge across settings layers. However, managed policy CLAUDE.md (the organization-wide file at the OS-level path) cannot be excluded — this ensures enterprise governance cannot be bypassed.

---

**9. Answer: B** — It runs shell commands before Claude sees the prompt, creating potential shell injection vulnerabilities if arguments are interpolated unsafely

**Rationale:** The `!` preprocessor runs commands before Claude sees the prompt, substituting output into the prompt content. This is convenient for injecting live context (diffs, PR metadata), but if arguments are interpolated unsafely, it creates shell injection vulnerabilities. Anthropic recommends using deterministic scripts that validate/escape inputs, restricting tools, and enabling sandboxing.

---

**10. Answer: C** — GitLab CI/CD

**Rationale:** The GitLab CI/CD integration is described as beta and maintained by GitLab (not Anthropic). In contrast, the GitHub Actions integration (`anthropics/claude-code-action@v1`) is official and maintained by Anthropic. Jenkins and CircleCI have no first-party plugins — they use CLI/SDK patterns directly.
