# Performance and cost: firebase profile
Firestore, Realtime Database, Storage and Cloud Functions. You pay per document
read, write and delete, so performance mistakes show up on the bill.

- **Every query has a `limit`**, and lists paginate with `startAfter`. Reading a
  whole collection to show ten items is a billing defect.
- **Listeners are reads.** A realtime listener on a large query re-bills on
  every change; listen to the narrowest query that serves the screen, and detach
  listeners when the screen goes away.
- **Count with aggregation queries** (`count()`, `sum()`, `average()`), not by
  reading every document.
- **Shape data for the screen that reads it.** Denormalize where one screen
  would otherwise read many documents; say what keeps the copies in sync.
- **Composite indexes** for every multi-field query (`firestore.indexes.json`
  checked in).
- **Cloud Functions triggers must not re-trigger themselves.** A function that
  writes to the document that triggered it loops, and each loop is billed. Guard
  with a changed-field check or write elsewhere.
- **Cold starts:** keep function dependencies lean; set minimum instances only
  where latency matters and the cost is accepted.
- **Batch writes** (`writeBatch`, up to the batch limit) instead of one call per
  document.
