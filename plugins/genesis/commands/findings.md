---
description: Review and act on the findings backlog — out-of-scope issues logged during other work
argument-hint: (none = list open) | a finding to act on
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
---

Work the findings backlog at `FINDINGS.md` (repo root).

## No arguments — triage view
Read `FINDINGS.md` and show me the **open** findings, ranked by severity
(critical → high → medium → low). For each: id, title, where, and the suggested
next step. Summarize counts (e.g. "2 open: 1 high, 1 low"). If there's no
`FINDINGS.md` or nothing open, say so plainly — an empty backlog is a good result.
Do not fix anything in this mode; this is the "what's outstanding" view.

## With a finding named — act on it
Take the finding I point to (by id or title) and route it to the right flow —
don't hand-fix it here beyond the trivial:
- A specific reproducible defect → run the `/genesis:fix` flow for it.
- A security issue → `/genesis:security-check` (scoped) or `/genesis:fix` if it's a
  discrete, well-understood fix (e.g. move a hardcoded key to env).
- A structural/design change → `/genesis:spec` (→ build) or `/genesis:audit-feature`
  → change, as fits.
- Trivial (a one-liner) → fix it directly, with my okay.

Whatever the route, when it's resolved:
- Mark the finding **done** in `FINDINGS.md` (append the resolution + date; do NOT
  delete the entry — the log is history).
- If acting on it revealed more out-of-scope issues, append those as new open
  findings rather than chasing them now.

## Committed-secret findings
If the finding is a hardcoded/committed secret, note that logging + moving it to
env is only half the fix — the secret must also be **rotated** and, ideally,
**purged from git history** (it's still in old commits). Remind me of both;
don't assume moving it to `.env` makes it safe.

## The backlog format (for reference)
`FINDINGS.md` entries, appended by any command that finds an out-of-scope issue:

```
## [severity] <short title>   <!-- severity: critical | high | medium | low -->
- Found: <date>, during <command / spec>
- Where: <file:line or area>
- What: <the issue and why it matters>
- Suggested next step: </genesis:fix … | /genesis:security-check | manual | …>
- Status: open   <!-- → done (<date>): <how it was resolved> -->
```
