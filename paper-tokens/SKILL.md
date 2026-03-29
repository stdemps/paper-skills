---
name: paper-tokens
description: >
  Extract design tokens (colors, fonts, spacing) from Paper frames and map them against the project's token definitions.
  Use when the user says paper-tokens, sync tokens, extract tokens, /paper-tokens, pull colors from paper,
  update tokens from design, or wants to keep the design system aligned between Paper and code.
---

# Paper tokens (design token sync)

Extract design values from Paper and map them against the project's existing token system. **Report first, write only after user approval.**

## Input

The user provides:
- **Frame name** (or "all" to scan the full artboard)
- Optionally which token types to focus on: `colors`, `fonts`, `spacing`, or `all` (default: `all`)

## Workflow

1. **Scan Paper**
   - `get_tree_summary` on the target frame(s).
   - Extract every unique value:
     - Colors: backgrounds, text, borders (hex values)
     - Font families, sizes, weights, line heights
     - Spacing values: padding, gaps, margins (px)

2. **Locate project tokens**
   - Find where the project defines its design system:
     - Tailwind config (`tailwind.config.ts` → `theme.extend`)
     - CSS variables (`:root` block in global CSS)
     - Theme file (e.g. `theme.ts`, `tokens.ts`)
   - Read and parse the existing token definitions.

3. **Produce a mapping report**

   Three categories:

   **Matched** (Paper value has a corresponding token)
   - e.g. `#18181b` → `--background` / `bg-zinc-900`

   **New** (in Paper, no matching token exists)
   - e.g. `#1e1b4b` → no match (used on 3 nodes: badge bg, card accent, header)
   - Group by usage count — tokens used on more nodes = higher priority.

   **Drift** (token exists but Paper uses a different value)
   - e.g. `--accent`: defined as `#6366f1`, Paper uses `#818cf8` (lighter)

4. **User decides**
   - For each "New" item: add a new token, or ignore (intentional one-off).
   - For each "Drift" item: update the token, keep as-is, or note as intentional.
   - Wait for explicit approval before writing anything.

5. **Write tokens** (only after approval)
   - Update the specific token file with approved changes.
   - Follow the project's existing naming conventions for new tokens.
   - Don't touch tokens the user didn't approve.

## Rules

- **Never auto-write** — always show the report and wait for approval.
- **Group by usage count** — more-used values are higher priority.
- **Respect naming conventions** — new tokens follow the project's existing patterns.
- **Flag ambiguity** — e.g. "this could be `--accent` or `--primary`, used in similar contexts."
- **Note raw palette vs semantic** — distinguish between raw color values and semantic token names.

## Installation

Copy this folder to your AI coding agent's skills directory:

- **Claude Code:** `~/.claude/skills/paper-tokens/`
- **Cursor:** `~/.cursor/skills/paper-tokens/`

Then use `/paper-tokens` or say "sync tokens from [frame name]."
