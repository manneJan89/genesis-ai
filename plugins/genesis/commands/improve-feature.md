---
description: Improve an existing feature from an approved audit spec, safely
argument-hint: [path/to/spec.md]
allowed-tools: Read, Write, Edit, Grep, Glob, Agent, Skill, Bash(rm HANDOFF.md), Bash(date *)
---

Orchestrate a safe improvement of an existing feature from the spec at: $ARGUMENTS

Read the spec first. If its status isn't `approved` or it has no "Current
behavior (as-built)" section, stop and tell me to run `/genesis:audit-feature`. Re-read
the spec whenever you delegate, and pass the relevant details in the delegation
prompt — subagents start fresh and only see what you give them. Delegate to
subagents **by name explicitly**.

**Read `${CLAUDE_PLUGIN_ROOT}/shared/parallel-and-checks.md` now.** It holds the
rules for running agents in parallel, timing phases, splitting work into areas
around a contract, and the checks and fix loop (Phases 5 and 6 below).

## Handoff (keep it current as you go)
This run keeps `HANDOFF.md` current so a later session can carry on if this one
ends. Format: `${CLAUDE_PLUGIN_ROOT}/templates/HANDOFF.template.md`.
- **At the start:** if a `HANDOFF.md` exists for different work, tell me before
  replacing it.
- **At the end of every phase:** overwrite it with where this run stands (phase
  reached, done, half-done, the exact next step, open questions, decisions not yet
  in the spec). Take **Branch** and **HEAD** from git. Keep it short. Don't commit it.
  Plan mode blocks file writes, so while it's on, skip the write and do it as
  soon as I approve the plan.
- **Whenever the run stops early** (waiting for my approval, a gate or stop rule,
  the fix-loop cap, blocked on an asset, or me telling you to stop): write it
  before you stop. That is when it matters most. Make **Next step** say what
  unblocks the work.
- **When the work is done:** move anything in its decisions or open questions
  that isn't recorded permanently into the spec, the roadmap or `FINDINGS.md`,
  then delete it with exactly `rm HANDOFF.md` (the one deletion this command
  pre-approves).

## Phase 1 — Safety net (before touching anything)
Delegate to the **characterization-tester** subagent to lock in every behavior
tagged **Keep** in the spec (one per area, in parallel, when the code spans
areas). If this run applies a new design (see Phase 4), the net pins
**behaviour, not markup**: what the screen does and shows, not its element
structure or classes, which are being replaced. Confirm the whole net is green
against the current,
unchanged code before proceeding. This is the step that makes the change safe —
do not skip it.

## Phase 2 — Plan (read-only)
Enter plan mode. Lay out the change: **exactly which files, and which functions/
sections within them, you will touch** (the touch budget), how it maps to the
target acceptance criteria, and — for a refactor — your assurance that the
characterization tests stay green throughout. Everything not on that list stays
untouched. If the change spans areas (backend, frontend, rules), split it as
**Areas and the contract** describes. For new UI, name the `COMPONENTS.md`
component each element will use.

**If this run applies a new design** (the spec points at an approved design in
`design/`), the plan also lists:
- the **render states** the screen has (loading, empty, error, populated,
  permission-gated, list item variants), each mapped to the part of the design
  that covers it; a state the design doesn't cover is a question for me, not a
  guess;
- **every edit to the backing class or controller**, one line each: what changes
  and why. A redesign should need few; any change to how a value is computed,
  or to what gets passed into a binding, is a behaviour change and must trace to
  an acceptance criterion. Recreating values in a getter or template for
  convenience is not allowed (see the Angular performance rules).

Stop for my approval of the touch budget before writing. Prefer small reviewable
chunks over one big pass.

**Data operations:** if the plan includes anything that writes to data outside a
local or emulator database (migration, backfill, bulk update or delete, data-fix
script, import), name it in the plan and apply the **data-operation gate** from
`genesis:standards-security`: you build and test it locally, I run it against
real data after confirming a backup.

## Phase 3 — Target tests first (red)
Skip for a pure **refactor** (no new behavior — the characterization net is the
test). Otherwise, scaffold any new public surface named in the plan (signatures
only, no logic) and delegate to the **test-writer** subagent in `tests-first` mode
to write unit tests for the *new/changed* acceptance criteria, derived from the
spec, before the change is made (one test-writer per area, in parallel). Confirm they fail for the right reason (assertion
or "not implemented", not a broken import). An absence test (denied user doesn't
see X) may pass already; keep it as long as its positive counterpart fails. These
sit alongside the characterization net, which must still be green.

