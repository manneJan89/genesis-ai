---
description: Build a feature from an approved spec, then test, fix, and verify it
argument-hint: [path/to/spec.md]
allowed-tools: Read, Write, Edit, Grep, Glob, Agent, Skill, Bash(rm HANDOFF.md), Bash(date *)
---

Orchestrate the full build pipeline for the spec at: $ARGUMENTS

Read that spec first. If its status is not `approved`, stop and tell me to run
`/genesis:spec` to finish it. The spec is the source of truth for every phase below —
re-read it whenever you hand work to a subagent, and pass the relevant criteria
in the delegation prompt (subagents start with a fresh context and only see what
you give them).

Delegate to subagents **by name explicitly** at each step — don't assume Claude
will auto-route to them.

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

## Phase 0 — Route silently (new vs build-upon-existing)
`build-feature` is the single entry point: "I planned it, now build it" — whether
or not the target already exists. Detect which case you're in and handle it; do
NOT ask me to run a different command.

Glob/grep for the files, screens, or modules this spec targets:
- **Targets don't exist → new construction.** Create them fresh.
- **Targets already exist → build UPON them.** You are adding to existing code, not
  replacing it. Before changing an existing file, lay a **characterization net**
  over its current behavior (delegate to the **characterization-tester** subagent)
  so any accidental change to existing behavior turns a test red. When the
  touched files fall in more than one area, run one characterization-tester per
  area, in parallel. Then apply the
  **modification contract**: add/modify only what this spec names, and leave all
  other code — and all existing UI/design — byte-for-byte intact. Never rewrite a
  working page/function the spec didn't ask you to change. This is what stops a new
  spec from killing earlier work.

Handle both cases in this one command. The only thing you surface to me here is the
reuse check below.

## Phase 0b — Reuse check (copy-paste is a defect)
Before building, search the codebase for what you're about to create:
- **Exact match found** (behaviorally identical component or function already
  exists) → do not duplicate it. Tell me it exists and recommend reusing it; if the
  spec needs a variation, propose extracting the shared part so both use one copy.
  Ask before extracting (it touches existing call sites).
- **Only partially similar** → mention it, but **default to leaving it separate**
  (KISS over DRY — don't force merely-similar code into one flag-driven
  abstraction). Extract only if I confirm the shared concept is real.
- **Reusable-forward**: if what you're building is generic enough to be reused
  elsewhere, ask whether to make it a shared component (respecting the project's
  Component libraries setting) rather than burying it inline.

"Exact" means behaviorally identical (same job, different variable names still
counts), not character-identical.

## Phase 1 — Plan (read-only)
Enter plan mode. State the **touch budget**: for new code, the exact files you'll
create; for existing code, the exact files AND the functions/sections within them
you'll modify — everything else stays untouched. Name the **public surface** you'll
add (functions, classes, components, endpoints — with signatures), since the tests
in Phase 3 are written against it before any logic exists. Map the work to each
acceptance criterion.

If the feature spans more than one area (backend, frontend, security rules, a
game module), split it as **Areas and the contract** describes.

**UI: name the house component for each element.** For every button, input,
dialog, list or other UI element the plan adds, name the `COMPONENTS.md`
component it will use, or "raw — none registered". Choosing at plan time is
cheaper than swapping after the build.

Stop and let me approve the touch budget (and the area split, if any) before
writing code. For anything large, propose reviewable chunks rather than one big
pass.

**Data operations:** if the plan includes anything that writes to data outside a
local or emulator database (migration, backfill, bulk update or delete, data-fix
script, import), name it in the plan and apply the **data-operation gate** from
`genesis:standards-security`: you build and test it locally, I run it against
real data after confirming a backup.

## Phase 2 — Design gate (UI features only)
If this feature renders UI, **consult the `ux` skill** and apply its reasoning
(hierarchy, spacing, the required states, affordance, accessibility, avoiding the
generic AI look). The spec must have a resolved **design source** (a file in
`design/`, or an explicit "assume" that was approved). If the spec's design gate
is unresolved, stop and tell me to settle it in `/genesis:spec` first — do not
invent a design at build time. When a stored design exists, implement it in the
**project's styling system** (Tailwind/Bootstrap/theme — whatever the project
uses); do not copy the design file's raw CSS. Minor pixel drift from existing
project styles is acceptable.

Skip this phase for backend/data/logic-only features.

## Phase 3 — Tests first (red)
Unless the spec's Test plan says to skip unit tests, the tests are written
**before the implementation**, from the spec alone:

