---
name: push-to-paper
description: >
  Sync a product screen from implementation code into Paper (design tool) via Paper MCP — or emit chunked HTML for manual paste.
  Use when the user says push to paper, paper sync, sync screen to Paper, code to Paper, /push-to-paper, match implementation in Paper,
  artboard from code, or wants design–dev alignment starting from the real UI. Works for any product/repo when given token paths and component paths.
  Target pixel-faithful reproduction: follow the Pixel QA section before calling finish_working_on_nodes.
---

# Paper sync (code → design)

Build or update **Paper** so a frame matches **what the codebase actually renders** — not a reinterpretation. Works in **any repo** once the user points you at the right files.

## Pixel fidelity goal (≥90% reliable passes)

**Aim:** visually and numerically indistinguishable from a screenshot of the implementation at the chosen breakpoint, within the tolerances below.

| Dimension | Target | Acceptable |
|-----------|--------|------------|
| Spacing from Tailwind scale (`p-*`, `gap-*`, `m-*`) | Exact px mapping | ±1 px only if engine rounding |
| Font size / line-height / weight | Matches classes | Exact |
| Border radius | From `rounded-*` or `--radius` | Exact |
| Semantic colors | From theme tokens | Same hex as browser computed **or** same `hsl()` triple as `index.css` |
| Layout | Same hierarchy and order | No omitted regions |

**Reliability (9/10):** Treat the **Pixel QA checklist** as a gate. Do **not** call `finish_working_on_nodes` until it passes. If a run fails QA, fix with `update_styles` or small `write_html` deltas and re-run QA — that iteration counts as one attempt; systematic use of this gate is what keeps long-run pass rates high.

### Pre-flight: measurement spec (required for pixel mode)

Before writing HTML:

1. List **every** layout/aesthetic class on the target DOM subtree (including CVA `variant` / `size` defaults — e.g. shadcn `Button` applies `px-5` even for `variant="ghost"` unless overridden).
2. Build a **spec table**: for each logical block (root, icon, title, body, row, button…): `fontFamily`, `fontSize`, `lineHeight`, `fontWeight`, `padding`, `gap`, `maxWidth`, `borderRadius`, `color`, `backgroundColor`, `opacity`.
3. Map Tailwind spacing to px (default 4 px unit): `1→4`, `2→8`, `3→12`, `4→16`, `5→20`, `6→24`, `8→32`, `16→64`, etc.; `min-h-11` → 44 px.
4. Resolve theme colors: read semantic CSS (`hsl(var(--primary))`, …) from the project’s theme file (e.g. `:root` / `.dark`). Prefer **sampling computed styles from a running build** for final hex if Paper and the browser differ in HSL rounding.
5. **Paper layout rules** (from `write_html`): use **flex** + **padding** + **gap**; **no** `display: grid`, **no** margin-driven layout for containers, **no** emoji icons — use SVG (e.g. Lucide paths) at the size implied by classes (`h-4 w-4` → 16 px).

### Post-flight: Pixel QA checklist (mandatory)

After the structure is stable:

1. `get_computed_styles` on **at least** these anchors: screen root, primary heading text, primary body text, primary CTA container, secondary CTA if any, key icon container.
2. Compare each numeric to the **spec table**. Flag any delta outside tolerance.
3. `get_screenshot` at **2×** on the spec root for a visual read of hierarchy, alignment, and contrast.
4. Paper **design review** (from MCP instructions): spacing rhythm, type hierarchy, contrast, alignment, clipping. Fix issues before shipping.
5. **Targeted fixes:** prefer **`update_styles`** for 1–3 property corrections. If horizontal padding does not stick as `paddingInline`, set **`paddingLeft` / `paddingRight`** explicitly (verified pattern).

Only then: `finish_working_on_nodes`.

## Default assumption

- **Prefer an existing Paper artboard** the user is already working in.
- Insert into a **dedicated parent frame** inside that artboard (e.g. `Spec from code — [Screen]`) so other explorations are not overwritten.
- **Create a new artboard only** if the user says there is no suitable target; size it from **layout constraints in code** (breakpoint, `max-w-*`, full bleed).

## Design tool & MCP

- **Primary:** **Paper MCP** (`user-paper` or equivalent). **Read tool schemas** (`write_html`, `get_basic_info`, `get_tree_summary`, `get_selection`, `get_computed_styles`, `get_screenshot`, `create_artboard`, `update_styles`, `finish_working_on_nodes`, etc.) before calling.
- **If Paper MCP is unavailable:** run **paste mode** (end of this skill).

### Operational limits (Paper MCP)

- **Tool quotas:** Paper’s MCP may enforce **weekly or session limits** on tool calls (`write_html`, `get_computed_styles`, …). If you get a quota or rate-limit response, **do not keep calling** — switch immediately to **[Paste mode](#paste-mode-no-mcp)**: emit chunked HTML, the full **measurement spec table**, and the **Pixel QA** checklist so the user can paste when quota resets or upgrade their Paper plan.
- **Partial pushes:** Record the last successful parent `targetNodeId`, layer names, and the **next chunks** still needed so a follow-up session can continue without redoing work.
- **`paddingInline`:** If `get_computed_styles` still shows wrong horizontal padding after `update_styles`, set **`paddingLeft` and `paddingRight`** explicitly.

## Workflow (Paper MCP)

1. **Discover context:** `get_basic_info`; use `get_selection` / `get_tree_summary` to resolve **`targetNodeId`** for the parent of new content.
2. **Pre-flight spec table** (pixel mode): as above.
3. **Write incrementally:** `write_html` in **small chunks** (e.g. chrome → header → sections → footer). After **2–3 chunks**, visually checkpoint (`get_screenshot`): spacing, type hierarchy, contrast, alignment.
4. **Modes:** Prefer **`insert-children`**; use **`replace`** only when the user asked to swap a specific node.
5. **Layers:** Use sensible names (`layer-name` on roots where supported; mirror real regions: header, body scroll, list, footer).
6. **Pixel QA:** run checklist; fix until pass.
7. **Done:** `finish_working_on_nodes` when stable **and** QA passed.

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
2. A short **checklist**: token references, key measurements, breakpoint width assumption, any **TBD — verify in code** items, and the **Pixel QA** spec table for manual verification.

---

## Installation

**Canonical source:** this repo — copy `push-to-paper/` to your agent skills directory, or vendor into an app repo under `.cursor/skills/push-to-paper/`.

```bash
# Claude Code
cp -r push-to-paper ~/.claude/skills/

# Cursor
cp -r push-to-paper ~/.cursor/skills/
```

Then use `/push-to-paper` or say "push this screen to Paper."
