# Performance: client profile, Flutter
- `const` constructors wherever possible; keep `build` methods cheap (no
  parsing, sorting or allocation-heavy work in `build`).
- `ListView.builder` (or similar lazy builders) for long lists.
- Dispose controllers, streams and listeners in `dispose`.
