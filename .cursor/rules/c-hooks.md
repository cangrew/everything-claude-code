---
description: "C hooks extending common rules"
globs: ["**/*.c", "**/*.h", "**/Makefile", "**/Makefile.*"]
alwaysApply: false
---
# C Hooks

> This file extends the common hooks rule with C specific content.

## PostToolUse Hooks

Configure in `~/.claude/settings.json`:

- **clang-format**: Auto-format `.c`/`.h` files after edit
- **cppcheck**: Run static analysis after editing source files
- **make**: Incremental build check after edits
