# genesis — a spec-driven workflow plugin for Claude Code

Bundles the agents and commands for a spec-driven build / change / optimize
workflow into an installable Claude Code plugin. Stack-agnostic — works on
Angular/Node, Flutter, Go, Python, anything. You fill in the stack once per
project with `/genesis:setup`.

## Install
This repo is both the plugin and its marketplace (self-referencing), so:

```
/plugin marketplace add manneJan89/genesis-ai
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
    "genesis": { "source": { "source": "github", "repo": "manneJan89/genesis-ai" } }
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
| Replace a screen/component's look (keep behavior) | `/genesis:redesign <unit>` | New design approved, then UI replaced with behavior held constant |
| See/manage which shared components to use | `/genesis:components` | The house-component registry; scan to seed it |
| See / work out-of-scope issues found during other work | `/genesis:findings` | The backlog, ranked; or route one to its fix flow |
| Audit security across the app | `/genesis:security-check [scope]` | Attacker's-eye, checklist-driven vulnerability report routed to `/genesis:fix` |
| Fix a reported bug | `/genesis:fix <what's broken>` | A failing test that reproduces it, a minimal fix, and that test left behind as a regression guard |
| Make working code faster | `/genesis:audit-feature <thing>` (Change type = `refactor`) → `/genesis:optimize-feature specs/<name>.md` | Measured before/after numbers, behavior provably unchanged |
| Stop mid-task, or carry on from a previous session | `/genesis:handoff` (or `resume`) | `HANDOFF.md`: task, phase, the exact next step, checked against git by the next session |
| Aggressively red-team your own code | `/genesis:hack [scope]` | White-box attack chains and blast radius, each with how to replicate and fix; never runs live attacks |
| Pull new rules into a project after updating the plugin | `/genesis:sync` | Your `CLAUDE.md` brought up to date, per-project block untouched, diff shown first |

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
- `/genesis:build-feature specs/<name>.md` — plan (you approve) → design gate →
  **tests first** (test-writer writes them from the spec; they must fail) → build
  until green → acceptance check (e2e-tester) → fix loop (bug-fixer) → perf check
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
- `/genesis:components [scan|name]` — view/manage `COMPONENTS.md`, the registry of
  shared UI components agents must use instead of raw elements. No args lists it
  (empty is normal on a new project); `scan` seeds it from the codebase.
- `/genesis:hack [scope]` — white-box adversarial red-team of your OWN code. Reads
  the source and reasons like an attacker with the blueprints: attack chains (authz/
  IDOR, injection, auth, business-logic) plus blast-radius (what's exposed if a layer
  fails — unencrypted PII at rest, recoverable secrets). Describes exploits + how to
  replicate + how to fix; never runs live attacks, only your code. Best on the
  strongest model. More aggressive than `security-check` (which is a static audit).
- `/genesis:findings [item]` — no args lists the open `FINDINGS.md` backlog
  (out-of-scope issues logged during other commands), ranked. Name an item to route
  it to its fix flow and mark it done. The backlog is append-only history.
- `/genesis:fix <bug>` — capture the report → investigate (read-only) → **reproduce
  with a failing test** → minimal fix (bug-fixer) → verify the full suite → check
  whether the same bug exists elsewhere. Won't fix what it can't reproduce.
- `/genesis:redesign <screen-or-component>` — replaces the visuals of ONE screen or
  shared component while keeping behavior. Render-surface audit (not behavioral),
  new design written to `design/` and approved before code, behavior netted, new
  features found are logged not built. Multi-screen redesign → use a roadmap.
- `/genesis:handoff [resume|clear]` — writes `HANDOFF.md`, a one-page note of what
  this session was busy with, how far it got, and the exact next step, so the next
  session carries on instead of rediscovering. The build commands (build-feature,
  improve-feature, redesign, optimize-feature, fix) write it at every phase
  boundary and when they stop early, and delete it when the work is done. Every
  session reads it first and checks it against git (branch, commits since, what's
  uncommitted); where they disagree, the repo wins. Approvals never carry over.
  It's committed with your work (not by Genesis), so commit and push to pick up on
  another machine. Never holds secrets or production data.
- `/genesis:improve-feature specs/<name>.md` — characterization net first → plan (you
  approve) → target tests first (red) → change (green) → acceptance → fix loop → perf → summary.
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
- `e2e-tester` runs on **Haiku** and reports tersely (no pasted logs); it runs
  on every fix cycle, so it's where a cheap model saves the most. The agents
  whose misses become silent bugs stay on Sonnet: `test-writer`, `bug-fixer`,
  and `characterization-tester` (a behavior it fails to pin has no safety net,
  so a later change can break it unnoticed).
- **Smaller slices win**: a big feature that needs several fix cycles re-emits far
  more than small slices that pass first try. Lean on roadmap slicing.
- `/clear` between unrelated features so you're not re-processing stale context.

## Safety: what's enforced and what's only instructed
Genesis's rules in CLAUDE.md are instructions. A model can reason its way past an
instruction, so the protections are layered, strongest first:

1. **No production credentials on the dev machine** (your infrastructure; setup's
   Data safety step checks and warns). The only layer that can't be bypassed.
