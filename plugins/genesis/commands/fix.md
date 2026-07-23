---
description: Investigate, reproduce, and fix a reported bug — failing test first
argument-hint: [description of the bug]
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
---

Handle this reported bug: $ARGUMENTS

Read CLAUDE.md first for the project's commands, standards, and codebase map.
The rule that governs this whole flow: **do not fix anything you have not first
reproduced with a failing test.** A fix without a reproduction is a guess.

## Phase 0 — Capture the report
From what I've told you, fill in what you can and ask me **only** for what's
genuinely missing (one question at a time, max three):
- What actually happens (the symptom, exact error/message if any)
- What should happen instead
- Steps to reproduce, and whether it's consistent or intermittent
- Where: platform/environment, and when it started (after which change, if known)

Write the report to `specs/bugs/<kebab-case-name>.md` using
`${CLAUDE_PLUGIN_ROOT}/templates/bug-template.md`. Keep it short — this is a bug
report, not a feature spec.

## Phase 1 — Investigate (read-only)
Locate the code involved. Trace the path from the reproduction steps to the
suspect logic. Check recent changes to those files if the bug is a regression.
State a specific root-cause hypothesis — where and why — before touching anything.
Do not start editing during this phase.

If you cannot form a hypothesis, say so and tell me what additional information
(logs, a stack trace, a failing input) would let you proceed.

## Phase 2 — Reproduce with a failing test (the gate)
Write a test that fails **because of this bug** — at the smallest useful level
(unit if possible, widget/integration if the bug only appears there). Use the
project's test framework and the mocks/fixtures it already uses.

Run it and confirm it **fails for the expected reason**, not for an unrelated one
(typo, bad setup, wrong import). Show me the failure output.

- If the test passes, you have not reproduced the bug. **Stop.** Report that the
  bug does not reproduce as described and ask for more detail. Do not fix blind.
- Also confirm the rest of the suite is green *before* the fix, so you know what
  the fix is responsible for.

## Phase 3 — Fix
Delegate to the **bug-fixer** subagent with the failing test and your hypothesis.
The fix must be **minimal** and address the root cause, not mask the symptom. No
refactoring beyond what the fix requires; no scope creep.

## Phase 4 — Verify
- The new test now passes.
- Run the **full suite** (command from CLAUDE.md) — everything else still passes.
  A fix that breaks another test is not done.
- Confirm the original reproduction steps no longer produce the symptom.

## Phase 5 — Check for the same bug elsewhere
If the root cause was duplicated logic, the same bug likely exists in the other
copies — fixing one copy leaves the others broken. Search for the pattern and
report any siblings. If you find them, ask whether to fix them here or file them
separately (per the project's DRY standard, this may also be a signal to extract
the shared logic — propose it, don't do it unilaterally).

## Phase 6 — Summary
Report: the root cause in one or two sentences, exactly what changed, the test
that now guards it, full suite results, and any related issues found in Phase 5.
Note anything you deliberately left alone.

## When to escalate instead
If the real fix requires a design change, touches many modules, or means changing
intended behavior rather than correcting broken behavior, **stop and say so**.
That's a job for `/genesis:audit-feature` → `/genesis:improve-feature`, not a
patch here.
