# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code plugin** - a collection of production-ready agents, skills, hooks, commands, rules, and MCP configurations. The project provides battle-tested workflows for software development using Claude Code.

## Commands

```bash
# Run all validators + tests (CI pipeline)
npm test

# Run only the test suite (no validators)
node tests/run-all.js

# Run a single test file
node tests/lib/utils.test.js

# Lint (ESLint + markdownlint)
npm run lint

# Run the claw codemap generator
npm run claw

# Install to ~/.claude/ (default) or .cursor/
./install.sh typescript
./install.sh --target cursor typescript python
./install.sh cpp
./install.sh c cpp
```

`npm test` runs five CI validators (`scripts/ci/validate-*.js`) then all unit/integration tests in `tests/`.

## Architecture

```
agents/        # Specialized subagents (.md files with YAML frontmatter)
commands/      # Slash commands invoked with /command-name
contexts/      # Reusable context snippets (dev.md, research.md, review.md)
examples/      # Sample CLAUDE.md files for various project types
hooks/         # hooks.json — trigger-based automations
mcp-configs/   # MCP server configurations
plugins/       # Claude Code plugin manifests
rules/         # Always-loaded guidelines, organized by common/ + language/
schemas/       # JSON schemas for hooks.json, package-manager, and plugin
scripts/
  ci/          # validate-agents.js, validate-commands.js, etc. (run by npm test)
  hooks/       # Node.js scripts executed by hooks at runtime
  lib/         # Shared utilities: utils.js, package-manager.js, session-manager.js
skills/        # Domain knowledge modules (one subdirectory + SKILL.md each)
tests/         # Mirrors scripts/ with .test.js files; no test framework needed
```

### File Format Requirements

**Agents** (`agents/*.md`) — YAML frontmatter required fields: `name`, `description`, `tools`, `model` (`haiku` | `sonnet` | `opus`). Validated by `scripts/ci/validate-agents.js`.

**Skills** (`skills/<name>/SKILL.md`) — YAML frontmatter required fields: `name`, `description`. Validated by `scripts/ci/validate-skills.js`.

**Commands** (`commands/*.md`) — YAML frontmatter required fields: `description`. Validated by `scripts/ci/validate-commands.js`.

**Hooks** (`hooks/hooks.json`) — Must match `schemas/hooks.schema.json`. Hook scripts in `scripts/hooks/` are Node.js and read JSON from stdin.

**Rules** (`rules/`) — Plain markdown, organized into `common/` (always applies) and language subdirectories (`typescript/`, `python/`, `golang/`, `swift/`).

## Key Commands

- `/tdd` - Test-driven development workflow
- `/plan` - Implementation planning
- `/e2e` - Generate and run E2E tests
- `/code-review` - Quality review
- `/build-fix` - Fix build errors
- `/learn` - Extract patterns from sessions
- `/skill-create` - Generate skills from git history
- `/cpp-review` - C++ code review (RAII, memory safety, UB)
- `/cpp-build` - Fix C++ build/CMake/linker errors
- `/cpp-test` - C++ TDD with GoogleTest/Catch2
- `/c-review` - C code review (memory safety, CERT C)

## Development Notes

- Package manager detection: npm, pnpm, yarn, bun (configurable via `CLAUDE_PACKAGE_MANAGER` env var or project config)
- Cross-platform: Windows, macOS, Linux support via Node.js scripts
- Tests use Node.js `assert` directly — no test framework dependency
- The `scripts/lib/` utilities (utils.js, package-manager.js, etc.) are shared between hook scripts and tests

## Contributing

Follow the formats in CONTRIBUTING.md. File naming: lowercase with hyphens (e.g., `python-reviewer.md`, `tdd-workflow.md`).
