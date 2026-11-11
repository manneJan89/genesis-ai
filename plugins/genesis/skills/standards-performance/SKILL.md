---
name: standards-performance
description: Genesis performance, cost and scale standards: scalable algorithms, no unbounded growth, measure before optimizing, metered-service cost, plus per-profile rules (server data access, client rendering and change detection, Firebase cost, Unreal and Unity frame budgets, native hot paths). Load when writing or reviewing queries, list endpoints, loops over collections, rendering or per-frame code, caching, or calls to billable services.
user-invocable: false
---

# Performance, cost, and scale standards

Part of the Genesis Standards. These rules are binding whenever they apply; they
load on demand to save context, not because they are optional. Project-specific
additions live in the project's CLAUDE.md under **Conventions**; where the two
conflict, the project's CLAUDE.md wins (except over security, which always wins).

## Load only what applies
This file applies to every project. For each profile in CLAUDE.md →
**Profiles** that covers the files at hand, read
`${CLAUDE_PLUGIN_ROOT}/skills/standards-performance/profiles/<profile>.md`
(`server`, `client`, `firebase`, `unreal`, `unity`, `native`). For `client`,
also read `profiles/client-<framework>.md` for the framework that line names
(`angular`, `react`, `flutter`). Nothing else. No Profiles section in
CLAUDE.md? Work out the profile from the code, say which you assumed, and
suggest `/genesis:sync`.

## The universal rules
- **Performance is a first-class concern.** Don't ship an obviously wasteful
  approach and defer it to "later". Prefer algorithms that scale; flag
  quadratic blow-ups, repeated work inside loops, and anything whose cost grows
  with total data size rather than with what's being shown or processed.
- **No unbounded growth.** Lists, caches, queues, buffers and logs have a limit
  or an eviction rule.
- **Measure before optimizing.** Find the real hotspot with a profiler or
  timing first; an optimization without a before-and-after number is a guess.
  Budgets go in the spec as measurable criteria (p95 latency, frame time,
  memory).
- **Metered services cost money per call** (Firebase, Supabase, hosted queues,
  paid APIs, LLM APIs). Batch, cache where safe, avoid per-item round-trips and
  polling. When a change adds billable calls, say so and estimate the impact.
- **Call out trade-offs.** Don't silently accept a change that improves one of
  {correctness, performance, cost} at a real expense to another. Security is not
  on that list: it isn't traded for speed.
