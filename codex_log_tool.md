# Codex 对话 → 实验日志 + README 自动生成工具（V1）

---

## 一、功能概述

该工具用于将 Codex 本地 `.jsonl` 会话日志自动转换为结构化 Markdown 文档：

```
/output/
  experiment_log.md     # 实验过程记录
  README.md             # 项目总结
```

适用于：

* 科研实验记录（EEG / 机器学习 / 系统开发）
* Prompt 管理与复用
* 项目 README 自动生成
* 开发过程追踪与复现

---

## 二、核心原理

Codex 默认行为：

```
Codex = 记录对话日志（JSONL）
```

本工具的作用：

```
JSONL（原始日志） → Markdown（结构化文档）
```

即：

| 输入     | 输出   |
| ------ | ---- |
| 原始对话日志 | 实验记录 |
| 多轮交互   | 项目总结 |
| 非结构化数据 | 可读文档 |

---

## 三、输入数据

默认路径：

```
~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl
```

数据格式：JSONL（每行一个 JSON）

包含内容：

* 用户输入（user）
* 模型输出（assistant）
* 工具调用（部分版本）

---

## 四、输出格式

### 1. experiment_log.md

```markdown
# 实验日志

## 用户
xxx

## Codex
xxx

## 用户
xxx

## Codex
xxx
```

---

### 2. README.md

```markdown
# Project Summary

## Objectives
...

## Methods
...

## Results
...

## Next Steps
...
```

---

## 五、脚本代码（可直接运行）

```python
import json
import os
from pathlib import Path

INPUT_DIR = os.path.expanduser("~/.codex/sessions")
OUTPUT_DIR = "./output"

os.makedirs(OUTPUT_DIR, exist_ok=True)


def find_latest_jsonl():
    files = list(Path(INPUT_DIR).rglob("rollout-*.jsonl"))
    files.sort(key=lambda x: x.stat().st_mtime, reverse=True)
    return files[0] if files else None


def parse_jsonl(file_path):
    messages = []

    with open(file_path, "r", encoding="utf-8") as f:
        for line in f:
            try:
                data = json.loads(line)
            except:
                continue

            if data.get("type") == "message":
                role = data.get("role")
                content = ""

                if isinstance(data.get("content"), list):
                    for c in data["content"]:
                        if "text" in c:
                            content += c["text"]

                messages.append({
                    "role": role,
                    "content": content.strip()
                })

    return messages


def build_experiment_log(messages):
    md = "# 实验日志\n\n"

    for msg in messages:
        if msg["role"] == "user":
            md += f"## 用户\n{msg['content']}\n\n"
        elif msg["role"] == "assistant":
            md += f"## Codex\n{msg['content']}\n\n"

    return md


def summarize(messages):
    user_inputs = [m["content"] for m in messages if m["role"] == "user"]
    assistant_outputs = [m["content"] for m in messages if m["role"] == "assistant"]

    summary = f"""
# Project Summary

## Objectives
{user_inputs[0] if user_inputs else "未知"}

## Methods
使用 Codex 进行代码生成与迭代开发

## Results
生成了 {len(assistant_outputs)} 条模型输出

## Next Steps
- 优化模块结构
- 增加测试用例
"""

    return summary


def main():
    file_path = find_latest_jsonl()
    if not file_path:
        print("No session found.")
        return

    print(f"Using: {file_path}")

    messages = parse_jsonl(file_path)

    exp_log = build_experiment_log(messages)
    readme = summarize(messages)

    with open(f"{OUTPUT_DIR}/experiment_log.md", "w", encoding="utf-8") as f:
        f.write(exp_log)

    with open(f"{OUTPUT_DIR}/README.md", "w", encoding="utf-8") as f:
        f.write(readme)

    print("Done. Output in ./output/")


if __name__ == "__main__":
    main()
```

---

## 六、使用方法

```bash
python codex_to_log.py
```

执行后自动生成：

```
/output/
  experiment_log.md
  README.md
```

---

## 七、推荐项目结构

```
project/
  /logs/
    raw_jsonl/
    parsed_md/

  /scripts/
    codex_to_log.py

  /output/
```

---

## 八、限制与注意事项

### 1. JSONL 结构可能变化

不同版本 Codex 日志字段可能不同
→ 已做基础容错，但不保证完全兼容

---

### 2. 工具调用可能缺失

部分 tool_call 信息未包含在 message 中

---

### 3. 长对话可能被截断

属于系统限制，无法完全恢复完整上下文

---

## 九、适用场景

| 场景           | 是否适用 |
| ------------ | ---- |
| EEG 实验记录     | ✔    |
| 模型开发日志       | ✔    |
| Prompt 管理    | ✔    |
| 项目 README 生成 | ✔    |
| 论文素材整理（初级）   | ✔    |

---

## 十、核心价值

* 自动化实验记录
* 提高项目复现能力
* 减少手动整理成本
* 提高开发效率

---

## 十一、后续升级方向（V2）

建议后续扩展：

* 接入 GPT 自动生成总结
* 多 session 合并分析
* 自动提取模型参数
* 自动生成论文 Method / Experiment

---

## 十二、结论

```
Codex 只负责记录日志（JSONL）
该工具负责将日志转化为结构化文档（Markdown）
```

实现效果：

```
Codex → JSONL → 实验日志 + README（自动生成）
```
