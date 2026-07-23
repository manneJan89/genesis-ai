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
| Build something that doesn't exist yet | `/genesis:spec <feature>` | An interview, then `specs/<name>.md` for you to approve |
| …then actually build it | `/genesis:build-feature specs/<name>.md` | Working code + tests, all acceptance criteria passing |
| Understand / document existing code | `/genesis:audit-feature <thing>` | `specs/<name>.md` describing what the code does *today*, with gaps and bugs flagged |
| Change or extend existing code | `/genesis:audit-feature <thing>` → `/genesis:improve-feature specs/<name>.md` | The change, behind a characterization net that proves nothing else broke |
| Fix a bug in existing code | same pair (set Change type = `bugfix`) | Correct behavior + a regression test |
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
- `/genesis:spec <feature>` — interactive interview for a NEW feature; writes an
  approved spec. Writes no code.
- `/genesis:build-feature specs/<name>.md` — plan (you approve) → build → unit tests
  (test-writer) → acceptance check (e2e-tester) → fix loop (bug-fixer) → perf check
  → summary.
- `/genesis:audit-feature <thing>` — reads EXISTING code, reverse-engineers what it
  does today, flags gaps/bugs, interviews you on the target, tags each behavior
  Keep/Change/Wrong. Writes no code.
- `/genesis:improve-feature specs/<name>.md` — characterization net first → plan (you
  approve) → change → tests → acceptance → fix loop → perf → summary.
- `/genesis:optimize-feature specs/<name>.md` — baseline + profile → safety net →
  hypothesis → ONE change → re-measure vs baseline (revert if it doesn't beat noise)
  → repeat → summary.

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
