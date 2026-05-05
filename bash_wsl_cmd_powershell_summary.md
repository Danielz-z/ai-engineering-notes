# Bash, WSL, CMD, and PowerShell Summary

## What Is Bash?

Bash, short for Bourne Again Shell, is both a command interpreter and a scripting language. It is commonly used to run system commands and automate workflows on Linux and Unix-like systems.

Core capabilities:

- Variables, conditions, loops, and functions.
- Shell scripting for automation.
- Pipeline-style command composition.

## CMD vs PowerShell

### CMD

CMD is the older Windows command-line shell.

- Output is plain text.
- Commands are simple but not strongly structured.
- It is useful for legacy Windows commands and simple scripts.

Example:

```cmd
dir | find "txt"
```

### PowerShell

PowerShell is the modern Windows shell and scripting environment.

- Output is object-based rather than plain text.
- Commands can operate directly on object properties.
- It is better suited for Windows administration and structured automation.

Example:

```powershell
Get-ChildItem | Where-Object { $_.Length -gt 1MB }
```

## What Is WSL?

WSL, or Windows Subsystem for Linux, lets Windows run a Linux environment.

Basic relationship:

```text
Windows -> WSL -> Linux -> Bash
```

WSL2 uses a real Linux kernel, so it behaves much more like a normal Linux system and is usually the recommended version.

## Relationship Between the Tools

```text
Windows
|-- CMD
|-- PowerShell
`-- WSL
    `-- Bash
```

## Common Interactions

Enter WSL:

```powershell
wsl
```

Run a Linux command from Windows:

```powershell
wsl ls
```

Open the current Linux directory in Windows Explorer:

```bash
explorer.exe .
```

## Capability Comparison

| Capability | CMD | PowerShell | WSL |
| --- | --- | --- | --- |
| Windows operations | Yes | Yes | Indirect |
| Linux programs | No | No | Yes |
| Bash support | No | No | Yes |

## Summary

- Bash is best for Linux scripting and automation.
- CMD is an older text-based Windows command shell.
- PowerShell is the modern object-based Windows automation shell.
- WSL is the best bridge when Linux tools are needed on Windows.

## Practical Recommendation

- Local development: WSL + Bash.
- Remote servers: Linux + Bash.
- Windows administration: PowerShell.
