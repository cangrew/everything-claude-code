---
paths:
  - "**/*.c"
  - "**/*.h"
  - "**/Makefile"
  - "**/Makefile.*"
---
# C Testing

> This file extends [common/testing.md](../common/testing.md) with C specific content.

## Framework

Use **Unity**, **CMocka**, or **Check** for unit tests; integrate with **CTest** for test runner.

## Memory Checking

Prefer **AddressSanitizer** as the primary memory error detector (faster than Valgrind, catches overlapping issues):

```bash
gcc -fsanitize=address,undefined -g -o test_binary test.c src.c
./test_binary
```

Use **Valgrind** as a complement for leak detection and uninstrumented binaries:

```bash
valgrind --leak-check=full --error-exitcode=1 ./test_binary
```

## Coverage

```bash
gcc -fprofile-arcs -ftest-coverage -o test_binary test.c src.c
./test_binary
gcov src.c
```

## Fuzzing

For security-sensitive parsing or deserialization code, add **AFL++** or **libFuzzer** targets.
