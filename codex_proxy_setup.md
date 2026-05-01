# Codex CLI + 远程服务器 + 本地代理 完整配置指南

## 一、目标

在无 sudo 权限 + 服务器无法直接访问外网的情况下，实现：

本地代理 → SSH 反向隧道 → 服务器 → npm → Codex CLI

最终效果：
- 在远程服务器运行 Codex
- 所有请求通过本地代理转发
- 避免 reconnecting / ECONNREFUSED

---

## 二、系统架构

[本地电脑]
  代理(127.0.0.1:7897)
        ↓
SSH -R
        ↓
[服务器]
  127.0.0.1:7897
        ↓
npm / codex

---

## 三、核心步骤

### 1. 本地执行

ssh -R 7897:127.0.0.1:7897 Gold_Server

---

### 2. 服务器设置代理

export http_proxy=http://127.0.0.1:7897
export https_proxy=http://127.0.0.1:7897
export ALL_PROXY=socks5h://127.0.0.1:7897

---

### 3. 验证

curl https://registry.npmjs.org

---

### 4. 无 sudo 安装

mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=$HOME/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

---

### 5. 安装

npm install -g @openai/codex

---

### 6. 运行

codex

---

## 四、关键约束

- SSH 不能断
- 本地代理必须开

---

## 五、总结

通过 SSH 反向代理，将本地代理能力提供给服务器，从而在受限环境中运行 Codex。
