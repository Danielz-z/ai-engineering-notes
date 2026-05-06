# Claude Code and Codex Plugin Setup Guide

This is a beginner-friendly setup note for using the Codex plugin inside Claude Code.

The plugin command names and marketplace package names may change over time. Treat this note as a practical workflow record, and verify the current plugin name inside Claude Code before installing.

## Core Concept

This integration does not mean that Claude has Codex built in.

The architecture is:

```text
Claude Code
  -> Codex plugin
  -> Local Codex CLI
  -> OpenAI API
```

Key point:

- The plugin is only a bridge.
- The actual execution happens in the local Codex CLI.
- If the local Codex CLI is broken, the plugin will fail.

## Correct Installation Flow

Follow the steps in order and do not skip verification.

### Step 1: Install Codex CLI

```bash
npm install -g @openai/codex
```

Verify the installation:

```bash
codex --version
```

If this command fails, fix the Codex CLI installation before configuring the Claude plugin.

### Step 2: Sign In to Codex

Use the current Codex CLI login flow:

```bash
codex --login
```

If your environment still uses API-key based authentication, configure the OpenAI API key according to the Codex CLI instructions for your version.

Purpose:

- Store local credentials.
- Enable Codex CLI to call OpenAI models.
- Confirm that Codex works outside Claude Code first.

### Step 3: Install the Claude Code Plugin

Inside Claude Code, install the Codex plugin from the plugin marketplace. The commands may look like this:

```text
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
```

If the marketplace path or package name has changed, use Claude Code's plugin search or marketplace list command to find the current Codex plugin entry.

### Step 4: Reload Plugins

After installation, reload plugins:

```text
/reload-plugins
```

If the plugin still does not appear, restart Claude Code completely.

### Step 5: Initialize the Plugin

Run:

```text
/codex:setup
```

This step should:

- Detect the local Codex CLI.
- Verify the local login state.
- Attempt basic automatic fixes.

## Common Setup Error

### `Unknown command: /codex:setup`

Meaning:

```text
Claude Code did not load the Codex plugin.
```

Likely causes:

- The plugin was not installed.
- Plugins were not reloaded.
- Claude Code needs a full restart.
- The plugin command namespace changed.

## Troubleshooting

### Check Installed Plugins

Run:

```text
/plugin list
```

If `codex@openai-codex` does not appear, reinstall the plugin or search for the current package name.

### Reload Plugins

```text
/reload-plugins
```

### Restart Claude Code

Some plugin environments do not hot reload reliably. If commands are still missing, restart Claude Code completely.

### Verify Codex CLI

```bash
codex --version
```

If this fails, your `PATH` is probably incorrect or Codex CLI was not installed globally.

## Common Issues

### npm Permission Errors

Error:

```text
EACCES: permission denied
```

Fix:

```bash
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
export PATH="$HOME/.npm-global/bin:$PATH"
source ~/.bashrc
npm install -g @openai/codex
```

### Proxy or Network Issues

If Codex CLI cannot access the internet, configure proxy variables before running it:

```bash
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
```

Adjust the port based on your local proxy settings.

## How to Use the Plugin

### Standard Code Review

```text
/codex:review
```

Purpose:

- Find bugs.
- Detect edge cases.
- Suggest improvements.

### Adversarial Review

```text
/codex:adversarial-review
```

Purpose:

- Challenge the implementation.
- Find hidden assumptions.
- Identify risky logic.

This is especially useful for:

- Database systems.
- Permission and authentication logic.
- Infrastructure code.
- Core backend behavior.

### Rescue Mode

```text
/codex:rescue
```

Use this when Claude Code gets stuck and you want Codex to provide another implementation path.

### Background Tasks

Run a review in the background:

```text
/codex:review --background
```

Check status:

```text
/codex:status
```

View results:

```text
/codex:result
```

Cancel:

```text
/codex:cancel
```

## Recommended Workflow

```text
1. Claude writes or edits code.
2. Run /codex:review.
3. For high-risk modules, run /codex:adversarial-review.
4. Fix the issues.
5. Commit the final result.
```

## Engineering Positioning

Do not think of Codex as a replacement for Claude Code.

A more useful mental model is:

```text
Claude = builder
Codex = reviewer
```

## Shortest Successful Setup Path

```bash
npm install -g @openai/codex
codex --login
```

Then inside Claude Code:

```text
/plugin install codex@openai-codex
/reload-plugins
/codex:setup
/codex:review
```

If `/codex:review` works, the integration is operational.

## Summary

This workflow is a small multi-agent development setup:

```text
Claude Code handles building.
Codex provides review and second-pass reasoning.
```

Advantages:

- Second opinion.
- Stronger review process.
- Fewer unchecked assumptions.
- More production-oriented workflow.

Tradeoffs:

- More setup complexity.
- Higher API cost.
- Possible plugin instability.

Most important takeaway:

```text
Codex CLI is the real capability.
The plugin is the interface layer.
```
