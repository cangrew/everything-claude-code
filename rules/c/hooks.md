---
paths:
  - "**/*.c"
  - "**/*.h"
  - "**/Makefile"
  - "**/Makefile.*"
---
# C Hooks

> This file extends [common/hooks.md](../common/hooks.md) with C specific content.

## PostToolUse Hooks

Configure in `~/.claude/settings.json`:

- **clang-format**: Auto-format `.c`/`.h` files after edit
- **cppcheck**: Run static analysis after editing source files
- **make**: Incremental build check after edits
