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
# C++ Hooks

> This file extends [common/hooks.md](../common/hooks.md) with C++ specific content.

## PostToolUse Hooks

Configure in `~/.claude/settings.json`:

- **clang-format**: Auto-format `.cpp`/`.h` files after edit
- **clang-tidy**: Run static analysis after editing source files
- **cmake --build**: Incremental compilation check after edits
