---
name: standards-logging
description: Genesis logging and error-handling standards: log once where handled, never swallow errors, go through the logger abstraction, levels, no secrets or PII in logs. Load when writing or reviewing error handling (try/catch, error paths) or logging.
user-invocable: false
---

# Logging and error-handling standards

Part of the Genesis Standards. These rules are binding whenever they apply — they
were moved out of CLAUDE.md so they load only when relevant, not because they are
optional. Project-specific additions live in the project's CLAUDE.md under
**Conventions**; where the two conflict, the project's CLAUDE.md wins.

- **Log failures — through the abstraction, never raw, never secrets.**
  - Log where a failure is **handled** (the catch that decides what happens), once
    per failure — not at every layer it passes through. One meaningful entry, not
    ten.
  - **Never swallow errors silently.** An empty catch, or one that hides a failure
    with no log and no rethrow, is a defect. Every caught failure is either
    handled-and-logged or rethrown.
  - Log through the project's **logger abstraction** (see CLAUDE.md → Logging),
    never `print` / `console.log` / direct SDK calls. This is what lets the sink
    and level change per environment from one place.
  - Use **levels**: error (failed, needs attention) / warn (recovered or degraded)
    / info (notable events) / debug (dev only). Production runs at warn/error.
  - Include **actionable context** — the operation, relevant identifiers, the
    error type/message — but **never secrets, tokens, credentials, or PII** (this
    overrides the urge to "log everything to debug it"). Log an id, not the record.
  - Internal detail (stack traces) goes to the log sink, **never to a client
    response** (ties to the security standard).
