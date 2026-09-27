---
name: standards-security
description: Genesis security standards: server-side trust boundary, auth and per-resource authorization, input validation, injection, secrets, least privilege, fail closed. Load when writing or reviewing endpoints, auth or permissions, user input, file uploads, redirects, secrets/config, or database queries.
user-invocable: false
---

# Security standards

Part of the Genesis Standards. These rules are binding whenever they apply — they
were moved out of CLAUDE.md so they load only when relevant, not because they are
optional. Project-specific additions live in the project's CLAUDE.md under
**Conventions**; where the two conflict, the project's CLAUDE.md wins.

- **Security is not optional and not a feature — it's a property of everything.**
  - **The server is the only trust boundary.** Client-side validation is for UX;
    it can always be bypassed. Every rule enforced on the client MUST be enforced
    again on the server. Never trust input that arrived over the wire.
  - **Auth on every private endpoint.** Private/authenticated endpoints verify the
    caller's identity AND that they're authorized for *this* resource (an
    authenticated user is not automatically permitted). Deny by default: an
    endpoint with no explicit auth decision is treated as needing auth, not as
    public. Return 401 vs 403 correctly and don't leak which.
  - **Validate and sanitize all input, server-side.** Reject malformed input at
    the boundary; validate type, range, length, and shape. Use parameterized
    queries / the ORM's safe paths — never string-built queries. Encode/escape
    output to prevent injection (SQL, XSS, command). Treat file uploads and
    redirects as hostile until checked.
  - **No secrets in source, ever.** API keys, tokens, connection strings, and
    credentials come from environment/config (`.env`, platform secrets), which is
    git-ignored. Never log secrets or PII. If a secret would appear in code,
    output, or a commit, stop and flag it.
  - **Least privilege.** Request the narrowest scopes/permissions that work;
    don't run or connect as an admin/superuser for ordinary operations.
  - **Fail closed.** On an auth or validation error, deny access — never fall
    through to permitted. Don't expose stack traces or internal detail in errors
    sent to clients.
