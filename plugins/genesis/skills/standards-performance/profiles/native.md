# Performance: native profile
C and C++ outside Unreal.

- **No allocation in hot loops.** Hoist buffers out of loops, `reserve()`
  vectors when the size is known, reuse memory.
- **Don't copy what you can borrow or move:** pass large types by `const&` or
  `std::string_view` / `std::span`; `std::move` to transfer ownership; return
  by value and let the compiler elide.
- **Data layout:** prefer contiguous containers (`std::vector`) over node-based
  ones (`std::list`, `std::map`) on hot paths; iterate memory in order.
- **Virtual calls, exceptions and locks in tight loops** only when measurement
  says they're not the bottleneck.
- **Profile first** (perf, VTune, Visual Studio profiler, Tracy) and benchmark in
  a release build with optimizations on; debug-build numbers mean nothing.
