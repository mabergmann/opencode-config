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

## Notes

- This repo is not the opencode project itself; it is a configuration layer for it.
- If you prefer to keep your own fork, replace the clone URL accordingly.

## License

See `LICENSE` for details.
