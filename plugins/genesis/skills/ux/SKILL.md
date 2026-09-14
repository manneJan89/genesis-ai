---
name: ux
description: Design/UX reasoning for building or reviewing user interfaces — visual hierarchy, spacing and rhythm, the required UI states, affordance and feedback, form UX, accessibility, and avoiding the generic "AI-generated" look. Consult this whenever a task creates, redesigns, or reviews UI (a screen, page, component, form). Not relevant to backend, data, or pure-logic work.
---

# UX reasoning

Use this when building, redesigning, or reviewing UI. It's **principles, not a house
style** — it makes design decisions deliberate and correct; it does NOT impose one
look (every screen looking identically "Genesis-designed" is a failure). Force an
explicit choice per project/screen, then apply these principles to it.

How to apply it (ties into the command that called you):
- **Building/redesigning:** design against these principles. If the obvious
  implementation violates one, fix it rather than shipping it. Where there's a real
  judgment call or you can do materially better than asked, **prompt the user**;
  if there's no response, proceed with the better option and note the assumption.
- **Reviewing:** treat violations as findings, severity-ranked, with the concrete
  fix — not vague "improve the UX" advice.

## 1. Design for intent, not the schema
The most common AI-UI failure: rendering the data model instead of the user's job.
A 12-column table with every field, Edit and Delete equally weighted on every row,
is a schema browser, not a product. Before laying out a screen, know: **who uses
it, what they're trying to do, and the worst mistake they could make.** Then:
- Show the few columns/fields that serve the job; put the rest behind detail/expand.
- Make the *primary* action prominent; demote the rest.
- Guard the destructive action (see §5).

## 2. Visual hierarchy — rank everything
Every element is primary, secondary, or tertiary. If everything is emphasized,
nothing is. Enforce it with:
- **One clear primary action** per screen/section (one filled button); secondary
  actions are lower-weight (stroked/text), tertiary are quiet (icon/overflow).
- **Size, weight, colour, and position** signal importance — the eye should land on
  the most important thing first. Establish a scanning path (top-left→down for LTR).
- Don't compete: two equally-loud buttons force a decision the design should have
  made.

## 3. Spacing and rhythm
Sloppy spacing is the fastest "AI-made" tell.
- Use a **consistent spacing scale** (e.g. 4/8/12/16/24/32) — never arbitrary
  one-off pixel values. Reuse the project's tokens if they exist.
- **Group by proximity**: related things close, unrelated things apart. Whitespace
  is what creates groups, not borders everywhere.
- Consistent gutters and alignment — things line up on a grid. Ragged, near-but-not-
  aligned edges read as amateur.
- Give content room to breathe; cramped density signals "dumped the data."

## 4. All the states — not just the happy one
Every data-driven view needs its states designed up front, not just the populated
one. Cover, as applicable:
- **Loading** — a skeleton matching the final layout beats a spinner; first paint
  shouldn't feel broken.
- **Empty** — a helpful message + a next-step CTA ("Invite your first user"), never
  blank rows.
- **Partial / few items** — layout holds with 1 item as well as 100.
- **Error** — what went wrong and what to do, not a raw exception.
- **Success / confirmation** — visible feedback that the action worked.
This maps to the render-surface list a redesign inventories, and to the states tests
should cover.

## 5. Affordance and feedback
- **Clickable looks clickable**; non-interactive things don't. Hover/focus/active
  states exist and are visible.
- **Feedback is immediate** (<100ms perceived) — a button press acknowledges at
  once, even if the result is still loading (disable + spinner on the control).
- **Destructive actions are protected**: move Delete out of the equal-weight row
  into an overflow, and require confirmation (typed confirmation for the truly
  dangerous). Never sit Delete next to Edit at equal weight.
- **Disabled controls say why** — a disabled Save with no reason is a dead end;
  show what's missing.

## 6. Form UX
- **Group related fields**; order them the way the user thinks, not the way the
  table is columned.
- **Labels always visible** (not placeholder-as-label — it vanishes on input).
- **Validate helpfully**: inline, on blur/submit, saying how to fix it — not a wall
  of red on load.
- **Smart defaults** and progressive disclosure — don't show advanced options until
  needed. Every field you can remove or default is a win.

## 7. Accessibility (baseline, not optional)
- Keyboard operable end to end; visible focus; logical tab order.
- Correct roles/labels on interactive elements; meaningful `aria-*` where text
  doesn't convey it.
- **Colour is never the only signal** — pair it with icon/label/shape (a green
  "Active" text alone fails colourblind users; use a pill with icon + label).
- Adequate contrast and hit-target size.

## 8. Don't look AI-generated
Models fall back on a statistically-safe look; that sameness is the tell. Counter it:
- Make **one deliberate decision per axis** — type, colour, spacing, finish —
  rather than defaulting. State the choice.
- Common tells to avoid: everything centered, evenly-weighted everything, generic
  gradient hero, no real hierarchy, identical card grids, emoji as icons (banned
  anyway), placeholder-as-label, no empty/loading states.
- The lineup test: would this screen be indistinguishable from ten other AI UIs for
  the same prompt? If yes, it hasn't made a decision.

## Boundaries
- Principles guide; they don't override the project's chosen design system, an
  approved design file in `design/`, or `COMPONENTS.md`. Conform to those first,
  apply these within them.
- This is visual/interaction reasoning — it never changes behavior, data, or logic.
