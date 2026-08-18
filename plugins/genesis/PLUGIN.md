# genesis — a spec-driven workflow plugin for Claude Code

Bundles the agents and commands for a spec-driven build / change / optimize
workflow into an installable Claude Code plugin. Stack-agnostic — works on
Angular/Node, Flutter, Go, Python, anything. You fill in the stack once per
project with `/genesis:setup`.

## Install
This repo is both the plugin and its marketplace (self-referencing), so:

```
/plugin marketplace add YOUR_GITHUB_USERNAME/genesis
/plugin install genesis@genesis
/reload-plugins
```

Then, inside any project:

```
/genesis:setup
```

`/genesis:setup` infers your stack from an existing repo (or asks, for a new one),
then writes `CLAUDE.md` and `specs/_TEMPLATE.md` with the per-project block filled
in. It won't overwrite an existing CLAUDE.md without asking.

### Team auto-registration (optional)
To have collaborators prompted to install automatically, add to the project's
`.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "genesis": { "source": { "source": "github", "repo": "YOUR_GITHUB_USERNAME/genesis" } }
  }
}
```

## Which command do I need?
Start from what you're trying to do:

| I want to… | Run | What you get |
|---|---|---|
| Set up genesis in a project (once) | `/genesis:setup` | Writes `CLAUDE.md` (stack + rules) and `specs/_TEMPLATE.md` |
| Plan a whole system (multi-slice) | `/genesis:roadmap <system>` | A thin plan + ordered slices, each ready to spec |
| Build something that doesn't exist yet | `/genesis:spec <feature>` | An interview, then `specs/<name>.md` for you to approve |
| …then actually build it | `/genesis:build-feature specs/<name>.md` | Working code + tests, all acceptance criteria passing |
| Understand / document existing code | `/genesis:audit-feature <thing>` | `specs/<name>.md` describing what the code does *today*, with gaps and bugs flagged |
| Change or extend existing code | `/genesis:audit-feature <thing>` → `/genesis:improve-feature specs/<name>.md` | The change, behind a characterization net that proves nothing else broke |
| Find out what's wrong (don't know yet) | `/genesis:review <thing>` | A ranked list of real defects, perf hypotheses, and standards violations — each routed to the right command |
| Turn a handed design into editable HTML | `/genesis:design <screen>` | A human-editable `design/<screen>.html` you can tweak directly |
| See / work out-of-scope issues found during other work | `/genesis:findings` | The backlog, ranked; or route one to its fix flow |
| Audit security across the app | `/genesis:security-check [scope]` | Attacker's-eye, checklist-driven vulnerability report routed to `/genesis:fix` |
| Fix a reported bug | `/genesis:fix <what's broken>` | A failing test that reproduces it, a minimal fix, and that test left behind as a regression guard |
| Make working code faster | `/genesis:audit-feature <thing>` (Change type = `refactor`) → `/genesis:optimize-feature specs/<name>.md` | Measured before/after numbers, behavior provably unchanged |

Rules of thumb:
- **Every code change starts with a spec.** New code → `spec`. Existing code → `audit-feature`.
- **Nothing gets built without your approval.** `spec`/`audit-feature` stop for you
  to approve the spec; the orchestrators stop again to approve the plan.
- **Existing code is never changed without a safety net** — `improve-feature` and
  `optimize-feature` characterize current behavior before touching anything.

## Command reference
- `/genesis:setup` — infers your stack (or asks) and writes the project's `CLAUDE.md`
  + spec template. Re-run when the stack changes. Won't overwrite an existing
  CLAUDE.md without asking.
- `/genesis:roadmap <system>` — interview-driven planning for a system too big for
  one spec: settles cross-cutting decisions (schema, enums, side effects, failure
  semantics, permissions) and breaks it into ordered slices. Run with no arguments
  to see where you left off. Optional — single features don't need one.
- `/genesis:design <screen>` — converts a handed image, HTML/CSS, or Claude Design
  export into ONE human-editable `design/<screen>.html` (semantic markup, CSS
  variables, commented sections) you can tweak without an LLM. Writes no app code.
- `/genesis:spec <feature>` — interactive interview for a NEW feature; writes an
  approved spec. Writes no code.
- `/genesis:build-feature specs/<name>.md` — plan (you approve) → build → unit tests
  (test-writer) → acceptance check (e2e-tester) → fix loop (bug-fixer) → perf check
  → summary.
- `/genesis:audit-feature <thing>` — reads EXISTING code, reverse-engineers what it
  does today, flags gaps/bugs, interviews you on the target, tags each behavior
  Keep/Change/Wrong. Writes no code.
- `/genesis:review <thing>` — read-only inspection for bugs, performance risks,
  standards violations, and coverage gaps. Reports and routes; changes nothing.
- `/genesis:security-check [scope]` — deliberate whole-surface security audit
  (default: whole app). Maps every entry point, works a vulnerability checklist
  (authn, authz/IDOR, injection, secrets, exposure, transport, abuse, deps),
  reports and routes to `/genesis:fix`. Read-only. A structured self-review, not a
  pentest — it says what it can't verify statically.
