---
description: Turn a handed design (image, HTML/CSS, or Claude Design export) into a human-editable HTML file in design/
argument-hint: [screen/page name] — attach or point to the design
allowed-tools: Read, Write, Edit, Grep, Glob
---

Produce a human-editable design file for: $ARGUMENTS

This creates the stored, tweakable source of truth for a screen's look, saved in
`design/`. It does NOT write any app code — `/genesis:spec` and
`/genesis:build-feature` consume this file later and re-implement it in the
project's styling system.

## 1. Get the input
Identify what I've handed you:
- **An image** (screenshot, mockup, photo of a sketch) → you'll build editable
  HTML/CSS that reproduces it. Pixel-perfect isn't the goal; a clean, close,
  editable version is (I'll nudge it myself afterward).
- **HTML/CSS** (a file, a snippet, or a Claude Design export) → adopt its structure
  and look, then TIDY it to meet the editable rule below (exports are often dense
  and unreadable — that defeats the purpose).
- **Nothing attached** → ask me for the image or file before proceeding. Don't
  invent a design here; inventing belongs in `/genesis:spec`'s design gate where I
  approve it.

If a design file for this screen already exists in `design/`, ask whether to
update it or create a new one — don't silently overwrite my edits.

## 2. Write it — the editable rule (hard requirement)
Write ONE self-contained file to `design/<kebab-name>.html`. It must be editable by
a human with no LLM involved:
- Styles in a single `<style>` block — no external stylesheet. Not minified;
  indented and formatted.
- **Design tokens as CSS variables at the top** (`:root { --color-primary; --gap;
  --radius; --font-size-… }`) so I change one value, not twenty.
- **Semantic, shallow structure** — real elements (`<header>`, `<nav>`, `<main>`,
  `<button>`), not deep `<div>` nesting.
- **Commented sections** marking each region (`<!-- Search bar -->`,
  `<!-- Results list -->`, `<!-- Footer -->`).
- A short **header comment** telling me how to edit it: change the variables for
  colour/spacing/type, edit the marked sections for layout.
- Uses placeholder/sample content so it renders standalone in a browser.

**Acceptance test for the file:** I can open it in a browser, change a colour
variable and move a section, save, refresh, and see the change — without asking an
LLM. If it wouldn't pass that, it's not done.

## 3. Hand back
Tell me the path (`design/<name>.html`), remind me I can open and edit it directly,
and note that when this screen is built, `/genesis:build-feature` will re-implement
it in the project's styling system (Tailwind/Bootstrap/theme) — it won't copy this
file's raw CSS, and minor pixel drift from existing project styles is expected.

Do not write application code. This command's only output is the design file.
