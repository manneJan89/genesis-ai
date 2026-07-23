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

## Phase 1 — Plan (read-only)
Enter plan mode. Restate your implementation approach: files you'll touch, the
shape of the change, and how it maps to each acceptance criterion. Stop and let
me approve the plan before writing code. For anything large, propose doing it in
reviewable chunks rather than one big pass.

## Phase 2 — Build
Implement the approved plan. Respect the "Technical notes / constraints" and the
out-of-scope list in the spec. Keep the change focused on this feature.

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

> Tip: for a hands-off version of this loop, I can instead run
> `/goal all tests pass and the acceptance criteria in <spec> are met`
> and let Claude iterate build → test → fix → re-test on its own.

## Phase 6 — Performance
Check the spec's performance budget. If one exists, verify it with a concrete
measurement (a benchmark, a timed run, a load test — whatever fits). Report the
numbers against the target. If no budget was specified, say so and move on.

## Phase 7 — Summary
Report: what was built, which acceptance criteria now pass, test results,
performance numbers, and anything left open or descoped.
