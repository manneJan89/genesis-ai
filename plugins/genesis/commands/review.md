---
description: Review existing code for bugs, performance risks, and standards violations
argument-hint: [file, feature, module, or "recent changes"]
allowed-tools: Read, Grep, Glob, Bash
---

Review this and report what's wrong with it: $ARGUMENTS

You are **read-only** in this command. You find and report problems; you fix
nothing. Every finding gets routed to the command that should handle it.

Read CLAUDE.md first — its **Standards**, **Conventions**, and **Component
libraries** sections are the criteria you review against, not your personal taste.

## Phase 0 — Scope
Confirm what's in scope (a file, a feature, a module, or the most recent changes).
If the target is large, say what you'll cover and what you're leaving out rather
than skimming everything shallowly. Ask me only if the scope is genuinely unclear.

## Phase 1 — Read
Read the code in scope and the tests that cover it. Note what the tests actually
assert — untested paths are where defects hide.

## Phase 2 — Findings
Look for these, in priority order:

1. **Correctness / likely bugs** — unhandled errors, ignored failure paths, race
   conditions, off-by-one, null/empty handling, incorrect edge-case behavior,
   state that can go stale, resource leaks (undisposed controllers, listeners,
   subscriptions).
2. **Performance & cost risks** — N+1 access, work inside loops that could be
   hoisted, unbounded growth, quadratic behavior, redundant network/DB calls,
   missing caching, unnecessary rebuilds/renders, and (per Standards) extra
   billable operations against metered services.
3. **Standards violations** — measured against CLAUDE.md, not preference:
   duplicated logic (DRY), over-clever abstractions with flag parameters (KISS),
   magic strings where an enum belongs, non-exhaustive branching over a fixed
   value set, dependencies or component libraries the project didn't opt into.
4. **Test coverage gaps** — behavior with no test, especially error paths and
   edge cases. Name the specific missing case, not "needs more tests".
5. **Maintainability risks** — functions doing too much, unclear naming, hidden
   coupling, anything that will make the next change dangerous.

## Rules for findings — follow these strictly
- **Cite evidence.** Every finding names the file and line/symbol. No vague claims.
- **Performance findings are HYPOTHESES, not verdicts.** You have not measured
  anything. Say "likely hot path — worth measuring" and never "this is slow."
  Anything requiring a real fix goes through `/genesis:optimize-feature`, which
  profiles before changing. Do not propose optimizations as facts.
- **Separate defects from taste.** A defect is something that is wrong, will break,
  or violates a written standard. "I'd have structured it differently" is taste —
  either leave it out or label it explicitly as optional.
- **Prioritize, don't pad.** Rank by (impact × confidence). A short list of real
  problems beats an exhaustive list of nitpicks. If you find nothing serious, say
  so plainly — that's a valid result, don't manufacture findings.
- **Flag uncertainty.** If something looks wrong but you can't confirm it without
  running the code, mark it "unconfirmed — needs reproduction."

## Phase 3 — Report and route
Write the report to `specs/reviews/<kebab-case-name>.md`. For each finding give:

- **Severity**: blocker / major / minor / optional
- **Confidence**: confirmed (evidence in code) / suspected (needs verification)
- **Location**: file + line/symbol
- **Why it matters**: the concrete consequence, not an abstraction
- **Next step**: exactly one of —
  - `/genesis:fix <description>` — a specific reproducible defect
  - `/genesis:optimize-feature` (via `/genesis:audit-feature`) — a performance
    hypothesis that must be measured before anything changes
  - `/genesis:audit-feature` → `/genesis:improve-feature` — a design or structural
    change, or a standards violation needing rework
  - "trivial — fix directly" — a one-line correction not worth a full flow

Then show me the ranked list and ask which findings I want to act on. **Do not
start fixing anything.**