1. **Write the contract, then scaffold the surface.** If the plan has areas,
   write the contract files first, in full (types and shapes are not behavior).
   Then create the files and public signatures named in the plan with no
   behavior — bodies that throw "not implemented" (or the stack's equivalent).
   This is only so the tests compile; write no logic. For build-upon-existing,
   lay the characterization net first (Phase 0).
2. **Delegate to the test-writer subagent in `tests-first` mode**, one per area,
   all in the same message so they run in parallel (see **Areas and the
   contract**):

       Use the test-writer subagent in tests-first mode to write unit tests for
       <spec path>, <area> only (<folders>). The approved surface is:
       <signatures>. The contract is: <contract files>. Derive every test from the
       acceptance criteria for this area; no implementation exists yet. Frontend
       tests mock the backend through the contract.

3. **Confirm red.** Every new test must fail, and fail for the right reason — an
   assertion or "not implemented", not a syntax error, bad import, or broken
   fixture. A new test that passes against the empty scaffold is testing nothing;
   send it back to the test-writer. **Exception:** an absence test (denied user
   doesn't see X, empty list renders nothing) passes on the scaffold by nature.
   Keep it, as long as its positive counterpart exists and fails. Characterization
   tests (build-upon-existing) must still be green.

Writing tests first is the point of a separate test-writer: tests written after
the code tend to encode what the code does, bugs included. Tests written from the
spec against an empty surface can only encode what the spec promised.

## Phase 4 — Build (green)
Load the Genesis standards skills that apply to this feature (see CLAUDE.md →
Standards) before writing code. Respect the "Technical notes / constraints" and
the out-of-scope list in the spec, and stay strictly within the approved touch
budget. If building upon existing code, wire new behavior *into* existing
structure (for UI, into the existing widgets/layout), never rebuild what's
already there.

Build as **Areas and the contract → Building per area** describes: yourself
for one area, parallel **area-builder** subagents for several, then the join
(affected tests and lint once).

**Don't edit a new test to make it pass.** If a Phase 3 test looks wrong (it
contradicts the spec, or asserts something the spec never said), stop and show me
the test, the criterion, and why you think it's wrong. The test or the spec
changes only with my say-so. **Existing tests** that fail because of this change
follow CLAUDE.md's "trace each one, then act": updated without asking when the
failure traces to a change in the spec, the code fixed when it doesn't.

## Phase 5 — Checks, in parallel
Follow **Checks, in parallel** in the shared file: e2e-tester, security-reviewer
(when the change touches the trust boundary) and component-moderator (when it
touched UI), launched together.

## Phase 6 — Fix loop
Follow **Fix loop** in the shared file (related tests while fixing, re-run only
checks that had findings, cap 3 cycles and 2 for security, affected tests once at
the end).

## Phase 7 — Performance
Check the spec's performance budget. If one exists, verify it with a concrete
measurement (a benchmark, a timed run, a load test — whatever fits). Report the
numbers against the target. If no budget was specified, say so and move on.

## Phase 7.5 — Component manifest + completion gate
- **Manifest write-back:** if this feature created or changed a shared UI
  component, update `COMPONENTS.md` (use-for, API, can/can't) so it stays the
  authoritative registry. The component-moderator's "unregistered" list goes
  here. If the project has a component guard test (`/genesis:components
  guard`), add the new component's Replaces list to it.
- **Completion gate — required assets present?** If the feature needs an
  icon/image/asset that was NOT provided (see the icons-and-assets standard), you
  may finish all the code, but you **must NOT report the feature `done`.** Mark it
  **blocked-on-asset**, ensure a finding is logged in `FINDINGS.md`, and tell me:
  "Code complete, but <asset> is still missing — provide it and I'll finish, or add
  it yourself and I'll verify and mark it done." Any placeholder or emoji stand-in
  is not acceptable. I can override and mark it done myself; that's my call.

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

    Built: <one line>
    Tests: <passed>/<total> (<n> new, <n> of them security); affected tests green (<n> projects)
    Updated: <n> existing tests to the spec's changes (test → spec line, one per line), or "none"
    Checks: security <clean | n fixed | skipped: why>, components <clean | n swapped>
    Time: plan <m>, tests <m>, build <m>, checks <m>, fixes <m>
    Needs you: <(manual) criteria to check, partial component matches, open items; or "nothing">

Add performance numbers only if the spec had a budget. Don't recap the steps or
list every file; the diff shows that.

If the work is finished, clear `HANDOFF.md` as described under Handoff. If it
ended blocked, leave it, with **Next step** saying what unblocks it.
