---
description: Improve an existing feature from an approved audit spec, safely
argument-hint: [path/to/spec.md]
allowed-tools: Read, Write, Edit, Grep, Glob, Agent, Skill, Bash(rm HANDOFF.md)
---

Orchestrate a safe improvement of an existing feature from the spec at: $ARGUMENTS

Read the spec first. If its status isn't `approved` or it has no "Current
behavior (as-built)" section, stop and tell me to run `/genesis:audit-feature`. Re-read
the spec whenever you delegate, and pass the relevant details in the delegation
prompt — subagents start fresh and only see what you give them. Delegate to
subagents **by name explicitly**.

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
tagged **Keep** in the spec. Confirm the whole net is green against the current,
unchanged code before proceeding. This is the step that makes the change safe —
do not skip it.

## Phase 2 — Plan (read-only)
Enter plan mode. Lay out the change: **exactly which files, and which functions/
sections within them, you will touch** (the touch budget), how it maps to the
target acceptance criteria, and — for a refactor — your assurance that the
characterization tests stay green throughout. Everything not on that list stays
untouched. Stop for my approval of the touch budget before writing. Prefer small
reviewable chunks over one big pass.

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
spec, before the change is made. Confirm they fail for the right reason (assertion
or "not implemented", not a broken import). An absence test (denied user doesn't
see X) may pass already; keep it as long as its positive counterpart fails. These
sit alongside the characterization net, which must still be green.

## Phase 4 — Make the change (green)
Load the Genesis standards skills that apply (see CLAUDE.md → Standards), then
implement the approved plan. **Modify only what the touch budget names; leave all
other code — and all existing UI/design — byte-for-byte intact** (per the
modification contract in Standards). Do not restyle or re-lay-out an existing
screen. If integrating data into an existing page, wire the data *into* the
existing widgets — do not rebuild the page.

> **Design is a Keep here.** `improve-feature` changes behavior and preserves the
> existing look. If this task is actually a *redesign* (replacing the layout/visuals
> of an existing screen or component), STOP and tell me to use `/genesis:redesign`
> — that command keeps behavior and changes design, the inverse of this one. Don't
> restyle from within improve-feature. If the feature adds NEW UI to an existing
> screen and its design gate wasn't resolved in the spec, stop and settle it (a
> design in `design/`, or an approved "assume") before building that UI — don't
> improvise a look.
- **Change type = refactor:** the characterization net must stay green the entire
  time; run it frequently. Green net = behavior preserved.
- **Change type = extend / bugfix:** the behaviors tagged **Change** or **Wrong**
  will make their characterization tests fail — that's expected. Update those
  specific tests to the new target behavior; the Keep tests must still pass.

Make the Phase 3 tests pass without editing them. If one looks wrong, stop and
show me rather than bending it. Run the lint/type-check command before moving on.

## Phase 5 — Acceptance / e2e check
Delegate to the **e2e-tester** subagent to run the full suite (characterization +
new tests + integration/e2e) and exercise the feature against the target
acceptance criteria. Collect its structured bug list.

## Phase 6 — Fix loop
Send bugs to the **bug-fixer** subagent, then re-run the **e2e-tester** on the
whole feature. Repeat until everything is green — including the untouched
characterization tests, which guard against regressions. Stop and report any
blocker rather than working around the spec.

**Cap the loop at 3 fix cycles.** The cap holds even if a `/goal` or other
loop is set: at three, stop and report. If it's not green after three passes, stop and
report what's still failing and your best hypothesis rather than regenerating the
change again. If a characterization test keeps failing, check whether it was a
Keep behavior that this change legitimately alters (update the test) versus a real
regression (fix the code) — don't loop blindly on it.

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
Report: what changed, which characterization tests were kept vs updated (and why),
new acceptance criteria now passing, full test results, the security pass result
(or why it was skipped), performance numbers, any `(manual)` criteria for me to
check by hand, and anything left open or descoped.

If the work is finished, clear `HANDOFF.md` as described under Handoff. If it
ended blocked, leave it, with **Next step** saying what unblocks it.
