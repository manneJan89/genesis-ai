---
name: standards-code
description: Genesis code-quality standards: DRY/KISS reuse rules, enums over magic strings, descriptive naming, and matching the project's existing structure. Load before writing, refactoring, or reviewing any code.
user-invocable: false
---

# Code quality standards

Part of the Genesis Standards. These rules are binding whenever they apply — they
were moved out of CLAUDE.md so they load only when relevant, not because they are
optional. Project-specific additions live in the project's CLAUDE.md under
**Conventions**; where the two conflict, the project's CLAUDE.md wins.

- **DRY — reuse before you write.** Before adding a function, widget, model, or
  service, search for an existing one that already does it (or nearly does it) and
  extend that instead. Never copy-paste a block and tweak it. If the same logic
  appears a third time, extract it.
  **Search order:** (1) this project's own code, (2) libraries already in this
  project's dependencies, (3) any shared library this project has *explicitly*
  opted into under "Component libraries" below. Never introduce a new dependency
  to satisfy DRY without asking.
  **Before building anything, search for what already exists.** If an **exact**
  match (behaviorally identical component or function — same job, different names
  still counts) is already in the codebase, reuse it, or extract it into a shared
  component so there's one copy; ask before extracting. If something is only
  **partially similar**, note it but leave it separate by default (see KISS) —
  merging merely-similar code into one flag-driven abstraction is worse than the
  duplication. Copy-paste of an existing block is a defect, not a shortcut.
- **KISS — but not at the cost of readability.** DRY loses to KISS when they
  conflict. Do NOT collapse similar-looking code into one function bristling with
  boolean flags, optional params, and branches — that's harder to maintain than the
  duplication it removed. Two clear functions beat one clever one. Prefer
  extracting a genuinely shared *concept*, not merely shared *characters*.
  Duplication is cheaper than the wrong abstraction: if two blocks look alike but
  change for different reasons, leave them alone and say why.
- **Enums over magic strings.** Any fixed set of values — status, role, type,
  gender, state — is an enum (or sealed/const type), never a bare string or int
  compared with `==`. Prefer `if (gender == Gender.male)` over
  `if (gender == 'male')`. Strings are allowed only at the boundary (JSON, DB,
  API); parse them into the enum on the way in and serialize on the way out, in
  one place. Exhaustive `switch` over an enum is preferred to if/else chains, so
  the compiler catches a missing case when a value is added later.

- **Descriptive, self-documenting names (all code, front and back).** A name must
  say what the thing is without decoding. Booleans read as questions
  (`isTrueFalseQuestion`, `canManageQuiz` — not `flag`, not `isTF`). No cryptic
  abbreviations, no single letters except loop indices. Clear beats both cryptic
  AND bloated — don't pad names to be "descriptive"; say exactly what it is.
  **If you can't tell what a name means from context, that is itself the finding:**
  say so plainly ("I can't tell what `isTrueFalse` refers to — true/false *what*?")
  and propose a clearer name, rather than silently accepting it. Not understanding a
  name is a signal to rename it, not to move on.

- **Follow the project's existing structure.** Before creating a new file or
  deciding where code goes, look at how the project already organizes this kind of
  thing and **match it.** If API calls for a service live in one file, add the new
  call there — don't spin up a parallel file. Put models where models already live,
  follow the existing foldering, layering, and naming patterns. Detect the
  convention from the actual codebase (find the existing file/folder and conform),
  don't impose a structure that seems reasonable in isolation — that's how a
  codebase grows two or three competing patterns for the same thing. Introduce a
  new structural pattern ONLY if the project has none for this, if I explicitly ask,
  or if `/genesis:review` flagged the current structure for change. If you think the
  existing structure is genuinely wrong, don't quietly work around it — log a
  finding (`FINDINGS.md`) or raise it, don't fork the pattern.
