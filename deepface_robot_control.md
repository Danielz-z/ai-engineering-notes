# DeepFace -> Astribot Realtime Emotion Control (Final Project README)

## 1. Project Goal
Use local DeepFace emotion recognition to drive Astribot chassis movement in realtime.

Final behavior:
- `happy` -> forward
- `neutral` -> stop/brake
- `surprise` -> backward

## 2. What We Built (Final Architecture)
We migrated from keyboard injection to direct motion API control.

Final chain:
1. `realtime_emotion.py` on Windows  
2. `robot_receiver.py` on Windows (`127.0.0.1:8000`)  
3. `wsl_motion_server.py` in WSL (`0.0.0.0:9001`)  
4. Astribot SDK direct control (`set_joints_position`) in control loop

Why this is final:
- no pseudo-tty keyboard relay dependency
- fewer fragile points
- cleaner safety strategy with auto-timeout idle

## 3. Key Lessons and Changes We Made
Historical path:
1. Started with keyboard relay approach (`w/s` via WSL)  
2. Hit tty/input limitations and environment issues (`conda`, user context, tmux)  
3. Confirmed direct manual movement works in WSL  
4. Rewrote to direct motion service (`wsl_motion_server.py`)  
5. Added safety timeout auto-brake  
6. Added startup scripts and standard operating sequence

Important issues we resolved:
- PowerShell execution policy blocking `.ps1`
- WSL distro/user mismatch
- `conda` not found in non-interactive shell
- Port conflict (`9001 already in use`)
- Robot control rights occupied by another process
- TTY key injection instability

## 4. Current File Roles
- `realtime_emotion.py`  
  DeepFace realtime emotion inference and command posting.
- `robot_receiver.py`  
  Windows relay API from emotion command to WSL motion server.
- `wsl_motion_server.py`  
  WSL motion service, direct Astribot chassis control loop.
- `run_realtime_emotion.ps1`  
  Start emotion inference with recommended parameters.
- `run_receiver.ps1`  
  Start relay receiver with endpoint defaults.
- `run_motion_server.sh`  
  Start WSL motion server using your preferred command order.

## 5. Command Mapping
At receiver/motion API level:
- `start`: forward (`+x`)
- `idle`: zero velocity
- `stop`: backward (`-x`)

At emotion level:
- `happy` -> `start`
- `neutral` -> `idle`
- `surprise` -> `stop`

## 6. Standard Startup Sequence (Production Use)
Use three terminals.

### Pre-check in WSL (important)
Before starting services, verify `/etc/hosts` contains:

```text
<robot-x86-ip>     x86     # robot hostname
<robot-orin-ip>    orin    # robot compute unit hostname
```

In your environment, WSL may refresh `/etc/hosts` after restart, so re-add these lines when needed.

### Terminal A (WSL): Start motion server
```bash
bash /mnt/c/Users/<windows-user>/deepface_local/run_motion_server.sh
```

`run_motion_server.sh` currently does:
1. `su <robot-user>`
2. `conda activate astribot`
3. `source env.sh`
4. run `python examples/wsl_motion_server.py ...`

### Terminal B (Windows): Start receiver
```powershell
cd <project-root>
.\run_receiver.ps1
```

### Terminal C (Windows): Start emotion pipeline
```powershell
cd <project-root>
.\run_realtime_emotion.ps1
```

Stop emotion window with `q`.

## 7. Pre-Flight Manual Test (Must Run)
Before face control, verify control chain manually:

```powershell
Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/robot/control -ContentType "application/json" -Body '{"command":"start","emotion":"happy","score":0.99}'
Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/robot/control -ContentType "application/json" -Body '{"command":"idle","emotion":"neutral","score":0.99}'
```

Move then auto-stop quick test:

```powershell
Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/robot/control -ContentType "application/json" -Body '{"command":"start","emotion":"happy","score":0.99}'; Start-Sleep -Milliseconds 400; Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/robot/control -ContentType "application/json" -Body '{"command":"idle","emotion":"neutral","score":0.99}'
```

## 8. Safety Strategy
Implemented safety:
- command timeout in motion server (`--command_timeout`, default `0.6`)
- auto-idle if no fresh command in timeout window
- emotion-side neutral is easier/faster to trigger stop

Manual emergency brake:

```powershell
Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/robot/control -ContentType "application/json" -Body '{"command":"idle","emotion":"neutral","score":1.0}'
```

## 9. Recommended Runtime Parameters
Emotion side recommended start values:
- `EMOTION_SCORE_THRESHOLD=0.70`
- `EMOTION_STREAK_REQUIRED=2`
- `COMMAND_COOLDOWN_SECONDS=0.15`
- `IDLE_STREAK_REQUIRED=1`
- `IDLE_COOLDOWN_SECONDS=0.12`

Motion side recommended start values:
- `--freq 200`
- `--linear_speed 0.12`
- `--angular_speed 0.35`
- `--command_timeout 0.6`
- `--mode local`

## 10. Troubleshooting Cookbook
### A) `Port 9001 is in use`
```bash
lsof -i :9001
kill -9 <pid>
```
Note: do not include angle brackets in actual command.

### B) `You don't have control rights of the robot`
Another control process is active. Kill old keyboard/motion scripts first.

### C) `conda: command not found`
Ensure conda init script is sourced for the target user/session.

### D) Robot hostname cannot be resolved after WSL reboot
Re-check `/etc/hosts` and add:
```text
<robot-x86-ip>     x86
<robot-orin-ip>    orin
```

### E) WSL service seems alive but robot not moving
Check manual test in section 7 first. If manual fails, issue is not emotion; fix motion service/control rights first.

### F) Windows cannot run `.ps1`
Temporary bypass in current shell:
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

## 11. Current Status
This project is now in a stable state:
- manual API motion works
- emotion pipeline integrated
- safety timeout and quick brake are in place
- startup scripts are available

Next optional enhancements:
- add `left/right/turn` emotion or gesture mappings
- add command audit logging file
- add one-click stop script (`stop_robot.ps1`)
