# Project rules

Keep this file lean. It loads into every session **and** every subagent, so it's
the one place to encode rules the whole pipeline obeys.

> This file was set up by the **genesis** plugin (`/genesis:setup`). It's
> stack-agnostic: everything specific to *this* project — language, commands,
> conventions, layout — lives in the **"PER-PROJECT — FILL THIS IN"** block at the
> bottom. The rules above it are the same in every project. Re-run `/genesis:setup`
> if the stack changes.

## How we build features (spec-driven)

The spec file is the source of truth that carries scope across phases — agents do
not share a conversation.

- New feature:      `/genesis:spec <feature>`  →  `/genesis:build-feature specs/<name>.md`
- Existing feature: `/genesis:audit-feature <thing>`  →  `/genesis:improve-feature specs/<name>.md`
- Make it faster:   `/genesis:audit-feature <thing>` (type=refactor)  →  `/genesis:optimize-feature specs/<name>.md`

Don't write implementation code for a feature until an approved spec exists in `specs/`.

## Testing

- Unit tests are the default gate, not optional. Only skip when the spec's "Test
  plan" says to.
- Tests are derived from the spec's acceptance criteria, not from whatever the
  implementation happens to do. If code and spec disagree, the spec wins and the
  discrepancy gets flagged.
- Use the **commands defined below** to run tests, lint, build, and benchmark —
  never assume a command; read it from this file.
- A feature is "done" only when its acceptance criteria pass and its performance
  budget (if any) is met.

## Performance budgets

Define targets in the spec as measurable acceptance criteria (e.g. "p95 < 200ms").
A perf check with no target is meaningless; treat a missing budget as "none
required" unless stated.

## Exploring efficiently (token hygiene)

- Prefer targeted reads over broad exploration. When delegating, name the exact
  files or directories so subagents don't scan the whole tree.
- If a queryable code index exists (e.g. a graphify graph under `graphify-out/`,
  or a `/graphify query` skill), use it for structural questions before grepping
  or reading files wholesale. Fall back to grep/read when none is present.
- Use `/clear` between unrelated tasks to reset context.

## Standards (apply to every project)
Durable rules every phase and subagent must respect. These ship in the genesis
template, so they're inherited by every project. Edit them in the plugin's
`templates/CLAUDE.template.md` to change them everywhere; add project-specific
standards under Conventions below.

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

- **DRY — reuse before you write.** Before adding a function, widget, model, or
  service, search for an existing one that already does it (or nearly does it) and
  extend that instead. Never copy-paste a block and tweak it. If the same logic
  appears a third time, extract it.
  **Search order:** (1) this project's own code, (2) libraries already in this
  project's dependencies, (3) any shared library this project has *explicitly*
  opted into under "Component libraries" below. Never introduce a new dependency
  to satisfy DRY without asking.
- **KISS — but not at the cost of readability.** DRY loses to KISS when they
  conflict. Do NOT collapse similar-looking code into one function bristling with
  boolean flags, optional params, and branches — that's harder to maintain than the
  duplication it removed. Two clear functions beat one clever one. Prefer
  extracting a genuinely shared *concept*, not merely shared *characters*.
  Duplication is cheaper than the wrong abstraction: if two blocks look alike but
  change for different reasons, leave them alone and say why.
- **Enums over magic strings.** Any fixed set of values — status, role, type,
  gender, state — is an enum (or sealed/const type), never a bare string or int
  compared with `==`. Prefer `if (gender == Gender.male)` over
  `if (gender == 'male')`. Strings are allowed only at the boundary (JSON, DB,
  API); parse them into the enum on the way in and serialize on the way out, in
  one place. Exhaustive `switch` over an enum is preferred to if/else chains, so
  the compiler catches a missing case when a value is added later.

<!-- =================================================================== -->
<!-- PER-PROJECT — FILL THIS IN  (the only section that changes per repo) -->
<!-- =================================================================== -->

## Project

- Name:
- Stack (language / framework / runtime):

## Commands
The canonical commands agents use. Fill in the ones that apply to this stack;
delete the rest. (Examples in the README show a Node/Angular and a Flutter fill.)

- Install deps:
- Run / serve:
- Build:
- Test (all):
- Test (single file/pattern):
- Lint / static analysis:
- Format:
- Benchmark / profile (for optimization work):

## Component libraries
Which shared UI/component library this project uses. **Default: none.**

- Component source: **none — this project uses its own components only**

> Genesis's workflow is independent of any component library. Unless a shared
> library is named above, build components local to this project using plain
> framework code, and do NOT import or suggest Genesis (or any other external)
> component packages. If a shared library would genuinely help, propose it and
> wait for approval — never add the dependency unilaterally.

## Conventions

- (naming, file layout, error handling, state management, etc.)

## Codebase map
Where things live, so agents don't rediscover the layout every session. Keep it
short — a pointer, not documentation.

- Entry points:
- Core modules / packages:
- Tests live in:
- Config / infra:
- Don't bother reading (generated, vendored, build output):