- `/genesis:sync` — after a plugin update, pulls new/changed Standards and rules
  into this project's `CLAUDE.md` (commands update with the plugin, but CLAUDE.md
  doesn't — this closes that gap). Preserves your per-project block; shows a diff
  and asks before applying.
- `/genesis:findings [item]` — no args lists the open `FINDINGS.md` backlog
  (out-of-scope issues logged during other commands), ranked. Name an item to route
  it to its fix flow and mark it done. The backlog is append-only history.
- `/genesis:fix <bug>` — capture the report → investigate (read-only) → **reproduce
  with a failing test** → minimal fix (bug-fixer) → verify the full suite → check
  whether the same bug exists elsewhere. Won't fix what it can't reproduce.
- `/genesis:improve-feature specs/<name>.md` — characterization net first → plan (you
  approve) → change → tests → acceptance → fix loop → perf → summary.
- `/genesis:optimize-feature specs/<name>.md` — baseline + profile → safety net →
  hypothesis → ONE change → re-measure vs baseline (revert if it doesn't beat noise)
  → repeat → summary.

## Getting Opus for planning, Sonnet for building
The planning-heavy commands think better on a stronger model; the mechanical build
work doesn't need it. Claude Code's `opusplan` gives you both — Opus while Plan
Mode is on, Sonnet after — but it keys off **Plan Mode**, not off these commands'
phases. So the model switch only happens if you toggle Plan Mode yourself.

Recommended:
1. `/model opusplan` (set it once as your default — press Enter in the picker).
2. Before an interview/planning command — `/genesis:roadmap`, `/genesis:spec`,
   `/genesis:audit-feature` — enter **Plan Mode** (Shift+Tab). It plans on Opus and,
   as a bonus, is read-only so it can't write by accident.
3. For `/genesis:build-feature` / `/genesis:improve-feature` / `/genesis:optimize-feature`,
   stay in Plan Mode through the plan phase (Opus); approving the plan exits Plan
   Mode and the build drops to Sonnet automatically.

Revising a plan on a smarter model: a roadmap/spec is just markdown — enter Plan
Mode (Opus) and ask Claude to pressure-test the existing file
(`specs/roadmaps/<name>.md`). No special command needed; Plan Mode is read-only so
it critiques without editing until you apply the changes.

> Requires Opus access on your plan. If `opusplan` isn't in your `/model` picker,
> everything runs on Sonnet regardless of Plan Mode — the workflow still works,
> you just don't get the planning boost.

### Keeping output-token cost down
Most output cost is in **code generation and the test loops**, not planning or doc
writing (a spec is a few KB). If your bill runs high:
- The fix loops cap at **3 cycles** — a feature that won't go green is stopped and
  reported rather than regenerating code indefinitely. Runaway loops are the #1
  cost.
- `e2e-tester` and `characterization-tester` run on **Haiku** and report tersely
  (no pasted logs). The reasoning agents (`test-writer`, `bug-fixer`) stay on Sonnet.
- **Smaller slices win**: a big feature that needs several fix cycles re-emits far
  more than small slices that pass first try. Lean on roadmap slicing.
- `/clear` between unrelated features so you're not re-processing stale context.

## Agents (used by the commands — you don't call these directly)
test-writer · characterization-tester · perf-profiler · e2e-tester · bug-fixer.
They read the actual test/lint/benchmark commands from the project's `CLAUDE.md`,
so the same agents work on any stack. Under the plugin they're namespaced
`genesis:<name>`.

## Worked example (Flutter)
```
/genesis:setup                                   # once per project
/genesis:spec authentication — register and login
      ↳ answer its questions, approve specs/authentication.md
/genesis:build-feature specs/authentication.md
      ↳ approve the plan, it builds + tests + fixes until green
```

## Updating
After `/plugin marketplace update genesis`, run **`/genesis:sync`** in each
existing project to pull new Standards/rules into its `CLAUDE.md` — the plugin
update refreshes commands and agents but not a project's already-written CLAUDE.md.

Bump `version` in `.claude-plugin/plugin.json`, push, and users run
`/plugin marketplace update genesis`. That refreshes agents and commands — but not
a CLAUDE.md already written into a project. Re-run `/genesis:setup` (or edit by
hand) to pull template changes into an existing project's CLAUDE.md.

## Repo layout
```
genesis/
├── .claude-plugin/
│   ├── plugin.json          # manifest (name, version, components)
│   └── marketplace.json     # self-referencing catalog (source "./")
├── commands/                # slash commands → /genesis:<name>
├── agents/                  # subagents → genesis:<name>
├── templates/               # CLAUDE.template.md + spec-template.md (used by setup)
└── README.md
```

## Before you publish
- Replace `YOUR_NAME` / `YOUR_GITHUB_USERNAME` in `plugin.json`, `marketplace.json`,
  and this README.
- Push to a public (or team-accessible) git repo named to match the marketplace source.
- Optional: add `graphify-out/` to consuming projects' `.gitignore` if you trial a code graph.

MIT.