## Phase 4 — Make the change (green)
Load the Genesis standards skills that apply (see CLAUDE.md → Standards), then
implement the approved plan: yourself for one area, parallel **area-builder**
subagents for several, then the join (see **Building per area**). **Modify only what the touch budget names; leave all
other code intact** (per the modification contract in Standards).

**The look: keep it, unless an approved design replaces it.**
- **No approved design for the screen:** the existing look is a Keep. Don't
  restyle or re-lay-out it; wire new data *into* the existing widgets. New UI on
  an existing screen needs its design gate settled first (a design in
  `design/`, or an approved "assume"); never improvise a look.
- **The spec points at an approved design in `design/`:** build the new look and
  the new behaviour together in this run, without asking. Implement the design in
  the project's styling system and house components (`COMPONENTS.md`), never by
  copying the design file's raw CSS, and style every render state from the plan.
  The approved design is the only UI source: anything it doesn't show stays as
  it is or comes to me.
- **Look only, behaviour unchanged:** that's `/genesis:redesign`, not this
  command.
- **Change type = refactor:** the characterization net must stay green the entire
  time; run the related part of it after each step. Green net = behavior
  preserved.
- **Change type = extend / bugfix:** the behaviors tagged **Change** or **Wrong**
  will make their characterization tests fail — that's expected. Update those
  specific tests to the new target behavior; the Keep tests must still pass.
- **Older tests from earlier specs** that now fail follow CLAUDE.md's "trace
  each one, then act": updated without asking when the failure traces to a
  change this spec makes, the code fixed when it doesn't, and only a genuinely
  unclear one comes to me.

Make the Phase 3 tests pass without editing them (new tests are never edited to
pass). If one looks wrong, stop and
show me rather than bending it. Run the lint/type-check command before moving on.

## Phase 5 — Checks, in parallel
Follow **Checks, in parallel** in the shared file: e2e-tester (including the
untouched characterization tests, which guard against regressions),
security-reviewer (when the change touches the trust boundary) and
component-moderator (when it touched UI), launched together.

**Design check (only if this run applied a new design):** after the checks,
walk the built screen against the design file, element by element and state by
state (every render state from the plan): each element present, using the right
house component, in the right place, with the design's content and states. List
mismatches as defects for the fix loop. Minor pixel drift from existing project
styles is fine.

## Phase 6 — Fix loop
Follow **Fix loop** in the shared file. The Keep characterization tests must be
green at the end; a fix that breaks one is a regression, not progress.

## Phase 7 — Performance
Verify the spec's performance budget with a real measurement. For refactors aimed
at performance, report before-vs-after numbers. If no budget, say so.

## Phase 7.5 — Component manifest + completion gate
- If a shared UI component was created or changed, update `COMPONENTS.md`.
- If a required icon/image/asset wasn't provided, finish the code but do **not**
  mark the feature `done` — set **blocked-on-asset**, log a finding, and tell me.
  No emoji/placeholder stand-ins. I can override.

## Phase 8 — Update the roadmap (if any)
If the spec header names a roadmap (`Roadmap: specs/roadmaps/<name>.md (slice N …)`),
open that roadmap and:
- set this slice's status from the outcome, not the code: `done` only if Phase
  7.5 passed; if the work ended **blocked-on-asset** (or blocked for any other
  reason), set `blocked — <what's missing>` and leave **Next up** pointing at
  this slice,
- fill in its Spec column with a relative link to the spec,
- if (and only if) it's `done`, move **Next up** to the first remaining non-done
  slice,
- update Last updated.

If a shared decision changed during this build (schema, enum, side-effect rule),
update the roadmap's Shared foundations too and tell me what changed — later
slices depend on it. Do NOT rewrite older specs; they are point-in-time history.

If the spec names no roadmap, skip this phase.

## Phase 9 — Summary
Short. A few lines, in this shape, and nothing else unless something needs me:

    Changed: <one line>
    Tests: <passed>/<total>; Keep net green; <n> characterization tests updated (Change/Wrong)
    Updated: <n> older tests to the spec's changes (test → spec line, one per line), or "none"
    Checks: security <clean | n fixed | skipped: why>, components <clean | n swapped>, design <matched | n fixed | not applied>
    Time: net <m>, plan <m>, tests <m>, build <m>, checks <m>, fixes <m>
    Needs you: <(manual) criteria, partial component matches, open items; or "nothing">

Add performance numbers only if the spec had a budget, and a line per updated
characterization test only if one changed for a reason other than the spec's
Change/Wrong tags.

If the work is finished, clear `HANDOFF.md` as described under Handoff. If it
ended blocked, leave it, with **Next step** saying what unblocks it.
