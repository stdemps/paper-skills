---
name: build-from-paper
description: >
  Match code to a Paper design frame — pixel-perfect implementation from design.
  Use when the user says build from paper, match paper, match design, code from paper, /build-from-paper,
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
   - `get_jsx` (format: `tailwind`) — get the Tailwind-annotated JSX as a **structural reference**. Do NOT copy-paste this directly — it uses hardcoded hex values and raw HTML that won't match the project's conventions.
   - Parse the tree summary + JSX for: layout hierarchy, text content (exact strings), spacing values, colors (hex), font sizes/weights/families, border radii, opacity.
   - Note every concrete value — these are what the code must reproduce.

3. **Locate the code**
   - Search the codebase for the matching component/page using: layer names, text content, route structure, file names.
   - If no matching code exists, create a new page/component in the appropriate location.
   - If ambiguous, ask the user which file maps to this frame.
   - Read the component files and all imported sub-components.

4. **Inventory existing components**
   - Scan the project's `components/` directory (especially `components/ui/`) to discover reusable components.
   - Read each relevant component to understand its API: variants, sizes, props.
   - Map design elements to existing components. Common mappings:
     - Buttons (outlined, filled, ghost) → project's `Button` component with appropriate `variant`
     - Pills/tags/labels → project's `Badge` component
     - Cards → project's `Card` component (or card token classes)
     - Inputs, dialogs, dropdowns → project's existing UI primitives
   - **Always prefer an existing component over raw HTML.** Only use raw elements when no suitable component exists.
   - When using a component, override styles via `className` prop to match the design — don't fight the component's API.

5. **Understand existing tokens**
   - Read the project's design system: Tailwind config, CSS variables, theme files.
   - Map Paper's pixel/hex values to the closest existing tokens.
   - If a Paper value has **no matching token**, flag it to the user — don't invent tokens.
   - **Dark mode awareness**: Never use hardcoded hex/rgba values for backgrounds, text, or borders. Always use CSS variable-based tokens (`bg-card`, `text-foreground`, `border-border`, etc.) that adapt automatically to light/dark mode. Inline `style={{}}` with hardcoded colors is a red flag — use Tailwind classes with project tokens instead. The only exception is truly unique values that have no token equivalent (e.g. a specific status color).

6. **Produce an inline diff checklist**
   Before editing, list the specific differences:
   - Spacing: e.g. "`gap-4` in code → frame shows ~12px gap, should be `gap-3`"
   - Colors: e.g. "header bg is `bg-zinc-900` → frame shows `#1a1a2e`, map to `bg-primary`"
   - Typography: sizes, weights, tracking, line height
   - Layout: flex direction, alignment, order of elements
   - Content: label/copy differences
   - Missing or extra elements
   - **Components**: which design elements map to existing components

