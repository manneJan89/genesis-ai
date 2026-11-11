# Performance: unreal profile
Unreal Engine projects. The budget is the frame: 16.6 ms at 60 fps, 33.3 ms at
30. Specs for gameplay features state the target frame rate and platform.

- **Tick only when needed.** Set `PrimaryActorTick.bCanEverTick = false` (and
  the component equivalent) unless the actor genuinely updates every frame;
  prefer timers, events and delegates.
- **Nothing expensive in `Tick`:** no `GetAllActorsOfClass`,
  `FindComponentByClass`, string building, or allocation per frame. Cache
  references in `BeginPlay`.
- **Heavy loops in C++, not Blueprint.** Blueprint is fine for wiring; loops
  over many items or per-frame maths belong in C++.
- **Containers:** `Reserve()` `TArray`s when the size is known; pass large types
  by `const&`; use `MoveTemp` to hand off ownership instead of copying.
- **Profile first** with Unreal Insights or `stat unit` / `stat game`, and report
  frame-time numbers before and after any optimization.
- **Asset loading:** soft references and async loading for anything not needed
  at startup; avoid hard references that pull large assets into memory.
