---
description: Optimize existing working code, measure-driven, without changing behavior
argument-hint: [path/to/spec.md]
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
---

Orchestrate a performance optimization of existing code from the spec at:
$ARGUMENTS

Read the spec first. It must have an approved target metric, a budget, and a
representative workload (from `/genesis:audit-feature`); if not, stop and say so. This is
a refactor — **behavior does not change** — so correctness is guarded throughout
and every claimed speedup must be measured, not asserted. Delegate to subagents
**by name explicitly**.

## Phase 1 — Baseline + find the real hotspot
Delegate to the **perf-profiler** subagent (Mode A) to establish a baseline (with
spread) and rank the real hotspots. **Record the baseline number in the spec.**
Do not optimize anything the profiler didn't flag. If the profiler reports the
bottleneck is architectural, stop and surface that to me — it's a design call,
not something to grind out here.

## Phase 2 — Correctness safety net
Delegate to the **characterization-tester** subagent to lock in current behavior
and confirm the net is green against the unchanged code. This is what lets you
change the implementation freely while proving behavior stayed identical.

## Phase 3 — Hypothesis + plan (read-only)
Enter plan mode. Pick the top hotspot worth attacking (critical path, meaningful
share of the metric). State a specific hypothesis: "X is slow because Y; doing Z
should cut it by roughly N." Stop for my approval before changing code.
- *If a queryable code graph exists*, use it to see what depends on the hotspot
  before you change it, so an optimization doesn't quietly break a caller you
  didn't know about.

## Phase 4 — One change
Make exactly one change implementing the hypothesis. Don't bundle unrelated
optimizations — one change per cycle so we know what moved the number.

## Phase 5 — Verify the change
Two gates, both must pass:
- **Correctness:** the characterization net stays green (behavior unchanged). Run
  it; if anything went red, the optimization broke behavior — revert or fix.
- **Speed:** delegate to the **perf-profiler** (Mode B) to re-measure vs baseline.
  Keep the change only if the gain is real (bigger than measurement noise) and
  worth the complexity it adds. If it didn't help, **revert it** — a change that
  doesn't move the number but complicates the code is a net loss.

## Phase 6 — Next hotspot or stop
If the budget isn't met and worthwhile hotspots remain, return to Phase 3 for the
next one. Stop when the budget is met, or when remaining gains are too small to
justify the added complexity (diminishing returns — say so explicitly).

## Phase 7 — Summary
Report: baseline vs final numbers with spread and total % gain, each change that
was kept (and its individual contribution), anything tried and reverted and why,
confirmation the characterization net is still green, and any hotspot left on the
table as a deliberate call.
