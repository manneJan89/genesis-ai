---
description: Set up the genesis workflow in this project (writes CLAUDE.md + spec template; infers or asks your stack)
argument-hint: (no arguments)
allowed-tools: Read, Write, Edit, Grep, Glob
---

Set up the genesis spec-driven workflow in the current project. Do this carefully
and confirm with me before writing files.

## 1. Detect: new or existing project
Look for stack signals in the repo root (and one level down): `package.json`,
`angular.json`, `pubspec.yaml`, `go.mod`, `Cargo.toml`, `pyproject.toml`,
`requirements.txt`, `pom.xml`, `build.gradle`, `Gemfile`, `composer.json`,
`firebase.json`, `firestore.rules`, C and C++ build files (`CMakeLists.txt`,
`Makefile`, `meson.build`, `vcpkg.json`, `conanfile.txt` / `conanfile.py`,
`*.sln` / `*.vcxproj`), Unreal (`*.uproject`, a `Source/` folder with
`*.Build.cs`), Unity (`ProjectSettings/ProjectVersion.txt` with an `Assets/`
folder; `*.asmdef` files), other .NET (`*.csproj`), etc., plus the presence of a
`src/`, `lib/`, or `test/` tree.

- **Existing project** (signals found): infer the stack, package manager, test
  runner, lint/format tool, and likely build/run commands from those files. Read
  the manifest(s) to get exact script names (e.g. the `scripts` block, or the
  test config). Don't guess where you can read.
  - Also scan dependencies for **metered third-party services** (firebase,
    firestore, supabase, algolia, hosted queues, paid APIs, etc.). If any are
    present, note them and add a project-specific cost note under Conventions
    (e.g. "Firestore is metered — batch reads, cache, avoid per-item round-trips"),
    reinforcing the universal cost standard.
- **New / empty project** (no signals): ask me which language/framework I'm using
  (one question), then propose sensible default commands for that stack.

## 2. Propose the per-project config
Show me a filled-in **Commands** block (install, run, build, test-all,
test-single, lint, format, benchmark, and for C, C++ and Unreal a sanitizer test
run) and a first-draft **Codebase map** based on what you found. State your
confidence and flag anything you're unsure about. Let me correct it before you
write anything.

**Profiles.** Classify the code into the profiles the security and performance
standards use, each with the folders it covers, and show me the list:
- `server`: APIs, backends, Cloud Functions, anything receiving requests from
  clients you don't control;
- `client`: Angular, React, Vue, Flutter, desktop UI; name the framework on
  the line (`angular`, `react`, `flutter`), since its rules load from a
  separate file;
- `firebase`: clients reading Firestore, RTDB or Storage directly (security
  rules are the server); usually alongside `client`, and `server` for Functions;
