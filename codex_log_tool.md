# Codex JSONL to Experiment Log and README Tool

## Overview

This tool converts local Codex `.jsonl` session logs into structured Markdown documents:

```text
/output/
  experiment_log.md
  README.md
```

It is useful for:

- Research experiment records, such as EEG, machine learning, or system development.
- Prompt management and reuse.
- Automatic project README generation.
- Tracking and reproducing development processes.

## Core Idea

Codex stores conversation history as JSONL session logs. This tool transforms those raw logs into readable Markdown.

```text
Raw JSONL logs -> Structured Markdown documents
```

| Input | Output |
| --- | --- |
| Raw conversation logs | Experiment log |
| Multi-turn interaction | Project summary |
| Unstructured session data | Readable documentation |

## Input Data

Default Codex session path:

```text
~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl
```

Data format:

- JSONL, one JSON object per line.
- User messages.
- Assistant messages.
- Tool calls, depending on the Codex version and log format.

## Output Format

### `experiment_log.md`

```markdown
# Experiment Log

## User
...

## Codex
...
```

### `README.md`

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

## Script

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
            except json.JSONDecodeError:
                continue

            if data.get("type") == "message":
                role = data.get("role")
                content = ""

                if isinstance(data.get("content"), list):
                    for item in data["content"]:
                        if "text" in item:
                            content += item["text"]

                messages.append({
                    "role": role,
                    "content": content.strip(),
                })

    return messages


def build_experiment_log(messages):
    markdown = "# Experiment Log\n\n"

    for message in messages:
        if message["role"] == "user":
            markdown += f"## User\n{message['content']}\n\n"
        elif message["role"] == "assistant":
            markdown += f"## Codex\n{message['content']}\n\n"

    return markdown


def summarize(messages):
    user_inputs = [m["content"] for m in messages if m["role"] == "user"]
    assistant_outputs = [m["content"] for m in messages if m["role"] == "assistant"]

    summary = f"""
# Project Summary

## Objectives
{user_inputs[0] if user_inputs else "Unknown"}

## Methods
Use Codex for code generation and iterative development.

## Results
Generated {len(assistant_outputs)} assistant responses.

## Next Steps
- Improve the module structure.
- Add test cases.
"""

    return summary


def main():
    file_path = find_latest_jsonl()
    if not file_path:
        print("No session found.")
        return

    print(f"Using: {file_path}")

    messages = parse_jsonl(file_path)

    experiment_log = build_experiment_log(messages)
    readme = summarize(messages)

    with open(f"{OUTPUT_DIR}/experiment_log.md", "w", encoding="utf-8") as f:
        f.write(experiment_log)

    with open(f"{OUTPUT_DIR}/README.md", "w", encoding="utf-8") as f:
        f.write(readme)

    print("Done. Output written to ./output/")


if __name__ == "__main__":
    main()
```

## Usage

```bash
python codex_to_log.py
```

After running the script, the output directory contains:

```text
/output/
  experiment_log.md
  README.md
```

## Recommended Project Structure

```text
project/
  logs/
    raw_jsonl/
    parsed_md/
  scripts/
    codex_to_log.py
  output/
```

## Limitations

### JSONL Structure May Change

Different Codex versions may store session fields differently. The script includes basic error handling, but it is not guaranteed to support every future format.

### Tool Call Details May Be Missing

Some tool call information may not appear inside `message` entries.

### Long Conversations May Be Truncated

If the session log was truncated by the system, this tool cannot recover the missing context.

## Use Cases

| Use case | Suitable |
| --- | --- |
| EEG experiment records | Yes |
| Model development logs | Yes |
| Prompt management | Yes |
| Project README generation | Yes |
| Early-stage paper material organization | Yes |

## Value

- Automates experiment record generation.
- Improves project reproducibility.
- Reduces manual documentation work.
- Makes development traces easier to review.

## Future Improvements

- Use an LLM to generate richer summaries.
- Merge and analyze multiple sessions.
- Extract model parameters automatically.
- Generate paper-style method and experiment sections.

## Conclusion

Codex records the development conversation as JSONL. This tool converts those logs into structured Markdown:

```text
Codex -> JSONL -> Experiment log + README
```