2. **Branch protection on GitHub** for `main` (you set it; setup checks).
3. **Permission guardrails** (`templates/permissions.template.json`, offered by
   `/genesis:setup` and `/genesis:sync`). Enforced by Claude Code itself in every
   mode. Tested against Claude Code 2.1.299:
   - denied: force-push (incl. `--force-with-lease`, `-f`, env-prefixed and
     compound forms), `push --delete`, `reset --hard`, `clean -f`, `branch -D`,
     and reading `.env`/`.env.local`/`.env.production` or key files, by Claude's
     file tools *and* by `cat`, `grep`, `sed`, `head`;
   - prompts (even in auto mode): any command with `--delete`/`--drop`/
     `--purge`/`--wipe`/`--truncate`, `branch -d`, `source .env`, `bash -c` /
     `sh -c`, database command-line tools, migrations, `firebase deploy`;
   - still allowed: `git clean -n`, `--dry-run`, `.env.example`.
   - **Not covered:** `git push origin :branch` (the colon delete form can't be
     expressed as a rule), a full program path for git (`/usr/bin/git …`), `git -c
     … push`, and a script with a destructive action hard-coded inside and no
     flag. Rule matching is case-sensitive.
   - Since v0.25 also prompted: recursive deletes (`rm -r`, `rm -rf`, `rm -vrf`,
     `rm dir -rf`, `rm --recursive`), `find -delete`, and `xargs rm` (tested
     against Claude Code 2.1.322; a plain single-file `rm` is not caught). The
     PowerShell delete aliases (`rm`, `del`, `erase`, `rd`, `ri`, `rmdir`) are in
     the rule set but untested, since no Windows machine was available.
   - The only `allow` rules setup offers are your exact test and lint commands.
4. **Genesis's rules** (stop means stop, never use found credentials, never act
   destructively on live data, never route around a denial).

**What Genesis commands pre-approve.** A command's `allowed-tools` line doesn't
restrict tools; it approves them without a prompt while the command runs.
Before v0.25, most Genesis commands listed bare `Bash`, which silently approved
*every* shell command not caught by a deny or ask rule (tested: `rm -rf` and
`node <script>` ran with no prompt inside a Genesis command, and prompted
outside it). Now no Genesis command pre-approves the shell except for read-only
git (`status`, `diff`, `log`, `show`), dependency audits in `security-check`,
and the single command `rm HANDOFF.md`. Everything else follows your own
permission settings, so in default mode Claude asks. To stop the build loop
prompting on every test run, allow your exact test command (setup offers this).

## Skills
- **ux** — design/UX reasoning (visual hierarchy, spacing & rhythm, the required UI
  states, affordance & feedback, form UX, accessibility, avoiding the generic
  AI-generated look). The UI commands (`build-feature`, `redesign`, `design`,
  `review`) consult it automatically when a task involves UI; it stays out of
  backend/logic work. Principles, not a house style — it makes design decisions
  deliberate without imposing one look. Invoke directly with "use the ux skill on
  this screen" if you want it on a plain prompt.

- **standards-code / standards-security / standards-performance / standards-ui /
  standards-logging** — the detailed Genesis Standards, split by topic. CLAUDE.md
  keeps a short always-on core and a table saying which of these to load for which
  work; commands and agents load them before writing or reviewing code. Hidden from
  the `/` menu (Claude loads them; you don't need to).
- **finding-confidence** — the disprove-then-score filter `review`, `security-check`
  and `hack` run on every finding: 70+ is reported, 40–69 is listed as unverified,
  below 40 is dropped (critical security findings are never dropped silently).

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
To release: bump `version` in `.claude-plugin/plugin.json` (and the
`genesis-standards-version` marker in `templates/CLAUDE.template.md` if the
template changed), then push.

To take an update: `/plugin marketplace update genesis`, `/reload-plugins`, then
**`/genesis:sync`** in each existing project. The plugin update refreshes
commands, agents and skills, but not a project's already-written `CLAUDE.md`;
sync closes that gap and offers any new permission rules.

## Independence: the workflow never requires a component kit

The workflow plugin stands alone. Install `genesis` and nothing else, and it works
fully — it generates plain Flutter / Angular / React code with no Genesis imports
and no Genesis dependency in your `pubspec.yaml` or `package.json`.

Even with a component kit installed, a project only uses it if that project's own
`CLAUDE.md` opts in under **Component libraries**. The default written by
`/genesis:setup` is `none`, which instructs Claude to build project-local
components and not to suggest external component packages. Adding a dependency
always requires your approval.

So: use Genesis on a project with a bespoke design system, a client's existing
component library, or nothing at all — the workflow doesn't care.

## Your standards live here

The rules every project inherits live in two places:

- **`skills/standards-*/`** — the detailed Standards (code quality,
  security, performance/cost/scale, UI, logging). They load only when the work
  calls for them, so they don't cost context on every call. Edit a skill once and
  every project gets it on the next `/plugin marketplace update genesis` — no
  `/genesis:sync` needed.
- **`templates/CLAUDE.template.md`** — the short always-on core
  (honesty, no secrets, live-data safety, stop means stop, change only what was
  asked, log out-of-scope findings, session handoff, no emojis) plus the table
  telling Claude which standards skill to load when. Changes here reach existing
  projects via `/genesis:sync`.

Project-specific rules go in that project's own `CLAUDE.md` under Conventions.

## Plugin layout
```
plugins/genesis/
├── .claude-plugin/
│   └── plugin.json          # manifest (name, version, metadata)
├── commands/                # slash commands → /genesis:<name>
├── agents/                  # subagents → genesis:<name>
├── skills/                  # ux, standards-*, finding-confidence (load on demand)
├── templates/               # CLAUDE.template.md + spec/roadmap/etc. templates
└── PLUGIN.md
```
The marketplace catalog lives at the repo root in `.claude-plugin/marketplace.json`.

Optional: add `graphify-out/` to consuming projects' `.gitignore` if you trial a
code graph.

MIT.
