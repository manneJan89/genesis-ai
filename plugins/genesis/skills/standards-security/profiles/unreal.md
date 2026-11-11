# Security: unreal profile
Unreal Engine projects (C++ and Blueprint). Two concerns: multiplayer trust, and
memory safety in C++.

**Multiplayer (if the game has networking)**
- **The server is authoritative.** Damage, currency, inventory, scores,
  cooldowns and movement corrections are decided on the server. The client
  requests; the server checks and applies.
- **Validate every Server RPC.** Use `WithValidation` and a `_Validate`
  function that rejects out-of-range or impossible values (negative amounts,
  distances beyond what's reachable, items the player doesn't own). Returning
  false disconnects the cheating client; that's intended.
- **Authority checks** (`HasAuthority()`) guard any code that changes
  replicated state; replicated properties are written only by the server.
- **Save games, config, mod and downloaded content** are untrusted input:
  validate before use.

**Memory safety (C++)**
- **UObject pointers:** a raw pointer to a `UObject` that isn't a `UPROPERTY()`
  is invisible to the garbage collector, which can free the object and leave the
  pointer dangling. Hold UObjects in `UPROPERTY()` members (`TObjectPtr` in
  UE5) or `TWeakObjectPtr`, and check `IsValid()` before use.
- **Creation:** `NewObject`, `SpawnActor` or `CreateDefaultSubobject` for
  UObjects, never `new`/`delete`. Non-UObject data uses `TUniquePtr` or
  `TSharedPtr`.
- **Bounds:** check `IsValidIndex()` before indexing a `TArray` with any value
  that isn't a loop index; check sizes read from files or the network before
  using them.
- **Invariants:** `check()` for conditions that must never fail, `ensure()` for
  ones you want reported but survivable. Never silently continue past a broken
  invariant.
- **Sanitizer builds:** where the platform supports it, run the tests with
  AddressSanitizer enabled (the Commands block's sanitizer test command) to catch
  memory bugs the tests themselves don't notice.

**Security tests for this profile:** each Server RPC's `_Validate` rejects
out-of-range values; a client-side attempt to change replicated state has no
effect on the server.
