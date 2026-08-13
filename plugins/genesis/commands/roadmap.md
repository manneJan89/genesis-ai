---
description: Plan a multi-slice system — interview, settle cross-cutting decisions, and break it into specs
argument-hint: [system/epic name, or blank to show status]
allowed-tools: Read, Write, Edit, Grep, Glob
---

Plan the system: $ARGUMENTS

## If called with no arguments
List the roadmaps in `specs/roadmaps/`, and for each show its **Next up** slice
and slice statuses. Then stop. This is the "where did we leave off" view.

## What this command is for
A **roadmap** plans a system too big for one spec (a quoting module, a booking
flow) by settling the decisions that span slices and breaking the work into
slice-sized features. Each slice later gets its own `/genesis:spec`.

**A roadmap is a thin index, not a mega-spec.** Hard rule:

> It must be **impossible to implement anything from the roadmap alone.**

It holds the *shape* of a decision ("create → sends supplier email"); the slice's
spec holds the *detail* (which template, retry policy, acceptance criteria). If
you find yourself writing acceptance criteria or behavior steps, stop — that
belongs in a spec.

If the request turns out to be a single feature, say so and tell me to use
`/genesis:spec` instead. Don't manufacture slices.

## 1. Explore first (read-only)
Read CLAUDE.md (Standards, Conventions, Component libraries, Codebase map). Check
for existing code, models, or services this system would touch or reuse — per the
DRY standard, existing components must be named as dependencies, not rebuilt.

## 2. Interview me — this is the main event
Ask **one question at a time** and wait. Push back, propose alternatives, and tell
me what I appear to be missing. Be direct: if a plan has a flaw, say so.

**Grill hard on what is expensive to change later. Leave cheap decisions to the
slice specs.**

- Expensive (ask now): data shape and schema, enums and their value sets, state
  and lifecycle transitions, side-effect rules, failure/partial-failure semantics,
  permissions, soft-delete and its consequences for every read path, cost shape on
  metered services, which existing components get reused.
- Cheap (do NOT ask now): form layout, copy, button placement, which error message
  appears where, per-screen validation detail.

Cover at least:
- **Lifecycle** — what states exist, which transitions are legal, what happens at
  the ends (can it be edited after X? is anything versioned or audited?).
- **Side effects** — for each operation, what else must happen. Ask whether there
  is a *rule* behind them or whether they're ad hoc; an ad hoc pattern now means
  an arbitrary one later.
- **Failure semantics** — if the main write succeeds and a side effect fails, what
  is the correct outcome? Ask this for each side effect.
- **Cross-cutting flags** — anything (soft delete, tenancy, archiving) that must
  exist in the schema from slice one because later reads depend on it.
- **Permissions** — who may do each operation.
- **Security model** — the system-wide decisions: how callers authenticate, the
  authorization model (roles? per-resource ownership?), which endpoints are public
  vs private, where the trust boundary is, and where secrets/config live. Settle
  this once so every slice inherits it instead of re-deciding per endpoint.
- **Scale shape** — expected data volumes and growth; which entities need
  pagination and indexing from the start. Anything that must be built to scale in
  slice one because retrofitting it later is costly.
- **Cost** — expected volumes, and whether any read pattern grows unbounded.
- **Design source** — how UI should be produced across this system. Establish it
  once here so every slice inherits it. Resolve in this order and record the
  outcome under Shared foundations:
  1. **Existing design system in the codebase** — theme, design tokens, a shared
     component library, established layout patterns. Detect these; if present, they
     are the default and slices conform to them.
  2. **A design file/export in the repo** — mockups, a style guide, exported specs.
     Name the path.
  3. **An external reference I point you at** — e.g. a Figma file (note: you can
     only read it if the Figma MCP connector is set up — if the design lives there
     and it isn't connected, tell me to connect it rather than guessing), or a
     design exported/pasted from Claude Design into the repo.
  4. **Your own initiative, constrained** — if none of the above, build UI
     consistent with whatever partial conventions exist (framework defaults, any
     tokens found).
  5. **Invent one (last resort)** — only if there's genuinely nothing to follow.
     State the conventions you chose (spacing, colour, type scale) so they're
     reviewable and can become the system.

When I say I don't want something, ask whether I've **decided against it** or
simply **hadn't considered it**. Record the two differently: a deliberate
exclusion goes under "Deliberately not doing" **with its reason**, so it isn't
re-litigated in every slice interview.

For each **non-obvious cross-cutting decision** we settle (a schema choice, an
enum's value set, a side-effect rule, a failure policy), capture a short
**Decision** entry: what was chosen, *why*, and the alternatives we rejected and
why not. This is the part that stops a future session from reopening settled
ground or mistaking a deliberate call for an arbitrary one. Offer a recommended
answer with each question and let me override it — record which way it went.

## 3. Propose the slicing
Propose an ordered set of slices and explain the ordering. Rules:
- Cut **vertically** — each slice should be a thin end-to-end path that is
  independently shippable and verifiable. Not "all the models", then "all the UI".
- Merge slices that aren't independently verifiable (e.g. create + view usually
  ship together; view alone proves nothing and create alone can't be checked).
- Slice one carries the shared foundation — say so explicitly.
- Name each slice in **kebab-case**; that name becomes its spec filename and is
  how `/genesis:spec` finds it.
- Flag cross-slice dependencies, especially anything slice one must include for a
  later slice to work.

Let me correct the slicing before you write anything.

## 4. Write the roadmap
Write to `specs/roadmaps/<kebab-case-name>.md` using
`${CLAUDE_PLUGIN_ROOT}/templates/roadmap-template.md`. All slices start as
`planned`; set **Next up** to the first one. Keep it short — if it's running past
~40 lines, you're writing a spec, not an index.

Use relative markdown links (`[create](../quote-create.md)`), never wikilinks, so
they work on GitHub and in any markdown editor.

## 5. Hand off
Show me the roadmap and confirm. Then tell me to start the first slice with
`/genesis:spec <first-slice-name>`, which will pick up the roadmap's shared
decisions automatically.

Remind me the roadmap is a **living document**: revise it as slices are built and
we learn. Later slices are expected to change — a stale roadmap misleads, whereas
old specs are harmless history and should NOT be rewritten.
