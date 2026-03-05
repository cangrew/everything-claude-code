---
description: "C coding style extending common rules"
globs: ["**/*.c", "**/*.h", "**/Makefile", "**/Makefile.*"]
alwaysApply: false
---
# C Coding Style

> This file extends the common coding style rule with C specific content.

## Standard

Target **C11** minimum; use C17/C23 features where the toolchain supports them.

## Formatting

- **clang-format** is mandatory -- commit a `.clang-format` file to the project

## Naming

- `snake_case` for all identifiers (variables, functions, types)
- `SCREAMING_SNAKE_CASE` for macros and constants
- Prefix public API symbols with a module name: `mylib_create()`, `mylib_destroy()`
- Avoid the `_t` suffix for application-defined types -- POSIX reserves `_t` endings broadly. Prefer a project prefix instead: `mylib_config` rather than `config_t`

## Key Principles

- Use `static_assert` (C23) or `_Static_assert` (C11/C17) for compile-time invariants
- Use designated initializers for struct initialization: `{ .x = 1, .y = 2 }`
- Protect headers with include guards or `#pragma once`
- Mark read-only pointer parameters `const`

## Note

This file targets pure C projects. For C++ projects, use `rules/cpp/` instead.
