# Instructions for LLMs

- This repo is always cloned into ~/.config/opencode
- The repo is public in github, so be careful with sensitive information.
- Use markdown configs over json configs

# Agents

There are **two agent types**.
**Primary agents** are the main assistants you talk to directly; you can switch between them during a session.
**Subagents** are specialists that primary agents can invoke automatically or that you can call explicitly with `@`.

You can **create or customize agents** via:
* **Markdown files** inside the `agent` folder, where the filename becomes the agent name.

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
