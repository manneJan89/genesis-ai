---
description: Set up the genesis workflow in this project (writes CLAUDE.md + spec template; infers or asks your stack)
argument-hint: (no arguments)
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
---

Set up the genesis spec-driven workflow in the current project. Do this carefully
and confirm with me before writing files.

## 1. Detect: new or existing project
Look for stack signals in the repo root (and one level down): `package.json`,
`angular.json`, `pubspec.yaml`, `go.mod`, `Cargo.toml`, `pyproject.toml`,
`requirements.txt`, `pom.xml`, `build.gradle`, `Gemfile`, `composer.json`, etc.,
plus the presence of a `src/`, `lib/`, or `test/` tree.

- **Existing project** (signals found): infer the stack, package manager, test
  runner, lint/format tool, and likely build/run commands from those files. Read
  the manifest(s) to get exact script names (e.g. the `scripts` block, or the
  test config). Don't guess where you can read.
  - Also scan dependencies for **metered third-party services** (firebase,
    firestore, supabase, algolia, hosted queues, paid APIs, etc.). If any are
    present, note them and add a project-specific cost note under Conventions
    (e.g. "Firestore is metered — batch reads, cache, avoid per-item round-trips"),
    reinforcing the universal cost standard.
- **New / empty project** (no signals): ask me which language/framework I'm using
  (one question), then propose sensible default commands for that stack.

## 2. Propose the per-project config
Show me a filled-in **Commands** block (install, run, build, test-all,
test-single, lint, format, benchmark) and a first-draft **Codebase map** based on
what you found. State your confidence and flag anything you're unsure about.
Let me correct it before you write anything.

Also set **Component libraries**. Detect whether the project already depends on a
shared component library; if none is found, set it to **none** (the default) — do
not assume a Genesis component kit is wanted. Only name a shared library here if
it's already a dependency or I explicitly ask for it. This keeps the project's
code independent of any component package.

## 3. Write the files
Use the bundled templates (do not invent structure):
- `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.template.md`
- `${CLAUDE_PLUGIN_ROOT}/templates/spec-template.md`

Then:
- Write `specs/_TEMPLATE.md` from the spec template (create `specs/` if missing).
- **CLAUDE.md handling:**
  - If the project has **no** `CLAUDE.md`, write one from the template with the
    per-project block filled in from step 2.
  - If a `CLAUDE.md` **already exists**, do NOT overwrite it. Show me the genesis
    rules + filled per-project block and ask whether to append them to the
    existing file or write them to a separate `CLAUDE.genesis.md` that the main
    file can reference. Wait for my choice.

## 4. Confirm and hand off
Show me the final CLAUDE.md, then tell me the available commands:
- `/genesis:spec <feature>` → `/genesis:build-feature specs/<name>.md`  (new feature)
- `/genesis:audit-feature <thing>` → `/genesis:improve-feature specs/<name>.md`  (change existing)
- `/genesis:audit-feature <thing>` (type=refactor) → `/genesis:optimize-feature specs/<name>.md`  (make it faster)

Remind me that I can re-run `/genesis:setup` anytime the stack changes, and that
updating the plugin (`/plugin marketplace update genesis`) refreshes the agents and
commands but not an already-written CLAUDE.md.
