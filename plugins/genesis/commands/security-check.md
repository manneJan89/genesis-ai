---
description: Deliberate whole-surface security audit — attacker's-eye, checklist-driven, read-only
argument-hint: [scope: whole app (default), a feature, or an area]
allowed-tools: Read, Grep, Glob, Skill, Bash(git status*), Bash(git diff*), Bash(git log*), Bash(git show*), Bash(npm audit*), Bash(pnpm audit*), Bash(yarn audit*), Bash(flutter pub outdated*), Bash(dart pub outdated*), Bash(pip-audit*)
---

Run a security audit of: ${ARGUMENTS:-the whole application}

You are **read-only**. You find and report vulnerabilities and route them; you fix
nothing. Load the `genesis:standards-security` and `genesis:standards-performance` skills
first (with the profile files they load for this project) — they are the bar
you audit against —
and read CLAUDE.md for project-specific Conventions.

Security is meant to be built in when a feature is specced and built (with its
security tests and the build's security pass). This command is for the rest:
auditing existing code that predates that, and catching what a first draft
missed.

## What this is — and isn't
This is a **structured self-review against common vulnerability classes**, run with
an attacker's mindset over the whole attack surface. It is **not** a penetration
test, a runtime scan, or a guarantee. State that in your report. Where a finding
can only be confirmed by running the app (actual auth bypass, live injection),
mark it **needs runtime verification** rather than asserting it.

## 1. Map the attack surface (don't skip)
Enumerate every place untrusted input or an external caller can enter, before
judging any of it:
- HTTP endpoints / routes / API handlers (note each: public or authenticated?)
- Forms and client inputs
- File uploads, downloads, redirects
- Query params, headers, cookies, webhooks, message consumers
- Auth flows (login, register, reset, token refresh, session)
- Anything reading secrets/config or talking to a database or third party
- Security rules files (Firestore, RTDB, Storage)
- Games: Server RPCs, replicated state, save files, mod or downloaded content
- Native code: every parser of files, packets, or command-line input

List the surface first. A vulnerability in an endpoint you never looked at is the
one that gets exploited — coverage matters more than depth on any single item.

## 2. Audit each against the checklist
Load `genesis:standards-security`; it loads the profile files listed in
CLAUDE.md → Profiles. For every entry point, work the checklist below **and**
the rules of each profile that covers it (MongoDB operator injection and mass
assignment for `server`, `bypassSecurityTrust*` for `client`, rules ownership
for `firebase`, RPC validation and authority for `unreal` and `unity`, memory
safety for `native` and `unreal`). Skip classes that don't apply to the
project's profiles, and list what you skipped under coverage. For each entry
point, also check that its security tests exist ("Prove it with tests"); a
missing test is a finding. Cite file + line for each finding.

**Authentication**
- Private endpoints actually verify identity (not just assume a session).
- No auth logic that can be bypassed by a missing/malformed token defaulting to
  "allowed" (fail-closed, not fail-open).

**Authorization**
- Authenticated ≠ authorized: is there a **per-resource / ownership** check? (Can
  user A read or mutate user B's record by changing an id?) This (IDOR) is the
  most common real breach — check it on every resource access.
- Role checks where roles matter; least privilege honored.

**Input validation & injection**
- Server-side validation on ALL input (client validation doesn't count).
- Parameterized queries / safe ORM paths — no string-built SQL/NoSQL, and no
  request value reaching a MongoDB filter without its type checked (`{"$ne":
  ""}` matches everything).
- Output encoded/escaped (XSS); no unsanitized HTML injection.
- Command/path injection on anything reaching the shell or filesystem.
- File uploads: type/size checked, stored safely, not executable.

**Secrets & config**
- No hardcoded keys, tokens, connection strings, credentials.
- Secrets from env/secret store; env files git-ignored; not logged.

**Data exposure**
- No PII/secrets in logs or client-facing errors.
- Errors don't leak stack traces or internals to clients.
- Responses don't over-return fields (e.g. password hashes, other users' data).

**Transport & session**
- Sensitive traffic over HTTPS; secure/httpOnly/sameSite cookies where relevant.
- Sessions/tokens expire; logout/refresh handled; no long-lived static tokens.

**Availability / abuse**
- Rate limiting or abuse protection on auth and expensive endpoints.
- Unbounded queries (see `genesis:standards-performance`) that enable resource
  exhaustion.

**Dependencies**
- Known-vulnerable packages. If the stack has an audit tool (e.g. `npm audit`,
  `pip-audit`, `flutter pub outdated`), run it and summarize — don't auto-upgrade.

## 3. Report and route
Before writing, load the `genesis:finding-confidence` skill: run every candidate
through its disprove pass (look for the guard, middleware, rule, or framework
default that already covers it) and score it. Its thresholds decide what goes in
the main list, the "Unverified — worth a look" section, or gets dropped.

Write to `specs/security/<scope-or-date>.md`. Start with a one-line scope + the
"not a pentest" caveat. Then, ranked by severity:

- **Severity**: critical / high / medium / low
- **Class**: authn / authz-IDOR / injection / secrets / exposure / transport /
  abuse / dependency
- **Confidence**: 0–100 score (finding-confidence skill); for 70–89, what needs
  runtime verification
- **Location**: file + line, and which entry point
- **Impact**: what an attacker gains
- **Fix direction**: the correct remedy (not code yet)
- **Failing test**: the exact request or input and the expected rejection, so
  `/genesis:fix` can reproduce it first
- **Next step**: `/genesis:fix <specific vuln>` for a discrete fix, or
  `/genesis:audit-feature` → `/genesis:improve-feature` if it needs a design change
  (e.g. the whole authz model is missing).

End with **coverage**: what you audited, and what you could NOT verify statically
(so the gaps are visible). If you found nothing serious in a category, say so —
don't pad. Do not fix anything; wait for me to choose what to act on.
