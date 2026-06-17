# Fine-tuning π0.5 on Custom Aloha Pick Tasks

This document covers the complete workflow for fine-tuning the π0.5 (pi0.5) model on custom Aloha bimanual robot grasping tasks using the openpi framework, including configuration, common pitfalls, and fixes applied.

---

## Environment Overview

| Item | Path |
|------|------|
| Codebase | `$PROJECT_ROOT` |
| Training data | `$HF_LEROBOT_HOME` |
| Base model | `$OPENPI_DATA_HOME/pi05_base/` |
| Checkpoints | `$PROJECT_ROOT/checkpoints/` |
| Training logs | `$PROJECT_ROOT/logs/` |
| wandb offline runs | `$PROJECT_ROOT/wandb/` |

**Required environment variables**:

```bash
export PROJECT_ROOT=/path/to/openpi
export HF_LEROBOT_HOME=/path/to/lerobot/datasets
export OPENPI_DATA_HOME=/path/to/openpi/model/assets
```

---

## Datasets

LeRobot format, stored under `$HF_LEROBOT_HOME`:

| Task | `repo_id` | Episodes | Frames |
|------|-----------|----------|--------|
| Pick book | `pick_book_joint` | 31 | 6457 |
| Pick cup | `pick_bhc_joint` | 31 | 6655 |
| Pick can | `pick_cola_joint` | 30 | 7089 |

Format: parquet (`data/chunk-000/episode_XXXXXX.parquet`), 30 fps, 14-DOF actions (dual-arm Aloha, 7 joints each).

**Camera views** (3 streams):
- `observation.images.cam_high`
- `observation.images.cam_left_wrist`
- `observation.images.cam_right_wrist`

---

## Model Architecture

**Base model**: π0.5, loaded from `$OPENPI_DATA_HOME/pi05_base/params`

**Fine-tuning setup** (LoRA):
- Language backbone: PaliGemma gemma-2B (LoRA)
- Action expert: Gemma 300M (LoRA)
- Non-LoRA parameters are frozen via `freeze_filter`
- `ema_decay=None` (EMA disabled)

**Norm stats**: loaded from `$OPENPI_DATA_HOME/pi05_base/assets/trossen/norm_stats.json`, covering mean/std/quantiles for actions and state.

---

## Training Config (`config.py`)

Config name: `pi05_aloha_pick_book`, defined in `src/openpi/training/config.py`:

```python
TrainConfig(
    name="pi05_aloha_pick_book",
    model=Pi0Config(
        pi05=True,
        paligemma_variant="gemma_2b_lora",
        action_expert_variant="gemma_300m_lora",
    ),
    data=LeRobotAlohaDataConfig(
        repo_id="pick_book_joint",
        assets=AssetsConfig(
            assets_dir="$OPENPI_DATA_HOME/pi05_base/assets",
            asset_id="trossen",
        ),
        default_prompt="pick up the book",
        repack_transforms=Group(inputs=[
            RepackTransform({...}),
            SliceActions(n=14),    # keep first 14 action dims (drop mobile base)
            NormalizeGripper(),    # normalize gripper from meters to [0, 1]
        ]),
    ),
    weight_loader=CheckpointWeightLoader("$OPENPI_DATA_HOME/pi05_base/params"),
    freeze_filter=...,
    ema_decay=None,
    num_train_steps=20_000,
    batch_size=32,
)
```

**Data transform pipeline**:
1. `RepackTransform` — remap LeRobot field names to model-expected keys
2. `SliceActions(n=14)` — keep first 14 action dims (dual-arm, no mobile base)
3. `NormalizeGripper(open_val=0.1, close_val=0.036)` — normalize gripper dims (6, 13) from meters to [0, 1]
4. `AlohaInputs` + `DeltaActions` — convert to delta action representation
5. `ResizeImages(224×224)` + `TokenizePrompt` + `PadStatesAndActions`

---

## Training Commands

```bash
cd $PROJECT_ROOT
mkdir -p logs

nohup bash -c '
CUDA_VISIBLE_DEVICES=1,2 \
HF_LEROBOT_HOME=$HF_LEROBOT_HOME \
OPENPI_DATA_HOME=$OPENPI_DATA_HOME \
WANDB_MODE=disabled \
uv run scripts/train.py pi05_aloha_pick_book \
  --exp_name pick_book_v1 \
  --checkpoint-base-dir $PROJECT_ROOT/checkpoints
' > $PROJECT_ROOT/logs/pick_book_v1.log 2>&1 &
echo "PID: $!"
```

**Key environment variables**:

| Variable | Example | Purpose |
|----------|---------|---------|
| `CUDA_VISIBLE_DEVICES` | `1,2` | **Must specify.** JAX claims all visible GPUs by default — if other GPUs are nearly full, NCCL will OOM and crash. |
| `HF_LEROBOT_HOME` | `/data/lerobot` | Root directory for LeRobot datasets |
| `OPENPI_DATA_HOME` | `/data/openpi` | Root directory for base model weights and norm stats |
| `WANDB_MODE` | `disabled` | Disable wandb when no API key is available; sync manually after training |

**Overwrite an existing checkpoint directory**:
```bash
uv run scripts/train.py pi05_aloha_pick_book --exp_name pick_book_v1 \
  --checkpoint-base-dir $PROJECT_ROOT/checkpoints --overwrite
```

