# Claude Commands

Shared slash commands for Claude Code sessions. These work in any repo, any project.

## Commands

| Command | What It Does |
|---|---|
| `/verify` | Evidence-based verification before claiming work is done |
| `/debug <thing>` | Hypothesis-driven debugging. Prevents "retry and hope" loops |
| `/plan <what to build>` | Break work into precise, granular tasks before building |

## Install

```bash
# Clone to a temp location and copy
git clone https://github.com/Working-Mono/claude-commands.git /tmp/claude-commands
mkdir -p ~/.claude/commands
cp /tmp/claude-commands/*.md ~/.claude/commands/
rm -rf /tmp/claude-commands
```

Or one-liner:

```bash
mkdir -p ~/.claude/commands && curl -sL https://raw.githubusercontent.com/Working-Mono/claude-commands/main/{verify,debug,plan}.md -o ~/.claude/commands/#1.md
```

## Update

Re-run the install. Files are overwritten.

## How It Works

Files in `~/.claude/commands/` are loaded as slash commands in every Claude Code session. They're user-level, so they apply across all repos and projects.

Project-specific commands (like `/build`, `/proposal`, `/client`) still live in each repo's `.claude/commands/` directory. These shared commands are deliberately generic so they work everywhere.

## Origin

Adapted from [obra/superpowers](https://github.com/obra/superpowers), filtered for what's useful in integration-heavy GTM infrastructure work.
