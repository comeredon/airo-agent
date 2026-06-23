---
name: airo-setup
description: "Install, set up, and configure airo-mono packages. Use when: install, setup, getting started, pip install, uv, conda, dependencies, sister repo, airo-drake, airo-planner, airo-models, airo-teleop, airo-ipc, airo-tulip, PyPI, hardware setup, ZED SDK, RealSense, pre-commit, mypy, pytest, versioning, releasing, contributing."
---

# AIRO-Mono Setup & Installation

Source: https://github.com/airo-ugent/airo-mono

## When to Use
- Installing airo-mono packages (from PyPI or source)
- Setting up a development environment
- Understanding hardware-specific dependencies
- Choosing which sister repos to use
- Setting up pre-commit hooks, testing, and type checking
- Troubleshooting dependency issues

## Quick Install from PyPI

```bash
pip install airo-typing airo-spatial-algebra airo-robots airo-camera-toolkit airo-dataset-tools
```

Each package can also be installed individually:
```bash
pip install airo-robots          # includes airo-typing + airo-spatial-algebra
pip install airo-camera-toolkit  # includes all of the above + airo-dataset-tools
```

## Install from Source (Recommended for Development)

### Option A: uv (fastest)

```bash
git clone https://github.com/airo-ugent/airo-mono.git
cd airo-mono
uv venv --python 3.10
# On Windows: .venv\Scripts\activate
# On Linux/macOS: source .venv/bin/activate
uv pip install -e airo-typing/ airo-spatial-algebra/ airo-robots/ airo-camera-toolkit/ airo-dataset-tools/
uv pip install -r dev-requirements.txt
pre-commit install
```

### Option B: Conda

```bash
git clone https://github.com/airo-ugent/airo-mono.git
cd airo-mono
conda env create -f environment.yaml
conda activate airo-mono
```

### Option C: pip

```bash
git clone https://github.com/airo-ugent/airo-mono.git
cd airo-mono
python -m venv .venv
# activate the venv
pip install -e airo-typing/ airo-spatial-algebra/ airo-robots/ airo-camera-toolkit/ airo-dataset-tools/
```

### Option D: Git Submodule

Add airo-mono as a submodule in your own project:
```bash
git submodule add https://github.com/airo-ugent/airo-mono.git
pip install -e airo-mono/airo-typing/ airo-mono/airo-spatial-algebra/  # etc.
```

## Hardware-Specific Setup

### UR Robots (airo-robots)
- `ur-rtde >= 1.5.7` is installed automatically
- The UR robot must be reachable on the network (default RTDE port: 30004)
- No additional SDK required

### ZED Cameras (airo-camera-toolkit)
- Requires **ZED SDK** installed separately from https://www.stereolabs.com/developers
- The `pyzed` Python package version must match the ZED SDK version
- See: `airo-camera-toolkit/airo_camera_toolkit/cameras/zed/installation.md`

### RealSense Cameras (airo-camera-toolkit)
- Requires `pyrealsense2`: `pip install pyrealsense2`
- See: `airo-camera-toolkit/airo_camera_toolkit/cameras/realsense/realsense_installation.md`

### USB Webcams
- No extra dependencies — uses OpenCV (already included)

## Optional Extras

```bash
# Hand-eye calibration (needs airo-robots)
pip install airo-camera-toolkit[hand-eye-calibration]

# Dataset augmentation
pip install airo-dataset-tools[augmentations]

# FiftyOne visualization
pip install airo-dataset-tools[fiftyone]
```

## Key Dependencies

| Dependency | Version | Why |
|-----------|---------|-----|
| Python | ≥ 3.10 | Required |
| numpy | ≥ 2.0 | Array operations everywhere |
| opencv-contrib-python | == 4.10.0.84 | Pinned — ArUco/ChArUco API |
| spatialmath-python | latest | SE3 math backend |
| scipy | latest | Spatial algebra helpers |
| pydantic | > 2.0 | Dataset tool models |
| ur-rtde | ≥ 1.5.7 | UR robot driver |
| rerun-sdk | ≥ 0.23.4 | 3D visualization |
| loguru | latest | Logging |

**Important**: `opencv-contrib-python` is pinned because the ArUco API changed between versions. Do not upgrade without checking compatibility.

## Sister Repositories — When to Use What

| Need | Repository | Install |
|------|-----------|---------|
| Simulate a UR robot without hardware | [airo-drake](https://github.com/airo-ugent/airo-drake) | `pip install airo-drake` |
| Collision-free motion planning | [airo-planner](https://github.com/airo-ugent/airo-planner) | `pip install airo-planner` |
| Robot URDFs and 3D object models | [airo-models](https://github.com/airo-ugent/airo-models) | `pip install airo-models` |
| Teleoperate a robot arm | [airo-teleop](https://github.com/airo-ugent/airo-teleop) | `pip install airo-teleop` |
| Multi-process communication | [airo-ipc](https://github.com/airo-ugent/airo-ipc) | `pip install airo-ipc` |
| KELO mobile robot platform | [airo-tulip](https://github.com/airo-ugent/airo-tulip) | `pip install airo-tulip` |
| Fast analytic IK for UR arms | [ur-analytic-ik](https://github.com/Victorlouisdg/ur-analytic-ik) | `pip install ur-analytic-ik` |
| Synthetic data with Blender | [airo-blender](https://github.com/airo-ugent/airo-blender) | `pip install airo-blender` |

### Hackathon Starter Kit

For a robotics hackathon with a UR arm and a camera:

```bash
pip install airo-robots airo-camera-toolkit airo-planner airo-drake airo-models ur-analytic-ik
```

For a vision-only hackathon (no robot hardware):

```bash
pip install airo-camera-toolkit airo-dataset-tools
pip install airo-dataset-tools[augmentations]  # if training ML models
```

## Development Setup

### Pre-commit Hooks
```bash
pip install -r dev-requirements.txt
pre-commit install
# Run manually:
pre-commit run --all-files
```

Hooks: Black (formatter, 119 chars), isort, autoflake, Flake8.

### Testing
```bash
make pytest airo-robots/        # single package with coverage
pytest airo-camera-toolkit/ -m "not expensive"  # skip hardware tests
```

### Type Checking
```bash
mypy airo-robots/               # per package
```

### Versioning
All packages share one CalVer version: `YYYY.M.PATCH` (e.g., `2026.5.0`). Use `bump-my-version` to update.

## Common Issues

- **`opencv-contrib-python` conflict**: If another package installs `opencv-python` or `opencv-python-headless`, it may overwrite `cv2` and break ArUco detection. Fix: `pip install opencv-contrib-python==4.10.0.84 --force-reinstall`
- **numpy 2.0 incompatibility**: Some older packages don't support numpy ≥ 2.0. Check compatibility before mixing with other libraries.
- **ZED SDK mismatch**: The `pyzed` version must exactly match the installed ZED SDK version.
- **Windows `time.sleep` accuracy**: On Windows, `time.sleep` has ~15ms resolution. Set `sleep_resolution >= 0.015` in `AwaitableAction.wait()`.

## GitHub Source

- README: `https://github.com/airo-ugent/airo-mono/blob/main/README.md`
- Getting started: `https://github.com/airo-ugent/airo-mono/blob/main/docs/getting_started.md`
- Releasing: `https://github.com/airo-ugent/airo-mono/blob/main/docs/releasing.md`
- Environment file: `https://github.com/airo-ugent/airo-mono/blob/main/environment.yaml`
