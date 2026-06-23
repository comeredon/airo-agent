---
name: camera-vision
description: "Work with RGB-D cameras and point clouds using airo-camera-toolkit. Use when: camera, ZED, RealSense, RGB, depth, point cloud, image, grab_images, retrieve, intrinsics, calibration, hand-eye, ChArUco, ArUco, multiprocess, image transform, pinhole, projection, undistort, crop point cloud."
---

# Camera & Vision with airo-camera-toolkit

Source: https://github.com/airo-ugent/airo-mono/tree/main/airo-camera-toolkit

## When to Use
- Capturing images from ZED, RealSense, or USB cameras
- Working with depth maps and point clouds
- Projecting 3D points to 2D pixels (or unprojecting)
- Applying image transforms (resize, crop, rotate)
- Hand-eye calibration (camera-to-robot)
- Detecting ArUco/ChArUco markers

## Installation

```bash
pip install airo-camera-toolkit
# For hand-eye calibration with a robot:
pip install airo-camera-toolkit[hand-eye-calibration]
```

**Hardware-specific setup:**
- **ZED**: Requires ZED SDK installed separately (see `cameras/zed/installation.md`)
- **RealSense**: Requires `pyrealsense2` (see `cameras/realsense/realsense_installation.md`)
- **USB webcam**: No extra deps (uses OpenCV)

## Core Interfaces

### Camera Hierarchy

```
Camera (ABC)
├── RGBCamera (ABC)
│   └── provides: retrieve_rgb_image(), retrieve_rgb_image_as_int()
└── DepthCamera (ABC)
    └── provides: retrieve_depth_map(), retrieve_depth_image(), retrieve_confidence_map()
```

### The grab + retrieve Pattern (CRITICAL)

Capture and retrieval are **explicitly decoupled**:

```python
camera.grab_images()                    # Capture a frame from hardware
rgb = camera.retrieve_rgb_image()       # float32 RGB [0,1], shape (H, W, 3)
rgb_int = camera.retrieve_rgb_image_as_int()  # uint8 RGB [0,255], shape (H, W, 3)
depth = camera.retrieve_depth_map()     # float32, shape (H, W), values in meters
pc = camera.retrieve_colored_point_cloud()  # PointCloud dataclass
```

**Rules:**
1. Always call `grab_images()` before any `retrieve_*()` call
2. Calling `retrieve_*()` before the first `grab_images()` raises `RuntimeError`
3. Multiple `retrieve_*()` calls after one `grab_images()` return data from the **same frame** — this is how you get synchronized RGB + depth

### Camera Properties
- `intrinsics_matrix()` → `np.ndarray` shape `(3, 3)` — camera intrinsics K matrix
- `resolution` → `tuple[int, int]` — (height, width)
- `fps` → `int` — frames per second

## Camera Implementations

### ZED (Stereo RGB-D)
```python
from airo_camera_toolkit.cameras import Zed

camera = Zed(resolution=Zed.RESOLUTION_720, fps=30)
# Also: Zed.RESOLUTION_1080, Zed.RESOLUTION_2K

camera.grab_images()
rgb = camera.retrieve_rgb_image_as_int()
depth = camera.retrieve_depth_map()
point_cloud = camera.retrieve_colored_point_cloud()
intrinsics = camera.intrinsics_matrix()
```

### RealSense (RGB-D)
```python
from airo_camera_toolkit.cameras import Realsense

camera = Realsense(resolution=Realsense.RESOLUTION_720, fps=30)
camera.grab_images()
rgb = camera.retrieve_rgb_image_as_int()
depth = camera.retrieve_depth_map()
```

### USB Webcam
```python
from airo_camera_toolkit.cameras import OpenCVVideoCapture

camera = OpenCVVideoCapture(camera_index=0)
camera.grab_images()
rgb = camera.retrieve_rgb_image_as_int()
```

## Point Cloud Operations

