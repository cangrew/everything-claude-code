---
description: "C security extending common rules"
globs: ["**/*.c", "**/*.h", "**/Makefile", "**/Makefile.*"]
alwaysApply: false
---
# C Security

> This file extends the common security rule with C specific content.

## Unsafe Functions -- Never Use

| Avoid | Use Instead |
|-------|-------------|
| `gets` | `fgets` |
| `strcpy` | `snprintf(dst, sizeof(dst), "%s", src)` or `strlcpy` (if available) |
| `strcat` | `snprintf` to append, or `strlcat` (if available) |
| `sprintf` | `snprintf` |
| `scanf("%s")` | `scanf("%Ns", ...)` with explicit width |

**Note on `strncpy`/`strncat`**: These are **not** safe drop-in replacements. `strncpy` does not guarantee NUL-termination (CERT STR32-C). Prefer `snprintf` for portable, always-terminated string copying. If you must use bounded copies, always explicitly NUL-terminate the destination.

## Integer Safety

- Always check `malloc`/`calloc` return value for `NULL`
- Validate sizes before arithmetic to prevent integer overflow
- Use `size_t` for sizes and lengths, not `int`

## Static Analysis

```bash
cppcheck --enable=warning,style,performance,portability --std=c11 src/
clang --analyze src/*.c
```

Follow the **CERT C Coding Standard** for security-critical code.
