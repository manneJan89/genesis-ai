---
name: finding-confidence
description: How Genesis scores and filters findings before reporting them — a disprove pass, a 0–100 confidence score, and thresholds for what reaches the main list. Used by /genesis:review, /genesis:security-check and /genesis:hack; load whenever producing a list of defects or vulnerabilities.
user-invocable: false
---

# Finding confidence

A report full of false alarms trains the reader to ignore it. Every finding gets
tested and scored before it reaches the user, and only the ones that survive go in
the main list.

## 1. Collect candidates
Gather everything that looks wrong while reading. At this stage, don't filter.

## 2. Disprove pass (for every candidate)
Try to prove the finding **wrong** before reporting it. Actively look for the thing
that would make it a non-issue:
- a guard, middleware, interceptor, decorator, or security rule elsewhere that
  already handles it (route-level auth, a global validation pipe, Firestore rules);
- a framework default that covers it (auto-escaping templates, ORM parameterization);
- validation or normalization done by the caller before this code runs;
- the path being unreachable, dead, test-only, or behind a disabled flag;
- a test that already pins the behavior as intended.

Read the code that would disprove it — don't assume it's absent. If the disprove
pass finds the guard, drop the finding. If it finds a *partial* guard, note what
still gets through.

## 3. Score 0–100

| Score | Meaning | Evidence required |
|---|---|---|
| 90–100 | Confirmed | Traced end to end in code: the trigger, the path, and the wrong result. You could write the failing test now. |
| 70–89 | Likely | Strong code evidence, but it depends on one thing you couldn't verify statically (runtime config, deploy settings, caller behavior). Name that thing. |
| 40–69 | Plausible | The pattern is there, but the trigger or impact is unproven. |
| 0–39 | Speculative | "This could be a problem if…" with no evidence it is. |

Score the evidence, not the severity. A critical-looking issue with no evidence is
still speculative. Performance findings from static reading cap at 69 — nothing
has been measured. Where the report has a separate "Performance hypotheses"
section (`/genesis:review`), they go there instead of the Unverified section.

## 4. Report by threshold
- **70 and above → main findings list.** Include the score and, for 70–89, the one
  thing that would confirm it.
- **40–69 → "Unverified — worth a look" section.** One line each: location, the
  suspicion, what would confirm or kill it. No routing to a fix flow until confirmed.
  **Exception:** a critical/blocker-severity security finding at 40–69 goes in the
  main list, marked *unverified*. Missing a real one costs more than checking a
  false one.
- **Below 40 → drop it.** State only the count ("7 speculative candidates dropped")
  so the user knows the filter ran.

If nothing reaches 70, say so plainly. An empty main list is a valid result;
don't lower the bar to fill it.

## 5. Parallel passes
When findings come from several focused passes or sub-agents, run the disprove
pass and scoring on the merged list, and merge duplicates first (the same root
cause found by two passes is one finding; agreement between independent passes is
evidence and can raise the score).
