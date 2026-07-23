---
description: Reverse-engineer a spec from existing code, then agree what to improve
argument-hint: [path, module, or description of the existing feature]
allowed-tools: Read, Grep, Glob, Write
---

You are auditing an **existing** feature so we can safely improve it: $ARGUMENTS

This is interactive and stays in the main session — don't delegate the interview.
Follow @specs/_TEMPLATE.md.

## Your job

1. **Locate and read the existing implementation.** Map its real entry points,
   dependencies, and the code paths that make up this feature. Use read-only
   exploration; don't change anything.
   - *If a queryable code graph exists* (e.g. `graphify-out/graph.json`, or a
     `/graphify query` skill), use it first to trace structure — callers,
     dependencies, what flows through this feature — and then read only the
     specific files it points you to. Fall back to grep/read if no graph exists.

2. **Reverse-engineer the "Current behavior (as-built)" section** — describe what
   the code actually does today, faithfully, including quirks and likely bugs.
   Don't describe what you assume it's *supposed* to do.

3. **Assess it.** Call out: missing or thin test coverage, correctness bugs,
   edge cases it mishandles, performance concerns, and risky areas that will make
   change hard. Be concrete about where.

4. **Interview me on the target.** Ask, one question at a time:
   - What does "improve" mean here — refactor (same behavior), extend/change
     behavior, or fix a bug?
   - Which current behaviors must be preserved exactly, and which should change?
   - New acceptance criteria and any performance budget?

5. **Tag each as-built behavior** as Keep / Change / Wrong (per the template),
   because that decides what gets locked into characterization tests and what
   doesn't.

6. **Write the spec** to `specs/<kebab-case-name>.md`: fill the as-built section,
   set `Change type`, and write the target Behavior + Acceptance criteria for the
   improvement. Put unknowns in "Open questions".

7. **Show me the spec and ask me to approve.** Only then set `Status: approved`
   and tell me to run: `/genesis:improve-feature specs/<name>.md`

Write no implementation code in this phase.
