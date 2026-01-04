# Instructions for LLMs

- This repo is always cloned into ~/.config/opencode
- The repo is public in github, so be careful with sensitive information.
- Use markdown configs over json configs

# Agents

There are **two agent types**.
**Primary agents** are the main assistants you talk to directly; you can switch between them during a session.
**Subagents** are specialists that primary agents can invoke automatically or that you can call explicitly with `@`.

You can **create or customize agents** via **Markdown files** in `agent/`, where the filename becomes the agent name.

Key configuration options include:

* **Description** (required): what the agent does.
* **Model**: choose different LLMs per agent.
* **Prompt**: custom system instructions.
* **Temperature**: control determinism vs creativity.
* **Max steps**: cap agent iterations to control cost.
* **Tools**: enable/disable capabilities like write, edit, bash.
* **Permissions**: allow, deny, or require approval per tool or even per bash command.
* **Mode**: primary, subagent, or both.
* **Disable**: turn an agent off entirely.
* **Provider-specific options**: pass advanced model parameters through directly.

Here is an example agent config in markdown:

```Markdown
---
description: Code review without edits
mode: subagent
permission:
  edit: deny
  bash:
    "git diff": allow
    "git log*": allow
    "*": ask
  webfetch: deny
---

Only analyze code and suggest changes.
```

```Markdown
---
description: Performs security audits and identifies vulnerabilities
mode: subagent
tools:
  write: false
  edit: false
---

You are a security expert. Focus on identifying potential security issues.

Look for:

- Input validation vulnerabilities
- Authentication and authorization flaws
- Data exposure risks
- Dependency vulnerabilities
- Configuration security issues
```

For more info about agents read https://opencode.ai/docs/agents/

# Tools

Agents can be granted capabilities via a Tools section in their markdown config. Tools control what an agent is allowed to do (e.g., read files, edit, run bash) and how.

Key points:

- Enable/disable per tool: Set booleans under `tools` to allow or block capabilities.
- Fine-grained permissions: Use `permission` to allow, deny, or require approval for specific tools or individual bash commands.
- Prefer markdown configs: Define tool settings directly in agent markdown front matter.
- Safety first: Avoid granting broad bash access unless necessary; prefer allowlists.

Common tools:

- `read`: Allows reading files from the filesystem.
- `write`: Allows creating or overwriting files.
- `edit`: Allows modifying existing files, with safeguards (must read before edit).
- `bash`: Allows running shell commands (git, npm, etc.). Supports command-level rules.
- `webfetch`: Allows fetching content from URLs for analysis.
- `task`: Allows launching subagents for multi-step autonomous work.

Permission patterns:

- `allow`: Command/tool can run without approval.
- `deny`: Command/tool is blocked.
- `ask`: Command/tool requires approval before running.

Examples:

```Markdown
---
description: Doc-only reviewer
mode: subagent
tools:
  read: true
  edit: false
  write: false
  bash: false
  webfetch: false
permission:
  read: allow
---

Only read files and suggest documentation improvements.
```

```Markdown
---
description: Git-aware code reviewer
mode: subagent
tools:
  read: true
  bash: true
permission:
  bash:
    "git status": allow
    "git diff": allow
    "git log*": allow
    "*": deny
---

Analyze changes using limited git commands.
```

```Markdown
---
description: Network researcher
mode: subagent
tools:
  webfetch: true
permission:
  webfetch: allow
---

Fetch and summarize external documentation.
```

Approval workflows:

- Use `ask` for sensitive operations (e.g., `bash` with network or destructive commands).
- Combine `tools` booleans with `permission` rules to enforce both capability and policy.

Recommendations:

- Start with minimal tool access and expand as needed.
- For `bash`, prefer explicit allowlists over wildcards.
- Do not grant `write` or `edit` unless the agent’s role requires file changes.
- Document the intent of each tool grant in the agent description.

For more info about tools read https://opencode.ai/docs/tools/

# Commands

Create custom commands for repetitive tasks that run predefined prompts via the TUI.

- Overview:
  - Commands are triggered with `/name` in the TUI.
  - Built-in commands include `/init`, `/undo`, `/redo`, `/share`, `/help`.
  - Custom commands can override built-ins when using the same name.

