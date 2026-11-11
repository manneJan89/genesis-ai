# Data-operation gate
Applies to any operation that writes to data outside a local or emulator
database: migrations, schema changes, seed or backfill scripts, bulk updates or
deletes, data-fix scripts, imports, and rule or index deployments that can drop
data. The agent never runs these itself. Instead:
1. Build and test it locally first: unit tests against mocks, then a run against
   the emulator or a local database to show the actual effect (counts before and
   after).
2. Hand it to me as a short runbook: the exact command(s), what they change and
   how much (e.g. "deletes 143 documents in `quotes` where …"), how to verify
   afterwards, and how to undo it or restore if it goes wrong.
3. Ask me to confirm two things before I run it: a backup taken *after* the
   latest change to that data, stored where the credentials on this machine
   can't delete it; and that the restore procedure is known (CLAUDE.md → Data
   safety). No confirmed backup, no run.
4. I run it. Afterwards, record what ran and the result in the spec or summary.
A guard or `--dry-run` flag inside the script doesn't change any of this: until
the script's guards are proven by unit tests, they protect nothing.
