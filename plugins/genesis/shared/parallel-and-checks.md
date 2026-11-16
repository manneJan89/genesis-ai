# Parallel work, checks and fixes (shared by the build commands)

Read by `/genesis:build-feature` and `/genesis:improve-feature`. Follow the
sections the command points you to.

## Running agents in parallel
**Run independent agents at the same time.** Whenever two or more agents don't
depend on each other's output and don't write the same files (test-writers for
different areas, builders for different areas, the post-build checks), launch
them together: all their Agent calls in **one message**, so they run in
parallel. One after the other is only for work that genuinely waits on earlier
work. Parallel uses about the same tokens (each agent starts fresh either way)
and finishes in the time of the slowest one.

**Time each phase.** At the start of each phase, note the time
(`date +%H:%M`). The summary reports how long each phase took, so slow phases
are visible instead of guessed at.

## Areas and the contract
**Split by area, for parallel work.** If the feature spans more than one area
(backend, frontend, security rules, a game module), give each area its own touch
budget. The budgets must not share a file; anything both sides need (endpoint
paths, request and response types, DTOs, enums, events) goes in the **contract**:
the shared files you will write first, before any area starts, using the spec's
Contract section (if the spec has none, propose the contract in the plan for my
approval). Then the areas can be tested and built at the same time. If
two areas can't be separated cleanly (they must edit the same file), say so and
build them as one area. A feature in one area is built by you directly, as before.

**Tests per area.** In the tests-first phase, launch one **test-writer** per
area in the same message, each told its area, folders, the approved surface, and
the contract files. Frontend tests mock the backend through the contract.

**Building per area.**
- **One area:** implement the plan yourself until the Phase 3 tests pass,
  running only the related tests while you iterate.
- **Several areas:** delegate to one **area-builder** subagent per area, all in
  the same message so they build in parallel. Give each: the spec path, its
  area and folders, its touch budget, the contract files, its tests, and which
  standards skills to load. If one reports a contract gap or an outside-budget
  file, stop the affected area, settle it with me, update the contract yourself,
  and restart only the areas it affects.
- **Join:** when every area is green, run the **affected** tests and lint once
  (CLAUDE.md "affected" commands; the full suite only if the stack has none). This is where the areas meet: a mismatch between
  frontend and backend shows up here.

## Checks, in parallel
Launch these in **one message**, so they run at the same time. None of them
edits code:
- **e2e-tester:** run the feature against the spec's acceptance criteria;
  report bugs. Tell it the affected tests just ran green at the join, so it doesn't
  run it again.
- **security-reviewer**, if the change touches the trust boundary: endpoints or
  route handlers, RPCs or callable functions, auth or permission code, security
  rules files, database queries built from input, input parsing or
  deserialization, file uploads or paths, or anything named in the spec's
  Security section. Size doesn't matter: the most dangerous edit is often one
  line that drops a check. Give it the spec path, the changed files, and the
  entry points. If the change touched none of these, skip it and say so.
- **component-moderator**, if the change touched UI: give it the changed UI
  files. Its exact swaps are defects for the fix loop; its partial matches go to
  me (extend the house component, or accept a one-off), never a fork.

## Fix loop
Collect every finding from the checks: e2e bugs, security findings scored 70 or
above, exact component swaps.
- **Security findings first get a failing test:** the **test-writer** turns the
  reviewer's "failing test that would prove it" into a real test, and you
  confirm it fails.
- Delegate the fixes to the **bug-fixer** subagent, one per area in parallel
  when the fixes touch different areas, one for everything otherwise.
- After fixes land, run only the **related** tests, then re-run only the checks
  that had findings (an e2e re-run for e2e bugs, the security-reviewer for
  security findings). A clean check isn't re-run.

**Cap the loop at 3 fix cycles** (security findings: 2). The cap holds even if a
`/goal` or other loop is set: at the cap, stop and report. If it's not green,
report what's still failing, your best hypothesis, and what you'd try next — do
not keep regenerating code. Repeated failure usually means the spec or the
approach is wrong, not that another cycle will fix it. Critical or high security
findings still open at the cap block `done`: stop and report, don't ship around
them. Medium and low security findings go to `FINDINGS.md`, as does the
reviewer's "outside this change" list (critical or high there: log it and stop,
per the core rule).

When the loop ends green, run the **affected** tests and lint once to confirm.
Never the whole repo unless I ask.
