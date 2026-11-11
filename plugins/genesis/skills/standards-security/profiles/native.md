# Security: native profile
C and C++ outside Unreal (Unreal has its own profile). The language lets memory
bugs pass silently, so these rules are about making them impossible or loud.

- **Ownership is explicit.** RAII for every resource. `std::unique_ptr` by
  default, `std::shared_ptr` only for genuinely shared ownership, raw pointers
  and references only as non-owning views. No owning `new`/`delete` in
  ordinary code.
- **No unsafe C string or memory functions:** `strcpy`, `strcat`, `sprintf`,
  `gets`, `scanf("%s")`, unbounded `memcpy`. Use `std::string`,
  `std::string_view`, `std::format` or `snprintf`, `std::span`.
- **Bounds:** check sizes and indexes that come from input (files, network,
  command line) before using them; prefer `std::span` and range-for over raw
  pointer arithmetic.
- **Integer overflow:** sizes and counts read from input are checked before
  arithmetic that allocates or indexes (`count * size` can wrap).
- **No undefined behavior:** no uninitialized reads, no signed overflow, no
  use after move, no dangling references to temporaries.
- **Format strings** are never user-controlled.
- **Threads:** shared mutable state is guarded (mutex, atomic) or not shared.
- **Parsing untrusted input** (files, packets, save data) validates every
  length and offset before reading.
- **Compiler warnings are errors** (`-Wall -Wextra -Werror`, or `/W4 /WX` on
  MSVC), and new code adds no warning suppressions without a stated reason.
- **Sanitizer builds:** tests also run under AddressSanitizer and
  UndefinedBehaviorSanitizer (`-fsanitize=address,undefined`; MSVC
  `/fsanitize=address`), and ThreadSanitizer for threaded code, using the
  Commands block's sanitizer test command. A sanitizer report is a failing test.

**Security tests for this profile:** malformed and oversized input to every
parser (truncated, too-long lengths, negative or huge counts) is rejected
cleanly, with the sanitizer build reporting nothing.
