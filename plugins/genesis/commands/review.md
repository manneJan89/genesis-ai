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
   subscriptions). Also **silently swallowed errors** (empty catches, failures
   with no log and no rethrow) and logging that bypasses the abstraction
   (`print`/`console.log`/direct SDK calls) or logs secrets/PII.
2. **Performance & cost risks** — N+1 access, work inside loops that could be
   hoisted, unbounded growth, quadratic behavior, redundant network/DB calls,
   missing caching, unnecessary rebuilds/renders, and (per Standards) extra
   billable operations against metered services. Also scale risks: unpaginated
   list queries, filters/sorts on unindexed fields, whole-collection reads.
3. **Security** — per the Standards, a first-pass smell check on the code in
   scope: private endpoints missing auth or per-resource authorization checks;
   input trusted without server-side validation/sanitization; string-built queries
   (injection risk); secrets or credentials hardcoded or logged; errors leaking
   internals; anything that fails open instead of closed. For a deliberate
   whole-surface audit (attacker's-eye, full checklist), route to
   `/genesis:security-check`.
4. **Standards violations** — measured against CLAUDE.md, not preference:
   duplicated logic (DRY), over-clever abstractions with flag parameters (KISS),
   magic strings where an enum belongs, non-exhaustive branching over a fixed
   value set, dependencies or component libraries the project didn't opt into.

   **UI files — walk every element, don't skim.** This check is mechanical, not
   impressionistic. Enumerate the actual elements in the file and verify each:
   - Every raw `<button>`, `<input>`, `<select>`, `<textarea>`, checkbox, toggle,
     table, card, chip, etc. → is there a house component in `COMPONENTS.md` for
     it? If yes, using the raw element is a **confirmed finding** (cite the line).
     Go element by element; do not sample.
   - Every emoji character anywhere (UI, labels, titles, copy) → **confirmed
     finding**; icons must be `surespace-icon`/SVG.
   - Repeated markup structure (same block 2+ times) → should be a shared
     component; flag it.
   - Non-descriptive names (`isTF`, `flag`, cryptic abbreviations) → flag.
   - `COMPONENTS.md` drift: a registered component that no longer matches the code,
     or a shared component missing from the manifest.
   Produce one verdict per relevant element — a rich file with interesting logic is
   NOT a reason to glance past the UI pass.
5. **Test coverage gaps** — behavior with no test, especially error paths and
   edge cases. Name the specific missing case, not "needs more tests".
6. **Maintainability risks** — functions doing too much, unclear naming, hidden
   coupling, anything that will make the next change dangerous. Also **structural
   drift**: new code that departed from the project's existing patterns (a parallel
   API file when one already exists, models placed inconsistently, competing
   foldering) — and, conversely, an existing structure that has genuinely outgrown
   itself and is worth refactoring (flag it as a deliberate change, with the reason).

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
start fixing anything.** Offer to append any I don't act on now to `FINDINGS.md`
so they aren't lost — especially anything medium+ — and log any committed secret
there immediately while flagging it to me.
