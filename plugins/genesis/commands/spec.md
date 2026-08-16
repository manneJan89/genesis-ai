---
description: Interview me about a feature, then write an approved spec file
argument-hint: [feature-name or short description]
allowed-tools: Read, Write, Grep, Glob
---

You are running the **planning + spec** phase for: $ARGUMENTS

This phase is interactive and happens here in the main conversation — do NOT
delegate it to a subagent (subagents can't interview me).

Follow the template at @specs/_TEMPLATE.md.

## 0. Check for a roadmap (before interviewing)
Glob `specs/roadmaps/*.md` and look for a slice matching $ARGUMENTS.

- **Exactly one match** — say so plainly ("This is slice N of the <name> roadmap"),
  list the shared decisions it must respect (data shape, enums, cross-cutting
  flags, side-effect rules, failure semantics, reused components), and treat those
  as **fixed constraints** — do not re-open them in the interview. Record the
  back-link in the spec header:
  `Roadmap: specs/roadmaps/<name>.md (slice N — <slice-name>)`
  Mark that slice `in progress` in the roadmap.
  Note the slice's `new`/`modify` type in the spec for context, but the build step
  is always `/genesis:build-feature` — it detects and protects existing code
  itself, so you don't switch commands for modify slices.
  Also read the roadmap's **Decisions** — treat them as settled rationale, not open
  questions. If this slice would contradict a recorded decision, flag it to me
  explicitly rather than silently going along; that's a decision to revisit at the
  roadmap level, not to quietly override in a slice spec.
- **No match** — proceed standalone. A roadmap is optional; most one-off features
  don't need one.
- **Ambiguous / fuzzy match** — ask which roadmap and slice I mean. Don't guess.

If the request is clearly a whole system rather than one feature (it would need
several specs, or the interview keeps expanding in scope), stop and recommend
`/genesis:roadmap <system>` instead of writing a mega-spec.

## Your job

1. **Interview me to extract the real requirement.** Ask focused questions
   **one at a time** and wait for my answer before the next. Don't dump a
   questionnaire. Prioritize the questions that most reduce ambiguity:
   - What problem does this solve, and who for?
   - What is explicitly out of scope?
   - What are the concrete acceptance criteria (including edge and error cases)?
   - Are there performance targets?
   - **Security (any feature touching data, endpoints, or user input):** Is this
     public or authenticated? Who is authorized — which roles/owners, and is it
     per-resource (can user A act on user B's record)? What input crosses a trust
     boundary, and what's the server-side validation for it? Any secrets/config
     involved (must come from env, never hardcoded)? Default to requiring auth
     unless I say it's genuinely public.
   - **Scale:** expected volume for any list/query here — does it need pagination,
     and are the filtered/sorted fields indexed?
   - What existing code/constraints must this respect or avoid touching?
   - **If this feature has UI, run the DESIGN GATE before finalizing the spec.**
     (Skip entirely for backend/database/API/logic — no design needed there.)
     1. If a roadmap set a **Design source**, inherit it.
     2. Check the `design/` folder for a stored design for this screen:
        - **Design exists and covers this feature** (e.g. the report-page design
          already shows a search bar) → say so, reference it, proceed.
        - **Design exists but doesn't cover this feature** → STOP and ask: "the
          <screen> design doesn't include <feature> — will you update the design,
          or should I assume it?"
        - **No design exists** → STOP and ask: "there's no design for <screen> —
          hand me one (image, HTML/CSS, or Claude Design export), or should I
          assume it?"
     3. **Only if I say assume:** design it FIRST, before any build. Produce a
        human-editable HTML/CSS file into `design/` (see the editable-design rule)
        and **get my approval of the design as part of approving the spec.** No
        code is written until the look is signed off — design precedes build.
     Reuse existing components before designing new ones (DRY), and respect the
     project's **Component libraries** setting. Record the resolved design source in
     the spec's UI/design section. The spec is not `approved` until the design gate
     is resolved (design in hand/approved, or explicitly deferred by me).

     **Editable-design rule** — anything written to `design/` MUST be hand-editable
     by a non-LLM human:
     - one self-contained `.html` file, styles in a `<style>` block, no external
       sheet, not minified, indented;
     - semantic shallow structure (`<header>`, `<nav>`, `<button>`), not deep
       `<div>` nesting;
     - commented sections (`<!-- Search bar -->`, `<!-- Results -->`);
     - colours/spacing/type as CSS variables at the top so one edit changes many;
     - a short header comment explaining how to edit it.
     Test it must pass: I can open it, change a colour and a position, save, refresh,
     and see the change — no LLM involved. Hand HTML/CSS or a Claude Design export →
     store in `design/` as-is (tidy to meet the rule). Hand an image → generate the
     editable HTML/CSS from it. (This conversion is exactly what `/genesis:design`
     does — you may run its logic inline here, or tell me to run `/genesis:design`
     first if I'd rather iterate on the look before speccing.) When this design is
     later coded into the app, use the **project's** styling system
     (Tailwind/Bootstrap/theme), NOT the design's raw CSS; minor pixel drift is fine.

2. **Push back and improve the idea.** If something is vague, contradictory,
   over-scoped, or missing an obvious edge case, say so and propose a sharper
   version. I'd rather you challenge me now than build the wrong thing later.
   Suggest alternatives where they'd be better.

3. **Explore the codebase (read-only) as needed** to ground the spec in what
   actually exists — but keep exploration brief.

4. **When we've converged, write the spec** to `specs/<kebab-case-name>.md`
   using the template. Fill every section. Move anything unresolved into
   "Open questions" rather than guessing.

5. **Show me the finished spec and ask me to confirm.** Only set
   `Status: approved` after I explicitly approve (and, for UI features, after the
   design gate is resolved). Then tell me to run:
   `/genesis:build-feature specs/<name>.md` — it's the single build entry point and
   detects for itself whether this is new code or a change to existing code,
   protecting existing work automatically.

Do not write any implementation code in this phase.
