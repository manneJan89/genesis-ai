# Components

The authoritative registry of this project's **shared UI components**. Agents read
this (not a codebase scan) to know which house component to use for a given
primitive, and what it can and can't do. If a component is listed here, using a raw
HTML/native equivalent instead is a defect.

Kept current automatically: when a shared component is created or changed, the
build updates its entry. You can also add/edit entries by hand. Manage with
`/genesis:components`. If this file and the code disagree, the code is truth —
`/genesis:review` checks for drift.

<!-- Format, one entry per component:

## <ComponentSelector or Name>   — e.g. surespace-button
- **Use for**: <the primitive it replaces — e.g. all buttons>
- **Location**: <path>
- **API**: <key inputs/props — e.g. tone="brand|danger", [disabled], (click)>
- **Can**: <what it supports>
- **Can't / not yet**: <known gaps — extend the component (ask first), don't fork>

-->

_No components registered yet. They're added here automatically as shared
components are built, or manually by you. A new project having none is normal._
