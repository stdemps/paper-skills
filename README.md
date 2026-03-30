# Paper Skills

AI coding agent skills for keeping [Paper](https://paper.design) designs and code in sync. Works with Claude Code and Cursor.

These skills close the loop between designing in Paper and building in code — so you can move between the two without losing fidelity.

## The skills

| Skill | Direction | What it does |
|-------|-----------|--------------|
| `push-to-paper` | Code → Paper | Push what your code renders into a Paper frame (pixel QA checklist, paste fallback when MCP is limited) |
| `build-from-paper` | Paper → Code | Update code to pixel-perfectly match a Paper frame |
| `diff-with-paper` | Compare | Structured report of differences between Paper and code |
| `pull-tokens` | Tokens | Extract and map design tokens from Paper to your codebase |

## How they work together

```
Design in Paper ──→ /build-from-paper ──→ Code updates to match
                                              │
Code changes ─────→ /push-to-paper ────→ Paper updates to match
                                              │
Not sure what's off? → /diff-with-paper ─→ See exactly what differs
                                              │
Tokens drifting? ───→ /pull-tokens ──────→ Sync your design system
```

## Prerequisites

- [Paper](https://paper.design) with MCP server running
- An AI coding agent that supports skills ([Claude Code](https://claude.ai/code) or [Cursor](https://cursor.com))

## Install

Clone this repo, then copy the skill folders to your agent's skills directory:

```bash
# Claude Code
cp -r push-to-paper build-from-paper diff-with-paper pull-tokens ~/.claude/skills/

# Cursor
cp -r push-to-paper build-from-paper diff-with-paper pull-tokens ~/.cursor/skills/
```

**Optional:** vendor `push-to-paper/` (or the full set) under your app’s `.cursor/skills/` so the team shares the same revision in git — see `push-to-paper/SKILL.md` → Installation.

## Usage

Once installed, use the skills by name in your AI agent:

- `/push-to-paper` or "push this screen to Paper"
- `/build-from-paper` or "build from the [frame name] frame"
- `/diff-with-paper` or "diff [frame name] with Paper"
- `/pull-tokens` or "pull tokens from [frame name]"

## License

MIT
