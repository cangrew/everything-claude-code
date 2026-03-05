---
description: "C++ testing extending common rules"
globs: ["**/*.cpp", "**/*.cxx", "**/*.cc", "**/*.hpp", "**/*.hxx", "**/*.h", "**/CMakeLists.txt", "**/*.cmake"]
alwaysApply: false
---
# C++ Testing

> This file extends the common testing rule with C++ specific content.

## Framework

Use **GoogleTest** (preferred) or **Catch2** for unit tests; integrate with **CTest** as the test runner.

## Sanitizers

Always run tests with sanitizers enabled:

```bash
cmake -DCMAKE_CXX_FLAGS="-fsanitize=address,undefined" -DCMAKE_BUILD_TYPE=Debug ..
cmake --build . && ctest
```

## Coverage

```bash
cmake -DCMAKE_CXX_FLAGS="--coverage" -DCMAKE_BUILD_TYPE=Debug ..
cmake --build . && ctest
gcovr --root .. --html --html-details -o coverage.html
```

## Reference

See skill: `cpp-testing` for detailed C++ testing patterns, fixtures, mocks (GoogleMock), and CMake integration.
