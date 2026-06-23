---
name: dataset-tools
description: "Create, load, merge, split, and transform COCO datasets, convert CVAT annotations, and work with segmentation masks using airo-dataset-tools. Use when: COCO, dataset, annotation, CVAT, segmentation mask, YOLO, FiftyOne, merge dataset, split dataset, bounding box, keypoint, instance segmentation, data augmentation, albumentations, resize images, dataset CLI."
---

# Dataset Tools with airo-dataset-tools

Source: https://github.com/airo-ugent/airo-mono/tree/main/airo-dataset-tools

## When to Use
- Creating, loading, merging, or splitting COCO-format datasets
- Converting CVAT XML annotations to COCO format
- Converting COCO to YOLO format
- Applying data augmentations to COCO datasets
- Working with segmentation masks
- Visualizing datasets with FiftyOne
- Resizing dataset images and updating annotations

## Installation

```bash
pip install airo-dataset-tools

# Optional extras:
pip install airo-dataset-tools[augmentations]  # albumentations transforms
pip install airo-dataset-tools[fiftyone]       # FiftyOne visualization
```

## COCO Dataset Operations

### Pydantic Models

All COCO structures are Pydantic v2 models for validation:

```python
from airo_dataset_tools.coco_tools.coco_instances_dataset import (
    CocoInstancesDataset,
    CocoImage,
    CocoInstanceAnnotation,
    CocoCategory,
)

# Load a COCO dataset
dataset = CocoInstancesDataset.model_validate_json(Path("annotations.json").read_text())

# Access components
images = dataset.images          # list[CocoImage]
annotations = dataset.annotations  # list[CocoInstanceAnnotation]
categories = dataset.categories    # list[CocoCategory]
```

### Merging Datasets

```python
from airo_dataset_tools.coco_tools.merge_datasets import merge_coco_datasets

merge_coco_datasets(
    json_path_1="dataset1/annotations.json",
    json_path_2="dataset2/annotations.json",
    target_json_path="merged/annotations.json",
)
# Images are automatically copied, preserving nested subdirectory structure
```

### Splitting Datasets

```python
from airo_dataset_tools.coco_tools.split_dataset import split_and_save_coco_dataset

split_and_save_coco_dataset(
    coco_json_path="annotations.json",
    target_dir="./splits",
    ratios=[0.7, 0.15, 0.15],  # train, val, test
    split_names=["train", "val", "test"],
)
```

### Applying Transforms (Augmentations)

```python
from airo_dataset_tools.coco_tools.transform_dataset import apply_transform_to_coco_dataset
import albumentations as A

transform = A.Compose([
    A.HorizontalFlip(p=0.5),
    A.RandomBrightnessContrast(p=0.3),
], bbox_params=A.BboxParams(format='coco'))

apply_transform_to_coco_dataset(
    coco_json_path="annotations.json",
    target_dir="./augmented",
    transform=transform,
    n_augmentations_per_image=3,
)
```

### Creating a COCO Dataset Programmatically

```python
from airo_dataset_tools.coco_tools.coco_instances_dataset import (
    CocoInstancesDataset,
    CocoImage,
    CocoInstanceAnnotation,
    CocoCategory,
    CocoInfo,
)

dataset = CocoInstancesDataset(
    info=CocoInfo(description="My dataset", version="1.0"),
    images=[
        CocoImage(id=1, file_name="img001.jpg", width=640, height=480),
    ],
    categories=[
        CocoCategory(id=1, name="object", supercategory="none"),
    ],
    annotations=[
        CocoInstanceAnnotation(
            id=1,
            image_id=1,
            category_id=1,
            bbox=[100, 150, 200, 300],  # [x, y, width, height]
            area=60000,
            iscrowd=0,
        ),
    ],
)

Path("annotations.json").write_text(dataset.model_dump_json(indent=2))
```

### COCO to YOLO Conversion

```python
from airo_dataset_tools.coco_tools.coco_to_yolo import (
    create_yolo_dataset_from_coco_instances_dataset,
)

create_yolo_dataset_from_coco_instances_dataset(
    coco_json_path="annotations.json",
    target_dir="./yolo_dataset",
    use_segmentation=False,  # True for polygon labels
)
```

## CVAT Annotation Conversion

```python
from airo_dataset_tools.cvat_labeling.cvat_image_to_coco import cvat_image_to_coco

cvat_image_to_coco(
    cvat_xml_path="annotations.xml",
    categories_json_path="categories.json",
    target_json_path="coco_annotations.json",
)
```

## Segmentation Mask Conversion

```python
from airo_dataset_tools.segmentation_mask_converter import (
    binary_mask_to_rle,
    rle_to_binary_mask,
    binary_mask_to_polygon,
    polygon_to_binary_mask,
)

# Convert between mask representations
rle = binary_mask_to_rle(binary_mask)
mask = rle_to_binary_mask(rle, height, width)
polygons = binary_mask_to_polygon(binary_mask)
mask = polygon_to_binary_mask(polygons, height, width)
```

## Camera Intrinsics & Pose Formats

```python
from airo_dataset_tools.data_parsers.camera_intrinsics import CameraIntrinsics
from airo_dataset_tools.data_parsers.pose import Pose

# Pydantic models for serialization
intrinsics = CameraIntrinsics(
    image_resolution=ImageResolution(width=640, height=480),
    focal_lengths=FocalLengths(fx=600.0, fy=600.0),
    principal_point=PrincipalPoint(cx=320.0, cy=240.0),
)

pose = Pose(
    translation=[0.5, 0.0, 0.3],
    rotation=Rotation(quaternion=Quaternion(x=0, y=0, z=0, w=1)),
)
```

## CLI Entry Points

```bash
# View dataset with FiftyOne
airo-dataset-tools fiftyone-coco-viewer annotations.json --dataset-dir ./images

# Convert CVAT XML to COCO
airo-dataset-tools convert-cvat-to-coco-keypoints cvat.xml categories.json

# Split dataset
airo-dataset-tools split-coco-dataset annotations.json ./splits --ratios 0.7 0.15 0.15

# Resize images + update annotations
airo-dataset-tools resize-coco-dataset annotations.json ./resized --height 480 --width 640

# Convert COCO to YOLO
airo-dataset-tools coco-to-yolo annotations.json ./yolo_data --use-segmentation
```

## Common Pitfalls

- **COCO bbox format**: `[x, y, width, height]` (top-left corner) — not `[x1, y1, x2, y2]`
- **Image IDs**: Must be unique integers; merging auto-remaps IDs to avoid collisions
- **FiftyOne**: Now an optional extra — install with `pip install airo-dataset-tools[fiftyone]`
- **albumentations**: Also optional — install with `pip install airo-dataset-tools[augmentations]`

## GitHub Source

Fetch the latest from:
- COCO tools: `airo-dataset-tools/airo_dataset_tools/coco_tools/`
- CVAT conversion: `airo-dataset-tools/airo_dataset_tools/cvat_labeling/`
- Data parsers: `airo-dataset-tools/airo_dataset_tools/data_parsers/`
- Segmentation masks: `airo-dataset-tools/airo_dataset_tools/segmentation_mask_converter.py`
- CLI: `airo-dataset-tools/airo_dataset_tools/cli.py`
