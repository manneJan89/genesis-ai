# Genesis

A personal toolkit for starting and building projects with Claude Code.

This repo is a **Claude Code marketplace** — a catalog that can host several
plugins. Today it ships one; component kits and other tools slot in alongside it
without anyone needing to re-add the marketplace.

| Plugin | Commands | What it does |
|---|---|---|
| **genesis** | `/genesis:*` | Spec-driven workflow — build, change, and optimize features behind a test safety net. Stack-agnostic. |
| _(planned)_ genesis-angular | `/genesis-angular:*` | Reusable Angular components + scaffolding |
| _(planned)_ genesis-react | `/genesis-react:*` | Reusable React components + scaffolding |
| _(planned)_ genesis-flutter | `/genesis-flutter:*` | Reusable Flutter widgets + scaffolding |

## Install

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
in. It won't overwrite an existing `CLAUDE.md` without asking.

> `/plugin install genesis@genesis` reads as *plugin `genesis` from marketplace
> `genesis`* — the repeated name is expected.

See [`plugins/genesis/PLUGIN.md`](plugins/genesis/PLUGIN.md) for the full command
reference and worked examples.

## Quick start (new Flutter app)

```
/genesis:setup                                     # once per project
/genesis:spec authentication — register and login  # interview → spec, you approve
/genesis:build-feature specs/authentication.md     # plan → build → test → fix → verify
```

## Which command do I need?

| I want to… | Run |
|---|---|
| Set up Genesis in a project (once) | `/genesis:setup` |
| Plan a whole system (multi-slice) | `/genesis:roadmap <system>` |
| See where I left off | `/genesis:roadmap` (no arguments) |
| Build something new | `/genesis:spec <feature>` → `/genesis:build-feature specs/<name>.md` |
| Find out what's wrong with existing code | `/genesis:review <thing>` |
| Fix a reported bug | `/genesis:fix <what's broken>` |
| Understand existing code | `/genesis:audit-feature <thing>` |
| Change / extend existing code | `/genesis:audit-feature <thing>` → `/genesis:improve-feature specs/<name>.md` |
| Make working code faster | `/genesis:audit-feature <thing>` (type=refactor) → `/genesis:optimize-feature specs/<name>.md` |

## Independence: the workflow never requires the component kits

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

`plugins/genesis/templates/CLAUDE.template.md` holds the rules every project
inherits — performance, cost-consciousness with metered services, DRY/KISS, and
enums over magic strings. Edit them **once** here and every project picks them up
the next time you run `/genesis:setup`. Project-specific rules go in that project's
own `CLAUDE.md`.

## Team auto-registration (optional)

Add to a project's `.claude/settings.json` so collaborators are prompted to install:

```json
{
  "extraKnownMarketplaces": {
    "genesis": { "source": { "source": "github", "repo": "YOUR_GITHUB_USERNAME/genesis" } }
  }
}
```

## Adding another plugin to the suite

1. Create `plugins/<name>/` with `.claude-plugin/plugin.json`, plus any
   `commands/`, `agents/`, or `skills/` directories.
2. Add an entry to `.claude-plugin/marketplace.json` pointing at
   `./plugins/<name>`.
3. Push. Users run `/plugin marketplace update genesis` and can then
   `/plugin install <name>@genesis`.

Keep each plugin's name short — it becomes the prefix on every one of its commands.

## Repo layout

```
genesis/
├── .claude-plugin/
│   └── marketplace.json          # catalog of all Genesis plugins
├── plugins/
│   └── genesis/                  # the spec-driven workflow plugin
│       ├── .claude-plugin/plugin.json
│       ├── commands/             # → /genesis:<name>
│       ├── agents/               # → genesis:<name> (called by the commands)
│       ├── templates/            # CLAUDE.md + spec templates used by /genesis:setup
│       └── PLUGIN.md
└── README.md
```

## Updating

Bump `version` in the relevant `plugin.json`, push, and run
`/plugin marketplace update genesis`. That refreshes commands and agents — but not
a `CLAUDE.md` already written into a project. Re-run `/genesis:setup` (or edit by
hand) to pull template changes into an existing project.

## Before you publish

- Replace `YOUR_NAME` / `YOUR_GITHUB_USERNAME` in `.claude-plugin/marketplace.json`,
  `plugins/genesis/.claude-plugin/plugin.json`, and this README.
- Push to a git repo named `genesis` (public, or accessible to your team).

MIT.
