# Claude Code and Codex Project Template

## 1. Problem

A project that uses both Claude Code and Codex as collaborating agents needs a shared structure, shared rules, and a clear handoff workflow. Without this, the two agents step on each other's edits, drift from the same source of truth, and produce unreviewable diffs.

## 2. Environment

- OS: any
- Tools: Claude Code, Codex, Git
- Context: a repository where Claude Code implements changes and Codex independently reviews them

## 3. Cause

- No shared rules file means each agent applies different conventions.
- No clear division of responsibility leads to conflicting edits in the same files.
- Architecture, status, and decisions are not recorded, so context is lost between sessions.
- Missing acceptance criteria allow tasks to be marked done prematurely.

## 4. Solution

### 4.1 Standard project layout

```text
project/
├── src/                  # Source code
├── tests/                # Tests
├── docs/
│   ├── ARCHITECTURE.md   # Architecture and module boundaries
│   ├── PROJECT_STATUS.md # Current progress and known issues
│   └── DECISIONS.md      # Important technical decisions
├── AGENTS.md             # Shared rules for Claude Code and Codex
├── CLAUDE.md             # Claude Code-specific instructions
├── .env.example          # Environment variable template
└── README.md
```

### 4.2 README scaffold

```markdown
# Project Name

## Overview
Briefly describe the problem this project solves, its target users, and its core functionality.

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

Fill in required values locally. Never commit real secrets, passwords, API keys, or tokens.

### 4.5 Agent responsibilities

- Claude Code: analyze requirements, create implementation plans, implement features, and fix issues.
- Codex: review Git diffs, identify defects, add missing tests, and perform independent code reviews.
- Only one agent may modify the same working directory at a time.
- Use separate Git branches or Git worktrees for parallel development.

### 4.6 Standard workflow

```text
Define the task and acceptance criteria
→ Claude Code implements the changes
→ Run tests, lint, and build
→ Codex reviews the Git diff
→ Claude Code fixes identified issues
→ Run verification again
→ Human review and merge
```

### 4.7 Development rules

All agents must follow `AGENTS.md`. Baseline requirements:

- Read the relevant code and documentation before making changes.
- Do not perform unrelated refactoring.
- Do not upgrade dependencies unless explicitly required.
- Do not modify production configuration or real secrets.
- Update `docs/ARCHITECTURE.md` when the architecture changes.
- Update `docs/PROJECT_STATUS.md` when features are completed.
- Record important technical decisions in `docs/DECISIONS.md`.

## 5. Verification

A task is complete only when all of the following are satisfied:

- The implementation meets the acceptance criteria.
- Tests pass.
- Lint checks pass.
- The project builds successfully.
- No unrelated files were modified.
- Required documentation has been updated.
- The Git diff has been reviewed.

## 6. Common Issues

- Both agents editing the same directory at the same time: enforce branch or worktree separation.
- Silent dependency upgrades: forbid unless the task explicitly requires them.
- Untracked decisions: every non-trivial choice goes into `docs/DECISIONS.md` before merging.
- `.env` committed by accident: keep only `.env.example` in the repo.

## 7. Summary

Pin a shared layout, a shared rules file, and a fixed implement-then-review workflow so Claude Code and Codex stay on the same page and every task has an objective Definition of Done.
