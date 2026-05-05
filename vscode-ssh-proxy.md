# VS Code Remote SSH Proxy

This note explains how to fix external network access for VS Code Remote SSH on a restricted remote server. The same setup can also help tools such as Copilot, Codex, pip, git, and Hugging Face downloads.

## Problem

Some remote servers, such as lab GPU nodes, cannot directly access the public internet. In that environment:

- VS Code extensions may fail.
- Copilot may not connect.
- `pip install`, `git clone`, and model downloads may time out.

The solution is to let the remote server use the local machine's proxy through SSH remote forwarding.

## Core Idea

```text
Remote server -> SSH tunnel -> Local proxy -> Internet
```

More specifically:

```text
VS Code remote process
  -> 127.0.0.1:6152 on the remote server
  -> SSH RemoteForward
  -> 127.0.0.1:7897 on the local machine
  -> Internet
```

## Requirements

- The local machine can access the internet through a proxy, such as Clash, V2Ray, or another local proxy tool.
- You know the local proxy port. In this example, it is `7897`.
- SSH access to the remote server is already configured.

## Setup

### 1. Configure SSH on the Local Machine

Edit:

```text
~/.ssh/config
```

Example:

```sshconfig
Host 4090
    HostName 192.168.3.28
    User zhouzheng

    RemoteForward 6152 127.0.0.1:7897
    RemoteForward 6153 127.0.0.1:7897
```

In this setup:

- `6152` is used as the remote HTTP proxy port.
- `6153` is used as the remote SOCKS proxy port.
- `7897` is the local proxy port.

### 2. Connect Through the SSH Host Alias

```bash
ssh 4090
```

Use the configured `Host` alias. If you connect directly by IP address, the `RemoteForward` settings from the SSH config may not be applied.

### 3. Configure VS Code on the Remote Server

Run this on the remote server:

```bash
mkdir -p ~/.vscode-server/data/Machine
```

Create or update:

```bash
cat > ~/.vscode-server/data/Machine/settings.json << 'EOF'
{
  "http.proxy": "http://127.0.0.1:6152",
  "http.proxyStrictSSL": false,
  "http.proxySupport": "on",

  "remote.env": {
    "HTTP_PROXY": "http://127.0.0.1:6152",
    "HTTPS_PROXY": "http://127.0.0.1:6152",
    "ALL_PROXY": "socks5://127.0.0.1:6153"
  }
}
EOF
```

### 4. Verify the Proxy

Run this on the remote server:

```bash
curl google.com -I -x http://127.0.0.1:6152
```

Expected result:

```text
HTTP/1.1 200 OK
```

or:

```text
HTTP/1.1 301 Moved Permanently
```

### 5. Reload VS Code

In VS Code:

```text
Ctrl + Shift + P -> Reload Window
```

## Expected Result

After the setup works:

- Copilot and Codex can connect.
- `pip install` can reach external package indexes.
- `git clone` works through the proxy.
- Hugging Face downloads can use the forwarded proxy.

## Troubleshooting

### `curl` Times Out

Check whether the remote port is listening:

```bash
netstat -tlnp | grep 6152
```

If there is no output, the SSH remote forwarding did not start. If there is output, check whether the local proxy is running.

### `RemoteForward` Does Not Work

Common causes:

- The SSH connection did not use the configured host alias.
- VS Code connected directly by IP instead of using the host entry.
- The SSH session was restarted without the forwarding configuration.

### Copilot Does Not Work

Check:

- Whether VS Code was reloaded.
- Whether `settings.json` is in the remote VS Code server path.
- Whether `remote.env` contains the proxy variables.

### Wrong Proxy Port

Confirm the actual local proxy port. This note uses:

```text
7897
```

Replace it with the port used by your own proxy tool.

## Notes

- `curl` does not automatically use the proxy unless `-x` is provided or proxy environment variables are set.
- `ping` cannot verify whether an HTTP or SOCKS proxy works.
- The `.vscode-server` path is important because VS Code Remote reads machine-level settings from that location.

## Keywords

- VS Code Remote SSH proxy
- Copilot remote server proxy
- SSH `RemoteForward`
- Linux proxy through SSH

## Summary

SSH remote forwarding lets a remote server borrow the local machine's network access. This is useful when a server can be reached through SSH but cannot directly access the public internet.
