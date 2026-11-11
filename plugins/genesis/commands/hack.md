---
description: White-box red-team of your OWN code — adversarial attack-chain + blast-radius analysis, logged with replication and fix
argument-hint: [scope: whole app (default), a feature, or an area]
allowed-tools: Read, Grep, Glob, Agent, Skill, Bash(git status*), Bash(git diff*), Bash(git log*), Bash(git show*)
---

Red-team the codebase: ${ARGUMENTS:-the whole application}

You have the source (white-box) — so reason like an attacker **who already has the
blueprints**. Don't fumble for a back door; read the code and find the real holes:
the one route missing an auth check, the business-logic flaw, the race condition,
the data that's exposed if a layer fails. That's the advantage — use it.

Load `genesis:standards-security` first (it loads the profile files listed in
CLAUDE.md → Profiles). It is the floor, not the ceiling: every violation of it is
a finding, but your job is to go past the checklist to the chains and logic flaws
a checklist misses.

**What this is for.** Security is built in when a feature is specced and built,
and checked per change by the build's security pass. Hack is the adversarial
test of that implementation: it hunts for the holes that got through anyway, and
proves each one with a test so the fix is verifiable and the hole can't quietly
come back.

## Rules (read first — these are hard)
- **Analyze and DESCRIBE exploits; never fire live ones.** Aggression is in the
  thoroughness of the attack reasoning, not in execution. Do not actually run SQL
  injection, auth-bypass requests, data mutation, or anything that hits a live/prod
  system or a third party. Construct each exploit as a described proof-of-concept +
  replication steps the user runs in a safe environment.
- **Only your own code.** Scope is THIS codebase and infrastructure the user owns,
  read locally. Never probe live external services, third-party APIs, or anything
  not owned — even ones this app calls. White-box means you don't need to.
- **Read-only.** No edits, no fixes here. You find each hole and write the exact
  test that proves it; `/genesis:fix` then writes that test into the project,
  runs it against local code to confirm it fails, and fixes it. Tests only ever
  run against local code, mocks or an emulator, never a live system.
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

Add the passes that match the project's profiles, and skip passes that don't
apply (a single-player Unity game has no IDOR pass; say so in the report):
- **unreal / unity (multiplayer):** client-authoritative values (damage,
  currency, position), Server RPCs that trust their arguments, replicated state
  the client can write, save-file and mod tampering, purchase validation done
  only on the device.
- **native / unreal (C++):** memory corruption reachable from input: lengths and
  counts from files or packets, integer overflow in size maths, unchecked
  indexes, use-after-free across object lifetimes.
- **firebase:** rules that allow more than the app needs: `request.auth != null`
  as the whole rule, client-writable ownership or role fields, collections with
  no rules.

## Report — the format is the point
First merge the passes' results and load the `genesis:finding-confidence` skill:
dedupe, run the disprove pass on each chain (is there a guard somewhere along it
that breaks the chain?), and score. Only 70+ goes in the exploit list; critical
chains at 40–69 go in marked *unverified*; the rest follow the skill's thresholds.

Write to `specs/security/hack-<date>.md`, findings ranked critical→low. **Every
finding MUST have all five:**
- **What** — the vulnerability, and the attack chain if it's a chain.
- **Replicate** — written as **the failing test**: the exact request, call or
  input, and the assertion it should meet (e.g. "POST /api/login with body
  `{"email":"a@b.c","password":{"$ne":""}}` → expect 400; today it returns
  200"), in the project's test framework where you can, plus the file:line
  evidence that proves the hole exists.
- **Fix** — the concrete remedy.
- **Severity** — critical / high / medium / low, and the impact (what the attacker
  gains).
- **Confidence** — the 0–100 score, and for anything under 90, which link in the
  chain you couldn't confirm statically.

Add an **"Also worth hardening"** section for defense-in-depth suggestions that
aren't a live exploit but are real risk — e.g. "ID numbers stored unencrypted; if
the DB is ever accessed, they're exposed — encrypt at rest." Don't force these into
the exploit list; they're advice, still valuable.

Append a one-line summary of each finding to `FINDINGS.md` (so the backlog stays the
single triage view) pointing at the full report. Then show me the ranked list and
ask which to act on. Each one I pick goes to `/genesis:fix` with its failing test
(fix's reproduce-first gate runs it locally, sees it fail, then fixes it), or to
`/genesis:spec` when the fix is a design change (for example, there's no
authorization model at all). Recommend fixing critical and high first, in order.
Fix nothing yourself.

## Honest limits (state these in the report)
This is white-box adversarial *analysis*, not a live penetration test — no dynamic
fuzzing, no real exploitation, no runtime discovery. It finds design- and code-level
attack surface well; it won't find what only appears when exploits actually run. For
a system handling real PII (e.g. client data under POPIA/GDPR), this is a strong
first pass, not a substitute for a professional pentest before production.
