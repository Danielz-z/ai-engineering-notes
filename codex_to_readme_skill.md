# Codex → Experiment Log & README Skill

## Overview
该 Skill 用于将 Codex 对话、代码修改记录（diff / commit）自动整理为：
- 实验日志（Experiment Log）
- 项目 README（可直接用于 GitHub）

目标：将“开发过程”转化为“可复现的结构化文档”。

---

## Done Criteria
- 输出包含完整实验日志（时间、任务、方法、修改、结果、问题）
- 输出 README 可直接用于 GitHub 项目首页
- 所有步骤具备可复现性（命令、流程明确）
- 支持多轮实验追加记录

---

## Input
- codex_chat: Codex 对话记录（必须）
- code_diff: 代码修改记录（可选）
- project_name: 项目名称
- task_description: 当前任务描述

---

## Output
- experiment_log.md
- README.md

---

## Workflow

### Step 1: 信息抽取
提取：
- 任务目标（Task）
- 方法（Method）
- 修改内容（Changes）
- 实验结果（Result）
- 问题与修复（Issues）

---

### Step 2: 生成实验日志

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

---

### Step 3: 生成 README

# {project_name}

## Overview
项目目标与背景说明

## Pipeline
数据 → 预处理 → 模型 → 输出

## Usage
```bash
python main.py
```

## Results
实验结果总结（指标/表现）

## Project Structure
project/
├── src/
├── data/
├── models/
├── logs/
└── README.md

---

## Prompt Template（用于 Codex）

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
- Must extract real changes (not guess)

Done when:
- README is GitHub-ready
- Experiment log is structured

Input:
{paste codex chat here}

---

## 自动化建议

结合 Git：
git log -p > diff.txt

---

## 多轮实验记录

## Experiment Log

### Day 1
...

### Day 2
...

---

## Summary

Codex 对话 → 实验知识沉淀 → 可复现文档