**Resume from checkpoint**:
```bash
uv run scripts/train.py pi05_aloha_pick_book --exp_name pick_book_v1 \
  --checkpoint-base-dir $PROJECT_ROOT/checkpoints --resume
```

---

## Checkpoint Save Policy

- A permanent checkpoint is kept every **5000 steps** (`keep_period=5000`)
- The **latest checkpoint** is always kept (`max_to_keep=1`)
- Path: `checkpoints/pi05_aloha_pick_book/{exp_name}/{step}/`

---

## Training Results

### pick_book_v1 (completed)

- **Total steps**: 20,000
- **Batch size**: 32
- **Speed**: ~1.3 s/step on 2× H800
- **Loss**: initial ~0.0027 → final ~0.0006
- **grad_norm**: ~0.050 → ~0.028

### pick_book_v2 (in progress)

- Same config, second run for verification
- GPUs: 2× H800, `CUDA_VISIBLE_DEVICES=1,2`
- Speed: ~1.3 s/step, ~7.5 hours total

---

## GPU Management

### JAX claims all GPUs by default

JAX automatically detects and uses every visible GPU for multi-device parallelism. On a shared server where other GPUs are nearly full, this triggers an NCCL OOM crash:

```
NCCL operation ncclGroupEnd() failed: unhandled cuda error 'out of memory'
```

**Fix**: always set `CUDA_VISIBLE_DEVICES` to restrict to specific free GPUs.

```bash
# Check GPU availability before starting
nvidia-smi --query-gpu=index,memory.used,memory.free,utilization.gpu --format=csv
```

### Zombie process after training completes

JAX's background threads (async checkpoint writers, etc.) sometimes prevent the process from exiting after training finishes. The process stays alive in `S (sleeping)` state with hundreds of threads, holding GPU file descriptors but doing no useful work.

**Detection**:
```bash
# Check process state and thread count
cat /proc/<PID>/status | grep -E "State|Threads"

# Confirm training is actually done (last checkpoint == final step)
ls checkpoints/pi05_aloha_pick_book/pick_book_v1/
```

**Fix**: verify training is complete, then kill the process:
```bash
kill <PID>
```

### Killing the `uv` wrapper does not kill the child Python process

```bash
# Wrong: only kills the uv wrapper; Python subprocess survives
kill <uv-wrapper-pid>

# Correct: find and kill the actual Python process
ps aux | grep train.py | grep -v grep
kill <python-pid>
```

---

## wandb: Offline Training and Manual Sync

### Disable wandb during training

Set `WANDB_MODE=disabled` in the training command. Offline run data is saved to `wandb/offline-run-*/`.

### Sync after training

```bash
# Pass API key via environment variable (no interactive login needed)
WANDB_API_KEY=<your-key> uv run wandb sync --sync-all

# Sync a single run
WANDB_API_KEY=<your-key> uv run wandb sync wandb/offline-run-20260615_035408-xxxxx
```

> **Note**: Older versions of the wandb SDK (included in this project's `.venv`) do not support the new `wandb_v1_...` key format via `wandb login`. Pass the key through `WANDB_API_KEY` instead.

---

## Bug Fix: `NormalizeGripper` incompatible with PyTorch Tensors

**File**: `src/openpi/training/config.py`

**Problem**: `NormalizeGripper` is applied inside PyTorch DataLoader worker processes as part of `repack_transforms`. At that point `data["actions"]` is a `torch.Tensor`, but the original code called `.copy()` — a NumPy-only method.

**Error**:
```
AttributeError: 'Tensor' object has no attribute 'copy'
```

**Fix**:
```python
# Before
actions = data["actions"].copy()

# After — works for both Tensor and ndarray
actions = data["actions"].clone() if hasattr(data["actions"], "clone") else data["actions"].copy()
```

---

## Monitoring Training

```bash
# Live log tail
tail -f $PROJECT_ROOT/logs/pick_book_v1.log

# Step/loss only
grep "^Step" $PROJECT_ROOT/logs/pick_book_v1.log | tail -20

# Progress bar with ETA
grep "Progress on" $PROJECT_ROOT/logs/pick_book_v1.log | tail -5
```

Progress bar format:
```
Progress on: 4.90kit/20.0kit rate:1.3s/it remaining:5:28:08 elapsed:1:43:54
```

---

## Directory Structure

```
$PROJECT_ROOT/
├── scripts/
│   └── train.py                     # training entry point
├── src/openpi/
│   ├── training/
│   │   ├── config.py                # TrainConfig definitions + custom transforms
│   │   ├── data_loader.py           # DataLoader (applies repack_transforms in workers)
│   │   └── checkpoints.py           # checkpoint save/restore logic
│   └── policies/
│       └── policy_config.py         # inference-time model loading
├── checkpoints/
│   └── pi05_aloha_pick_book/
│       ├── pick_book_v1/            # completed (20 000 steps)
│       └── pick_book_v2/            # in progress
├── logs/
│   ├── pick_book_v1.log
│   └── pick_book_v2.log
└── wandb/                           # offline run data

$HF_LEROBOT_HOME/
├── pick_book_joint/
├── pick_bhc_joint/
└── pick_cola_joint/

$OPENPI_DATA_HOME/
└── pi05_base/
    ├── params/                      # base model weights
    └── assets/trossen/
        └── norm_stats.json          # normalization statistics
```
