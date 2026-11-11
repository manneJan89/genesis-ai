# Genesis

A personal toolkit for starting and building projects with Claude Code.

This repo is a **Claude Code marketplace**: a catalog that can host several
plugins. Today it ships one; component kits and other tools slot in alongside it
without anyone needing to re-add the marketplace.

| Plugin | Commands | What it does |
|---|---|---|
| **genesis** | `/genesis:*` | Spec-driven workflow: build, change, and optimize features behind a test safety net. Stack-agnostic. |
| _(planned)_ genesis-angular | `/genesis-angular:*` | Reusable Angular components + scaffolding |
| _(planned)_ genesis-react | `/genesis-react:*` | Reusable React components + scaffolding |
| _(planned)_ genesis-flutter | `/genesis-flutter:*` | Reusable Flutter widgets + scaffolding |

## Install

```
/plugin marketplace add manneJan89/genesis-ai
/plugin install genesis@genesis
/reload-plugins
```

Then, inside any project:

```
/genesis:setup
```

> `/plugin install genesis@genesis` reads as *plugin `genesis` from marketplace
> `genesis`*; the repeated name is expected.

## Quick start

```
/genesis:setup                                     # once per project
/genesis:spec authentication — register and login  # interview → spec, you approve
/genesis:build-feature specs/authentication.md     # plan → build → test → fix → verify
```

**Everything else** (which command to use when, the safety model, standards,
agents, updating) is in [`plugins/genesis/PLUGIN.md`](plugins/genesis/PLUGIN.md).
This README only covers the marketplace, so the two can't drift apart.

## Updating

```
/plugin marketplace update genesis
/reload-plugins
```

Then run `/genesis:sync` in each project: a plugin update refreshes commands,
agents and skills, but not a project's already-written `CLAUDE.md`.

## Adding another plugin to the suite

1. Create `plugins/<name>/` with `.claude-plugin/plugin.json`, plus any
   `commands/`, `agents/`, or `skills/` directories.
2. Add an entry to `.claude-plugin/marketplace.json` pointing at
   `./plugins/<name>`.
3. Push. Users run `/plugin marketplace update genesis` and can then
   `/plugin install <name>@genesis`.

Keep each plugin's name short: it becomes the prefix on every one of its commands.

MIT.
