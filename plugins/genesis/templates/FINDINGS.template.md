# Findings

Append-only backlog of important issues noticed **out of scope** during other
work — bugs, security holes, hardcoded secrets, performance traps, tech debt.
Commands log here instead of fixing inline (scope creep) or letting it vanish into
a summary. Work the backlog with `/genesis:findings`.

Rules: append new findings; never delete — mark resolved ones `done` with a date
so the history survives. A committed secret is logged here **and** raised
immediately (it needs rotating + purging from git history, not just a note).

<!-- Newest at top. Format:

## [high] Hardcoded API key in payment client
- Found: 2026-08-18, during /genesis:build-feature checkout-flow
- Where: lib/payments/stripe_client.dart:14
- What: Stripe secret key committed in source; exposed in git history.
- Suggested next step: /genesis:fix — move to env, then rotate the key and purge history
- Status: open

-->

_No findings yet._
