# Spec: <feature name>

Status: draft | approved
Roadmap: <specs/roadmaps/<name>.md (slice N — <slice-name>)>, or omit if standalone
Change type: new | refactor | extend | bugfix   <!-- drives how tests are handled -->
Owner:
Date:

## Problem / goal
What are we actually trying to achieve, and for whom? One or two sentences.
Why now — what breaks or stays painful if we don't build this?

## Current behavior (as-built) — existing features only
Fill this in when auditing something that already exists. Describe what the code
does *today*, faithfully, bugs and all. Note which of these behaviors are:
- **Keep** — locked in by characterization tests before any change
- **Change** — the target behavior below replaces it
- **Wrong** — a bug; the target below is the correct behavior (do NOT lock this in)

(Leave this section out for brand-new features.)

## Scope
**In scope**
- ...

**Out of scope** (explicitly, so it can't creep in)
- ...

## Behavior
The feature described as observable behavior — what the user or caller does and
what happens. User stories or a numbered flow both work.

1. ...
2. ...

## Acceptance criteria
The testable checklist. Each item must be verifiable by a test or an observable
outcome. This is what the test-writer and e2e-tester work from, so be precise.

- [ ] ...
- [ ] ...
- [ ] Edge case: ...
- [ ] Error case: ...

## Performance budget
Measurable targets, or "none required".
- e.g. p95 response < 200ms under 50 concurrent requests
- e.g. handles input of up to N items without degradation

**Optimization work only** — fill these when the goal is to make existing code faster:
- **Baseline**: current measured number (filled in by the profiler, with spread)
- **Target**: the budget to hit (absolute, e.g. "< 200ms p95", or relative, e.g. "≥ 2× faster")
- **Representative workload**: the input/load the benchmark runs against
- **Metric**: what's being measured (latency, throughput, memory, allocations…)

## Technical notes / constraints
Anything the builder must respect: existing modules to reuse, APIs, data shapes,
security requirements, things NOT to touch.

## Test plan
- Unit tests: yes (default) / skip — if skip, say why
- Integration / e2e / acceptance: what to exercise end-to-end
- Data or fixtures needed:

## Open questions
Anything unresolved during the interview. Resolve before marking "approved".
- ...
