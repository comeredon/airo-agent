---
name: spatial-algebra
description: "Work with SE3 poses, transforms, quaternions, and rotation matrices using airo-spatial-algebra and airo-typing. Use when: SE3, pose, transform, quaternion, rotation matrix, homogeneous matrix, euler angles, spatial algebra, coordinate frame, rotation vector, axis angle, change frame, type alias, JointConfigurationType, HomogeneousMatrixType, PointCloud."
---

# Spatial Algebra & Types

Sources:
- https://github.com/airo-ugent/airo-mono/tree/main/airo-spatial-algebra
- https://github.com/airo-ugent/airo-mono/tree/main/airo-typing

## When to Use
- Creating or converting SE3 poses (rotation + translation)
- Converting between quaternion, Euler, rotation matrix, axis-angle representations
- Transforming 3D points between coordinate frames
- Understanding airo-mono type aliases and conventions
- Fixing numerical issues with rotation matrices

## Installation

```bash
pip install airo-spatial-algebra   # includes airo-typing
```

## Conventions (CRITICAL)

| Convention | Format |
|-----------|--------|
| **Quaternions** | Scalar-last: `[x, y, z, w]` |
| **Euler angles** | XYZ extrinsic (roll, pitch, yaw) in radians |
| **Homogeneous matrices** | 4×4 NumPy array, row-major |
| **Units** | Meters, radians |
| **Rotation matrices** | 3×3 SO(3), `det(R) = 1`, `R @ R.T = I` |

## Type Aliases (from airo-typing)

```python
from airo_typing import (
    # Spatial
    Vector3DType,              # np.ndarray shape (3,)
    QuaternionType,            # np.ndarray shape (4,) — [x, y, z, w]
    RotationMatrixType,        # np.ndarray shape (3, 3)
    HomogeneousMatrixType,     # np.ndarray shape (4, 4)
    EulerAnglesType,           # np.ndarray shape (3,) — XYZ extrinsic
    RotationVectorType,        # np.ndarray shape (3,)
    Vector2DType,              # np.ndarray shape (2,)

    # Robot
    JointConfigurationType,    # np.ndarray shape (N,)
    JointPathType,             # np.ndarray shape (M, N)
    JointPathContainer,        # dataclass: path + joint_speeds
    SingleArmTrajectory,       # dataclass: positions + speeds + times
    DualArmTrajectory,         # dataclass: two SingleArmTrajectory

    # Camera
    CameraIntrinsicsMatrixType,  # np.ndarray shape (3, 3)
    CameraExtrinsicsMatrixType,  # np.ndarray shape (4, 4)
    NumpyFloatImageType,       # np.ndarray float32 [0,1]
    NumpyIntImageType,         # np.ndarray uint8 [0,255]
    NumpyDepthMapType,         # np.ndarray float32 in meters

    # Point Cloud
    PointCloud,                # dataclass: points, colors, attributes
    BoundingBox3DType,         # tuple of two Vector3DType
)
```

## SE3Container — The Main Class

```python
from airo_spatial_algebra import SE3Container
import numpy as np
```

### Creating Poses

```python
# From a 4×4 homogeneous matrix
pose = SE3Container.from_homogeneous_matrix(matrix_4x4)

# From rotation matrix + translation
pose = SE3Container.from_rotation_matrix_and_translation(
    rotation_matrix,  # (3, 3)
    translation,      # (3,)
)

# From quaternion + translation (scalar-last!)
pose = SE3Container.from_quaternion_and_translation(
    quaternion,    # [x, y, z, w]
    translation,   # (3,)
)

# From Euler angles + translation (XYZ extrinsic, radians)
pose = SE3Container.from_euler_angles_and_translation(
    euler_angles,  # [roll, pitch, yaw]
    translation,
)

# From orthogonal base vectors + translation
pose = SE3Container.from_orthogonal_base_vectors_and_translation(
    x_axis, y_axis, z_axis, translation
)

# Translation only (identity rotation)
pose = SE3Container.from_translation(np.array([0.5, 0.0, 0.3]))

# Random valid SE3 pose
pose = SE3Container.random()

# Identity
pose = SE3Container.identity()
```

### Reading Pose Components

```python
# Translation
t = pose.translation                    # np.ndarray (3,)

# Rotation in various formats
quat = pose.orientation_as_quaternion   # [x, y, z, w] scalar-last
euler = pose.orientation_as_euler_angles  # [roll, pitch, yaw] XYZ radians
axis_angle = pose.orientation_as_axis_angle  # (axis, angle) tuple
rot_vec = pose.orientation_as_rotation_vector  # np.ndarray (3,)
R = pose.rotation_matrix                # np.ndarray (3, 3)

# Full 4×4 matrix
H = pose.homogeneous_matrix            # np.ndarray (4, 4)

# Frame axes
x = pose.x_axis  # np.ndarray (3,)
y = pose.y_axis
z = pose.z_axis
```

### Composing and Inverting

```python
# Compose: A_from_C = A_from_B @ B_from_C
combined = SE3Container.from_homogeneous_matrix(
    pose_A_from_B.homogeneous_matrix @ pose_B_from_C.homogeneous_matrix
)

# Inverse: B_from_A = inv(A_from_B)
inverse = SE3Container.from_homogeneous_matrix(
    np.linalg.inv(pose.homogeneous_matrix)
)
```

## Transforming Points

```python
from airo_spatial_algebra import transform_points

# Transform N points (shape (N, 3)) by a 4×4 matrix
points_world = transform_points(
    homogeneous_matrix_4x4,  # world_from_camera
    points_camera,            # (N, 3) in camera frame
)
# Returns (N, 3) in world frame
```

## Fixing Numerical Rotation Issues

```python
from airo_spatial_algebra.operations import normalize_so3_matrix

# Fix a rotation matrix that drifted from SO(3) due to floating-point errors
R_fixed = normalize_so3_matrix(R_noisy)
# Ensures det(R) = 1 and R @ R.T = I via SVD projection
```

## Common Patterns

### Camera-to-World Transform
```python
# Given: camera extrinsics (world_from_camera)
extrinsics = SE3Container.from_homogeneous_matrix(world_from_camera_4x4)

# Transform point cloud from camera frame to world frame
from airo_spatial_algebra import transform_points
points_world = transform_points(extrinsics.homogeneous_matrix, points_camera)
```

### Creating a Grasp Pose
```python
# Grasp pose: Z-axis points down, X-axis along approach
import numpy as np
from airo_spatial_algebra import SE3Container

grasp_z = np.array([0, 0, -1])  # gripper points down
grasp_x = np.array([1, 0, 0])   # approach direction
grasp_y = np.cross(grasp_z, grasp_x)

grasp_pose = SE3Container.from_orthogonal_base_vectors_and_translation(
    grasp_x, grasp_y, grasp_z, target_position
)
```

## Common Pitfalls

- **Quaternion order**: airo-mono uses **scalar-last** `[x, y, z, w]` — many libraries (e.g., PyBullet) use scalar-first `[w, x, y, z]`
- **Euler convention**: XYZ **extrinsic** — not ZYX or intrinsic
- **Matrix multiplication order**: `world_from_camera @ camera_point` — read right-to-left
- **Numerical drift**: After many matrix multiplications, rotation matrices may leave SO(3) — use `normalize_so3_matrix()`

## GitHub Source

Fetch the latest from:
- SE3Container: `airo-spatial-algebra/airo_spatial_algebra/se3.py`
- Operations: `airo-spatial-algebra/airo_spatial_algebra/operations.py`
- Type aliases: `airo-typing/airo_typing/__init__.py`
