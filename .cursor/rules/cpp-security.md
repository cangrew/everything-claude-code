---
description: "C++ security extending common rules"
globs: ["**/*.cpp", "**/*.cxx", "**/*.cc", "**/*.hpp", "**/*.hxx", "**/*.h", "**/CMakeLists.txt", "**/*.cmake"]
alwaysApply: false
---
# C++ Security

> This file extends the common security rule with C++ specific content.

## Memory Safety

- Never use `gets`, `strcpy`, `sprintf` -- use `fgets`, `std::string`, `snprintf`
- Prefer `std::string`, `std::vector`, `std::span` over raw arrays
- Use RAII; never manually pair `new`/`delete`
- Run AddressSanitizer (`-fsanitize=address`) to detect memory errors in CI

## Integer Safety

- Validate before narrowing conversions; use `static_cast` explicitly
- Check for overflow before arithmetic on untrusted input
- Enable `-ftrapv` in debug builds to catch signed integer overflow

## Static Analysis

```bash
clang-tidy --checks='bugprone-*,modernize-*,readability-*,performance-*,cppcoreguidelines-*,misc-*' src/
cppcheck --enable=warning,style,performance,portability --std=c++17 src/
```

Run **CodeQL** or **Coverity** in CI for security-critical code.
