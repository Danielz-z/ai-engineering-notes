# Single-Agent Project Template

## 1. Problem

A repository worked on by a single coding agent still needs a shared scaffold so the agent stays inside scope, keeps documentation in sync with the code, and finishes each task against a clear Definition of Done. Without it, tasks drift, unrelated refactors slip in, and `README.md` rots out of date with the commands it documents.

## 2. Environment

- OS: any
- Tool: a single coding agent (e.g., Claude Code or any agent following an `AGENTS.md` convention)
- Context: a repository where one agent reads the code, plans, implements, and verifies its own changes

## 3. Cause

- No rules file means scope, refactoring, and dependency policies are ambiguous.
- Architecture and status are not recorded, so context is lost between sessions.
- Setup and run commands live only in the agent's memory, not in `README.md`.
- Missing acceptance criteria allow tasks to be marked done prematurely.

## 4. Solution

### 4.1 Standard project layout

```text
project/
├── src/                  # Source code
├── tests/                # Tests
├── docs/
│   ├── ARCHITECTURE.md   # Architecture and module boundaries
│   └── PROJECT_STATUS.md # Current progress and known issues
├── AGENTS.md             # Agent development rules
├── .env.example          # Environment variable template
└── README.md
```

When using only Claude Code, `AGENTS.md` may be replaced with `CLAUDE.md`.

### 4.2 README scaffold

```markdown
# Project Name

## Overview
Briefly describe the project goal, main functionality, and intended use cases.

## Tech Stack
* Frontend:
* Backend:
* Database:
* Deployment:
* Package Manager:
```

### 4.3 Setup and usage commands

```bash
# Install dependencies
<install-command>

# Start the development environment
<dev-command>

# Run tests
<test-command>

# Run lint checks
<lint-command>

# Build the project
<build-command>
```

### 4.4 Environment variables

```bash
cp .env.example .env
```

Fill in the required local configuration. Never commit real secrets, passwords, API keys, or tokens.

### 4.5 Agent workflow

```text
Understand the task
→ Read the relevant code and documentation
→ Create an implementation plan
→ Implement the changes
→ Run tests, lint, and build
→ Review the Git diff
→ Update the relevant documentation
```

### 4.6 Development rules

The agent must follow `AGENTS.md`. Baseline requirements:

- Do not modify files outside the task scope.
- Do not perform unrelated refactoring.
- Do not upgrade dependencies unless required.
- Do not remove compatibility code without understanding its purpose.
- Update `docs/ARCHITECTURE.md` when module boundaries or structure change.
- Update `docs/PROJECT_STATUS.md` when features are completed.
- Update `README.md` when setup or execution commands change.

## 5. Verification

A task is complete only when:

- The implementation meets the requirements.
- Tests pass.
- Lint checks pass.
- The project builds successfully.
- No unrelated changes are included.
- Documentation remains consistent with the code.

## 6. Common Issues

- Scope creep: unrelated cleanups land in the same diff. Reject them at review time.
- Silent dependency upgrades: forbid unless the task explicitly requires them.
- Stale `README.md`: any change to setup, dev, test, lint, or build commands must update the README in the same commit.
- `.env` committed by accident: keep only `.env.example` in the repo.

## 7. Summary

Pin a shared layout, a rules file (`AGENTS.md` or `CLAUDE.md`), and a fixed plan-implement-verify workflow so a single agent stays inside scope and every task has an objective Definition of Done.
