# Claude Code Custom Commands

This repository contains my custom Claude Code commands (skills) synced across machines.

## Available Commands

- `audit-docs.md` - Audit and migrate agent documentation
- `implement.md` - Implementation mode
- `init-docs.md` - Initialize agent documentation
- `jira.md` - Jira investigation protocol

## Setup on a New Machine

1. Clone this repo to `~/.claude/commands`:
   ```bash
   git clone <repo-url> ~/.claude/commands
   ```

2. Commands will be automatically available in Claude Code

## Syncing Changes

After modifying commands:
```bash
cd ~/.claude/commands
git add .
git commit -m "Update commands"
git push
```
