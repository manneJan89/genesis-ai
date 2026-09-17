---
description: White-box red-team of your OWN code — adversarial attack-chain + blast-radius analysis, logged with replication and fix
argument-hint: [scope: whole app (default), a feature, or an area]
allowed-tools: Read, Grep, Glob, Bash, Agent
---

Red-team the codebase: ${ARGUMENTS:-the whole application}

You have the source (white-box) — so reason like an attacker **who already has the
blueprints**. Don't fumble for a back door; read the code and find the real holes:
the one route missing an auth check, the business-logic flaw, the race condition,
the data that's exposed if a layer fails. That's the advantage — use it.

## Rules (read first — these are hard)
- **Analyze and DESCRIBE exploits; never fire live ones.** Aggression is in the
  thoroughness of the attack reasoning, not in execution. Do not actually run SQL
  injection, auth-bypass requests, data mutation, or anything that hits a live/prod
  system or a third party. Construct each exploit as a described proof-of-concept +
  replication steps the user runs in a safe environment.
- **Only your own code.** Scope is THIS codebase and infrastructure the user owns,
  read locally. Never probe live external services, third-party APIs, or anything
  not owned — even ones this app calls. White-box means you don't need to.
- **Read-only.** No edits, no fixes here — you find and report; fixing is the
  user's call via `/genesis:fix` or a spec.
- **Use the strongest model.** Finding non-obvious attack chains is exactly where a
  top model earns its cost — tell the user to run this on Opus (via Plan Mode) if
  they aren't; a weaker model will miss chains.

## Two lenses — run both
**A) Break in** — how an attacker gets in or escalates. Chain weaknesses, don't
list them in isolation ("if I combine the missing check at X with the leaky field
at Y, I can read any user's records").
**B) Blast radius** — assume a layer already failed (attacker in the DB, a token
leaked, an internal service compromised): what's exposed *then*? Unencrypted PII/ID
numbers at rest, secrets recoverable from config/backups, no field-level encryption
or tenant segmentation, over-broad DB permissions. This is often the more valuable
half — it turns a small breach into a catastrophic one.

## Method — parallel threat-class passes
Run these as focused passes (delegate to sub-agents where it helps breadth; each
takes a class and reports independently so no single tunnel-vision):
1. **AuthZ / IDOR / privilege escalation** — can user A reach B's data by changing
   an id? Can a normal user hit admin actions? Per-tenant isolation holes.
2. **Injection** — SQL/NoSQL, command, path, template; anywhere untrusted input
   reaches a query, shell, or filesystem.
3. **Auth / session / tokens** — bypassable guards, weak/again-usable tokens,
   fixation, missing expiry, fail-open logic.
4. **Business-logic abuse** — the stuff checklists miss: replay, race conditions,
   negative/overflow quantities, skipped workflow steps, price/qty tampering,
   approval bypass.
5. **Data exposure & secrets at rest** — the blast-radius lens: what's stored
   unencrypted, logged, over-returned, or recoverable that shouldn't be.

## Report — the format is the point
Write to `specs/security/hack-<date>.md`, findings ranked critical→low. **Every
finding MUST have all four:**
- **What** — the vulnerability, and the attack chain if it's a chain.
- **Replicate** — exact steps to reproduce it (to run in a safe env), with the
  file:line evidence that proves it exists in the code.
- **Fix** — the concrete remedy.
- **Severity** — critical / high / medium / low, and the impact (what the attacker
  gains).

Add an **"Also worth hardening"** section for defense-in-depth suggestions that
aren't a live exploit but are real risk — e.g. "ID numbers stored unencrypted; if
the DB is ever accessed, they're exposed — encrypt at rest." Don't force these into
the exploit list; they're advice, still valuable.

Append a one-line summary of each finding to `FINDINGS.md` (so the backlog stays the
single triage view) pointing at the full report. Then show me the ranked list and
ask which to act on — route each to `/genesis:fix` or `/genesis:spec`. Fix nothing
yourself.

## Honest limits (state these in the report)
This is white-box adversarial *analysis*, not a live penetration test — no dynamic
fuzzing, no real exploitation, no runtime discovery. It finds design- and code-level
attack surface well; it won't find what only appears when exploits actually run. For
a system handling real PII (e.g. client data under POPIA/GDPR), this is a strong
first pass, not a substitute for a professional pentest before production.
