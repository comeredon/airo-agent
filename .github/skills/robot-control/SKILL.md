---
name: robot-control
description: "Control UR robots and Robotiq grippers with airo-robots. Use when: UR robot, move robot, gripper, Robotiq, AwaitableAction, RTDE, joint configuration, TCP pose, manipulator, robot arm, move linear, move joint, servo, force torque, bimanual, parallel motion."
---

# Robot Control with airo-robots

Source: https://github.com/airo-ugent/airo-mono/tree/main/airo-robots

## When to Use
- Moving a UR robot arm to a pose or joint configuration
- Controlling a Robotiq 2F-85 gripper (open, close, move to width)
- Understanding the AwaitableAction async pattern
- Parallel arm + gripper motion
- Reading force/torque sensor data
- Troubleshooting robot communication (RTDE)

## Installation

```bash
pip install airo-robots
# or from source:
pip install -e airo-robots/
```

Requires `ur-rtde >= 1.5.7` (automatically installed). The UR robot must be reachable on the network.

## Core Interfaces

### PositionManipulator (ABC)
Base class for all position-controlled robot arms.

**Key methods** (all return `AwaitableAction`):
- `move_to_joint_configuration(joint_config, joint_speed=None)` — move joints
- `move_to_tcp_pose(tcp_pose, linear_speed=None)` — move TCP in Cartesian space
- `move_linear_to_tcp_pose(tcp_pose, linear_speed=None)` — straight-line TCP motion
- `servo_to_joint_configuration(joint_config, time)` — real-time servoing
- `servo_to_tcp_pose(tcp_pose, time)` — real-time TCP servoing

**Properties:**
- `joint_configuration` — current joint angles (radians)
- `tcp_pose` — current TCP pose as 4×4 homogeneous matrix
- `manipulator_specs` — `ManipulatorSpecs` dataclass with `.dof`, `.max_joint_speeds`, `.max_linear_speed`
- `gripper` — attached gripper (if any)
- `default_linear_speed` — default Cartesian speed (m/s)

### ParallelPositionGripper (ABC)
Base class for 2-finger parallel grippers.

**Key methods:**
- `open()` → `AwaitableAction`
- `close()` → `AwaitableAction`
- `move(width, speed=None, force=None)` → `AwaitableAction`
- `get_current_width()` → `float` (meters)

**Properties:**
- `speed` (get/set) — gripper speed (m/s)
- `max_grasp_force` (get/set) — grip force (N)
- `gripper_specs` — `ParallelPositionGripperSpecs` with `max_width`, `min_width`, `max_force`, etc.

### AwaitableAction
Async command pattern for hardware operations.

```python
from airo_robots.awaitable_action import AwaitableAction, ACTION_STATUS_ENUM

action = robot.move_to_tcp_pose(target_pose)
# ... do other work ...
status = action.wait(timeout=10.0, sleep_resolution=0.01)
# status is ACTION_STATUS_ENUM.SUCCEEDED or .TIMEOUT or .EXECUTING

if action.is_done():
    print("Move completed")
```

**Critical rules:**
1. Always `.wait()` on a command before sending the next one to the same device
2. Timeout is in seconds; choose based on expected motion duration
3. `sleep_resolution` controls busy-wait granularity (default ~0.01s; `time.sleep` on Windows has ~15ms accuracy)

## Hardware Implementations

### URrtde — UR Robot via RTDE

```python
from airo_robots.manipulators.hardware.ur_rtde import URrtde

robot = URrtde("192.168.1.10", URrtde.UR3E_CONFIG)
# Available configs: UR3_CONFIG, UR3E_CONFIG, UR5E_CONFIG, UR10E_CONFIG, UR16E_CONFIG, UR20_CONFIG

# Read current state
joints = robot.joint_configuration      # np.ndarray shape (6,)
pose = robot.tcp_pose                    # np.ndarray shape (4, 4)

# Move
robot.move_to_joint_configuration(target_joints).wait(timeout=10)
robot.move_linear_to_tcp_pose(target_pose, linear_speed=0.1).wait(timeout=15)
```

### Robotiq2F85 — Robotiq Gripper via URCap

```python
from airo_robots.grippers.hardware.robotiq_2f85_urcap import Robotiq2F85

gripper = Robotiq2F85("192.168.1.10")  # Same IP as the UR robot

gripper.open().wait(timeout=5)
gripper.close().wait(timeout=5)
gripper.move(width=0.04, speed=0.05, force=20).wait(timeout=5)

current_width = gripper.get_current_width()  # meters
```

## Common Patterns

### Sequential Pick-and-Place
```python
robot.move_to_joint_configuration(home_joints).wait(timeout=10)
gripper.open().wait(timeout=3)
robot.move_linear_to_tcp_pose(pre_grasp_pose).wait(timeout=10)
robot.move_linear_to_tcp_pose(grasp_pose).wait(timeout=5)
gripper.close().wait(timeout=3)
robot.move_linear_to_tcp_pose(pre_grasp_pose).wait(timeout=5)
robot.move_linear_to_tcp_pose(place_pose).wait(timeout=10)
gripper.open().wait(timeout=3)
```

### Parallel Arm + Gripper Motion
```python
arm_action = robot.move_to_joint_configuration(approach_joints)  # starts immediately
gripper.open().wait(timeout=3)  # runs while arm moves
arm_action.wait(timeout=10)  # then wait for arm to finish
```

## Common Pitfalls

- **Forgetting `.wait()`**: Commands are async — without `.wait()`, the next command may preempt the current one
- **Wrong timeout**: If timeout is too short, you get `TIMEOUT` status but the robot keeps moving
- **Windows sleep accuracy**: `time.sleep` on Windows has ~15ms accuracy; set `sleep_resolution >= 0.015`
- **Network issues**: The UR robot must be on the same network; default RTDE port is 30004

## GitHub Source

Fetch the latest interfaces and implementations from:
- Interfaces: `airo-robots/airo_robots/manipulators/position_manipulator.py`
- UR implementation: `airo-robots/airo_robots/manipulators/hardware/ur_rtde.py`
- Gripper interface: `airo-robots/airo_robots/grippers/parallel_position_gripper.py`
- Robotiq: `airo-robots/airo_robots/grippers/hardware/robotiq_2f85_urcap.py`
- AwaitableAction: `airo-robots/airo_robots/awaitable_action.py`