```python
from airo_camera_toolkit.point_clouds.operations import (
    crop_point_cloud,
    transform_point_cloud,
    filter_point_cloud,
)
from airo_typing import PointCloud

# Crop to a 3D bounding box
bbox = ((x_min, y_min, z_min), (x_max, y_max, z_max))
cropped = crop_point_cloud(point_cloud, bbox)

# Transform to world frame using a 4×4 matrix
pc_world = transform_point_cloud(point_cloud, camera_extrinsics_4x4)

# Filter by boolean mask
filtered = filter_point_cloud(point_cloud, mask)
```

## Pinhole Operations

```python
from airo_camera_toolkit.pinhole_operations.projection import project_points_to_image_plane
from airo_camera_toolkit.pinhole_operations.unprojection import unproject_pixels_to_points

# 3D → 2D projection
pixels = project_points_to_image_plane(points_3d, intrinsics_matrix)

# 2D + depth → 3D unprojection
points_3d = unproject_pixels_to_points(pixels, depths, intrinsics_matrix)
```

## Image Transforms

```python
from airo_camera_toolkit.image_transforms.transforms import Crop, Resize

# Create reversible transforms
crop = Crop(x_start=100, y_start=50, width=640, height=480)
resize = Resize(target_height=240, target_width=320)

# Apply to image
cropped_image = crop.transform_image(image)
resized_image = resize.transform_image(cropped_image)

# Apply to intrinsics (keeps projection consistent)
new_intrinsics = crop.transform_intrinsics(intrinsics)
```

## Image Format Conversion

```python
from airo_camera_toolkit.utils.image_converter import ImageConverter

converter = ImageConverter.from_numpy_format(float_rgb_image)
bgr_uint8 = converter.image_in_opencv_format  # BGR uint8 for cv2.imshow
```

## Hand-Eye Calibration

Find the pose of a camera relative to a robot arm.

**CLI usage:**
```bash
airo-camera-toolkit hand-eye-calibration --help
airo-camera-toolkit hand-eye-calibration \
    --mode eye_in_hand \
    --robot_ip 10.42.0.162
```

**Programmatic:**
```python
from airo_camera_toolkit.calibration.compute_calibration import compute_hand_eye_calibration
from airo_camera_toolkit.calibration.fiducial_markers import detect_charuco_board

# Detect ChArUco board pose in image
board_pose = detect_charuco_board(image, intrinsics)

# Compute calibration from collected pose pairs
extrinsics = compute_hand_eye_calibration(
    robot_poses, board_poses, method="eye_in_hand"
)
```

## Multiprocess Camera

For reading camera frames in a separate process (non-blocking robot control):

```python
from airo_camera_toolkit.cameras.multiprocess import MultiprocessVideoRecorder
# Requires the [recording] extra: pip install airo-camera-toolkit[recording]
```

## Common Pitfalls

- **Forgetting `grab_images()`**: Calling `retrieve_*()` without it raises `RuntimeError`
- **OpenCV BGR vs RGB**: airo cameras return RGB; use `ImageConverter` to get BGR for `cv2.imshow()`
- **ZED SDK version**: Must match the installed `pyzed` package; check ZED SDK docs
- **Depth units**: Depth maps are in **meters** (float32), not millimeters

## GitHub Source

Fetch the latest interfaces and implementations from:
- Interfaces: `airo-camera-toolkit/airo_camera_toolkit/interfaces.py`
- ZED: `airo-camera-toolkit/airo_camera_toolkit/cameras/zed/`
- RealSense: `airo-camera-toolkit/airo_camera_toolkit/cameras/realsense/`
- Point clouds: `airo-camera-toolkit/airo_camera_toolkit/point_clouds/`
- Calibration: `airo-camera-toolkit/airo_camera_toolkit/calibration/`
- Pinhole ops: `airo-camera-toolkit/airo_camera_toolkit/pinhole_operations/`
