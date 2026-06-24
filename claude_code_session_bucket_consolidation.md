# Claude Code Session Bucket Consolidation Across PowerShell cwds

## 1. Problem

A single Claude Code conversation is split across multiple project buckets under `~/.claude/projects/`, because the active PowerShell window's current working directory (cwd) determined which bucket the session JSONL was written to. Same UUID, different buckets — the session history is fragmented and the `claude --resume` UI can no longer show the full timeline.

## 2. Environment

- OS: Windows 11
- Tool: Claude Code CLI on Windows (PowerShell)
- Context:
  - Normal-user PowerShell defaults cwd to `C:\Users\<user>`
  - Administrator PowerShell defaults cwd to `C:\WINDOWS\system32`
  - Starting `claude` from either inherits the cwd, so JSONL lands in different `C--<encoded-cwd>` buckets

## 3. Cause

Claude Code keys its per-project session storage by the absolute cwd at session start:

```
~/.claude/projects/C--Users-asus-claude-main/    <- session started from claude-main
~/.claude/projects/C--WINDOWS-system32/          <- session started from admin PS
~/.claude/projects/C--Users-asus/                <- session started from user home
```

If the same session UUID continues writing after a cwd change (or is restarted from a different cwd), its lines end up in two physically separate JSONL files with the same UUID, in two different buckets.

## 4. Solution

### Step 1 — Back up affected buckets before touching anything

```powershell
Compress-Archive ~\.claude\projects\C--Users-asus-claude-main `
  ~\claude-projects-Users-asus-backup-20260624-150551.zip

Compress-Archive ~\.claude\projects\C--WINDOWS-system32 `
  ~\claude-WINDOWS-system32-backup-20260624-161648.zip
```

### Step 2 — Merge same-UUID JSONL halves with `Add-Content`

When the same UUID exists in two buckets (a single session split mid-stream), append the orphan half to the canonical one:

```powershell
Get-Content "$env:USERPROFILE\.claude\projects\C--WINDOWS-system32\b4160a1f-b3b5-4408-aa9f-299caba1cc3f.jsonl" |
  Add-Content "$env:USERPROFILE\.claude\projects\C--Users-asus-claude-main\b4160a1f-b3b5-4408-aa9f-299caba1cc3f.jsonl"

Remove-Item -Recurse "$env:USERPROFILE\.claude\projects\C--WINDOWS-system32"
```

### Step 3 — Move orphan JSONLs from incidental buckets

```bash
mv ~/.claude/projects/C--Users-asus/*.jsonl \
   ~/.claude/projects/C--Users-asus-claude-main/
rmdir ~/.claude/projects/C--Users-asus
```

### Step 4 — Prevent recurrence with a PowerShell `$PROFILE` auto-cd

Add to `$PROFILE` (e.g. `C:\Users\<user>\Documents\PowerShell\Microsoft.PowerShell_profile.ps1`):

```powershell
if (-not $env:CLAUDE_AUTO_CD) {
    $env:CLAUDE_AUTO_CD = '1'
    Set-Location 'C:\Users\asus\claude-main'
}
```

This pins every new PowerShell (including admin) to the canonical project directory before `claude` reads its cwd, so all future sessions write to one bucket.

## 5. Verification

Confirm the merged JSONL contains both original cwds and a continuous timestamp range:

```bash
F=~/.claude/projects/C--Users-asus-claude-main/b4160a1f-b3b5-4408-aa9f-299caba1cc3f.jsonl
grep -oE '"cwd":"[^"]*"' "$F" | sort -u
jq -r 'select(.timestamp) | .timestamp' "$F" | sort -u | head -1
jq -r 'select(.timestamp) | .timestamp' "$F" | sort -u | tail -1
```

Expected: at least two distinct `cwd` values, and timestamps spanning the full session window.

Confirm bucket layout after cleanup:

```powershell
Get-ChildItem ~\.claude\projects\ -Directory | Select-Object Name
```

Only the canonical `C--Users-asus-claude-main` (and unrelated project buckets) should remain.

## 6. Common Issues

- **`Add-Content` appends physically, not chronologically.** The merged JSONL is valid line-by-line but lines are not in `timestamp` order. Anything that replays the session should sort by the `timestamp` field, not by file offset.
- **Same-UUID collision is not a bug.** It means one session continued writing across a cwd change. Concatenation is the correct fix.
- **Admin PowerShell ignores the Windows Terminal default-terminal delegation.** It will still launch as a classic console window and still default cwd to `system32`; the `$PROFILE` auto-cd is what actually fixes the bucket split for admin sessions.
- **Encoding.** PowerShell 5.1's `Add-Content` defaults to UTF-8 without BOM on `.jsonl`, which is what Claude Code expects. If you write the profile or destination file with `Out-File` instead, force `-Encoding utf8` to avoid UTF-16.

## 7. Summary

Claude Code keys session storage by absolute cwd, so a single conversation can fragment across project buckets whenever the launching shell's cwd changes. Back up, concatenate the same-UUID halves with `Add-Content`, delete the empty buckets, then pin all future PowerShell sessions to one directory via `$PROFILE` to prevent the split from recurring.
