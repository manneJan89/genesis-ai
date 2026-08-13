---
description: Deliberate whole-surface security audit — attacker's-eye, checklist-driven, read-only
argument-hint: [scope: whole app (default), a feature, or an area]
allowed-tools: Read, Grep, Glob, Bash
---

Run a security audit of: ${ARGUMENTS:-the whole application}

You are **read-only**. You find and report vulnerabilities and route them; you fix
nothing. Read CLAUDE.md first — its **Security** and **Scale** standards are the
bar you audit against.

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

List the surface first. A vulnerability in an endpoint you never looked at is the
one that gets exploited — coverage matters more than depth on any single item.

## 2. Audit each against the checklist
For every entry point, work the checklist. Cite file + line for each finding.

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
- Parameterized queries / safe ORM paths — no string-built SQL/NoSQL.
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
- Unbounded queries (ties to the Scale standard) that enable resource exhaustion.

**Dependencies**
- Known-vulnerable packages. If the stack has an audit tool (e.g. `npm audit`,
  `pip-audit`, `flutter pub outdated`), run it and summarize — don't auto-upgrade.

## 3. Report and route
Write to `specs/security/<scope-or-date>.md`. Start with a one-line scope + the
"not a pentest" caveat. Then, ranked by severity:

- **Severity**: critical / high / medium / low
- **Class**: authn / authz-IDOR / injection / secrets / exposure / transport /
  abuse / dependency
- **Confidence**: confirmed in code / needs runtime verification
- **Location**: file + line, and which entry point
- **Impact**: what an attacker gains
- **Fix direction**: the correct remedy (not code yet)
- **Next step**: `/genesis:fix <specific vuln>` for a discrete fix, or
  `/genesis:audit-feature` → `/genesis:improve-feature` if it needs a design change
  (e.g. the whole authz model is missing).

End with **coverage**: what you audited, and what you could NOT verify statically
(so the gaps are visible). If you found nothing serious in a category, say so —
don't pad. Do not fix anything; wait for me to choose what to act on.
