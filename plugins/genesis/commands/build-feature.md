---
description: Build a feature from an approved spec, then test, fix, and verify it
argument-hint: [path/to/spec.md]
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
---

Orchestrate the full build pipeline for the spec at: $ARGUMENTS

Read that spec first. If its status is not `approved`, stop and tell me to run
`/genesis:spec` to finish it. The spec is the source of truth for every phase below —
re-read it whenever you hand work to a subagent, and pass the relevant criteria
in the delegation prompt (subagents start with a fresh context and only see what
you give them).

Delegate to subagents **by name explicitly** at each step — don't assume Claude
will auto-route to them.

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
you'll modify — everything else stays untouched. Map the work to each acceptance
criterion. Stop and let me approve the touch budget before writing code. For
anything large, propose reviewable chunks rather than one big pass.

## Phase 2 — Build
Implement the approved plan. Respect the "Technical notes / constraints" and the
out-of-scope list in the spec. Keep the change focused on this feature. If building
upon existing code, stay strictly within the approved touch budget — wire new
behavior *into* existing structure (for UI, into the existing widgets/layout),
never rebuild what's already there. If the design gate on the spec is unresolved
for a UI feature, stop (see Phase 1.5 below).

## Phase 1.5 — Design gate (UI features only)
If this feature renders UI, the spec must have a resolved **design source** (a file
in `design/`, or an explicit "assume" that was approved). If the spec's design gate
is unresolved, stop and tell me to settle it in `/genesis:spec` first — do not
invent a design at build time. When a stored design exists, implement it in the
**project's styling system** (Tailwind/Bootstrap/theme — whatever the project
uses); do not copy the design file's raw CSS. Minor pixel drift from existing
project styles is acceptable.

## Phase 3 — Unit tests
Unless the spec's Test plan says to skip unit tests, delegate to the
**test-writer** subagent:

    Use the test-writer subagent to write unit tests for this feature from
    <spec path>. Derive tests from the acceptance criteria, not from my
    implementation.

Using a separate agent here is deliberate: it stops tests from simply rubber-
stamping the implementation's bugs.

## Phase 4 — Acceptance / e2e check
Delegate to the **e2e-tester** subagent to run the suite and exercise the feature
against the spec's acceptance criteria. It reports bugs; it does not fix them.
Collect its structured bug list.

## Phase 5 — Fix loop
If Phase 4 found bugs, delegate each (or the batch) to the **bug-fixer** subagent
with the reproduction steps. After fixes land, run the **e2e-tester** again on the
whole feature. Repeat until the suite is green against every acceptance criterion,
or until you hit a blocker you can't resolve — in which case stop and report it
rather than hacking around the spec.

**Cap the loop at 3 fix cycles.** If it's not green after three build→test→fix
passes, stop and report what's still failing, your best hypothesis, and what you'd
try next — do not keep regenerating code. Repeated failure usually means the spec
or the approach is wrong, not that another cycle will fix it, and each cycle
re-emits the whole change. Getting my input is cheaper than a fourth blind pass.

> Tip: for a hands-off version of this loop, I can instead run
> `/goal all tests pass and the acceptance criteria in <spec> are met`
> and let Claude iterate build → test → fix → re-test on its own.

## Phase 6 — Performance
Check the spec's performance budget. If one exists, verify it with a concrete
measurement (a benchmark, a timed run, a load test — whatever fits). Report the
numbers against the target. If no budget was specified, say so and move on.

## Phase 6.5 — Component manifest + completion gate
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

## Phase 7 — Update the roadmap (if any)
If the spec header names a roadmap (`Roadmap: specs/roadmaps/<name>.md (slice N …)`),
open that roadmap and:
- mark this slice `done`,
- fill in its Spec column with a relative link to the spec,
- move **Next up** to the first remaining non-done slice,
- update Last updated.

If a shared decision changed during this build (schema, enum, side-effect rule),
update the roadmap's Shared foundations too and tell me what changed — later
slices depend on it. Do NOT rewrite older specs; they are point-in-time history.

If the spec names no roadmap, skip this phase.

## Phase 8 — Summary
Report: what was built, which acceptance criteria now pass, test results,
performance numbers, and anything left open or descoped.
