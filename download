---
description: Improve an existing feature from an approved audit spec, safely
argument-hint: [path/to/spec.md]
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
---

Orchestrate a safe improvement of an existing feature from the spec at: $ARGUMENTS

Read the spec first. If its status isn't `approved` or it has no "Current
behavior (as-built)" section, stop and tell me to run `/genesis:audit-feature`. Re-read
the spec whenever you delegate, and pass the relevant details in the delegation
prompt — subagents start fresh and only see what you give them. Delegate to
subagents **by name explicitly**.

## Phase 1 — Safety net (before touching anything)
Delegate to the **characterization-tester** subagent to lock in every behavior
tagged **Keep** in the spec. Confirm the whole net is green against the current,
unchanged code before proceeding. This is the step that makes the change safe —
do not skip it.

## Phase 2 — Plan (read-only)
Enter plan mode. Lay out the change: files you'll touch, how it maps to the
target acceptance criteria, and — for a refactor — your assurance that the
characterization tests stay green throughout. Stop for my approval. Prefer small
reviewable chunks over one big pass.

## Phase 3 — Make the change
Implement the approved plan.
- **Change type = refactor:** the characterization net must stay green the entire
  time; run it frequently. Green net = behavior preserved.
- **Change type = extend / bugfix:** the behaviors tagged **Change** or **Wrong**
  will make their characterization tests fail — that's expected. Update those
  specific tests to the new target behavior; the Keep tests must still pass.

## Phase 4 — Target tests
Delegate to the **test-writer** subagent to add unit tests for the *new/changed*
acceptance criteria (derived from the spec, not the implementation). These sit
alongside the characterization net.

## Phase 5 — Acceptance / e2e check
Delegate to the **e2e-tester** subagent to run the full suite (characterization +
new tests + integration/e2e) and exercise the feature against the target
acceptance criteria. Collect its structured bug list.

## Phase 6 — Fix loop
Send bugs to the **bug-fixer** subagent, then re-run the **e2e-tester** on the
whole feature. Repeat until everything is green — including the untouched
characterization tests, which guard against regressions. Stop and report any
blocker rather than working around the spec.

> For a hands-off loop: `/goal the full suite passes, the Keep characterization
> tests still pass, and the target acceptance criteria in <spec> are met`.

## Phase 7 — Performance
Verify the spec's performance budget with a real measurement. For refactors aimed
at performance, report before-vs-after numbers. If no budget, say so.

## Phase 8 — Summary
Report: what changed, which characterization tests were kept vs updated (and why),
new acceptance criteria now passing, full test results, performance numbers, and
anything left open or descoped.
