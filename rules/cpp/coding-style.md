---
paths:
  - "**/*.cpp"
  - "**/*.cxx"
  - "**/*.cc"
  - "**/*.hpp"
  - "**/*.hxx"
  - "**/*.h"
  - "**/CMakeLists.txt"
  - "**/*.cmake"
---
# C++ Coding Style

> This file extends [common/coding-style.md](../common/coding-style.md) with C++ specific content.

## Standard

Target **C++17** minimum; prefer C++20/23 features where available.

## Formatting

- **clang-format** is mandatory — commit a `.clang-format` to the project; no style debates

## Naming

`snake_case` for variables, functions, and namespaces; `PascalCase` for types and classes; `SCREAMING_SNAKE_CASE` for macros only. This is a common convention inspired by the C++ Core Guidelines (NL.10 prefers `underscore_style` as a default; this project chooses a mixed scheme for readability).

## Key Principles

- **RAII everywhere**: Tie every resource (memory, file, lock, socket) to object lifetime
- **`const` by default**: Mark variables and member functions `const` unless mutation is required
- **Smart pointers**: Use `std::unique_ptr` and `std::shared_ptr`; never raw `new`/`delete`
- **`enum class`**: Always prefer `enum class` over unscoped `enum`
- **`[[nodiscard]]`**: Mark functions where ignoring the return value is a bug

## Reference

See skill: `cpp-coding-standards` for comprehensive C++ idioms and patterns.