- `unreal`: Unreal Engine projects (its own C++ rules; don't add `native`);
- `unity`: Unity projects (C#);
- `native`: C or C++ outside Unreal.
A repo can have several (an Angular app with a Node API is `client: web/` and
`server: api/`). These decide which rules load: an Angular folder never loads
the game or C++ rules. Ask me about anything ambiguous, and whether a game is
multiplayer (the multiplayer rules only matter if it is).

**Game projects: what tests can and can't cover.** Logic kept in plain classes
(damage maths, inventory, state machines, save/load) is unit-testable and goes
through tests-first like anything else. "Does it feel right" isn't; those
criteria get tagged `(manual)` in specs. Say this once so it's expected.

Also set **Component libraries**. Detect whether the project already depends on a
shared component library; if none is found, set it to **none** (the default) — do
not assume a Genesis component kit is wanted. Only name a shared library here if
it's already a dependency or I explicitly ask for it. This keeps the project's
code independent of any component package.

## 3. Test infrastructure (blocking — do not skip)
The whole workflow shells out to this project's test command: the test-writer, the
acceptance gate, the fix loop, and the characterization safety net all depend on a
runner that actually works. Establish it now.

Detect what exists: a test framework in the dependencies, a test directory, runner
config, and whether any tests are actually present. Usual frameworks to look
for or propose by stack: Jest, Vitest or Karma/Jasmine (JavaScript and
TypeScript), `flutter test` / `dart test`, GoogleTest, Catch2 or doctest with
CTest (C, C++), Unreal's Automation tests (run headless, e.g.
`UnrealEditor-Cmd <project>.uproject -ExecCmds="Automation RunTests <filter>;Quit"
-unattended -nullrhi`), the Unity Test Framework in EditMode and PlayMode (run
headless with `Unity -batchmode -runTests -testPlatform EditMode -projectPath .
-testResults <file>`), and the Firebase emulator with
`@firebase/rules-unit-testing` for security rules. For C and C++, also propose a
sanitizer build of the tests (`-fsanitize=address,undefined`, or MSVC
`/fsanitize=address`) and record it as the sanitizer test command. Then handle
the case:

**(a) No test infrastructure at all**
Tell me plainly that this is a prerequisite, not a nice-to-have, and that without
it the safety-net phases can't function. Propose the standard framework for this
stack and exactly what you'd add. **Ask before installing anything** — per the
project Standards, never add a dependency unilaterally. On approval:
- add the test dependencies (dev/test scope),
- create the test directory following the stack's convention,
- write ONE trivial smoke test (assert something always true),
- **run it and show me it passes.**
The smoke test exists to prove the runner is wired up. A test command that has
never been executed successfully is not set up. If it fails, fix the wiring before
continuing — don't record a command you haven't seen work.

**(b) Framework present but no tests written**
Verify the runner actually executes (write and run the smoke test as above). Note
that existing code has no coverage.

**(c) Tests exist and pass**
Run them once to confirm and record the green baseline.

**(d) Tests exist but some FAIL**
Do not treat this as fine and do not try to fix them here. Report which fail, and
flag that a red baseline blocks the workflow: the e2e-tester can't tell new
breakage from pre-existing breakage, so the fix loop has no reliable signal. Ask
whether to (i) fix them first via `/genesis:fix`, or (ii) record the known-failing
tests in CLAUDE.md as an accepted baseline so later phases can distinguish them.

Also note lint/format tooling if absent, but treat that as optional — only the
test runner is blocking.

**Secrets hygiene (quick check).** Verify there's a `.gitignore` that excludes
env/secret files (`.env`, `*.key`, service-account JSON, `google-services.json`
where appropriate). If secrets appear to be committed or unignored, flag it —
this is a security issue per the project Standards. Add the ignore entries (with
my okay); do not delete committed secrets automatically, just report them so I can
rotate and purge them.

**Logging.** Establish where failures get logged, and fill the Logging section of
CLAUDE.md. **Do not provision or install any monitoring service** (Sentry,
Crashlytics) — that's a paid/metered dependency and a deliberate feature, not a
setup side-effect.
- Detect an existing log sink or logger in the dependencies. If one exists, record
  it as the destination.
- If none, set the **default**: a thin **logger abstraction** (one small file/
  module) that wraps the language's built-in console/stdout logging. Note its
  intended path in CLAUDE.md. Code will log through it, never directly — so a real
  sink can be swapped in later with a one-file change. Offer to create the
  abstraction stub now (with my okay).
- The abstraction **redacts before it writes**: anything credential-shaped
  (connection strings with a password in them, keys, tokens, auth headers) is
  masked at this one seam, so an accidental `log(config)` can't leak a secret. If
  the project already has a logger, check whether it redacts; if not, log a
  finding.
- Ask how the project distinguishes **environments** (dev vs prod) and fill the
  level/destination matrix. If unknown, default to dev=debug/console,
  prod=warn+error/console until a sink is added.

**Data safety (blocking — do not skip).** Fill the Data safety section of
CLAUDE.md by asking me and by reading config *structure*. Hard rules for this
step: **never connect to a database to test it, and never print, echo, or read
out a secret value.** Look at which env/config files exist and which variable
names they define, not their values. Never open an env or secrets file with Read
or `cat`: that puts every value into your context, which is itself a leak. Get
variable names from `.env.example` (or a similar names-only file) if one exists;
otherwise ask me to list them. Don't try other ways of reading the real env
file: once the permission guardrails below are installed, Claude Code blocks it
anyway, and routing around that is a violation in itself.
- Which databases does the project use, and which one do dev sessions point at
  (emulator, local DB, staging, production)?
- **Can this machine reach production?** Ask me directly, and look for signals
  without reading values: production project IDs or hostnames in non-secret
  config, a `.env.production` or similar on disk, prod variable names in the dev
  env file. If production is reachable, or you can't rule it out, **warn me
  loudly**, mark it `YES` in CLAUDE.md, and log a high finding: the fix is to move
  production credentials off this machine and point dev at an emulator, a local
  DB, or staging. Prompt rules can't make reachable production data safe; only
  not having the credentials can.
- Backups: what backs up each database, how often, and are the backups stored
  where the credentials on this machine can't delete them? When was a restore
  last tested? Missing or untested backups → high finding.
- Repo: is `main` protected against force-push and deletion? If not, log a
  medium finding with the fix (a branch protection rule; that's a GitHub setting
  you apply, not the agent).
- Genesis does not set up backups, secret managers, or branch protection itself —
  those are your infrastructure. It records what exists and flags what's missing.

**Permission guardrails (offer; I decide).** Claude Code's own permission rules
are enforced by Claude Code, not the model, in every mode, so no reasoning or
retrying gets past them. Offer to merge Genesis's rule set from
`${CLAUDE_PLUGIN_ROOT}/templates/permissions.template.json` into the project's
`.claude/settings.json`:
- **Deny:** destructive git (force-push, deleting remote branches, `reset --hard`,
  `clean -f`, `branch -D`, history rewrites), `gh repo delete`, Firebase
  delete/remove commands, and Claude's file tools reading env and secret files
  (`.env`, `.env.local`, `.env.production`, key and service-account files).
  `.env.example` stays readable.
- **Ask:** any command carrying `--delete`, `--drop`, `--purge`, `--wipe` or
  `--truncate` (this is what catches a project script run with a destructive
  flag); recursive deletes (`rm -r`, `rm -rf`, `rm --recursive`, in any flag
  order), `find -delete` and `xargs rm`; PowerShell's delete commands and their
  aliases; git commands that discard work; database command-line tools; migrations;
  `firebase deploy`; shell commands touching env files; and `bash -c` / `sh -c` /
  `zsh -c` (a fresh shell is the usual way around a rule, so it always prompts).
  Ask rules prompt even in auto mode. Note the env-file read deny also blocks
  `cat`, `grep`, `sed` and `head` on those files, not only Claude's Read tool.
- **Project-specific:** look at the project's own scripts (`package.json`
  scripts, Makefile targets, `melos`/`pubspec` scripts) for names like
  `reset`, `drop`, `wipe`, `purge`, `seed`, `migrate`, `delete`, `restore`, and
  propose an **ask** rule for each (e.g. `Bash(npm run db:reset*)`). Generic rules
  can't know your scripts; this is where the real coverage comes from.

