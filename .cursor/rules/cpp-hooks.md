---
description: "C++ hooks extending common rules"
globs: ["**/*.cpp", "**/*.cxx", "**/*.cc", "**/*.hpp", "**/*.hxx", "**/*.h", "**/CMakeLists.txt", "**/*.cmake"]
alwaysApply: false
---
# C++ Hooks

> This file extends the common hooks rule with C++ specific content.

## PostToolUse Hooks

Configure in `~/.claude/settings.json`:

- **clang-format**: Auto-format `.cpp`/`.h` files after edit
- **clang-tidy**: Run static analysis after editing source files
- **cmake --build**: Incremental compilation check after edits
