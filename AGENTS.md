# AIRO-Mono Project Context

This workspace uses the [airo-mono](https://github.com/airo-ugent/airo-mono) Python monorepo for robotic manipulation, developed by the AIRO lab (IDLab, Ghent University — imec).

## Source of Truth

**GitHub**: https://github.com/airo-ugent/airo-mono

When you need to look up source code, interfaces, or implementation details, fetch from this repository. The user may not have the repo cloned locally.

## Package Structure

```
airo-mono/
├── airo-typing/            → Type aliases & conventions (quaternions, matrices, point clouds)
├── airo-spatial-algebra/   → SE3 poses & transforms (wraps spatialmath-python)
├── airo-robots/            → UR robot + Robotiq gripper control via RTDE
├── airo-camera-toolkit/    → RGB(D) cameras, point clouds, calibration, image ops
└── airo-dataset-tools/     → COCO datasets, CVAT conversion, segmentation masks
```

**Dependency graph**: `airo-typing` → `airo-spatial-algebra` → `airo-robots` / `airo-camera-toolkit` → `airo-dataset-tools`

## Key Conventions

- **Quaternions**: scalar-last format `[x, y, z, w]`
- **Units**: metric (meters, radians)
- **Poses**: 4×4 homogeneous matrices (`HomogeneousMatrixType` = `np.ndarray` shape `(4, 4)`)
- **Images**: NumPy arrays — float32 RGB [0,1] by default, or uint8 RGB [0,255] via `_as_int()` methods
- **Point clouds**: `PointCloud` dataclass with `points`, `colors`, `attributes` fields
- **Async hardware**: `AwaitableAction` pattern — always call `.wait(timeout)` before sending the next command
- **Camera capture**: explicit `grab_images()` then `retrieve_*()` — never call retrieve before grab
- **Logging**: use `loguru`, not `print()`
- **Accessors**: Python properties, not `get_x()` methods
- **Return types**: native datatypes / NumPy arrays; dataclasses for structured data

## Python & Dependencies

- Python ≥ 3.10
- numpy ≥ 2.0
- opencv-contrib-python == 4.10.0.84 (pinned — ArUco API)
- spatialmath-python, scipy (spatial algebra)
- pydantic > 2.0 (dataset tools)
- ur-rtde ≥ 1.5.7 (UR robot driver — hardware only)
- rerun-sdk ≥ 0.23.4 (visualization)

## Code Style

- **Black** — 119-character line length
- **isort** — `--profile black --line-length 119`
- **Docstrings** — Google format
- **Type hints** — required on all public functions (`disallow_untyped_defs = True`)

## Sister Repositories

| Repo | Purpose |
|------|---------|
| [airo-drake](https://github.com/airo-ugent/airo-drake) | Simulation & rendering with Drake |
| [airo-planner](https://github.com/airo-ugent/airo-planner) | Motion planning (OMPL / cuRobo) |
| [airo-models](https://github.com/airo-ugent/airo-models) | Robot & object URDFs and 3D models |
| [airo-teleop](https://github.com/airo-ugent/airo-teleop) | Teleoperation of robot arms |
| [airo-ipc](https://github.com/airo-ugent/airo-ipc) | Inter-process communication |
| [airo-tulip](https://github.com/airo-ugent/airo-tulip) | KELO mobile robot platform driver |
| [ur-analytic-ik](https://github.com/Victorlouisdg/ur-analytic-ik) | Analytic IK for UR manipulators |
| [airo-blender](https://github.com/airo-ugent/airo-blender) | Synthetic data generation with Blender |

## Agent Run Traces

After completing any task, the agent **must** create a markdown file in `agent-runs/` to record what was done.

- **Path**: `agent-runs/YYYY-MM-DD_HH-MM-SS_<short-slug>.md`
- **Content**: task description, files created/modified, key decisions, commands run, and outcome
- Create the `agent-runs/` directory if it does not exist

Example:
```
agent-runs/
└── 2026-06-23_15-35-00_robot-motion-example.md
```

This folder is the persistent history of all agent work in this repository.

## Token Optimization — KISS / Caveman Rules

These rules apply globally to ALL agents and interactions in this workspace.

1. **Be a caveman** — use fewest words that convey meaning. No filler, no preamble, no "certainly", no "I'd be happy to".
2. **Code over prose** — show code, not paragraphs explaining code. One sentence of context max.
3. **Answer the question asked** — do not elaborate, suggest alternatives, or add "bonus tips" unless explicitly requested.
4. **No redundant reads** — if you already have file contents or search results, do not re-read/re-search.
5. **Batch tool calls** — combine independent operations into parallel calls. Never make 5 sequential reads when 5 parallel reads work.
6. **Skip the obvious** — do not explain what well-known functions do, what standard patterns are, or restate the user's question back.
7. **Minimal diffs** — edit only what's needed. Do not reformat, re-comment, or "improve" surrounding code.
8. **One-shot answers** — aim to resolve in a single response. If you need clarification, ask one focused question, not a list of five.
9. **No decorative formatting** — skip unnecessary headers, bullet lists, and markdown flourishes when a plain sentence suffices.
10. **Fetch only what's needed** — when looking up source code, read the specific file/function, not entire directories.
