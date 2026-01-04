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