How to apply it:
- **Merge, never replace.** Keep every rule already in the file. Add `deny` and
  `ask` rules freely; the only `allow` rules you may propose are the next point.
- **Allow the test and lint commands, exactly (offer).** Genesis commands don't
  pre-approve shell commands, so in default mode every test run asks. Offer
  `allow` rules for the exact test and lint commands from the Commands block
  (e.g. `Bash(npm test*)`, `Bash(npx ng test*)`, `Bash(flutter test*)`), so the
  build loop runs without a prompt per test. Only those commands: never a broad
  pattern like `Bash(npm *)`, `Bash(npm run *)` or `Bash(node *)`, because that
  approves every script the project has, destructive ones included. `deny` and
  `ask` rules still win over these.
- Show me the rules grouped as above, in plain words, and get my approval. I can
  drop any rule (for example, if `firebase deploy` prompting is too much
  friction). Ask whether to write to `.claude/settings.json` (committed, shared
  with anyone on the repo) or `.claude/settings.local.json` (just me).
- **Say plainly what this doesn't cover:** commands run another way (`bash -c
  '…'`, a full program path, `git -c … push`), a script with a destructive action
  hard-coded inside and no flag, and anything outside Claude Code. These rules are
  a seatbelt, not isolation: production credentials off this machine is still the
  real protection.
- Record the result in CLAUDE.md → Data safety ("Permission guardrails:
  installed, Genesis vX" or "declined"), and tell me to check them with
  `/permissions` and start a new session so they load.

Finally: if this is an **existing codebase with little or no test coverage**, tell
me that `/genesis:improve-feature`'s characterization tests become especially
important here — they're the only thing that will make changing this code safe,
and they'll take longer to establish on the first few features.

## 4. Write the files
Use the bundled templates (do not invent structure):
- `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.template.md`
- `${CLAUDE_PLUGIN_ROOT}/templates/spec-template.md`

Then:
- Write `specs/_TEMPLATE.md` from the spec template (create `specs/` if missing).
- Create `FINDINGS.md` at the repo root from
  `${CLAUDE_PLUGIN_ROOT}/templates/FINDINGS.template.md` if it doesn't exist — the
  backlog where out-of-scope issues get logged (worked via `/genesis:findings`).
- Create `COMPONENTS.md` at the repo root from
  `${CLAUDE_PLUGIN_ROOT}/templates/COMPONENTS.template.md` if it doesn't exist — the
  shared-UI-component registry. For an existing project with UI, offer to scan for
  components to seed it (as `/genesis:components scan` does); for a new project,
  leave it empty (normal).
- **CLAUDE.md handling:**
  - If the project has **no** `CLAUDE.md`, write one from the template with the
    per-project block filled in from step 2.
  - If a `CLAUDE.md` **already exists**, do NOT overwrite it. Show me the genesis
    rules + filled per-project block and ask whether to append them to the
    existing file or write them to a separate `CLAUDE.genesis.md`. Wait for my
    choice. If I choose the separate file, also add the line
    `@CLAUDE.genesis.md` to the top of `CLAUDE.md`: Claude Code only loads the
    separate file through that import line, so without it the Genesis rules are
    silently ignored. Then confirm it loads (`/memory` lists it).

## 5. Confirm and hand off
Show me the final CLAUDE.md, then tell me the available commands:
- `/genesis:spec <feature>` → `/genesis:build-feature specs/<name>.md`  (new feature)
- `/genesis:audit-feature <thing>` → `/genesis:improve-feature specs/<name>.md`  (change existing)
- `/genesis:audit-feature <thing>` (type=refactor) → `/genesis:optimize-feature specs/<name>.md`  (make it faster)
- `/genesis:handoff`  (stopping mid-task; the build commands also write `HANDOFF.md` on their own so the next session can carry on)

Remind me that I can re-run `/genesis:setup` anytime the stack changes, and that
updating the plugin (`/plugin marketplace update genesis`) refreshes the agents and
commands but not an already-written CLAUDE.md.
