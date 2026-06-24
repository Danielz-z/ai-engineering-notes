# AI Engineering Notes

This repository collects practical engineering notes from my local development and internship work. The notes focus on repeatable setup steps, debugging patterns, and small workflows that solved real problems.

## Notes

### Codex Conversation to Experiment Log and README

A reusable prompt and documentation workflow for turning Codex conversations, diffs, and commits into structured experiment logs and GitHub-ready README files.

[codex_to_readme_skill.md](codex_to_readme_skill.md)

### Codex CLI Remote Proxy Setup

Run Codex CLI on a remote server without direct internet access or sudo permissions by using an SSH reverse tunnel and a local proxy.

[codex_proxy_setup.md](codex_proxy_setup.md)

### Codex Log Tool

A small Python script that parses Codex `.jsonl` session logs into readable Markdown experiment logs and project summaries.

[codex_log_tool.md](codex_log_tool.md)

### VS Code Remote SSH Proxy

Configure VS Code Remote SSH, Copilot, pip, git, and Hugging Face access on restricted remote servers through SSH remote forwarding.

[vscode-ssh-proxy.md](vscode-ssh-proxy.md)

### Bash, WSL, CMD, and PowerShell

A short comparison of common Windows and Linux command-line environments and when to use each one.

[bash_wsl_cmd_powershell_summary.md](bash_wsl_cmd_powershell_summary.md)

### Claude Code and GLM Setup

Configure Claude Code to use GLM-compatible models directly or through CC Switch, with notes on installation, environment variables, and troubleshooting.

[claude_glm_full_guide.md](claude_glm_full_guide.md)

### Claude Code and Codex Plugin Setup

Install and troubleshoot a Claude Code plugin workflow that uses the local Codex CLI as a reviewer and second-pass coding assistant.

[codex_claude_plugin_setup.md](codex_claude_plugin_setup.md)

### Amor Fati Personal AI Infrastructure

Document a Docker-first personal website and AI infrastructure migration with Caddy, WordPress, FastAPI, and future agent services.

[amorfati-ai-infra-readme.md](amorfati-ai-infra-readme.md)

### DeepFace Robot Emotion Control

Use local DeepFace emotion recognition to drive Astribot robot movement through a Windows, WSL, and direct motion API control chain.

[deepface_robot_control.md](deepface_robot_control.md)

### AI-Powered Academic Writing Workflow

Use Overleaf, GitHub, local LaTeX tooling, and AI coding agents as a structured academic writing pipeline.

[ai_paper_workflow.md](ai_paper_workflow.md)

### Fine-tuning π0.5 on Custom Aloha Pick Tasks
 
Fine-tune the π0.5 vision-language-action model on bimanual robot manipulation tasks using the openpi framework, with LoRA fine-tuning, GPU resource management, and Weights & Biases offline training.

[pi05_aloha_finetune.md](pi05_aloha_finetune.md)

### OpenPI Inference Deployment on AgileX ALOHA

Deploy OpenPI policy serving and run dual-arm Piper inference on an AgileX ALOHA platform, covering hardware bring-up, three-terminal startup, prompt switching, and a left-gripper failed-grasp recovery loop driven by joint feedback.

[openpi_aloha_inference_deployment.md](openpi_aloha_inference_deployment.md)

### Claude Code and Codex Project Template

A reusable project scaffold for repositories where Claude Code implements changes and Codex performs independent diff review, covering shared rules, directory layout, workflow, and Definition of Done.

[claude_code_codex_project_template.md](claude_code_codex_project_template.md)

## Notes About This Repository

- The notes are based on personal practice and real setup problems.
- Some commands are environment-specific and may need small adjustments.
- The goal is practical reuse, not theoretical completeness.
