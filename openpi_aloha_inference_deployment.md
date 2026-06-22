# OpenPI Inference Deployment on AgileX ALOHA Robot

## 1. Problem

Deploy OpenPI policy serving and run dual-arm Piper inference on an AgileX ALOHA platform, covering hardware bring-up, policy server launch, ROS client startup, prompt switching, and failed-grasp recovery for the left gripper.

## 2. Environment

- OS: Ubuntu with ROS Noetic
- Tool: OpenPI, ROS Noetic, Python virtualenv, ROS topics on Piper arms and Astra cameras
- Context: AgileX ALOHA platform with dual Piper arms, three Astra RGB cameras, Tracer mobile base. All `<dev>` placeholders in paths are sanitized from the original developer account.

## 3. Cause

Inference requires three coordinated services running in separate terminals: hardware drivers, OpenPI policy server, and ROS client. In addition, the trained checkpoints use gripper normalization stats `q01 = 0.0` (fully closed) and `q99 = 1.0` (fully open), which is the opposite direction of the original client's `> 0.7` close threshold, so gripper logic and failed-grasp recovery had to be rewritten.

## 4. Solution

### 4.1 Machine Layout

- Robot user account: `agilex`
- OpenPI source: `/home/agilex/<dev>/vla/openpi`
- Working directory: `/home/agilex/<dev>/zz`
- Client code: `/home/agilex/<dev>/zz/openpi_infer`
- Python environment: `/home/<dev>/vla/openpi/.venv`
- Hardware startup script: `/home/agilex/<dev>/start_infer_base.sh`

### 4.2 Checkpoints

```bash
/home/agilex/<dev>/vla/openpi/checkpoints/pick_book_v1/19999
/home/agilex/<dev>/vla/openpi/checkpoints/pick_bhc_v1/19999
/home/agilex/<dev>/vla/openpi/checkpoints/pick_cola_v1/19999
```

Each checkpoint contains assets such as:

```bash
/home/agilex/<dev>/vla/openpi/checkpoints/pick_book_v1/19999/assets/trossen
```

### 4.3 Added OpenPI Configs

Three configs were added to `/home/agilex/<dev>/vla/openpi/src/openpi/training/config.py`:

```text
pi05_aloha_pick_book
pi05_aloha_pick_bhc
pi05_aloha_pick_cola
```

Prompts:

```text
pick up the book
pick up the iced tea
pick up the cola
```

### 4.4 Policy Service Scripts

Stored in `/home/agilex/<dev>/zz`:

```bash
bash /home/agilex/<dev>/zz/service_pick_book.bash
bash /home/agilex/<dev>/zz/service_pick_bhc.bash
bash /home/agilex/<dev>/zz/service_pick_cola.bash
```

Each script uses GPU `0`, sets `XLA_PYTHON_CLIENT_MEM_FRACTION=0.9` and `PYTHONPATH=/home/agilex/<dev>/vla/openpi/src`, then runs `serve_policy.py` with the `policy:checkpoint` subcommand. Example:

```bash
/home/<dev>/vla/openpi/.venv/bin/python \
  /home/agilex/<dev>/vla/openpi/scripts/serve_policy.py \
  policy:checkpoint \
  --policy.config pi05_aloha_pick_cola \
  --policy.dir /home/agilex/<dev>/vla/openpi/checkpoints/pick_cola_v1/19999
```

### 4.5 Startup Procedure (Three Terminals)

**Terminal 1 — Start Hardware**

```bash
cd /home/agilex/<dev>
bash /home/agilex/<dev>/start_infer_base.sh
```

This starts Astra cameras, CAN configuration, Piper arm nodes, and the Tracer base node. Keep this terminal open.

**Terminal 2 — Start Policy Server**

Stop any previous server first:

```bash
pkill -f serve_policy.py
```

Then start the desired task:

```bash
cd /home/agilex/<dev>/zz
bash /home/agilex/<dev>/zz/service_pick_book.bash
# or
bash /home/agilex/<dev>/zz/service_pick_bhc.bash
# or
bash /home/agilex/<dev>/zz/service_pick_cola.bash
```

The service is ready when it prints:

```text
server listening on 0.0.0.0:8000
```

**Terminal 3 — Start Client**

```bash
cd /home/agilex/<dev>/zz/openpi_infer
export PYTHONPATH=/home/agilex/<dev>/zz/openpi_infer/openpi-client/src:$PYTHONPATH
python inference_openpi.py
```

The robot first moves to an initial pose. After the prompt appears, press `Enter` to start the task.

### 4.6 Prompt Switching

The prompt is set in `/home/agilex/<dev>/zz/openpi_infer/inference_openpi.py`. Search for:

```python
obs["prompt"] = "pick up the cola"
```

Set it to match the active policy:

```python
obs["prompt"] = "pick up the book"
obs["prompt"] = "pick up the iced tea"
obs["prompt"] = "pick up the cola"
```

In-place switch to cola:

```bash
sed -i 's/pick up the iced tea/pick up the cola/g; s/pick up the book/pick up the cola/g' \
  /home/agilex/<dev>/zz/openpi_infer/inference_openpi.py
```

### 4.7 Gripper Semantics

Normalization stats:

```text
q01 = 0.0  # fully closed
q99 = 1.0  # fully open
```

Lower model gripper values therefore mean more closed. The original client used `> 0.7` to close, which is the wrong direction for these checkpoints. Corrected close threshold:

