# Claude Code and GLM Setup Guide

## Objective

This guide explains how to use Claude Code with GLM-compatible models on Windows. It covers:

- Installing Claude Code CLI.
- Connecting Claude Code to GLM, such as `glm-4.6`.
- Understanding two integration approaches:
  - Direct configuration through `settings.json`.
  - Proxy-based routing through CC Switch.

## Environment Preparation

### 1. Install Node.js

Download Node.js from:

```text
https://nodejs.org/
```

If the `.msi` installer cannot be opened, check the Windows Installer service:

1. Press `Win + R`.
2. Run `services.msc`.
3. Find `Windows Installer`.
4. Right-click it and start the service.

You can also install from an administrator command prompt:

```cmd
msiexec /package D:\nodejs.msi
```

Verify the installation:

```bash
node -v
npm -v
```

### 2. Install Git

Download Git for Windows from:

```text
https://git-scm.com/downloads/win
```

Verify the installation:

```bash
git --version
```

## Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

Verify:

```bash
claude
```

## Option 1: Direct GLM Configuration

### Idea

Claude Code sends requests to a GLM-compatible endpoint that exposes an Anthropic-style API.

```text
Claude Code -> GLM API
```

### 1. Get an API Key

Create an API key from the Zhipu AI platform or another GLM-compatible provider.

### 2. Configure Environment Variables

Use values similar to the following:

```text
ANTHROPIC_AUTH_TOKEN=your_api_key
ANTHROPIC_BASE_URL=https://api.z.ai/api/anthropic
API_TIMEOUT_MS=3000000
ANTHROPIC_DEFAULT_HAIKU_MODEL=glm-4.6
ANTHROPIC_DEFAULT_SONNET_MODEL=glm-4.6
ANTHROPIC_DEFAULT_OPUS_MODEL=glm-4.6
```

### 3. Update Claude Settings

Edit:

```text
C:\Users\<your-user-name>\.claude\settings.json
```

Example:

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "your_zai_api_key",
    "ANTHROPIC_BASE_URL": "https://api.z.ai/api/anthropic",
    "API_TIMEOUT_MS": "3000000",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.6",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-4.6",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-4.6"
  }
}
```

## Option 2: Use CC Switch

### Idea

CC Switch acts as a proxy and model router.

```text
Claude Code -> CC Switch -> GLM API
```

### Benefits

- Supports switching between multiple models, such as GPT, GLM, and Gemini.
- Adapts request formats automatically.
- Provides UI-based management.
- Is usually more stable for daily use.

## Comparison

| Dimension | Direct Configuration | CC Switch |
| --- | --- | --- |
| Architecture | Direct API call | Proxy routing |
| Latency | Lower | Slightly higher |
| Stability | Medium | Higher |
| Multi-model support | Limited | Supported |
| Ease of use | Lower | Higher |
| Extensibility | Lower | Higher |

## Verification

Run:

```bash
claude
```

Test with a simple prompt:

```text
Introduce yourself briefly.
```

Successful signs:

- The selected model is `glm-4.6`.
- Claude Code returns a normal response.

## Troubleshooting

### `claude` command not found

Check whether the npm global bin directory is in `PATH`.

### No API response

Check network access, account balance, and API endpoint configuration.

### Model did not switch

Restart the terminal after changing environment variables or settings.

### `401` or `403`

The API key is wrong, expired, or not allowed to access the selected model.

## Key Idea

The setup is essentially:

```text
Claude Code UI
  -> protocol adapter or proxy
  -> GLM model API
```

In other words, the important part is protocol adaptation and API forwarding.

## Recommendation

For daily use, CC Switch is usually easier and more stable. Direct configuration is simpler conceptually, but it can be less robust when switching models or providers.
