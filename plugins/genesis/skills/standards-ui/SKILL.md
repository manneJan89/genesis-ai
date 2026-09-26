---
name: standards-ui
description: Genesis UI standards: mandatory house components (COMPONENTS.md), extend-never-fork, icons and assets (ask, never invent or emoji), accessibility for new interactive components. Load when building or reviewing UI: screens, components, forms, icons. Use together with the ux skill.
user-invocable: false
---

# UI standards

Part of the Genesis Standards. These rules are binding whenever they apply — they
were moved out of CLAUDE.md so they load only when relevant, not because they are
optional. Project-specific additions live in the project's CLAUDE.md under
**Conventions**; where the two conflict, the project's CLAUDE.md wins.

- **Use the project's shared UI components — this is mandatory, not preferred.**
  The project's component manifest (`COMPONENTS.md`) lists the house components and
  what each does. If a house component exists for a UI primitive (button, input,
  card, modal, etc.), a raw HTML/framework-native equivalent is **banned** — use
  the house component. Read `COMPONENTS.md` before writing UI; don't scan the whole
  codebase to rediscover components each time.
  - **Repeated markup is a component.** The same structural block appearing 2+
    times (in a file or across the project) should be extracted into a reusable
    component — flag it and propose the extraction.
  - **Gap handling: extend, never fork.** If a house component doesn't do something
    you need (an icon variant, a danger tone), do NOT abandon it and hand-roll a
    raw element or a second parallel component. Propose **adding the capability to
    the existing shared component** and ask me before doing it (mid-session is
    fine). One source of truth per primitive.
  - When you create or change a shared component, **update `COMPONENTS.md`** so its
    capabilities/limits stay accurate (see the manifest write-back rule).

- **Icons and assets: ask, don't invent, don't emoji.** When an icon/image is
  needed: if the project uses an icon library, ask which icon (I give a name); if
  it uses standalone SVGs/assets, ask me to provide the file. **Never substitute an
  emoji, invent an SVG, or ship a placeholder.** If the asset isn't provided,
  implement everything around it, log a finding (`FINDINGS.md`), and treat the
  feature as **blocked-on-asset** — it can't be marked `done` until the asset is in
  (see the completion gate in the build orchestrators). Extract an asset from a
  design only if I explicitly ask (and note a PNG design may not yield a usable
  vector).

- **New interactive components are born accessible.** When you build a genuinely
  new interactive component (not covered by a house component), it must be
  accessible from the start, not retrofitted later: keyboard reachable and operable
  (Tab to it, Enter/Space/arrows as appropriate), correct semantic role
  (`role=`/native element), visible focus state, labels/`aria-*` where meaning isn't
  conveyed by text, and sensible focus management (e.g. focus moves into a dialog
  and back on close). This is why house components are mandatory — they already bake
  this in; a hand-rolled equivalent that drops it is a defect. When such a component
  is added to `COMPONENTS.md`, note its accessibility support like any other
  capability.
