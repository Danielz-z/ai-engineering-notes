# AI-Powered Academic Writing Workflow

This repository documents a practical workflow that integrates:

- Overleaf
- Claude Code / Codex
- Local LaTeX environment
- GitHub

The goal is to turn paper writing into a structured, version-controlled, and AI-augmented engineering process.

---

## Core Idea

Instead of writing papers directly in Overleaf, we treat the paper as a code project:

```text
Overleaf + GitHub + Local LaTeX + AI Agent
```

This enables:

- Human + AI collaboration
- Full version control
- Reproducible writing process
- Safer workflow (no accidental data loss)

---

## Basic Workflow

### 1. Create Template in Overleaf

- Start with a conference/journal template in Overleaf
- Requires Overleaf premium (for Git support)
- Sync project with GitHub (via Git)

### 2. Clone Repository Locally

```bash
git clone <your-overleaf-repo>
cd paper-project
```

Prepare your local environment:

- VSCode
- LaTeX (TeX Live / MacTeX)
- Git
- Claude Code / Codex

### 3. Write with AI Locally

All writing happens locally:

- Draft sections
- Revise paragraphs
- Polish language
- Prepare rebuttal

Then sync:

```bash
git add .
git commit -m "update section"
git push
```

Overleaf updates automatically.

---

## Key Advantages

### Human + AI Collaboration

- You, your advisor, collaborators, and AI all work on the same repo

### Clear Version History

- Git tracks every change:
  - What changed
  - Why it changed
  - When it changed

### Safety

- Avoid Overleaf crashes or data loss
- GitHub acts as backup

---

## Advanced Usage

### Iterative AI Polishing

Use Claude Code / Codex to:

- Improve introduction to match top-tier conference style
- Refine experiment analysis
- Strengthen logical structure

### Integrate with Auto-Research Workflow

Pipeline:

```text
Idea -> Experiment -> Logs -> Notes -> Paper
```

---

## Key Principle: Writing from Documents

Low-quality approach:

"Write me an introduction"

High-quality approach:

Use:

- Experiment logs
- Weekly reports
- Research notes
- Paper reading notes
- Debug records

Feed these into AI, then generate structured paper content.

---

## Tips

### Tip 1: Always Write from Documents

Feed structured materials into AI for better quality output.

### Tip 2: Let AI Help You Build Documents

```text
AI -> Documents -> AI -> Paper
```

---

## Summary

Paper writing is not typing LaTeX.

It is research-to-paper pipeline engineering.
