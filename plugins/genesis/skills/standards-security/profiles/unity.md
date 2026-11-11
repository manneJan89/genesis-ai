# Security: unity profile
Unity projects (C#). C# is memory-safe, so the concerns are multiplayer trust,
what ships in the build, and object lifetime.

**Multiplayer (if the game has networking)**
- **The server is authoritative.** Damage, currency, inventory, scores and
  cooldowns are decided on the server (Netcode for GameObjects, Mirror, Photon
  or similar). The client requests; the server checks and applies.
- **Validate every server RPC** (`[ServerRpc]`, `[Command]` or equivalent):
  check the sender owns what it's acting on and that values are in range.
- **Network variables** are writable only by the server unless there's a stated
  reason.

**What ships in the build**
- C# builds decompile easily (Mono and IL2CPP alike). No API keys with write
  access, no admin endpoints, no hidden cheats left behind a flag.
- **In-app purchases:** validate receipts on a server, never only on the
  device.
- **Save files and `PlayerPrefs`** are trivially editable: never store secrets
  there, and treat loaded values as untrusted input. Anything worth cheating
  over lives server-side.
- **Remote content** (asset bundles, Addressables, JSON from a URL) is checked
  for integrity before loading.

**Object lifetime**
- Unity overrides `==` so a destroyed object compares equal to `null`, but `?.`
  and `??` skip that override. Don't use `?.` or `??` on anything derived from
  `UnityEngine.Object`; use an explicit `== null` or `if (obj)` check.
- Unsubscribe from events in `OnDisable` or `OnDestroy`, or the handler keeps a
  destroyed object alive and fires on it.

**Security tests for this profile:** each server RPC rejects actions on objects
the sender doesn't own and out-of-range values (Unity Test Framework).