7. **Edit code**
   - Make targeted changes using the project's own classes/tokens and components.
   - **No inline styles** unless the project already uses them or no token exists.
   - **Preserve component logic and behavior** — only change visual styling.
   - Work incrementally — small, reviewable changes, not full rewrites.
   - **Accessible by default (WCAG 2.1 AA minimum)**:
     - Use semantic HTML: `<main>`, `<section>`, `<nav>`, `<h1>`–`<h6>` (in order), `<button>`, `<a>`, `<label>`, `<input>` — not `<div>` for everything.
     - All images need descriptive `alt` text (derive from the Paper layer name or visible caption). Decorative images get `alt=""`.
     - Buttons and links must have accessible names. Icon-only buttons need `aria-label`.
     - Form inputs must be associated with `<label>` elements (use `htmlFor`/`id` or wrap the input).
     - Color contrast: verify text/background combinations meet 4.5:1 for normal text, 3:1 for large text. If the design's colors fail contrast, flag it to the user rather than silently shipping inaccessible output.
     - Interactive elements must have visible focus states. Use the project's existing focus ring styles (e.g. `focus-visible:ring-*`).
     - Don't use `tabIndex` unless necessary. Native interactive elements are already focusable.
     - Status indicators that rely on color alone (e.g. a green dot) need a text label alongside them.
   - **Responsive by default**: The Paper frame represents one breakpoint. Write code that works across screen sizes:
     - Use `aspect-ratio` instead of fixed pixel heights for images (e.g. `aspect-video`, `aspect-square`, or `aspect-[W/H]` derived from the frame dimensions).
     - Multi-column grids should stack on mobile: use `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3` or `flex flex-col md:flex-row` patterns.
     - Use responsive text sizes where appropriate (e.g. `text-2xl md:text-3xl`).
     - Use `max-w-*` with responsive padding instead of fixed widths.
     - Container padding should adapt: `px-4 md:px-8` rather than just `px-8`.
     - If the frame name includes a viewport hint (e.g. "lg · 1024×1700"), treat the design as the **large breakpoint** and ensure smaller sizes degrade gracefully.
   - **Interactive polish**: The Paper frame is static — it can't show hover, active, or transition states. Add appropriate interaction feedback:
     - Links and clickable text need `hover:text-foreground transition-colors` (or a similar hover shift using project tokens).
     - Cards that act as links should get a subtle lift: `hover:shadow-md transition-shadow` or `hover:border-border transition-colors`.
     - Use `transition-colors`, `transition-shadow`, or `transition-all` on any element whose appearance changes on hover — don't let things snap.
     - Existing components (Button, Badge) already handle their own states — only add hover/transition to raw elements.

8. **Verify**
   - If a dev server is running, take a browser screenshot and compare against the Paper screenshot.
   - Check **both light and dark mode** if the project supports theming.
   - **Check mobile width** (~375px) to confirm layouts stack and nothing overflows.
   - **Flag image loading concerns**: If the page has large or remote images, flag to the user whether above-fold images should use `priority` (Next.js) or lazy loading, and whether `sizes` should be set for responsive images.
   - **Flag content overflow risks**: If card descriptions or headings could vary in length, flag to the user whether truncation (`line-clamp-*`) or overflow handling (`min-w-0` on flex children) is needed.
   - Flag any remaining gaps.

## Rules

- **Exact strings from Paper** — don't change copy unless it's clearly placeholder.
- **Never invent tokens** — use what the project already has. Flag gaps to the user.
- **Prefer existing components** — always use the project's UI components (Button, Badge, Card, etc.) over raw HTML elements. Match the design by selecting the right variant and overriding via `className`.
- **Dark-mode safe colors** — never use hardcoded hex/rgba for backgrounds, text, or borders. Use CSS variable-based tokens that adapt to both modes. Only use inline color values for truly unique values with no token equivalent.
- **Paper JSX is a reference, not copy-paste** — the `get_jsx` output provides structure and values but uses raw HTML and hardcoded colors. Always translate to project components and tokens.
- **Responsive by default** — the Paper frame is one breakpoint, not the only layout. Use fluid sizing (`aspect-ratio`, `max-w-*`, responsive grid columns) so the output works from mobile to desktop without a separate mobile design.
- **Accessible by default (WCAG 2.1 AA)** — semantic HTML, descriptive alt text, associated labels, 4.5:1 contrast ratio for text, visible focus states. Flag contrast failures to the user rather than shipping inaccessible output.
- **Interactive polish** — the design is static but production isn't. Add hover states and transitions to raw interactive elements. Existing components handle their own states.
- **Preserve behavior** — this skill changes appearance, not functionality.
- **Incremental edits** — small targeted changes, not rewrites.
- **Project conventions** — follow the existing code patterns (naming, file structure, component composition).

## Installation

Copy this folder to your AI coding agent's skills directory:

- **Claude Code:** `~/.claude/skills/build-from-paper/`
- **Cursor:** `~/.cursor/skills/build-from-paper/`

Then use `/build-from-paper` or say "build from the [frame name] frame."
