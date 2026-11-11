---
description: Optimize existing working code, measure-driven, without changing behavior
argument-hint: [path/to/spec.md]
allowed-tools: Read, Write, Edit, Grep, Glob, Agent, Skill, Bash(rm HANDOFF.md)
---

Orchestrate a performance optimization of existing code from the spec at:
$ARGUMENTS

Read the spec first. It must have an approved target metric, a budget, and a
representative workload (from `/genesis:audit-feature`); if not, stop and say so. This is
a refactor — **behavior does not change** — so correctness is guarded throughout
and every claimed speedup must be measured, not asserted. Delegate to subagents
**by name explicitly**.

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

**Data operations:** if the plan includes anything that writes to data outside a
local or emulator database (migration, backfill, bulk update or delete, data-fix
script, import), name it in the hypothesis and apply the **data-operation gate** from
`genesis:standards-security`: you build and test it locally, I run it against
real data after confirming a backup.

- *If a queryable code graph exists*, use it to see what depends on the hotspot
  before you change it, so an optimization doesn't quietly break a caller you
  didn't know about.

## Phase 4 — One change
Load `genesis:standards-performance` and `genesis:standards-code` first (plus any
other standard the change touches, per CLAUDE.md → Standards). A faster version
still has to meet them: no new cache that leaks, no pagination dropped, no
duplicated logic.

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

## Phase 6.5 — Security pass (when the change touches the trust boundary)
Optimizations are a common way to lose a check: a cache key without the user or
tenant in it, a query rewrite that drops an ownership filter, a fast path that
skips validation. Treat any of those as touching the trust boundary.

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

## Phase 7 — Summary
Report: baseline vs final numbers with spread and total % gain, each change that
was kept (and its individual contribution), anything tried and reverted and why,
confirmation the characterization net is still green, and any hotspot left on the
table as a deliberate call.

If the work is finished, clear `HANDOFF.md` as described under Handoff. If it
ended blocked, leave it, with **Next step** saying what unblocks it.
