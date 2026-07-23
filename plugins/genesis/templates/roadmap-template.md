# Roadmap: <system name>

Status: active | complete | on hold
Last updated: <date>

**Next up:** <first non-done slice, or "all slices complete">

> A thin index, not a spec. It holds the *shape* of cross-cutting decisions; each
> slice's spec holds the detail. Nothing here should be implementable on its own.
> Living document — revise as slices are built and we learn.

## Goal
One or two sentences: what this system is for and who uses it.

## Shared foundations
Decisions that span slices. **Slice 1 commits these** — later slices inherit them.

- **Data shape**: <core entity + its fields at a high level>
- **Enums**: <fixed value sets, e.g. Status = draft | sent | accepted | rejected>
- **Cross-cutting flags**: <e.g. soft delete via `deletedAt`; ALL reads exclude
  deleted by default — must exist from slice 1>
- **Permissions**: <who may do what>
- **Failure semantics**: <what happens when the main write succeeds but a side
  effect fails>

## External dependencies (reuse — do not rebuild)
Existing components this system uses.

- <component> — used by <which slices>

## Side effects
One row per operation. Detail lives in each slice's spec.

| Operation | Effect A | Effect B |
|---|---|---|
| create | | |
| edit | | |
| delete | | |

## Slices
Each becomes one `/genesis:spec <slice-name>`.

| # | Slice | Status | Spec | Notes |
|---|---|---|---|---|
| 1 | `<slice-name>` | planned | — | carries shared foundation |
| 2 | `<slice-name>` | planned | — | |
| 3 | `<slice-name>` | planned | — | |

Status: `planned` → `in progress` → `done` (or `blocked — <reason>`).
The build orchestrators update this automatically when a slice's spec is built.

## Cross-slice dependencies
Things an early slice must include for a later one to work.

- <e.g. `deletedAt` exists from slice 1 so list views filter correctly before
  delete ships>

## Deliberately not doing
Excluded on purpose, with the reason — so it isn't re-litigated in every slice
interview. (Distinct from "not thought of yet", which becomes a slice.)

- <thing> — <reason>

## Open questions
Unresolved cross-cutting questions. Resolve before the affected slice is specced.

- <question>
