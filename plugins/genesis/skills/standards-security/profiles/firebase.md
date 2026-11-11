# Security: firebase profile
Projects where clients talk to Firestore, Realtime Database or Storage
directly. Here the **security rules are the server**: they are the only thing
standing between any user and all of your data.

- **Default deny.** Every collection and path has explicit rules; anything not
  matched is refused. Never ship `allow read, write: if true` or
  `if request.auth != null` on its own (that lets every signed-in user read and
  change everyone's data).
- **Ownership in the rules.** Reads and writes check
  `request.auth.uid` against the document's owner field or path, and roles come
  from custom claims or a server-written document, never from a field the
  client can write.
- **Validate writes in the rules.** Check the shape of
  `request.resource.data`: allowed keys (`keys().hasOnly([...])`), types,
  lengths, and that protected fields (`ownerId`, `role`, `createdAt`, balances)
  can't be set or changed by the client.
- **The Admin SDK bypasses all rules.** Use it only in trusted server code
  (Cloud Functions, your backend), and re-check authorization there yourself,
  since no rule will.
- **Cloud Functions callable and HTTP endpoints** follow the server profile:
  check the caller (`request.auth` in v2 functions, `context.auth` in v1) and
  validate input.
- **App Check** on for production, as an extra layer, not a replacement for
  rules.

**Security tests for this profile:** rules tests against the emulator
(`@firebase/rules-unit-testing`) for every rule change: the owner can, another
signed-in user can't, a signed-out user can't, and a write setting a protected
field or an unexpected key is refused.
