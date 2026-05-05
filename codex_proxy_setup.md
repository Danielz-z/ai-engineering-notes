# Codex CLI Remote Server Proxy Setup

## Objective

This guide explains how to run Codex CLI on a remote Linux server when:

- The server has no direct internet access.
- The account has no sudo privileges.
- npm, Codex, or other tools need access to external package registries.

The solution is:

```text
Local proxy -> SSH reverse tunnel -> Remote server -> npm -> Codex CLI
```

Final result:

- Codex runs on the remote server.
- Network requests are forwarded through the local proxy.
- Common `reconnecting` and `ECONNREFUSED` issues are avoided.

## Architecture

```text
Local machine
  Proxy: 127.0.0.1:7897
        |
        | ssh -R
        v
Remote server
  127.0.0.1:7897
        |
        v
npm / Codex
```

## Setup

### 1. Start the SSH Reverse Tunnel

Run this command on the local machine:

```bash
ssh -R 7897:127.0.0.1:7897 Gold_Server
```

Keep this SSH session alive while using Codex on the remote server.

### 2. Configure Proxy Variables on the Remote Server

Run these commands on the remote server:

```bash
export http_proxy=http://127.0.0.1:7897
export https_proxy=http://127.0.0.1:7897
export ALL_PROXY=socks5h://127.0.0.1:7897
```

### 3. Verify Connectivity

```bash
curl https://registry.npmjs.org
```

If the request returns package registry data, the proxy path is working.

### 4. Install npm Packages Without sudo

```bash
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=$HOME/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### 5. Install Codex CLI

```bash
npm install -g @openai/codex
```

### 6. Run Codex

```bash
codex
```

## Common Issues

### `ECONNREFUSED`

The proxy is unreachable. Check whether the local proxy is running and whether the SSH tunnel is still connected.

### `Permission denied (publickey)`

The SSH key or server login configuration is wrong. Fix SSH authentication before debugging the proxy.

### `EACCES`

npm is trying to write to a system directory. Use the user-level npm prefix shown above.

### `command not found`

The npm global bin directory is not in `PATH`. Reload the shell or run `source ~/.bashrc`.

### `reconnecting`

The proxy chain is broken. Check the local proxy, SSH tunnel, and remote proxy environment variables.

## Key Constraints

- The SSH session must stay connected.
- The local proxy must remain running.
- The remote server should use `127.0.0.1:7897`, because that address is created by the reverse tunnel.

## Summary

SSH reverse tunneling lets a restricted remote server reuse the local machine's proxy. This makes it possible to install and run Codex CLI even when the server itself cannot directly access the internet.
