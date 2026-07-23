# Review: <what was reviewed>

Date:
Scope: <files / feature / module covered>
Not covered: <what was deliberately left out>

## Summary
One or two sentences: overall health, and the single most important thing to act on.
If nothing serious was found, say that plainly.

## Findings

### 1. <short title>
- **Severity**: blocker | major | minor | optional
- **Confidence**: confirmed | suspected (needs verification)
- **Category**: correctness | performance/cost | standards | test coverage | maintainability
- **Location**: `path/to/file.dart:123` (`functionName`)
- **What**: what is actually wrong.
- **Why it matters**: the concrete consequence.
- **Next step**: `/genesis:fix …` | measure via `/genesis:optimize-feature` |
  `/genesis:audit-feature` → `/genesis:improve-feature` | trivial — fix directly

### 2. <short title>
(repeat)

## Performance hypotheses (unmeasured)
Listed separately because nothing here has been profiled. Each needs measurement
before any change — see `/genesis:optimize-feature`.

- `path/to/file:line` — <suspected issue>, <why it might matter>

## Notes
Anything observed but deliberately not raised as a finding (taste, known
tradeoffs, out-of-scope areas).