- Create command files:
  - Use markdown files in `command/` directories.
  - Location (this repo is the global root):
    - `command/`
  - The file name (without extension) becomes the command name (e.g., `test.md` -> `/test`).

- Configuration formats (prefer markdown over JSON):
  - Markdown front matter defines properties; file content is the template.
    ```Markdown
    ---
    description: Run tests with coverage
    agent: build
    model: anthropic/claude-3-5-sonnet-20241022
    ---
    Run the full test suite with coverage report and show any failures.
    Focus on the failing tests and suggest fixes.
    ```

- Prompt config helpers:
  - Arguments: use `$ARGUMENTS` or positional `$1`, `$2`, `$3` inside templates.
  - Shell output injection: use `!\`command\`` to include bash output (e.g., `!\`npm test\`` or `!\`git log --oneline -10\``).
  - File references: include files with `@path/to/file.ext` to inline contents.

- Options (front matter / JSON keys):
  - `template` (required): the prompt content sent to the LLM.
  - `description` (optional): short summary shown in the TUI.
  - `agent` (optional): which agent executes the command; defaults to current agent.
  - `subtask` (optional): if true, forces a subagent invocation to keep primary context clean.
  - `model` (optional): override the default model for this command.

- Notes:
  - Commands run in your project’s root directory.
  - In this repo, the root is `.`.
  - Use allowlists and permissions consistent with your agent/tool policies.

For more info about commands read https://opencode.ai/docs/commands/

# Permissions

Control which actions require approval to run. Configure at global scope or per-agent using markdown front matter.

- Concepts:
  - `allow`: run without approval
  - `ask`: prompt for approval before running
  - `deny`: disable the tool/command entirely

- Tools with permission controls:
  - `edit`, `bash`, `skill`, `webfetch`, `doom_loop`, `external_directory`

- Per-agent overrides (markdown):
  - Agent-specific `permission` overrides the global defaults configured in this repo.
    Create or edit: `agent/<name>.md`
    ```Markdown
    ---
    description: Build agent
    mode: primary
    permission:
      bash:
        "git push": allow
        "git status": allow
        "git diff": allow
        "*": ask
    ---
    Performs build and release tasks with elevated git permissions.
    ```

- Bash command rules:
  - Use exact matches or wildcards to target commands.
  - Wildcards supported: `*` (any chars), `?` (single char).
  - Examples:
    ```Markdown
    ---
    description: Terraform-safe reviewer
    mode: subagent
    permission:
      bash:
        "terraform *": deny
        "*": deny
        "pwd": allow
        "git status": ask
    ---
    Review code while blocking Terraform operations.
    ```

- Ask scope behavior:
  - "accept always" applies for the session, scoped to the first two command tokens (e.g., `git log *`).
  - Pipelines request approval per command parsed in the pipeline.

For more info about permissions read https://opencode.ai/docs/permissions/

# MCP Servers

Add local and remote MCP tools via the Model Context Protocol (MCP). Once enabled, MCP tools appear alongside built-in tools and can be invoked in prompts (e.g., "use the sentry tool").

- Overview:
  - MCP servers expose additional tools to the LLM.
  - Be selective: MCPs add context and can increase token usage.

- Enable and manage:
  - Define MCP servers in OpenCode config under `mcp` with unique names.
  - Use `enabled: true/false` to toggle servers.
  - Tools can be enabled per agent to avoid bloating global context.
  - In this repo, manage config files under `.`.

- Local servers:
  - `type: local` with a `command` to start the MCP (supports env vars and timeouts).
  - Example: `@modelcontextprotocol/server-everything` via `npx`.

- Remote servers:
  - `type: remote` with `url`; optional `headers` for API keys.
  - OAuth is auto-handled; you can also authenticate with `opencode mcp auth <name>`.

- OAuth basics:
  - Automatic detection, dynamic client registration when supported.
  - Credentials stored securely; can disable auto OAuth per server.

- Per-agent enabling:
  - Disable MCP tools globally and enable only for specific agents to reduce context.
  - Use glob patterns to manage multiple MCP tool names.

- Examples:
  - Sentry: interact with projects/issues; authenticate via `opencode mcp auth sentry`.
  - Context7: doc search; pass API key via headers if needed.
  - Grep by Vercel: search GitHub code snippets; invoke with its server name.

