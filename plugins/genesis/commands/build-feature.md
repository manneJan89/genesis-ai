---
description: Build a feature from an approved spec, then test, fix, and verify it
argument-hint: [path/to/spec.md]
allowed-tools: Read, Write, Edit, Grep, Glob, Agent, Skill, Bash(rm HANDOFF.md)
---

Orchestrate the full build pipeline for the spec at: $ARGUMENTS

Read that spec first. If its status is not `approved`, stop and tell me to run
`/genesis:spec` to finish it. The spec is the source of truth for every phase below —
re-read it whenever you hand work to a subagent, and pass the relevant criteria
in the delegation prompt (subagents start with a fresh context and only see what
you give them).

Delegate to subagents **by name explicitly** at each step — don't assume Claude
will auto-route to them.

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
  so any accidental change to existing behavior turns a test red. Then apply the
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
acceptance criterion. Stop and let me approve the touch budget before writing code.
For anything large, propose reviewable chunks rather than one big pass.

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

1. **Scaffold the surface.** Create the files and the public signatures named in
   the approved plan with no behavior — bodies that throw "not implemented" (or
   the stack's equivalent). This is only so the tests compile; write no logic.
   For build-upon-existing, lay the characterization net first (Phase 0).
2. **Delegate to the test-writer subagent in `tests-first` mode:**

       Use the test-writer subagent in tests-first mode to write unit tests for
       <spec path>. The approved surface is: <signatures from the plan>. Derive
       every test from the acceptance criteria; no implementation exists yet.

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
Standards) before writing code. Implement the approved plan until the Phase 3
tests pass. Respect the "Technical
notes / constraints" and the out-of-scope list in the spec. Keep the change focused
on this feature. If building upon existing code, stay strictly within the approved
touch budget — wire new behavior *into* existing structure (for UI, into the
existing widgets/layout), never rebuild what's already there.

**Don't edit a test to make it pass.** If a test looks wrong (it contradicts the
spec, or asserts something the spec never said), stop and show me the test, the
criterion, and why you think it's wrong. The test or the spec changes only with my
say-so.

When the tests are green, run the project's lint/type-check command and fix what
it reports before moving on.

## Phase 5 — Acceptance / e2e check
Delegate to the **e2e-tester** subagent to run the suite and exercise the feature
against the spec's acceptance criteria. It reports bugs; it does not fix them.
Collect its structured bug list.

## Phase 6 — Fix loop
If Phase 5 found bugs, delegate each (or the batch) to the **bug-fixer** subagent
with the reproduction steps. After fixes land, run the **e2e-tester** again on the
whole feature. Repeat until the suite is green against every acceptance criterion,
or until you hit a blocker you can't resolve — in which case stop and report it
rather than hacking around the spec.

**Cap the loop at 3 fix cycles.** The cap holds even if a `/goal` or other
loop is set: at three, stop and report. If it's not green after three build→test→fix
passes, stop and report what's still failing, your best hypothesis, and what you'd
try next — do not keep regenerating code. Repeated failure usually means the spec
or the approach is wrong, not that another cycle will fix it, and each cycle
re-emits the whole change. Getting my input is cheaper than a fourth blind pass.

## Phase 6.5 — Security pass (when the change touches the trust boundary)
Run this if the change touched any of: endpoints or route handlers, RPCs or
callable functions, auth or permission code, security rules files, database
queries built from input, input parsing or deserialization, file uploads or
paths, or anything named in the spec's Security section. Size doesn't matter:
the most dangerous edit is often one line that drops a check. Skip it, and say
so, only when the change touched none of these.

1. Delegate to the **security-reviewer** subagent with the spec path, the
   changed files, and the entry points added or altered. It reads, it doesn't
   edit.
2. For each finding scored 70 or above: the **test-writer** turns its "failing
   test that would prove it" into a real test, you confirm it fails, then the
   **bug-fixer** fixes it and the full suite runs.
3. **Cap at 2 rounds.** Anything critical or high still open after that blocks
   `done`: stop and report it, don't ship around it. Medium and low go to
   `FINDINGS.md` with the reviewer's details.
4. Its "outside this change" list goes to `FINDINGS.md` (critical or high there:
   log it and stop, per the core rule).

## Phase 7 — Performance
Check the spec's performance budget. If one exists, verify it with a concrete
measurement (a benchmark, a timed run, a load test — whatever fits). Report the
numbers against the target. If no budget was specified, say so and move on.

## Phase 7.5 — Component manifest + completion gate
- **Manifest write-back:** if this feature created or changed a shared UI
  component, update `COMPONENTS.md` (use-for, API, can/can't) so it stays the
  authoritative registry. If it should have used a house component and didn't,
  that's a defect — fix it, don't record around it.
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
Report: what was built, which acceptance criteria now pass, test results
(including how many tests went red → green, and how many of those are security
tests), the security pass result (or why it was skipped), performance numbers,
any `(manual)` criteria for me to check by hand, and anything left open or
descoped.

If the work is finished, clear `HANDOFF.md` as described under Handoff. If it
ended blocked, leave it, with **Next step** saying what unblocks it.
