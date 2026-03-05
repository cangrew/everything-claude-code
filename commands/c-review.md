---
description: Comprehensive C code review for memory safety, CERT C compliance, POSIX patterns, and resource management. Invokes the c-reviewer agent.
---

# C Code Review

This command invokes the **c-reviewer** agent for comprehensive C-specific code review.

## What This Command Does

1. **Identify C Changes**: Find modified `.c` and `.h` files via `git diff`
2. **Run Static Analysis**: Execute `cppcheck` and `clang --analyze`
3. **Memory Safety Scan**: Check for buffer overflows, use-after-free, leaks
4. **Unsafe Function Audit**: Flag `gets`, `strcpy`, `sprintf`, unbounded `scanf`
5. **Integer Safety Check**: Detect overflow, truncation, sign/unsigned mixing
6. **Generate Report**: Categorize issues by severity

## When to Use

Use `/c-review` when:
- After writing or modifying C code
- Before committing C changes
- Reviewing code that handles untrusted input
- Auditing embedded or systems code for safety

## Review Categories

### CRITICAL (Must Fix)
- Buffer overflow via unsafe string functions
- Use-after-free or double-free
- `malloc` return value unchecked
- Resource leaks (memory, file descriptors) on error paths
- Hardcoded credentials

### HIGH (Should Fix)
- Signed integer overflow on untrusted input
- Implicit narrowing conversions (`size_t` to `int`)
- Missing cleanup in early-return error paths
- Unsafe functions (`gets`, `strcpy`, `sprintf`, `strncpy` without explicit NUL-termination)

### MEDIUM (Consider)
- Missing `const` on read-only pointer parameters
- Headers without include guards or `#pragma once`
- Magic numbers without named constants
- Non-portable size assumptions

## Automated Checks Run

```bash
# Static analysis
cppcheck --enable=warning,style,performance,portability --std=c11 src/

# Clang static analyzer
clang --analyze src/*.c

# Memory check (if test binary exists)
valgrind --leak-check=full --error-exitcode=1 ./binary
```

## Approval Criteria

| Status | Condition |
|--------|-----------|
| Approve | No CRITICAL or HIGH issues |
| Warning | Only MEDIUM issues (merge with caution) |
| Block | CRITICAL or HIGH issues found |

## Integration with Other Commands

- Use `/build-fix` if build errors occur first
- Use `/c-review` before committing
- Use `/code-review` for language-agnostic concerns

## Related

- Agent: `agents/c-reviewer.md`
