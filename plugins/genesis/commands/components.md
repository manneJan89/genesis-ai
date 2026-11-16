---
description: View or manage the shared UI component manifest (COMPONENTS.md)
argument-hint: (none = list) | "scan" to find candidates | "guard" to add the guard test | a component to add/update
allowed-tools: Read, Write, Edit, Grep, Glob
---

Manage `COMPONENTS.md` — the registry of shared UI components that agents consult
before writing UI (so they use house components instead of raw elements).

## No arguments — list
Read `COMPONENTS.md` and show the registered components with their "use for" and
key capabilities.

- **If it's empty** (a new project, or none registered yet): say so plainly — this
  is a normal, valid state, not a problem. Then offer the next step: "Want me to
  scan the codebase for likely shared-component candidates?" Do NOT invent entries
  to look populated — an empty honest manifest beats a fabricated one, because the
  whole point is that it's authoritative.
- **If there's no `COMPONENTS.md` at all**: create it from
  `${CLAUDE_PLUGIN_ROOT}/templates/COMPONENTS.template.md` first, then treat as empty.

## "scan" — find candidates
Search the codebase for existing shared/reusable UI components (component selectors,
`*-button`/`*-input`-style names, a components/ directory, the project's component
library). Propose entries — component, what it's for, its API, what it can/can't do
— and, on my okay, write them into `COMPONENTS.md`. Confirm each; don't guess an
API you didn't read. Then offer the guard test (`guard` below): the manifest says
what to use, the guard makes sure it's used.

## "guard" — add the component guard test
A check that can't be forgotten: a test in the project's own test framework that
fails when a raw element appears where a house component exists. It runs with
every test run, so the build loop catches it like any other failing test.

1. Collect each registered component's **Replaces** list (ask me to fill any that
   are missing; don't guess).
2. Write one test file (e.g. `component-guard.spec.ts`, `component_guard_test.dart`)
   that scans the project's UI files (templates, `.html`, `.tsx`, widget files),
   skips each house component's own folder, and fails with `file:line → use
   <component>` for every hit. Plain file reading and string or regex matching:
   no new dependencies.
3. For React, mention the alternative of ESLint's `react/forbid-elements` rule,
   which flags it in the editor too; I choose.
4. Show me the test and its first run before keeping it. Existing violations make
   it red: list them and ask whether to fix them now (each swap is a small
   change) or start the guard with them on an allowlist that only shrinks.

Re-run `guard` after adding a component, so the test learns its Replaces list.

## A component named — add or update
Add or update that component's entry (use-for, location, API, can, can't). Keep
entries short and accurate.

## Keeping it honest
This manifest is only useful if it's true. Two guards:
- **Write-back**: whenever a shared component is created or changed during a build,
  its entry here is updated (the build orchestrators do this).
- **Drift check**: `/genesis:review` verifies the manifest against the actual
  components and flags mismatches. If you suspect drift, run `/genesis:components
  scan` to reconcile.

Never fabricate capabilities. "Can't / not yet" gaps are extended by adding to the
component (ask first) — never by forking a parallel one.
