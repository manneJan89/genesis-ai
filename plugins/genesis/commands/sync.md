---
description: Pull the latest Genesis Standards/rules into this project's CLAUDE.md after a plugin update
argument-hint: (no arguments)
allowed-tools: Read, Write, Edit, Grep, Glob
---

Bring this project's `CLAUDE.md` up to date with the current Genesis template,
**without clobbering my project-specific edits.**

Context: plugin updates refresh commands and agents, but NOT a project's
`CLAUDE.md` (it's a per-project file written once at setup). So new rules — the
findings-capture rule, design gate, modification contract, logging, security,
scale, etc. — don't reach an existing project until this sync runs.

## 1. Compare
Read the project's `CLAUDE.md` and the current template at
`${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.template.md`. Focus on the **rule
sections** that ship in the template — Standards, and the workflow/testing/
token-hygiene guidance above the per-project block. Diff them:
- rules present in the template but **missing** from my CLAUDE.md → candidates to add
- rules **changed** in the template → candidates to update
- rules I appear to have **customized** locally → flag, don't overwrite

Do NOT touch my **PER-PROJECT block** (Project, Commands, Component libraries,
Conventions, Codebase map, Logging destinations, Test baseline) — those are mine
and project-specific. Sync only the shared rule sections.

## 2. Show me the diff and ask
Present a clear, grouped summary:
- **New rules to add** (e.g. "Out-of-scope findings: capture to FINDINGS.md")
- **Rules that changed** (old → new, briefly)
- **Local customizations detected** (I'll decide whether to keep mine or take the
  template's — default to keeping mine)

Ask which to apply. Don't apply anything until I confirm.

## 3. Apply
On my okay, merge the approved rules into `CLAUDE.md` in place — add missing ones,
update changed ones — preserving my per-project block and any customizations I
chose to keep. Also:
- If the template references files this project lacks because they postdate its
  setup (e.g. `FINDINGS.md`), create them from their templates
  (`${CLAUDE_PLUGIN_ROOT}/templates/FINDINGS.template.md`) so the newly-synced
  rules actually work.
- Note at the top of CLAUDE.md the plugin version synced to (read it from
  `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`), so next time it's obvious
  whether a sync is needed.

## 4. Confirm
Summarize what changed and confirm the per-project block is untouched. Remind me to
commit the CLAUDE.md change so the update is recorded.
