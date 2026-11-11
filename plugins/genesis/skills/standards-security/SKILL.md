---
name: standards-security
description: Genesis security standards: the trust boundary, auth and per-resource authorization, validating input at the boundary, injection, secrets, least privilege, fail closed, the data-operation gate, plus per-profile rules (server, client, firebase, unreal, unity, native). Load when writing or reviewing endpoints, RPCs, auth or permissions, user input, file parsing, security rules, secrets/config, database queries, or anything that writes to real data (migrations, seed/backfill/data-fix scripts, bulk updates or deletes, imports).
user-invocable: false
---

# Security standards

Part of the Genesis Standards. These rules are binding whenever they apply; they
load on demand to save context, not because they are optional. Project-specific
additions live in the project's CLAUDE.md under **Conventions**, and win over
these on anything else, but **a project convention can't weaken security**: one
that would is flagged to me, not followed.

## Load only what applies
This file applies to every project. The rest is in this skill's folder,
`${CLAUDE_PLUGIN_ROOT}/skills/standards-security/`; read only what the work
needs:
- **Profiles:** for each profile in CLAUDE.md → **Profiles** that covers the
  files at hand, read `profiles/<profile>.md` (`server`, `client`, `firebase`,
  `unreal`, `unity`, `native`). For `client`, also read
  `profiles/client-<framework>.md` for the framework that line names
  (`angular`, `react`, `flutter`). A change to the Angular app doesn't need the
  server file, and never the game or C++ ones.
- **Data operations:** if the work writes to data outside a local or emulator
  database (migration, seed, backfill, bulk update or delete, data-fix script,
  import), read `data-operations.md`: it's a gate you must follow.
- No Profiles section in CLAUDE.md? Work out the profile from the code, say
  which you assumed, and suggest `/genesis:sync`.

Security beats every other standard when they conflict (CLAUDE.md, "Security
and data safety win").

## The universal rules
- **Trust boundary.** Anything arriving from a party you don't control is
  hostile until checked: HTTP requests, game-client RPCs, files, save data,
  messages, deep links, IPC. The side that *receives* it enforces every rule.
  Checks on the sending side (form validation, client-side game logic, hidden
  buttons) are user experience only; an attacker skips the client entirely.
- **Authentication and authorization, every time.** Every private entry point
  verifies who the caller is AND that they may act on *this* resource
  (authenticated is not authorized: can user A read or change B's record by
  changing an id?). Deny by default: no explicit decision means no access.
- **Validate, don't clean.** At the boundary, check every value against a strict
  schema (type, shape, range, length, allowed keys) and **reject** what doesn't
  fit. Never try to repair hostile input. Type matters as much as content: an
  object, array or `null` where a string was expected is an attack, not a typo.
  Then keep data and code apart wherever the value goes: parameterized queries,
  argument arrays (not shell strings), path checks, and output encoding for the
  context (HTML, URL, shell). "Sanitize" (strip to a safe subset) is right in one
  case only: content you deliberately accept as rich text or HTML, cleaned with a
  maintained sanitizer, never your own regex.
- **Accept only the fields you expect.** Never pass a whole incoming object into
  a create or update. Pick the allowed fields explicitly, so a client can't set
  `role`, `ownerId`, `balance` or `isAdmin` by adding them.
- **No secrets in source, ever.** Keys, tokens, connection strings and
  credentials come from environment/config that is git-ignored. Anything shipped
  to a client (web bundle, mobile app, game build) is public, however it's
  obfuscated. Never log secrets or PII.
- **Least privilege.** Request the narrowest scopes that work; don't run or
  connect as an admin for ordinary operations.
- **Fail closed.** On an auth or validation error, deny. Never fall through to
  permitted. Errors sent to callers carry no stack traces or internals.

## Prove it with tests
Security rules are verified like acceptance criteria, not trusted. For every
entry point a change adds or alters, the tests include (the test-writer writes
them first, from the spec's Security section):
- no credentials → rejected;
- valid credentials, someone else's resource → rejected;
- wrong type or shape (object for a string, array, `null`, oversized, missing
  field) → rejected, and never treated as a match;
- unexpected extra fields (`role: "admin"`) → ignored or rejected, never stored;
- plus the profile-specific cases in the profile files.
These aren't skippable through the spec's Test plan. If the project can't run
them, say so plainly rather than dropping them.
