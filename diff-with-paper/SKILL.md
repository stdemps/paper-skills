---
name: diff-with-paper
description: >
  Compare a Paper design frame against the current code implementation and produce a structured diff report.
  Use when the user says diff with paper, compare paper, diff design, design vs code, /diff-with-paper,
  check against design, where does paper differ, or wants to see gaps between design and code.
---

# Paper diff (design vs code comparison)

Produce a **structured report** of differences between a Paper frame and the current code. **Report only — no edits.**

## Input

The user provides the **Paper frame name**. Optionally the code file/route — if not given, locate it by searching for matching components using layer names, text content, and route structure.

## Workflow

1. **Pull design state**
   - `get_tree_summary` + `get_screenshot` for the named frame.

2. **Pull code state**
   - Read the component source and its sub-components.
   - If a dev server is running, take a browser screenshot for visual comparison.

3. **Produce a structured report**

   Organize by category. Mark matches with checkmarks so the user sees what's aligned, not just what's broken:

   **Spacing**
   - Each item: what Paper shows (px) → what code has (class/token → px) → match or mismatch

   **Colors**
   - Each item: Paper hex → code token/class → match or mismatch

   **Typography**
   - Each item: Paper size/weight/family → code classes → match or mismatch

   **Layout**
   - Direction, alignment, order, wrapping, grid vs flex

   **Content**
   - Button labels, headings, body text, placeholder copy

   **Elements**
   - Missing in code (present in Paper but not implemented)
   - Extra in code (implemented but not in Paper)

4. **Summary line**
   - e.g. "7 differences found (3 spacing, 1 color, 1 typography, 1 layout, 1 content). 5 items match."

## Rules

- **Report only** — do not edit any code. The user decides what to act on.
- **Show matches too** — the user needs to see what's already aligned.
- **Map Paper values to project tokens** — so the fix is obvious from the report.
- **If the user wants fixes** — suggest running `build-from-paper` next.
- **Flag intentional deviations** — if a difference looks deliberate (e.g. responsive adaptation), note it.

## Installation

Copy this folder to your AI coding agent's skills directory:

- **Claude Code:** `~/.claude/skills/diff-with-paper/`
- **Cursor:** `~/.cursor/skills/diff-with-paper/`

Then use `/diff-with-paper` or say "diff [frame name] with Paper."
