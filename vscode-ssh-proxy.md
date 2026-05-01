# 🚀 VS Code Remote SSH Proxy

解决远程服务器无法访问外网的问题（Copilot / Codex / pip / git / HuggingFace）

---

## 📌 Overview

当远程服务器（如实验室/4090/GPU节点）无法访问外网时，VS Code Remote SSH 插件（如 Copilot）和开发工具（pip / git）会失效。

本方案通过：

> **SSH RemoteForward + 本地代理**

实现远程服务器“借用”本地网络能力。

---

## 🧠 Core Idea

```
Remote Server → SSH Tunnel → Local Proxy → Internet
```

---

## ⚙️ Requirements

- 本地电脑可科学上网（Clash / V2Ray / SakuraCat 等）
- 知道本地代理端口（例如：7897）
- 已配置 SSH 连接远程服务器

---

## 🛠️ Setup

---

### 1. Configure SSH

编辑本地：

```
~/.ssh/config
```

```bash
Host 4090
    HostName 192.168.3.28
    User zhouzheng

    RemoteForward 6152 127.0.0.1:7897
    RemoteForward 6153 127.0.0.1:7897
```

---

### 2. Connect via SSH

```bash
ssh 4090
```

> ❗ 必须使用 `Host`，不能直接用 IP

---

### 3. Configure VS Code (Remote)

在远程服务器执行：

```bash
mkdir -p ~/.vscode-server/data/Machine
```

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

---

### 4. Verify

```bash
curl google.com -I -x http://127.0.0.1:6152
```

Expected:

```
HTTP/1.1 200 OK
or
HTTP/1.1 301 Moved Permanently
```

---

### 5. Reload VS Code

```
Ctrl + Shift + P → Reload Window
```

---

## ✅ Result

成功后：

- Copilot / Codex ✔
- pip install ✔
- git clone ✔
- HuggingFace ✔

---

## 🔍 Troubleshooting

---

### ❌ curl timeout

```bash
netstat -tlnp | grep 6152
```

- 无输出 → SSH 转发失败
- 有输出 → 检查本地代理

---

### ❌ RemoteForward 不生效

原因：

- 没使用 `ssh 4090`
- VS Code 未使用 Host 连接

---

### ❌ Copilot 不工作

检查：

- 是否执行 Reload Window
- settings.json 是否正确
- remote.env 是否存在

---

### ❌ 端口错误

确认本地代理端口：

```
7897（或你的实际端口）
```

---

## 📎 Notes

- `curl` 默认不走代理，必须使用 `-x`
- `ping` 无法验证代理
- `.vscode-server` 路径不可更改

---

## 📚 Keywords

- VS Code Remote SSH proxy
- Copilot not working fix
- SSH RemoteForward tutorial
- Linux proxy via SSH

---

## ⭐ Summary

> Use SSH tunneling to give remote servers access to your local network.
