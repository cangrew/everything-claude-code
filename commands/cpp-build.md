---
description: Fix C++ build errors, CMake configuration issues, linker errors, and clang-tidy warnings incrementally. Invokes the cpp-build-resolver agent for minimal, surgical fixes.
---

# C++ Build and Fix

This command invokes the **cpp-build-resolver** agent to incrementally fix C++ build errors with minimal changes.

## What This Command Does

1. **Run Diagnostics**: Execute `cmake --build`, `clang-tidy`, `cppcheck`
2. **Parse Errors**: Group by file and sort by severity
3. **Fix Incrementally**: One error at a time, smallest change possible
4. **Verify Each Fix**: Re-run build after each change
5. **Report Summary**: Show what was fixed and what remains

## When to Use

Use `/cpp-build` when:
- `cmake --build` fails with errors
- `clang-tidy` reports issues
- Linker errors (`undefined reference`) appear
- Template instantiation failures occur
- After pulling changes that break the build

## Diagnostic Commands Run

```bash
# Primary build check
cmake -B build && cmake --build build

# Static analysis
clang-tidy --checks='bugprone-*,modernize-*,readability-*,performance-*,cppcoreguidelines-*,misc-*' src/

# Additional checks
cppcheck --enable=warning,style,performance,portability --std=c++17 src/
```

## Common Errors Fixed

| Error | Typical Fix |
|-------|-------------|
| `undefined reference to X` | Add `target_link_libraries` in CMakeLists.txt |
| `use of undeclared identifier` | Add missing `#include` or fix typo |
| `no matching function for call` | Check argument types; add cast |
| `implicit instantiation of undefined template` | Move template definition to header |
| `multiple definition of X` | Add `inline` or move to `.cpp` |
| `redefinition of X` | Add `#pragma once` or include guard |
| `member access into incomplete type` | Replace forward declaration with full include |

## Fix Strategy

1. **Compiler errors first** -- code must compile
2. **Linker errors second** -- symbols must resolve
3. **Static analysis third** -- clang-tidy and cppcheck
4. **One fix at a time** -- verify each change
5. **Minimal changes** -- don't refactor, just fix

## Stop Conditions

The agent will stop and report if:
- Same error persists after 3 attempts
- Fix introduces more errors
- Requires architectural changes
- Missing external library dependencies

## Related Commands

- `/cpp-test` -- Run tests after build succeeds
- `/cpp-review` -- Review code quality after build is clean
- `/verify` -- Full verification loop

## Related

- Agent: `agents/cpp-build-resolver.md`
- Skills: `skills/cpp-coding-standards/`
