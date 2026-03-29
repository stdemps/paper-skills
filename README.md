# Paper Skills

AI coding agent skills for keeping [Paper](https://paper.design) designs and code in sync. Works with Claude Code and Cursor.

These skills close the loop between designing in Paper and building in code — so you can move between the two without losing fidelity.

## The skills

| Skill | Direction | What it does |
|-------|-----------|--------------|
| `paper-sync` | Code → Paper | Push what your code renders into a Paper frame |
| `paper-match` | Paper → Code | Update code to pixel-perfectly match a Paper frame |
| `paper-diff` | Compare | Structured report of differences between Paper and code |
| `paper-tokens` | Tokens | Extract and map design tokens from Paper to your codebase |

## How they work together

```
Design in Paper ──→ /paper-match ──→ Code updates to match
                                          │
Code changes ─────→ /paper-sync ──→ Paper updates to match
                                          │
Not sure what's off? → /paper-diff ──→ See exactly what differs
                                          │
Tokens drifting? ───→ /paper-tokens ─→ Sync your design system
```

## Prerequisites

- [Paper](https://paper.design) with MCP server running
- An AI coding agent that supports skills ([Claude Code](https://claude.ai/code) or [Cursor](https://cursor.com))

## Install

Clone this repo, then copy the skill folders to your agent's skills directory:

```bash
# Claude Code
cp -r paper-sync paper-match paper-diff paper-tokens ~/.claude/skills/

# Cursor
cp -r paper-sync paper-match paper-diff paper-tokens ~/.cursor/skills/
```

## Usage

Once installed, use the skills by name in your AI agent:

- `/paper-sync` or "sync this screen to Paper"
- `/paper-match` or "match the [frame name] frame"
- `/paper-diff` or "compare [frame name] against code"
- `/paper-tokens` or "sync tokens from [frame name]"

## License

MIT
