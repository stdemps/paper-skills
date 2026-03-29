---
name: paper-match
description: >
  Match code to a Paper design frame — pixel-perfect implementation from design.
  Use when the user says paper-match, match paper, match design, code from paper, /paper-match,
  implement this frame, match this frame, or wants to update code to match a Paper design.
---

# Paper match (design → code)

Update code so it **pixel-perfectly matches a Paper frame** — using the project's own design system, not arbitrary values.

## Input

The user provides the **Paper frame name** to match. Optionally a code file/route — if not given, locate it (see Step 3).

## Workflow

1. **Pull design state**
   - `get_tree_summary` — find the named frame, get its node ID and full structure.
   - `get_screenshot` — capture the visual target. This screenshot is the source of truth.

2. **Read the design structure**
   - Parse the tree summary for: layout hierarchy, text content (exact strings), spacing values, colors (hex), font sizes/weights/families, border radii, opacity.
   - Note every concrete value — these are what the code must reproduce.

3. **Locate the code**
   - Search the codebase for the matching component/page using: layer names, text content, route structure, file names.
   - If ambiguous, ask the user which file maps to this frame.
   - Read the component files and all imported sub-components.

4. **Understand existing tokens**
   - Read the project's design system: Tailwind config, CSS variables, theme files.
   - Map Paper's pixel/hex values to the closest existing tokens.
   - If a Paper value has **no matching token**, flag it to the user — don't invent tokens.

5. **Produce an inline diff checklist**
   Before editing, list the specific differences:
   - Spacing: e.g. "`gap-4` in code → frame shows ~12px gap, should be `gap-3`"
   - Colors: e.g. "header bg is `bg-zinc-900` → frame shows `#1a1a2e`, map to `bg-primary`"
   - Typography: sizes, weights, tracking, line height
   - Layout: flex direction, alignment, order of elements
   - Content: label/copy differences
   - Missing or extra elements

6. **Edit code**
   - Make targeted changes using the project's own classes/tokens.
   - **No inline styles** unless the project already uses them.
   - **Preserve component logic and behavior** — only change visual styling.
   - Work incrementally — small, reviewable changes, not full rewrites.

7. **Verify**
   - If a dev server is running, take a browser screenshot and compare against the Paper screenshot.
   - Flag any remaining gaps.

## Rules

- **Exact strings from Paper** — don't change copy unless it's clearly placeholder.
- **Never invent tokens** — use what the project already has. Flag gaps to the user.
- **Preserve behavior** — this skill changes appearance, not functionality.
- **Incremental edits** — small targeted changes, not rewrites.
- **Project conventions** — follow the existing code patterns (naming, file structure, component composition).

## Installation

Copy this folder to your AI coding agent's skills directory:

- **Claude Code:** `~/.claude/skills/paper-match/`
- **Cursor:** `~/.cursor/skills/paper-match/`

Then use `/paper-match` or say "match the [frame name] frame."
