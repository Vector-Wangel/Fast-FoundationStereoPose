# Fast-FoundationStereo Real-Time Toolkit

Real-time stereo depth estimation and interactive 3D point cloud visualization using [Fast-FoundationStereo](https://github.com/NVlabs/Fast-FoundationStereo), with optional [SAM2](https://github.com/facebookresearch/sam2) object tracking and 6D oriented bounding box estimation.

Built on top of Fast-FoundationStereo (CVPR 2026).

## Features

- **Real-time stereo depth** with FFS zero-shot generalization (~7-10 FPS on RTX 3090)
- **Multiple camera support**: USB stereo cameras, Intel RealSense D415, OAK-D Lite
- **Stereo calibration pipeline**: ChArUco-based calibration for custom stereo rigs
- **SAM2 interactive tracking**: Click or drag to select objects, real-time mask tracking
- **3D point cloud visualization**: Interactive Open3D viewer with camera frustum
- **6D bounding box**: PCA-based oriented bounding box with temporal smoothing on tracked objects

## Supported Cameras

| Camera | Script | Notes |
|--------|--------|-------|
| USB stereo (side-by-side) | `demos/stereo_ffs_realtime.py` | Requires stereo calibration |
| Intel RealSense D415 | `demos/d415_ffs_realtime.py` | Uses IR stereo pair + RGB colorization |
| OAK-D Lite | `demos/oak_ffs_realtime.py` | Uses DepthAI SDK |

## Installation

### 1. Environment

```bash
conda create -n ffs python=3.12 && conda activate ffs
pip install torch==2.6.0 torchvision==0.21.0 xformers --index-url https://download.pytorch.org/whl/cu124
```

### 2. Clone this repo

```bash
git clone https://github.com/Vector-Wangel/Fast-FoundationStereoPose.git
cd Fast-FoundationStereoPose
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
pip install open3d pyyaml
# For RealSense:
pip install pyrealsense2
# For OAK-D:
pip install depthai
```

### 4. Download FFS weights

Download the entire `23-36-37` folder from the [official link](https://drive.google.com/drive/folders/1HuTt7UIp7gQsMiDvJwVuWmKpvFzIIMap?usp=drive_link) and place under `weights/`:

```
weights/23-36-37/
├── cfg.yaml
└── model_best_bp2_serialize.pth
```

### 5. SAM2 setup (optional, for tracking demos)

The SAM2 source code (including `CameraPredictor` for real-time streaming) is already included in `SAM2_streaming/`. You only need to download the checkpoint:

```bash
mkdir -p SAM2_streaming/checkpoints/sam2.1
wget -P SAM2_streaming/checkpoints/sam2.1 https://dl.fbaipublicfiles.com/segment_anything_2/092824/sam2.1_hiera_small.pt
```

## Stereo Calibration (USB stereo cameras)

For USB stereo cameras that output side-by-side frames, you need to calibrate first.

### Step 1: Diagnose your board (optional)

```bash
python calibration/diag_charuco.py
```

Verifies that the camera can detect your ChArUco board and shows detection results.

### Step 2: Collect calibration images

```bash
python calibration/collect_cali.py
```

- Point the camera at the ChArUco board from various angles and distances
- Press **Space** to capture (only saves when board is detected)
- Press **q** to quit
- Aim for 15-30 image pairs

Images are saved to `calibration/calib_imgs/left/` and `calibration/calib_imgs/right/`.

### Step 3: Run calibration

```bash
python calibration/calibrate.py
```

Generates `stereo_calib.yaml` with intrinsics, distortion, rectification maps, and baseline.

## Usage

### Basic stereo point cloud (USB camera)

```bash
python demos/stereo_ffs_realtime.py
```

Displays a real-time interactive 3D point cloud in an Open3D window.

### RealSense D415

```bash
python demos/d415_ffs_realtime.py
```

Uses IR stereo pairs for depth estimation with RGB colorization.

### OAK-D Lite

```bash
python demos/oak_ffs_realtime.py
```

### SAM2 tracking demo (RGB only)

```bash
python demos/sam2_rgb_demo.py
```

- **Drag** to draw a bounding box around the target
- **Click** to select a point on the target
- **r** to reset tracking
- **q** to quit

### SAM2 + Stereo point cloud + 6D BBox

```bash
python demos/combined_sam2_stereo.py
```

Two windows:
- **Left (OpenCV)**: RGB image with SAM2 mask overlay and mouse interaction
- **Right (Open3D)**: 3D point cloud with tracked object highlighted in red + green oriented bounding box

Controls (focus on OpenCV window):
- **Drag**: Select target with bounding box
- **Click**: Select target with point
- **r**: Reset tracking
- **q**: Quit

## Parameters

Key parameters in the demo scripts that you may want to tune:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `VALID_ITERS` | 6 | FFS refinement iterations (lower = faster, less accurate) |
| `MAX_DISP` | 192 | Maximum disparity (increase for close objects) |
| `PCD_STRIDE` | 2 | Point cloud downsampling (higher = fewer points, faster) |
| `ZFAR` | 5.0 | Maximum depth in meters |
| `ZNEAR` | 0.05 | Minimum depth in meters |

## Notes

- Camera images are rotated 180 degrees in some scripts (for upside-down mounted cameras). Remove `cv2.rotate(..., cv2.ROTATE_180)` if your camera is right-side up.
- SAM2's `fill_hole_area` is set to 0 to bypass the `_C.so` CUDA extension, which requires GLIBC >= 2.32. This is a cosmetic-only limitation.
- FFS uses the NVIDIA non-commercial license. See `LICENSE.txt`.

## Acknowledgments

- [Fast-FoundationStereo](https://github.com/NVlabs/Fast-FoundationStereo) (NVIDIA, CVPR 2026)
- [SAM 2](https://github.com/facebookresearch/sam2) (Meta, ECCV 2024)
