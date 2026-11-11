# Security: client profile
Code that runs on the user's device: Angular, React, Vue, Flutter, desktop UI.
Framework-specific rules are in `client-<framework>.md`.
The client is never the security boundary. Its checks are for the user's
experience; the server (or security rules) enforces the same rules again.

- **Everything shipped is public.** Angular `environment.ts`, Flutter
  `--dart-define` values, React env vars and anything in the bundle can be read
  by anyone. Only public identifiers belong there (a Firebase web config is
  fine; a service-account key or third-party secret is not).
- **Links and redirects from data:** allow only `https:` (and `mailto:` or
  `tel:` where intended); a `javascript:` URL in an `href` is script execution.
  Redirect targets after login are checked against a list of your own routes.
- **Tokens on the web:** prefer `httpOnly` cookies set by the server over
  `localStorage`, which any injected script can read.
- **Permission-based UI is cosmetic.** Hiding a button from a user who may not
  use it is good UX; the server must still refuse the action.

**Security tests for this profile:** a component given HTML or a `javascript:`
URL from data renders it inert; UI hidden for a denied user is absent (paired
with the server-side test that the action is refused).
