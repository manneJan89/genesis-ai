# Project rules

<!-- genesis-standards-version: 0.11.0 — run /genesis:sync after a plugin update to refresh these rules -->

Keep this file lean. It loads into every session **and** every subagent, so it's
the one place to encode rules the whole pipeline obeys.

> This file was set up by the **genesis** plugin (`/genesis:setup`). It's
> stack-agnostic: everything specific to *this* project — language, commands,
> conventions, layout — lives in the **"PER-PROJECT — FILL THIS IN"** block at the
> bottom. The rules above it are the same in every project. After a plugin update,
> run `/genesis:sync` to pull new rules into this file; re-run `/genesis:setup` if
> the stack changes.

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
  **Before building anything, search for what already exists.** If an **exact**
  match (behaviorally identical component or function — same job, different names
  still counts) is already in the codebase, reuse it, or extract it into a shared
  component so there's one copy; ask before extracting. If something is only
  **partially similar**, note it but leave it separate by default (see KISS) —
  merging merely-similar code into one flag-driven abstraction is worse than the
  duplication. Copy-paste of an existing block is a defect, not a shortcut.
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

- **Security is not optional and not a feature — it's a property of everything.**
  - **The server is the only trust boundary.** Client-side validation is for UX;
    it can always be bypassed. Every rule enforced on the client MUST be enforced
    again on the server. Never trust input that arrived over the wire.
  - **Auth on every private endpoint.** Private/authenticated endpoints verify the
    caller's identity AND that they're authorized for *this* resource (an
    authenticated user is not automatically permitted). Deny by default: an
    endpoint with no explicit auth decision is treated as needing auth, not as
    public. Return 401 vs 403 correctly and don't leak which.
  - **Validate and sanitize all input, server-side.** Reject malformed input at
    the boundary; validate type, range, length, and shape. Use parameterized
    queries / the ORM's safe paths — never string-built queries. Encode/escape
    output to prevent injection (SQL, XSS, command). Treat file uploads and
    redirects as hostile until checked.
  - **No secrets in source, ever.** API keys, tokens, connection strings, and
    credentials come from environment/config (`.env`, platform secrets), which is
    git-ignored. Never log secrets or PII. If a secret would appear in code,
    output, or a commit, stop and flag it.
  - **Least privilege.** Request the narrowest scopes/permissions that work;
    don't run or connect as an admin/superuser for ordinary operations.
  - **Fail closed.** On an auth or validation error, deny access — never fall
    through to permitted. Don't expose stack traces or internal detail in errors
    sent to clients.

- **Out-of-scope findings: capture, don't act.** When you notice something
  important that is NOT part of the current task — a hardcoded secret, a bug, a
  security hole, a performance trap, tech debt — do NOT fix it inline (that's scope
  creep and breaks the touch budget) and do NOT let it vanish into a summary.
  **Append it to `FINDINGS.md`** at the repo root (create it if missing) with
  severity, location, what it is, and a suggested next step, then continue your
  actual task. The backlog is worked separately via `/genesis:findings`.
  **Exception — a committed/hardcoded secret is live exposure, not a note for
  later:** log it AND stop to tell me immediately, because it needs rotating and
  purging from history, which can't wait for a backlog review.

- **Follow the project's existing structure.** Before creating a new file or
  deciding where code goes, look at how the project already organizes this kind of
  thing and **match it.** If API calls for a service live in one file, add the new
  call there — don't spin up a parallel file. Put models where models already live,
  follow the existing foldering, layering, and naming patterns. Detect the
  convention from the actual codebase (find the existing file/folder and conform),
  don't impose a structure that seems reasonable in isolation — that's how a
  codebase grows two or three competing patterns for the same thing. Introduce a
  new structural pattern ONLY if the project has none for this, if I explicitly ask,
  or if `/genesis:review` flagged the current structure for change. If you think the
  existing structure is genuinely wrong, don't quietly work around it — log a
  finding (`FINDINGS.md`) or raise it, don't fork the pattern.

- **Modifying existing code: change only what was asked.** When editing a file
  that already exists, add and modify exactly what the spec names — and leave
  everything else **byte-for-byte intact**. Do NOT rewrite working functions,
  restructure a screen, rename things, or "improve" code the spec didn't mention,
  however tempting. Existing UI/design is a **Keep**: never restyle or re-lay-out
  an existing screen unless the spec explicitly says "redesign". If you believe an
  untouched part genuinely needs changing, stop and propose it separately — don't
  fold it into this change. Rewriting existing work as you "see fit" is the single
  most destructive thing you can do; a modification that changes more than its
  spec named is a defect, not initiative.

- **Log failures — through the abstraction, never raw, never secrets.**
  - Log where a failure is **handled** (the catch that decides what happens), once
    per failure — not at every layer it passes through. One meaningful entry, not
    ten.
  - **Never swallow errors silently.** An empty catch, or one that hides a failure
    with no log and no rethrow, is a defect. Every caught failure is either
    handled-and-logged or rethrown.
  - Log through the project's **logger abstraction** (see CLAUDE.md → Logging),
    never `print` / `console.log` / direct SDK calls. This is what lets the sink
    and level change per environment from one place.
  - Use **levels**: error (failed, needs attention) / warn (recovered or degraded)
    / info (notable events) / debug (dev only). Production runs at warn/error.
  - Include **actionable context** — the operation, relevant identifiers, the
    error type/message — but **never secrets, tokens, credentials, or PII** (this
    overrides the urge to "log everything to debug it"). Log an id, not the record.
  - Internal detail (stack traces) goes to the log sink, **never to a client
    response** (ties to the security standard).
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

**Test baseline** — what a healthy run looks like, so agents can tell new breakage
from pre-existing breakage:
- Baseline: <e.g. "all tests pass" — or list known-failing tests accepted as-is>

## Logging
Where failures go, and how it varies by environment. Code logs through the
abstraction below — never `print`/`console.log`/direct SDK calls.

- **Logger abstraction**: <path, e.g. lib/core/logger.dart — the single seam all
  logging goes through>
- **Sink**: <console-only (default) | Sentry | Crashlytics | platform stdout | …>
- **Environment switch**: <how prod vs dev is detected — NODE_ENV, --dart-define,
  build flavor>

| Environment | Min level | Destination | Stack traces |
|---|---|---|---|
| dev | debug | console | shown |
| prod | warn/error | <sink> | to sink only, never to client |

> Genesis does not provision monitoring services. To add one (Sentry, Crashlytics),
> build it as a feature — `/genesis:spec add <service> error reporting` — so the
> DSN-from-env, cost note, and init go through the normal approval gates.

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
- Stored designs (human-editable HTML, from `/genesis:design`): `design/`
- Don't bother reading (generated, vendored, build output):
