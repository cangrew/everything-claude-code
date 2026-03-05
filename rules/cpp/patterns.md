---
paths:
  - "**/*.cpp"
  - "**/*.cxx"
  - "**/*.cc"
  - "**/*.hpp"
  - "**/*.hxx"
  - "**/*.h"
  - "**/CMakeLists.txt"
  - "**/*.cmake"
---
# C++ Patterns

> This file extends [common/patterns.md](../common/patterns.md) with C++ specific content.

## RAII Resource Management

```cpp
class FileHandle {
public:
    explicit FileHandle(const std::string& path)
        : file_(std::fopen(path.c_str(), "r")) {
        if (!file_) throw std::runtime_error("Cannot open: " + path);
    }
    ~FileHandle() { if (file_) std::fclose(file_); }

    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
    FileHandle(FileHandle&& other) noexcept : file_(other.file_) { other.file_ = nullptr; }
    FileHandle& operator=(FileHandle&& other) noexcept {
        if (this != &other) { if (file_) std::fclose(file_); file_ = other.file_; other.file_ = nullptr; }
        return *this;
    }
private:
    FILE* file_;
};
```

## Builder Pattern

```cpp
class ServerConfig {
public:
    ServerConfig& port(uint16_t p)  { port_ = p; return *this; }
    ServerConfig& threads(int n)    { threads_ = n; return *this; }
    Server build() const            { return Server{*this}; }
private:
    uint16_t port_ = 8080;
    int threads_ = 4;
};
```

## Closed Type Sets

Prefer `std::variant` + `std::visit` over inheritance hierarchies for closed type sets.

## Reference

See skill: `cpp-coding-standards` for comprehensive C++ patterns including templates, concurrency, and modern idioms.
