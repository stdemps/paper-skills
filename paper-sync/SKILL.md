---
name: paper-sync
description: >
  Sync a product screen from implementation code into Paper (design tool) via Paper MCP — or emit chunked HTML for manual paste.
  Use when the user says paper-sync, Paper sync, sync screen to Paper, code to Paper, /paper-sync, match implementation in Paper,
  artboard from code, or wants design–dev alignment starting from the real UI. Works for any product/repo when given token paths and component paths.
---

# Paper sync (code → design)

Build or update **Paper** so a frame matches **what the codebase actually renders** — not a reinterpretation. Works in **any repo** once the user points you at the right files.

## Default assumption

- **Prefer an existing Paper artboard** the user is already working in.
- Insert into a **dedicated parent frame** inside that artboard (e.g. `Spec from code — [Screen]`) so other explorations are not overwritten.
- **Create a new artboard only** if the user says there is no suitable target; size it from **layout constraints in code** (breakpoint, `max-w-*`, full bleed).

## Design tool & MCP

- **Primary:** **Paper MCP** (`user-paper` or equivalent). **Read tool schemas** (`write_html`, `get_basic_info`, `get_tree_summary`, `get_selection`, `create_artboard`, `finish_working_on_nodes`, etc.) before calling.
- **If Paper MCP is unavailable:** run **paste mode** (end of this skill).

## Workflow (Paper MCP)

1. **Discover context:** `get_basic_info`; use `get_selection` / `get_tree_summary` to resolve **`targetNodeId`** for the parent of new content.
2. **Write incrementally:** `write_html` in **small chunks** (e.g. chrome → header → sections → footer). After **2–3 chunks**, visually checkpoint (screenshot if available): spacing, type hierarchy, contrast, alignment.
3. **Modes:** Prefer **`insert-children`**; use **`replace`** only when the user asked to swap a specific node.
4. **Layers:** Use sensible names (`layer-name` on roots where supported; mirror real regions: header, body scroll, list, footer).
5. **Done:** `finish_working_on_nodes` when stable.

## Source of truth (user must supply or you must locate)

| Topic | What to use |
|--------|-------------|
| **Implementation** | The exact pages/components for this screen (paths the user gives or you find by search). |
| **Tokens / theme** | Project's **semantic** colors: CSS variables (`--*`), theme object, or Tailwind semantic colors — **not** inventing hex for chrome. **Exception:** if the code uses **explicit palette utilities** (e.g. brand blues on a chip), **match those** — they are product UI. |
| **Typography** | Fonts and scale from the project (global CSS, design tokens, Tailwind `font-*` / `text-*`). Match **sizes, weights, tracking** implied by the classes in the components. |
| **Structure** | Mirror **real composition**: section order, headings, buttons, empty states, badges — from the component tree, not a generic template. |
| **Copy** | **Exact strings** from code (labels, buttons, helper text). If something is unknown, use **`TBD — verify in code`** — do not invent product copy or fake entities. |
| **Icons** | Use the **same icon set** the app uses (often **Lucide** in React). Match **names and rendered size** (`h-4 w-4` → 16px). **Do not use emoji as icons.** |

## Spacing & layout fidelity

- Derive **px from the styling system** the project uses (e.g. Tailwind: `px-6` → 24px, `gap-2` → 8px, `-mt-6` → −24px; other systems: read the scale).
- **Paper HTML rules (typical):** **flex** + **inline styles**; avoid **grid**, **margin**-based layout, and unsupported patterns unless Paper docs confirm. Follow the **Paper server "workflow tips"** (small writes, checkpoints).

## Artboard sizing

- Infer from code: **max width** classes, **container** widths, **responsive** behavior.
- Name the artboard clearly, e.g. **`[Screen] — code match ([breakpoint] · W×H)`**.

## Handoff

The user refines in Paper and may share the file back for **implementation alignment** — preserve clear layer names to make diffs obvious.

---

## Paste mode (no MCP)

Emit:

1. **Chunked HTML** with inline styles (same structure as you would push via `write_html`).
2. A short **checklist**: token references, key measurements, breakpoint width assumption, and any **TBD — verify in code** items.

---

## Installation

Copy this folder to your AI coding agent's skills directory:

- **Claude Code:** `~/.claude/skills/paper-sync/`
- **Cursor:** `~/.cursor/skills/paper-sync/`

Then use `/paper-sync` or say "sync this screen to Paper."
