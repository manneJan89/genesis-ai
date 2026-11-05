---
description: Write, resume from, or clear HANDOFF.md, the note that lets the next session carry on where this one stopped
argument-hint: (none = write it now) | "resume" | "clear"
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

Handoff action: ${ARGUMENTS:-write}

`HANDOFF.md` at the repo root is the note one session leaves for the next: what
it was busy with, how far it got, and the exact next step. The build commands
write it at phase boundaries on their own; this command is for the moments they
don't cover (stopping mid-phase, ending a session in ordinary chat) and for
resuming or clearing it on purpose.

Format: `${CLAUDE_PLUGIN_ROOT}/templates/HANDOFF.template.md`. Follow it; don't
invent sections.

## No arguments (or "write"): write it now
1. Get the facts from git, not from memory: `git branch --show-current`,
   `git rev-parse --short HEAD`, `git status --short`. Read only; change nothing.
2. Write `HANDOFF.md` from the template, overwriting any existing one. Describe
   the state as it is right now, from this conversation and the repo:
   - **Next step** is one concrete action, not a plan. If you can't name it
     precisely, say what is unclear instead of guessing.
   - Point to the spec, roadmap and `FINDINGS.md`; don't copy them.
   - Decisions made this session that the spec doesn't record go under
     **Decisions not yet in the spec**. Better still: if the spec is the right
     home and I approved the decision, update the spec too and say so.
   - List uncommitted work honestly. It won't be on another machine.
   - Keep it under a page. It's a handoff, not a diary.
3. Never put secrets, credentials, connection strings, tokens, or production data
   in it. It gets committed.
4. Don't commit it. Tell me it's written and that it travels with my next commit,
   so if I want to pick up on another machine I need to commit and push first.
5. Show me the **Next step** line so I can check it reads right.

If there is nothing in progress (no open task in this conversation, clean working
tree, no handoff to update), say so and write nothing.

## "resume": pick up from it
Do the read-and-verify check from CLAUDE.md ("Session handoff") explicitly, and
report:
- what the handoff says (task, phase, next step),
- what git says (branch, commits since **HEAD when written**, uncommitted changes),
- where they disagree, and which you believe and why.

Then propose the next step and wait for my go-ahead. Resuming is not an approval:
anything that needed my approval before (a plan, a design, a data operation, a
push) still needs it now.

If `HANDOFF.md` doesn't exist, say so, and fall back to the roadmap's **Next up**
(`/genesis:roadmap` with no arguments shows it) and `git log`.

## "clear": delete it
Delete `HANDOFF.md`. Do this when the work it describes is finished or abandoned.
Before deleting, check its **Decisions not yet in the spec** and **Open questions**:
if anything there isn't recorded somewhere permanent (the spec, the roadmap,
`FINDINGS.md`), tell me first and offer to move it, because deleting the handoff
deletes that information.
