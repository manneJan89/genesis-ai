---
description: Redesign one screen or shared component's look — keep behavior, replace the visuals
argument-hint: [screen or component] — attach/point to the new design
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
---

Redesign the look of: $ARGUMENTS

**This is a redesign, not a feature build.** Design *changes*; behavior is a
**Keep**. Your focus is the visual replacement. If the new design implies new
functionality, you **log it, you do not build it** (see Phase 6). This is the
inverse of `/genesis:improve-feature`, which keeps the design and changes behavior.

Read CLAUDE.md first — Standards, `COMPONENTS.md`, and the styling system.

## Phase 0 — Scope gate (one unit per run)
Resolve what's being redesigned to a single unit:
- **One screen/page** → proceed.
- **One shared component** (even if it appears on many screens) → proceed; it's one
  unit with one contract. Note its consumers — you'll verify them in Phase 5.
- **Multiple screens** → STOP. That's a roadmap: tell me to run `/genesis:roadmap
  <redesign>` and slice it one screen per run. Don't do a multi-screen redesign in
  one pass.

## Phase 1 — Render-surface audit (NOT a behavioral audit)
You need to know every visual state you must restyle — not how anything works.
Read the target's template AND its backing class/controller, but only to inventory
the **render surface**:
- conditional branches (`*ngIf`/`@if` etc.) → loading, empty, error, populated,
  permission-gated states — each needs styling, not just the default view;
- loops and their item variants (selected / disabled / active);
- bound classes and bound content values (e.g. an enum that drives status colours).
Produce a short list of the visual states/variants the redesign must cover. Do NOT
audit business logic, click handlers, or data flow — that's behavior, which you're
keeping, not changing. This is lighter than `/genesis:audit-feature` on purpose.

## Phase 2 — Ingest the new design → editable, in `design/`
Take the design I handed you (Claude Design export, HTML/CSS, or image) and write it
to `design/<unit>.html` following the **editable-design rule** (self-contained,
semantic, CSS variables, commented sections, hand-editable with no LLM — same as
`/genesis:design`). If I gave a Claude Design/HTML export, tidy it to meet that rule;
if an image, generate the editable HTML/CSS.

## Phase 3 — Approve the design (before any code)
Show me the design and the render-surface list, and confirm the new design covers
every state from Phase 1 (if the new design has no empty/error state but the screen
has one, ask me what those should look like — don't leave them un-redesigned or
guess). **Get my explicit approval of the look before writing a single line of app
code.** Design precedes build, like a designer handing off.

## Phase 4 — Safety net + replace the UI
- Delegate to the **characterization-tester** to net the target's **behavior**
  (what it does), so you can prove the redesign didn't break function. Confirm green
  before changing anything.
- Enter plan mode; state the touch budget (the template + styles you'll change).
  Approve, then replace the visuals: implement the approved design in the
  **project's styling system** (Tailwind/Bootstrap/theme + house components from
  `COMPONENTS.md`) — do NOT copy the design file's raw CSS, and do NOT change
  behavior, data flow, or logic. Restyle every state from the Phase 1 list. Minor
  pixel drift from existing project styles is fine.
- Per Standards: no raw elements where a house component exists, no emojis, SVG
  icons only, and any new interactive element is accessible.

## Phase 5 — Verify
- The behavior characterization net is **still green** — behavior unchanged.
- The result matches the approved design across all states.
- If a **shared component** was redesigned: check its consuming screens still render
  correctly (the blast radius) — the net covers the component; the consumers need a
  render check.
- Component manifest: if a shared component's look/API changed, update
  `COMPONENTS.md`.

## Phase 6 — Log new features, don't build them
If the new design implies functionality that doesn't exist yet (a new filter, a new
action, a field that isn't wired), **log it to `FINDINGS.md`** as a new feature to
spec separately — do NOT build it here. Redesign changes the look only; new
behavior goes through `/genesis:spec`. Tell me what you logged.

## Phase 7 — Asset gate + summary
- If the design needs an icon/image asset that wasn't provided, finish what you can
  but don't mark it `done` — **blocked-on-asset**, log a finding (no emoji/placeholder
  stand-ins).
- If a spec/roadmap slice tracks this, update its status.
- Summarize: states restyled, behavior net still green, consumers checked (if a
  shared component), new features logged for separate speccing, anything left open.
