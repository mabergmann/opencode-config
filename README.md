# opencode-config

Personal configuration for the [opencode](https://github.com/anomalyco/opencode) CLI/agents. This repository is tailored to my workflow but is shared so others can use it as-is or take inspiration for their own agents and setup.

## Installation

This config is intended to live at `~/.config/opencode`.

### Quick install

1. Ensure the directory exists:
   ```bash
   mkdir -p ~/.config
   ```
2. Clone the repo into `~/.config/opencode`:
   ```bash
   git clone https://github.com/mabergmann/opencode-config ~/.config/opencode
   ```

### Updating

To pull the latest changes:
```bash
cd ~/.config/opencode
git pull
```

## What’s inside

- Opinionated settings and helpers tuned for my personal workflow
- Agent definitions and tips that others can adapt
- Files are structured to plug directly into opencode’s config directory

## My Workflow

This config supports a structured agent workflow used day-to-day:

- Orchestrator: One primary agent coordinates specialist subagents and enforces phase transitions.
- Design phase: Two agents iterate (Writer ↔ Reviewer) on a design doc or experiment plan until both agree it is ready and the user approves.
- Planning phase: A single agent decomposes the approved plan into small, atomic steps. Each step is intended to map to a future git commit.
- Implementation phase: Two agents iterate (Implementer ↔ Improver) to complete one planned step at a time, refining code and tests until convergence.
- Git phase: A git specialist reads changes, proposes a concise, "why"-focused commit message, and creates the commit upon user approval.
- Reporting: Two report-focused agents collaborate to produce technical and managerial summaries. Documentation assumes readers with deep code and business context.

### Typical Usage

- Start with the orchestrator to initiate a design or experiment plan.
- Alternate the writer and reviewer until the document meets acceptance criteria.
- Generate granular implementation steps with the planner (each a future commit).
- Implement steps with implementer and improver, one at a time.
- Use the git specialist to propose and finalize commit messages.
- Produce reports with the report writer and reviewer.

## External Artifacts (non-versioned)

Keep cross-repo assignment artifacts outside the repo for privacy and flexibility.

- Location: `~/opencode-artifacts/assignments/<assignment-id>/`
- Structure: `design/`, `planning/`, `reports/`, plus `meta.md` for links/owners/status
- IDs: short hyphenated slugs (e.g., `a-2026-01-auth-redesign`)
- Access: agents read/write via absolute paths; nothing here is committed
- Cross-links: reference repo files/PRs from artifacts; optional in-repo index file pointing to external paths

To start a new assignment, create the folder and copy your templates:
```bash
mkdir -p ~/opencode-artifacts/assignments/a-2026-01-example/{design,planning,reports}
cp ~/opencode-artifacts/templates/{design.md,plan.md,report.md} ~/opencode-artifacts/assignments/a-2026-01-example/
```