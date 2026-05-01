# Bash / WSL / CMD / PowerShell 总结

## 一、Bash 是什么
Bash（Bourne Again Shell）= 脚本语言 + 命令解释器  
用于自动化执行系统命令

### 核心能力
- 变量、条件、循环、函数
- 适合写自动化脚本（pipeline）

---

## 二、CMD vs PowerShell

### CMD
- 输出：字符串（文本）
- 特点：无结构，只能文本处理

示例：
dir | find "txt"

---

### PowerShell
- 输出：对象（结构化数据）
- 特点：可以直接操作属性

示例：
Get-ChildItem | Where-Object {$_.Length -gt 1MB}

---

## 三、WSL 是什么

WSL（Windows Subsystem for Linux）  
= 在 Windows 中运行 Linux 的环境

### 核心逻辑
Windows → WSL → Linux → Bash

### WSL2
- 使用真实 Linux 内核
- 接近真实 Linux（推荐）

---

## 四、三者关系

Windows
├── CMD
├── PowerShell
└── WSL
     └── Bash

---

## 五、交互方式

进入 WSL：
wsl

Windows 调用 Linux：
wsl ls

Linux 调用 Windows：
explorer.exe .

---

## 六、能力对比

| 能力 | CMD | PowerShell | WSL |
|------|-----|-----------|-----|
| Windows操作 | √ | √ | 间接 |
| Linux程序 | × | × | √ |
| Bash支持 | × | × | √ |

---

## 七、总结

- Bash：自动化脚本语言
- CMD：文本命令工具（旧）
- PowerShell：对象命令工具（现代）
- WSL：Linux环境

---

## 八、建议（你的场景）

推荐：
- 本地：WSL + Bash
- 服务器：Linux + Bash
- Windows管理：PowerShell
