# Codex CLI + Remote Server + Local Proxy Setup Guide (Detailed)

## 1. Objective

This guide explains how to run Codex CLI on a remote Linux server that:

- Has no direct internet access
- Has no sudo privileges

Solution:

Local Proxy → SSH Reverse Tunnel → Remote Server → npm → Codex CLI

---

## 2. Architecture Overview

[Local Machine]
  Proxy (127.0.0.1:7897)
        ↓
SSH Reverse Tunnel (-R)
        ↓
[Remote Server]
  127.0.0.1:7897
        ↓
npm / Codex access internet

---

## 3. Step-by-Step Setup

### Step 1: Start SSH Reverse Tunnel (Local Machine)

Run:

```bash
ssh -R 7897:127.0.0.1:7897 Gold_Server
```

---

### Step 2: Configure Proxy (Remote Server)

```bash
export http_proxy=http://127.0.0.1:7897
export https_proxy=http://127.0.0.1:7897
export ALL_PROXY=socks5h://127.0.0.1:7897
```

---

### Step 3: Verify Connectivity

```bash
curl https://registry.npmjs.org
```

---

### Step 4: Fix npm Permission (No sudo)

```bash
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=$HOME/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

### Step 5: Install Codex CLI

```bash
npm install -g @openai/codex
```

---

### Step 6: Verify Installation

```bash
which codex
codex
```

---

## 4. Common Errors

### ECONNREFUSED
Proxy unreachable

### Permission denied (publickey)
SSH key issue

### EACCES
No write permission

### command not found
PATH not loaded

### reconnecting
Proxy chain broken

---

## 5. Critical Constraints

- Keep SSH session alive
- Keep local proxy running

---

## 6. Final Result

Codex CLI runs on remote server with proxy access.

---

## 7. Key Insight

SSH reverse tunneling enables external network access in restricted environments.
