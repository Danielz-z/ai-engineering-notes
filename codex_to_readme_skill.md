# Codex to Experiment Log and README Skill

## Overview

This skill turns Codex conversations and code-change records, such as diffs or commits, into structured documentation:

- An experiment log.
- A GitHub-ready project README.

The goal is to transform the development process into reproducible structured documents.

## Done Criteria

- The output includes a complete experiment log with time, task, method, changes, results, and issues.
- The generated README can be used directly as a GitHub project landing page.
- All steps are reproducible, with clear commands and workflow descriptions.
- Multi-round experiments can be appended over time.

## Input

- `codex_chat`: Codex conversation log. Required.
- `code_diff`: Code change record. Optional.
- `project_name`: Project name.
- `task_description`: Current task description.

## Output

- `experiment_log.md`
- `README.md`

## Workflow

### Step 1: Extract Information

Extract:

- Task objective.
- Method.
- Changes.
- Result.
- Issues and fixes.

### Step 2: Generate the Experiment Log

```markdown
# Experiment Log

## [DATE]

### Task
{task_description}

### Method
{extracted_method}

### Changes
{extracted_changes}

### Result
{extracted_result}

### Issues
{extracted_issues}
```

### Step 3: Generate the README

````markdown
# {project_name}

## Overview
Explain the project goal and background.

## Pipeline
Data -> preprocessing -> model -> output

## Usage
```bash
python main.py
```

## Results
Summarize the experiment results, metrics, and observed performance.

## Project Structure
```text
project/
|-- src/
|-- data/
|-- models/
|-- logs/
`-- README.md
```
````

## Prompt Template for Codex

```text
You are a documentation agent.

Goal:
Convert the following Codex conversation into:
1. Experiment log
2. README.md

Context:
- Project: {project_name}
- Task: {task_description}

Constraints:
- Must be reproducible
- Must include pipeline
- Must extract real changes, not guesses

Done when:
- README is GitHub-ready
- Experiment log is structured

Input:
{paste codex chat here}
```

## Automation Tip

Use Git history as additional input:

```bash
git log -p > diff.txt
```

## Multi-Round Experiment Log

```markdown
# Experiment Log

## Day 1
...

## Day 2
...
```

## Summary

```text
Codex conversation -> captured experiment knowledge -> reproducible documentation
```
