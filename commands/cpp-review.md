---
description: Comprehensive C++ code review for RAII, memory safety, undefined behavior, type safety, and performance. Invokes the cpp-reviewer agent.
---

# C++ Code Review

This command invokes the **cpp-reviewer** agent for comprehensive C++-specific code review.

## What This Command Does

1. **Identify C++ Changes**: Find modified `.cpp`, `.hpp`, `.h` files via `git diff`
2. **Run Static Analysis**: Execute `clang-tidy` and `cppcheck`
3. **Memory Safety Scan**: Check for RAII violations, raw `new`/`delete`, dangling references
4. **Undefined Behavior Check**: Detect signed overflow, null dereference, uninitialized reads, data races
5. **Type Safety Review**: Verify casts, `[[nodiscard]]` usage, `enum class` usage
6. **Generate Report**: Categorize issues by severity

## When to Use

Use `/cpp-review` when:
- After writing or modifying C++ code
- Before committing C++ changes
- Reviewing pull requests with C++ code
- Auditing C++ code for memory safety

## Review Categories

### CRITICAL (Must Fix)
- Use-after-free, buffer overflow, dangling references
- Raw `new`/`delete` outside RAII wrappers
- Data races on shared mutable state
- Hardcoded credentials
- Signed integer overflow or undefined behavior

### HIGH (Should Fix)
- Rule of Five violations
- Exception-unsafe resource management
- C-style casts without justification
- `[[nodiscard]]` return values silently discarded
- Missing `override` on virtual overrides

### MEDIUM (Consider)
- Unnecessary copies where move suffices
- Missing `const` on member functions and parameters
- `using namespace std` in headers
- Magic numbers without named constants

## Automated Checks Run

```bash
# Static analysis
clang-tidy --checks='bugprone-*,modernize-*,readability-*,performance-*,cppcoreguidelines-*,misc-*' src/

# Additional checks
cppcheck --enable=warning,style,performance,portability --std=c++17 src/

# Sanitizer build (if CMake project)
cmake -DCMAKE_CXX_FLAGS="-fsanitize=address,undefined" -B build_san
cmake --build build_san
```

## Approval Criteria

| Status | Condition |
|--------|-----------|
| Approve | No CRITICAL or HIGH issues |
| Warning | Only MEDIUM issues (merge with caution) |
| Block | CRITICAL or HIGH issues found |

## Integration with Other Commands

- Use `/cpp-build` if build errors occur first
- Use `/cpp-test` to run tests with sanitizers
- Use `/cpp-review` before committing

## Related

- Agent: `agents/cpp-reviewer.md`
- Skills: `skills/cpp-coding-standards/`, `skills/cpp-testing/`
