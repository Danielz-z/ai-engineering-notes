# Claude Code + GLM 配置与 CC Switch 对比（完整指南）

---

## 一、目标

在 Windows 环境中实现：

- 安装 Claude Code CLI
- 接入 GLM（如 glm-4.6）
- 理解两种接入方式：
  - 直接配置（settings.json）
  - CC Switch 代理方式

---

## 二、环境准备

### 1. 安装 Node.js

https://nodejs.org/zh-cn/download

#### 如果 .msi 无法打开

方法1（推荐）：
1. Win + R
2. 输入：
   services.msc
3. 找到：Windows Installer
4. 右键 → 启动

方法2（命令行）：
1. Win + R → 输入：
   cmd
2. Ctrl + Shift + Enter（管理员）
3. 执行：
   msiexec /package 你的msi路径

示例：
msiexec /package D:\nodejs.msi

验证：
node -v
npm -v

---

### 2. 安装 Git

https://git-scm.com/downloads/win

验证：
git --version

---

## 三、安装 Claude Code

npm install -g @anthropic-ai/claude-code

验证：
claude

---

## 四、方案一：直接接入 GLM（无代理）

### 原理

Claude Code → GLM API（通过伪装 Anthropic 接口）

---

### 配置步骤

#### 1. 获取 API Key
智谱 AI 平台创建 API Key

#### 2. 环境变量

ANTHROPIC_AUTH_TOKEN=你的key  
ANTHROPIC_BASE_URL=https://api.z.ai/api/anthropic  
API_TIMEOUT_MS=3000000  
ANTHROPIC_DEFAULT_HAIKU_MODEL=glm-4.6  
ANTHROPIC_DEFAULT_SONNET_MODEL=glm-4.6  
ANTHROPIC_DEFAULT_OPUS_MODEL=glm-4.6  

---

#### 3. 修改配置文件

路径：
C:\Users\你的用户名\.claude\settings.json

内容：

{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "你的zai_api_key",
    "ANTHROPIC_BASE_URL": "https://api.z.ai/api/anthropic",
    "API_TIMEOUT_MS": "3000000",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.6",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-4.6",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-4.6"
  }
}

---

## 五、方案二：使用 CC Switch（推荐）

### 原理

Claude Code → CC Switch → GLM API

---

### 特点

- 支持多模型切换（GPT / GLM / Gemini）
- 自动适配接口
- 提供 UI 管理
- 更稳定

---

## 六、两种方案对比

| 维度 | 直接配置 | CC Switch |
|------|--------|----------|
| 架构 | 直连 | 代理 |
| 延迟 | 低 | 略高 |
| 稳定性 | 中 | 高 |
| 多模型 | 不支持 | 支持 |
| 易用性 | 较低 | 高 |
| 可扩展性 | 低 | 高 |

---

## 七、验证

运行：
claude

测试：
介绍一下你自己

成功标志：
- 出现 glm-4.6
- 正常返回结果

---

## 八、常见问题

1. claude 命令无效  
→ 检查 PATH  

2. API 无响应  
→ 检查余额 / 网络  

3. 模型未切换  
→ 重启终端  

4. 401 / 403  
→ API Key 错误  

---

## 九、关键理解（核心）

你当前架构：

Claude Code（UI）
↓
CC Switch（代理）
↓
GLM（模型）

本质是：

“协议适配 + API 转发”

---

## 十、推荐策略

### 当前阶段
使用 CC Switch（稳定 + 简单）

### 进阶阶段
自建 proxy server（FastAPI）

---

## 十一、总结

核心操作：

替换 Claude API → 指向 GLM

两种路径：

1. 直接改配置（简单但不稳定）
2. CC Switch（推荐，工程级方案）
