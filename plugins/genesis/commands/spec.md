---
description: Interview me about a feature, then write an approved spec file
argument-hint: [feature-name or short description]
allowed-tools: Read, Write, Grep, Glob
---

You are running the **planning + spec** phase for: $ARGUMENTS

This phase is interactive and happens here in the main conversation — do NOT
delegate it to a subagent (subagents can't interview me).

Follow the template at @specs/_TEMPLATE.md.

## 0. Check for a roadmap (before interviewing)
Glob `specs/roadmaps/*.md` and look for a slice matching $ARGUMENTS.

- **Exactly one match** — say so plainly ("This is slice N of the <name> roadmap"),
  list the shared decisions it must respect (data shape, enums, cross-cutting
  flags, side-effect rules, failure semantics, reused components), and treat those
  as **fixed constraints** — do not re-open them in the interview. Record the
  back-link in the spec header:
  `Roadmap: specs/roadmaps/<name>.md (slice N — <slice-name>)`
  Mark that slice `in progress` in the roadmap.
- **No match** — proceed standalone. A roadmap is optional; most one-off features
  don't need one.
- **Ambiguous / fuzzy match** — ask which roadmap and slice I mean. Don't guess.

If the request is clearly a whole system rather than one feature (it would need
several specs, or the interview keeps expanding in scope), stop and recommend
`/genesis:roadmap <system>` instead of writing a mega-spec.

## Your job

1. **Interview me to extract the real requirement.** Ask focused questions
   **one at a time** and wait for my answer before the next. Don't dump a
   questionnaire. Prioritize the questions that most reduce ambiguity:
   - What problem does this solve, and who for?
   - What is explicitly out of scope?
   - What are the concrete acceptance criteria (including edge and error cases)?
   - Are there performance targets?
   - What existing code/constraints must this respect or avoid touching?

2. **Push back and improve the idea.** If something is vague, contradictory,
   over-scoped, or missing an obvious edge case, say so and propose a sharper
   version. I'd rather you challenge me now than build the wrong thing later.
   Suggest alternatives where they'd be better.

3. **Explore the codebase (read-only) as needed** to ground the spec in what
   actually exists — but keep exploration brief.

4. **When we've converged, write the spec** to `specs/<kebab-case-name>.md`
   using the template. Fill every section. Move anything unresolved into
   "Open questions" rather than guessing.

5. **Show me the finished spec and ask me to confirm.** Only set
   `Status: approved` after I explicitly approve. Then tell me the exact command
   to run next: `/genesis:build-feature specs/<name>.md`

Do not write any implementation code in this phase.
