# Domain 2: Claude Code Configuration & Workflows — Practice Questions

**Instructions:** Choose the single best answer for each question.

---

**1.** What is the recommended maximum line count for a single CLAUDE.md file according to Anthropic's guidance?

A) 50 lines
B) 100 lines
C) ~200 lines
D) 500 lines

---

**2.** In a monorepo, when are CLAUDE.md files located in subdirectories loaded into context?

A) At session start along with all other CLAUDE.md files
B) Only when explicitly imported with `@path/to/file`
C) Lazy-loaded on demand when Claude reads files in those subdirectories
D) Never — only the root CLAUDE.md is loaded

---

**3.** How many recursive hops are supported when using `@path/to/import` in CLAUDE.md files?

A) 1
B) 3
C) 5
D) Unlimited

---

**4.** If a skill and a legacy command (`.claude/commands/*.md`) share the same name, which takes precedence?

A) The legacy command always wins
B) The skill takes precedence
C) Both are loaded and Claude chooses
D) An error is thrown at startup

---

**5.** What is the primary distinction between CLAUDE.md and `settings.json` in Claude Code's configuration hierarchy?

A) CLAUDE.md is for team settings; settings.json is for personal preferences
B) CLAUDE.md provides contextual guidance (soft); settings.json provides client-enforced rules (strong)
C) CLAUDE.md is loaded first; settings.json is loaded last
D) They serve identical purposes but use different file formats

---

**6.** What environment variable does Anthropic recommend setting in CI/CD automation contexts to prevent secrets from leaking into subprocess environments?

A) `CLAUDE_SAFE_MODE=1`
B) `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB=1`
C) `ANTHROPIC_SCRUB_SECRETS=true`
D) `CLAUDE_SECURE_ENV=1`

---

**7.** What is the purpose of the `allowed-tools` field in a skill's YAML frontmatter?

A) It defines which tools the skill creates
B) It grants tool access without extra approval prompts while the skill is active
C) It permanently changes the user's permission settings
D) It lists tools that are blocked during the skill's execution

---

**8.** How does the `claudeMdExcludes` setting work, and what is its limitation?

A) It excludes all CLAUDE.md files including managed policy; configured per-session
B) It uses glob patterns to skip selected CLAUDE.md or rules paths, but managed policy CLAUDE.md cannot be excluded
C) It removes CLAUDE.md from version control
D) It only applies to subdirectory CLAUDE.md files and has no limitations

---

**9.** What security risk does the `!`<command>`  preprocessor in skills introduce?

A) It allows Claude to execute arbitrary code without permissions
B) It runs shell commands before Claude sees the prompt, creating potential shell injection vulnerabilities if arguments are interpolated unsafely
C) It disables sandboxing for the duration of the skill
D) It exposes the API key to external processes

---

**10.** Which official CI/CD integration for Claude Code is described as beta and maintained by a third party (not Anthropic)?

A) GitHub Actions
B) Jenkins Plugin
C) GitLab CI/CD
D) CircleCI Orb
