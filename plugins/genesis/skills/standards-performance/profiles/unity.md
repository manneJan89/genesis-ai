# Performance: unity profile
Unity projects (C#). The budget is the frame (16.6 ms at 60 fps), and the
classic killer is garbage-collection spikes. Specs for gameplay features state
the target frame rate and platform.

- **Cache in `Awake` or `Start`, never look up per frame:** no `GetComponent`,
  `Find`, `FindObjectOfType` or `FindObjectsByType` in `Update`, `FixedUpdate`
  or `LateUpdate`.
- **No allocations in per-frame code:** no LINQ, string concatenation or
  interpolation, `new List<>`, lambdas that capture variables, or boxing inside
  `Update`. Each allocation feeds the garbage collector, which stalls a frame
  later.
- **Pool frequently spawned objects** (bullets, effects, enemies) instead of
  `Instantiate`/`Destroy` each time.
- **Use the non-allocating APIs:** `CompareTag` instead of `tag ==`,
  `Physics.RaycastNonAlloc` and similar.
- **Don't run `Update` on things that don't need it**; use events, coroutines
  or a single manager that updates many objects.
- **Profile first** with the Unity Profiler (and the GC Alloc column), and
  report frame time and allocations before and after any optimization.
