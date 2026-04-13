# Claude Commands & Skills

Shared slash commands and auto-triggering skills for Claude Code sessions. Work in any repo, any project.

## What's the Difference?

**Skills** (auto-trigger): Claude activates these automatically based on what you're doing. No slash command needed.

**Commands** (manual): Type `/command` to invoke explicitly. Useful as fallbacks or when you want to force a specific workflow.

## Skills (Auto-Triggering)

| Skill | Triggers When |
|---|---|
| `deliver` | You ask to build, implement, create, or deploy something non-trivial. Runs the full Orchestrate → Build → Verify loop with verification spec, subagent builder in worktree, and adversarial verifier. |
| `systematic-verify` | You're about to claim something is done. Fires before any completion claim to require fresh evidence. |
| `systematic-debug` | You report something is broken or not working. Forces hypothesis-driven investigation before any fix attempt. |

## Commands (Manual)

| Command | What It Does |
|---|---|
| `/deliver <task>` | Explicitly invoke the full delivery loop |
| `/verify` | Explicitly invoke verification |
| `/debug <thing>` | Explicitly invoke debugging |
| `/plan <what to build>` | Task breakdown without the full delivery loop |

## Install

### Full install (skills + commands)

```bash
# Clone
git clone https://github.com/Working-Mono/claude-commands.git /tmp/wm-claude

# Install commands
mkdir -p ~/.claude/commands
cp /tmp/wm-claude/*.md ~/.claude/commands/

# Install skills
mkdir -p ~/.claude/skills
cp -r /tmp/wm-claude/skills/* ~/.claude/skills/

# Cleanup
rm -rf /tmp/wm-claude
```

### Quick install (one-liner, commands only)

```bash
mkdir -p ~/.claude/commands && curl -sL https://raw.githubusercontent.com/Working-Mono/claude-commands/main/{deliver,verify,debug,plan}.md -o ~/.claude/commands/#1.md
```

## Update

Re-run the install. Files are overwritten.

## How It Works

- `~/.claude/commands/*.md` are loaded as slash commands in every Claude Code session
- `~/.claude/skills/*/SKILL.md` are loaded as auto-triggering skills in every session
- Both are user-level, so they apply across all repos and projects
- Project-specific commands (like `/build`, `/proposal`) still live in each repo's `.claude/commands/`

## Team Experience

After install, the team's experience changes:

**Before**: "Build me an enrichment connector for Apollo"
→ Claude jumps straight to coding, claims done, you find issues

**After**: Same request
→ Claude auto-triggers `deliver` skill
→ Explores codebase, writes verification spec, presents for approval
→ Dispatches builder subagent in worktree
→ Dispatches adversarial verifier that runs every assertion
→ Reports with evidence, or loops back to fix failures

No slash commands needed. It just works.

## Origin

Adapted from [obra/superpowers](https://github.com/obra/superpowers) and Working Mono's PI agent system (orchestrator/builder/verifier with tool gating).