- Agent guidance (markdown):
  - Teach agents to invoke MCP tools via their prompts or AGENTS.md rules.
    ```Markdown
    ---
    description: Research agent
    mode: subagent
    tools:
      webfetch: true
      # Enable only the needed MCP tool(s) per agent
      sentry: true
      context7: true
    permission:
      webfetch: ask
      sentry: allow
      context7: allow
    ---
    When investigating production issues, use the `sentry` tools.
    When searching documentation, use `context7`.
    Prefer MCP tools over generic webfetch when available.
    ```

For more info about MCP servers read https://opencode.ai/docs/mcp-servers/

# Agent Skills

Define reusable behavior via `SKILL.md` files that agents can load on demand using the built-in `skill` tool.

- Place files (global only):
   - Global: `skill/<name>/SKILL.md`

- Discovery:
   - OpenCode loads global skills from your home config under `skill/*/SKILL.md`.

- Frontmatter (required fields):
  - `name`: lowercase, hyphen-separated; must match directory name.
  - `description`: 1–1024 characters.
  - Optional: `license`, `compatibility`, `metadata` (map).

- Name rules:
  - 1–64 chars, lowercase alphanumeric with single hyphens, no leading/trailing hyphen, no `--`.

- Usage:
  - Agents see available skills listed in the `skill` tool description.
  - Load a skill by calling: `skill({ name: "git-release" })`.

- Permissions:
  - Control access with `permission.skill` patterns; values: `allow`, `ask`, `deny`.
   - Per-agent overrides supported in agent markdown frontmatter under `agent/<name>.md`.
  - To disable entirely for an agent, set `tools.skill: false` in its global agent config.

- Troubleshooting:
  - Ensure `SKILL.md` filename is uppercase.
  - Include required `name` and `description` in frontmatter.
  - Names must be unique across locations.
  - Skills with `deny` are hidden from agents.

For more info about skills read https://opencode.ai/docs/skills/

# Available models

These are the models available when creating new agents:

```
github-copilot/claude-3.5-sonnet
github-copilot/claude-3.7-sonnet
github-copilot/claude-3.7-sonnet-thought
github-copilot/claude-haiku-4.5
github-copilot/claude-opus-4
github-copilot/claude-opus-4.5
github-copilot/claude-opus-41
github-copilot/claude-sonnet-4
github-copilot/claude-sonnet-4.5
github-copilot/gemini-2.0-flash-001
github-copilot/gemini-2.5-pro
github-copilot/gemini-3-flash-preview
github-copilot/gemini-3-pro-preview
github-copilot/gpt-4.1
github-copilot/gpt-4o
github-copilot/gpt-5
github-copilot/gpt-5-codex
github-copilot/gpt-5-mini
github-copilot/gpt-5.1
github-copilot/gpt-5.1-codex
github-copilot/gpt-5.1-codex-max
github-copilot/gpt-5.1-codex-mini
github-copilot/gpt-5.2
github-copilot/grok-code-fast-1
github-copilot/o3
github-copilot/o3-mini
github-copilot/o4-mini
github-copilot/oswe-vscode-prime
```

# Effective Prompt Engineering for LLMs

## General principles

- Clarity and specificity: Provide clear, concise instructions; define exactly what you want, including any jargon or context up front. Put detailed instructions at the start and specify context, style, and scope. Prefer precise asks over vague ones.
- Structured formatting: Separate role, context, task, and output using headings or delimiters (e.g., ```markdown fences```, triple quotes, or tags like `<instructions>`, `<context>`, `<input>`). Keep language simple and sections distinct.
- Explicit output formats and constraints: Specify the desired format (bullet list, table, JSON schema) and limits (length, style). State constraints positively (e.g., “Use a professional tone without heavy jargon”).
- Relevant context and examples: Include necessary background and few‑shot examples showing the exact format and style you expect. Reference files or data and ask questions about them after the context.

## Autonomous agents

- Role: Define the agent’s persona with input/output expectations and usage rules.
- Task decomposition: Prompt the agent to outline a plan and work through numbered subtasks systematically.
- Long‑term planning: Ask for assumptions, risks, and strategy before executing steps.

