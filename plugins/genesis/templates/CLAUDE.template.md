# Project rules

<!-- genesis-standards-version: 0.21.0 — run /genesis:sync after a plugin update to refresh these rules -->

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

- Build a feature (new, or added onto existing code): `/genesis:spec <feature>` →
  `/genesis:build-feature specs/<name>.md`. `build-feature` detects existing code
  and protects it itself; there's no separate command for "existing".
- Too big for one spec: `/genesis:roadmap <system>`, then one spec per slice.
- Change existing behavior from an audit: `/genesis:audit-feature <thing>` →
  `/genesis:improve-feature specs/<name>.md`
- Replace a screen's look, keeping behavior: `/genesis:redesign <screen>`
- Make it faster: `/genesis:audit-feature <thing>` (type=refactor) →
  `/genesis:optimize-feature specs/<name>.md`
- Fix a reported bug: `/genesis:fix <bug>`
- Find problems: `/genesis:review`, `/genesis:security-check`, `/genesis:hack`

Don't write implementation code for a feature until an approved spec exists in `specs/`.

## Testing

- Unit tests are the default gate, not optional. Only skip when the spec's "Test
  plan" says to.
- Tests are derived from the spec's acceptance criteria, not from whatever the
  implementation happens to do. If code and spec disagree, the spec wins and the
  discrepancy gets flagged.
- **Test happy AND unhappy paths — always, even when the spec omits the unhappy
  one.** For every operation: success and failure. For every async/loading flow:
  state resets on both branches (the never-unset `loading` bug). For every
  permission: the allowed and the denied actor (assert the gated thing is absent
  for the denied one). For every input: valid and invalid/empty/boundary. Bugs
  live in the paths nobody asserted, so these are not optional.
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
Durable rules every phase and subagent must respect. The always-on core is below.
The detailed standards live in the plugin as skills that load when the work calls
for them — they are just as binding. **Before writing or reviewing code, load the
ones that apply:**

| Work involves | Load skill |
|---|---|
| Writing, refactoring, or reviewing any code | `genesis:standards-code` |
| Endpoints, auth/permissions, user input, secrets, queries | `genesis:standards-security` |
| Queries, lists, collections, caching, metered services (Firebase, Supabase, paid APIs) | `genesis:standards-performance` |
| Screens, components, forms, icons/assets | `genesis:standards-ui` + `genesis:ux` |
| Error handling or logging | `genesis:standards-logging` |

When delegating to a subagent, name the skills it should load in the prompt.
To change a standard everywhere, edit its skill in the plugin; add project-specific
standards under Conventions below.

### Tripwires (binding even if the skill didn't load)
The one rule from each skill that must never slip. If you're about to break one,
stop and load the skill.
- **Code:** reuse before you write; never copy-paste a block. Fixed value sets are
  enums, not strings. Match the project's existing structure. (`standards-code`)
- **Security:** the server enforces auth AND per-resource authorization on every
  private endpoint; never trust client input. (`standards-security`)
- **Performance:** no unbounded list queries; paginate, and filter at the query.
  (`standards-performance`)
- **UI:** if `COMPONENTS.md` lists a house component, a raw element is banned.
  Never invent an icon. (`standards-ui`)
- **Errors:** never swallow an error; log once where it's handled, through the
  logger abstraction. (`standards-logging`)

### Always-on core

- **Be direct and honest, not agreeable.** Say what's true, not what's easy to
  hear. If the user is wrong, say "you're wrong" and explain why — don't soften a
  real problem into a gentle suggestion. Challenge weak assumptions, weak plans,
  and weak code, including the user's and your own. Rate ideas honestly (including
  "this is a bad idea because X"); don't validate something just because the user
  proposed it or already built it. When uncertain, say so plainly and say what
  would resolve it — never guess with false confidence. Flag the risks in a
  decision even when not asked. **But don't manufacture disagreement to seem
  rigorous** — agreement, when the thing is actually sound, is honest too; the goal
  is truth and usefulness, not a contrarian reflex. This applies hardest to your
  own work: the failure mode is an agent that writes code and its tests with the
  same blind spot because nothing challenged the assumption.

- **No secrets in source, ever.** Keys, tokens, and credentials come from
  environment/config, never code, logs, or commits. If you find one hardcoded or
  committed, stop and tell me immediately — it needs rotating, not a backlog entry.

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

- **No emojis. Anywhere.** Not in UI, not in user-facing copy ("Thanks for
  registering 🎉" is banned), not in comments, logs, or commit messages. For
  iconography use **SVG icons**, never an emoji standing in for an icon.


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
