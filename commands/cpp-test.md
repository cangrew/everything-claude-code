---
description: Enforce TDD workflow for C++. Write GoogleTest/Catch2 tests first, then implement. Verify 80%+ coverage and run with sanitizers.
---

# C++ TDD Command

This command enforces test-driven development methodology for C++ using GoogleTest or Catch2.

## What This Command Does

1. **Define Interfaces**: Scaffold class/function signatures first
2. **Write Tests First**: Create comprehensive test cases (RED)
3. **Run Tests**: Verify tests fail for the right reason
4. **Implement Code**: Write minimal code to pass (GREEN)
5. **Refactor**: Improve while keeping tests green
6. **Check Coverage**: Ensure 80%+ coverage; run with sanitizers

## When to Use

Use `/cpp-test` when:
- Implementing new C++ functions or classes
- Adding test coverage to existing code
- Fixing bugs (write failing test first)
- Building memory-safe, tested business logic

## TDD Cycle

```
RED     -> Write failing GoogleTest/Catch2 test
GREEN   -> Implement minimal code to pass
REFACTOR -> Improve code, tests stay green
REPEAT  -> Next test case
```

## Test Patterns

### GoogleTest Fixture

```cpp
class UserServiceTest : public ::testing::Test {
protected:
    void SetUp() override {
        repo_ = std::make_unique<MockUserRepository>();
        service_ = std::make_unique<UserService>(*repo_);
    }
    std::unique_ptr<MockUserRepository> repo_;
    std::unique_ptr<UserService> service_;
};

TEST_F(UserServiceTest, CreateUser_ValidInput_ReturnsUser) {
    EXPECT_CALL(*repo_, save(::testing::_)).WillOnce(::testing::Return(true));
    auto result = service_->create("alice@example.com");
    ASSERT_TRUE(result.has_value());
    EXPECT_EQ(result->email, "alice@example.com");
}

TEST_F(UserServiceTest, CreateUser_EmptyEmail_ReturnsError) {
    auto result = service_->create("");
    EXPECT_FALSE(result.has_value());
}
```

### Parameterized Tests

```cpp
class ValidateEmailTest : public ::testing::TestWithParam<std::pair<std::string, bool>> {};

TEST_P(ValidateEmailTest, Validates) {
    auto [email, expected] = GetParam();
    EXPECT_EQ(validate_email(email), expected);
}

INSTANTIATE_TEST_SUITE_P(EmailCases, ValidateEmailTest, ::testing::Values(
    std::make_pair("user@example.com", true),
    std::make_pair("", false),
    std::make_pair("no-at-sign", false)
));
```

## CMake Test Setup

```cmake
enable_testing()
find_package(GTest REQUIRED)

add_executable(myapp_tests tests/user_service_test.cpp)
target_link_libraries(myapp_tests PRIVATE myapp_lib GTest::gtest_main gmock)
gtest_discover_tests(myapp_tests)
```

## Coverage Commands

```bash
# Build with coverage
cmake -DCMAKE_CXX_FLAGS="--coverage" -DCMAKE_BUILD_TYPE=Debug -B build_cov
cmake --build build_cov && ctest --test-dir build_cov

# Generate HTML report
gcovr --root . --html --html-details -o coverage.html
```

## Sanitizer Run

```bash
cmake -DCMAKE_CXX_FLAGS="-fsanitize=address,undefined" -B build_san
cmake --build build_san && ctest --test-dir build_san
```

## Coverage Targets

| Code Type | Target |
|-----------|--------|
| Critical business logic | 100% |
| Public APIs | 90%+ |
| General code | 80%+ |
| Generated code | Exclude |

## Related Commands

- `/cpp-build` -- Fix build errors
- `/cpp-review` -- Review code quality after implementation
- `/verify` -- Full verification loop

## Related

- Skills: `skills/cpp-testing/`, `skills/tdd-workflow/`