```python
if left_action[-1] < 0.08:
    left_action[-1] = 0.0
else:
    left_action[-1] = 0.068
```

The same threshold is used in `gripper_judge()`.

### 4.8 Measured Left Gripper Feedback

Measured from `/puppet/joint_left.position[6]`:

```text
Fully open:    0.0676 - 0.0679
Empty closed:  0.0046
Holding cola:  0.0397
```

Empty-grasp threshold:

```python
left_empty_grasp_threshold = 0.015
```

Values below `0.015` are treated as empty grasps; values around `0.03 - 0.04` indicate the object is likely held.

### 4.9 Failed-Grasp Recovery Logic

- Latch the left gripper closed once the model requests a close
- After a short delay, read `/puppet/joint_left.position[6]`
- If feedback is below `0.015`, treat it as an empty grasp: open the gripper, stop executing the current action chunk, return to the most recent local safe pose (not the full initial pose), and request a new policy inference
- Allow up to 3 retries

Parameters:

```python
left_empty_grasp_threshold = 0.015
left_grasp_check_after_steps = 5
left_slip_confirm_steps = 3
left_max_retries = 3
```

Expected logs:

```text
[gripper] left latch closed at step=...
[gripper] left feedback after close=...
[gripper] left empty grasp detected; opening and retrying
[gripper] left slip detected; feedback=...; opening and retrying
[gripper] stop current action chunk after failed grasp; opening and returning to recent local safe pose before next inference
```

## 5. Verification

### 5.1 ROS Topic Check

```bash
source /opt/ros/noetic/setup.bash
rostopic list | grep -E 'puppet/joint|master/joint|camera_'
```

Expected topics:

```text
/camera_f/color/image_raw
/camera_l/color/image_raw
/camera_r/color/image_raw
/master/joint_left
/master/joint_right
/puppet/joint_left
/puppet/joint_right
```

Frequency checks:

```bash
rostopic hz /puppet/joint_left
rostopic hz /puppet/joint_right
rostopic hz /camera_f/color/image_raw
```

Typical values: joint topics ~`200 Hz`, front camera ~`30 Hz`.

### 5.2 Manual Gripper Tests

Always keep joints 0–5 unchanged; only change index `6`.

Open left gripper:

```bash
source /opt/ros/noetic/setup.bash
python3 - <<PY
import rospy
from sensor_msgs.msg import JointState
rospy.init_node("open_left_gripper_once", anonymous=True)
msg = rospy.wait_for_message("/puppet/joint_left", JointState, timeout=5)
cmd = JointState()
cmd.header.stamp = rospy.Time.now()
cmd.name = msg.name
cmd.position = list(msg.position)
cmd.position[6] = 0.068
pub = rospy.Publisher("/master/joint_left", JointState, queue_size=1)
rospy.sleep(0.5)
for _ in range(10):
    cmd.header.stamp = rospy.Time.now()
    pub.publish(cmd)
    rospy.sleep(0.05)
PY
```

Close left gripper:

```bash
source /opt/ros/noetic/setup.bash
python3 - <<PY
import rospy
from sensor_msgs.msg import JointState
rospy.init_node("close_left_gripper_once", anonymous=True)
msg = rospy.wait_for_message("/puppet/joint_left", JointState, timeout=5)
cmd = JointState()
cmd.header.stamp = rospy.Time.now()
cmd.name = msg.name
cmd.position = list(msg.position)
cmd.position[6] = 0.0
pub = rospy.Publisher("/master/joint_left", JointState, queue_size=1)
rospy.sleep(0.5)
for _ in range(10):
    cmd.header.stamp = rospy.Time.now()
    pub.publish(cmd)
    rospy.sleep(0.05)
PY
```

Read left gripper feedback:

```bash
source /opt/ros/noetic/setup.bash
python3 - <<PY
import rospy
from sensor_msgs.msg import JointState
rospy.init_node("read_left_gripper_once", anonymous=True)
msg = rospy.wait_for_message("/puppet/joint_left", JointState, timeout=5)
print(msg.position[6])
PY
```

## 6. Common Issues

- **Inverted gripper threshold**: original client used `> 0.7` to close, but checkpoints expect `q01 = 0 → closed`. Use the corrected `< 0.08` close threshold.
- **Empty grasp not detected**: confirm `/puppet/joint_left.position[6]` falls below `0.015` after close; otherwise tune `left_empty_grasp_threshold`.
- **Missing topics**: do not start the client if `puppet/joint*` or `camera_*` topics are absent.
- **Stale policy server**: always `pkill -f serve_policy.py` before launching a new task.
- **Wrong prompt**: the prompt in `inference_openpi.py` must match the launched policy config; use the `sed` command above to switch.

### Stop Commands

```bash
pkill -f inference_openpi.py
pkill -f serve_policy.py
```

Stop hardware by pressing `Ctrl+C` in the terminal running `start_infer_base.sh`.

### Safety Notes

- Keep emergency stop available
- Do not run the client if arm/camera topics are missing
- Stop the client immediately if the arm presses into the table
- Recovery logic does not replace collision detection or force control
- Keep hands away from the gripper during tests

## 7. Summary

OpenPI inference on AgileX ALOHA runs as three coordinated services — hardware bring-up, policy server, and ROS client — with a corrected gripper close threshold (`< 0.08`) and a left-gripper feedback loop (`/puppet/joint_left.position[6] < 0.015 ⇒ empty grasp`) that opens the gripper, returns to the last local safe pose, and re-infers up to 3 times.
