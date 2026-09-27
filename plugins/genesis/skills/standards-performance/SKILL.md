---
name: standards-performance
description: Genesis performance, cost, and scale standards: scalable algorithms and data access, metered-service cost (Firebase/Firestore, Supabase, paid APIs), pagination, indexes, N+1. Load when writing or reviewing queries, list endpoints, loops over collections, caching, or calls to billable services.
user-invocable: false
---

# Performance, cost, and scale standards

Part of the Genesis Standards. These rules are binding whenever they apply — they
were moved out of CLAUDE.md so they load only when relevant, not because they are
optional. Project-specific additions live in the project's CLAUDE.md under
**Conventions**; where the two conflict, the project's CLAUDE.md wins.

- **Performance is a first-class concern, always.** Don't ship an obviously
  wasteful approach and defer performance to "later." Prefer algorithms and data
  access patterns that scale; flag anything with quadratic blow-up, N+1 access, or
  unbounded growth even when not explicitly asked.
- **Be cost-conscious with metered third-party services** (Firebase/Firestore,
  Supabase, hosted queues, paid APIs, etc.). Minimize billable operations: batch
  reads/writes, cache where safe, avoid per-item round-trips and chatty polling,
  and prefer a single query over many. When a change adds billable calls, say so
  and estimate the impact rather than silently increasing usage.
- Call out, don't silently accept, a tradeoff that improves one of
  {correctness, performance, cost} at a real expense to another.

- **Scale — design for growth from day one.** This extends the
  performance/cost rules above, at the data layer specifically:
  - **Never load unbounded result sets.** List/query endpoints paginate (limit +
    cursor/offset); no "fetch all rows/documents" that grows with the table.
  - Ensure queries are backed by an **index**; flag any filter/sort on an
    unindexed field.
  - Avoid N+1 access across a collection; batch or join.
  - Don't hold whole collections in memory to filter in code — filter at the
    query.
  - Prefer stateless request handling (state in the data store, not the process)
    so the service can run behind more than one instance.
  - Flag any operation whose cost grows with total data size rather than with the
    page being served.
