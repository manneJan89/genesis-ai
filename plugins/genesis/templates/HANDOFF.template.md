# Handoff

<!-- Written by Genesis so the next session can carry on where this one stopped.
     Overwritten, never appended: it describes work in progress right now, not
     history. Deleted when the work is done; history lives in git, specs, the
     roadmap and FINDINGS.md. Keep it under a page. Never put secrets, credentials,
     connection strings or production data in it: this file is committed. -->

- **Branch**: <git branch this work is on>
- **HEAD when written**: <short commit hash>
- **Written**: <date and time>, by <command, e.g. /genesis:build-feature, or /genesis:handoff>
- **Task**: <one line: what is being built or fixed>
- **Source of truth**: <specs/<name>.md, specs/bugs/<name>.md, or design/<screen>.html>
- **Roadmap**: <specs/roadmaps/<name>.md (slice N), or none>
- **Command and phase**: <e.g. /genesis:build-feature, Phase 4 (build, green)>

## Done
- <what is finished and verified, with the evidence: "12 new tests green", "plan approved">

## In progress
- <what is half-done and in what state: "RegisterForm validation written, not wired to the service">

## Next step
<The exact next action, specific enough to do without rereading the conversation.
One step, not a plan: "Wire RegisterForm.submit to AuthService.register and run
the auth tests." If the next step needs my approval, say so: approvals from a
previous session do not carry over.>

## Open questions and blockers
- <what is waiting on me, or on an asset, or on information; "none" if none>

## Decisions not yet in the spec
- <decisions made this session that the spec does not record yet, and why. Each
  one should end up in the spec; until then it lives here. "none" if none>

## Files touched
- <path>: <what changed, one line>

## Uncommitted work
<"none", or what is changed but not committed. A new session on another machine
will not have it.>
