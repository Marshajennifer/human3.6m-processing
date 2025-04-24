# human3.6m-processing
To preprocess and clean Human 3.6m dataset.
# Human3.6M Dataset Preparation for 2D-to-3D Pose Estimation

This repository provides code and guidance to preprocess the [Human3.6M dataset](http://vision.imar.ro/human3.6m/description.php) and convert it into `.npz` using [VideoPose3D](https://github.com/facebookresearch/VideoPose3D) and the original dataset was converted to produce 'data_3d_h36m.npz' and 'data_2d_h36m_gt.npz'.

## 🔧 What this repository does

- Converts raw `.cdf` files (D3_Positions) from Human3.6M into structured NumPy arrays (`.npz`)
- Projects 3D world coordinates into multiple 2D camera views
- Produces two key files:
  - `data_3d_h36m.npz`: Contains structured 3D pose data (in meters) for each subject and action
  - `data_2d_h36m_gt.npz`: Contains ground-truth 2D pose projections for each camera view

## 📦 Files Generated

| File | Description |
|------|-------------|
| `data_3d_h36m.npz` | 3D joint positions for each subject and action in Human3.6M,(in world coordinate) |
| `data_2d_h36m_gt.npz` | 2D projections of 3D poses into all 4 camera views using calibrated parameters |

data_2d_h36m_gt.npz is generated from data_3d_h36m.npz by (world coordinate-->camera coordinate-->2D projection).

## 📁 Dataset Used

- **Human3.6M Official Dataset**  
  Website: [http://vision.imar.ro/human3.6m/description.php](http://vision.imar.ro/human3.6m/description.php)  
  You need to request access and download the following folders for each subject:


- **Video Files**  
Download from [http://vision.imar.ro/human3.6m/description.php](http://vision.imar.ro/human3.6m/description.php)

## 📌 Camera Parameters

Camera extrinsic and intrinsic parameters are loaded from:
- [karfly/human36m-camera-parameters](https://github.com/karfly/human36m-camera-parameters)
- These include focal lengths, principal points, distortion coefficients, and camera orientations/translations.

## 📜 Credits

- 📦 Dataset: [Human3.6M](http://vision.imar.ro/human3.6m/description.php)
- 📘 Conversion Utilities: Adapted from [VideoPose3D by Facebook AI Research](https://github.com/facebookresearch/VideoPose3D)
- 📷 Camera Parameters: [karfly/human36m-camera-parameters](https://github.com/karfly/human36m-camera-parameters)

## 🛠️ Installation

To run the conversion scripts, make sure to install the required dependency:

```bash
pip install cdflib
```
Run `data_preprocessing.ipynb` to generate npz files.

Run `visualization.ipynb` to visualize and plot both 2D and 3D skeleton on video frames.

`3dto2dfromnpz.ipynb` shows manual projection of 3D to 2D. Its optional. For learning purposes.

## 📐 3D to 2D Projection from Human3.6M (Manual Implementation)

This repository contains a Jupyter notebook that **converts 3D world coordinates to 2D image coordinates** using **manual camera parameter transformations** — ideal for learning and debugging Human3.6M projections step by step.

---

### Files

- `3dto2dfromnpz.ipynb`  
  → Main notebook performing 3D-to-2D projection using manual equations.

- [`camera-parameters.json`](https://github.com/karfly/human36m-camera-parameters/blob/master/camera-parameters.json)  
  → JSON file containing both intrinsic and extrinsic parameters for all Human3.6M cameras.

- `data_3d_h36m.npz`  
  → Preprocessed file containing 3D joint positions in **world coordinates**, extracted from Human3.6M `.cdf` files.

---

### What This Notebook Does

For a selected `subject`, `action`, and `frame`:
- Loads 3D pose from `data_3d_h36m.npz` (in world coordinates).
- Loads camera intrinsics and extrinsics from the JSON file.
- Manually applies:
  - Rotation matrix `R` and translation vector `t` (extrinsics).
  - Perspective division (normalize by Z).
  - Focal length and principal point offset (intrinsics).
- Outputs final **2D pixel coordinates** of 17 keypoints.

---

### Why Manual?

Unlike OpenCV or bundled libraries that hide transformation steps, this notebook:
- **Explicitly shows every step**: from 3D world space to 2D image space.
- Helps you **understand projection math**, not just use it.
- Gives insight into why things go wrong (mismatch in camera space, resolution, etc.)

---

### 🧮 Core Equations

```python
# Step 1: World to Camera (Extrinsics)
X_cam = R @ X_world.T + t[:, np.newaxis]  # Shape: (3, 17)

# Step 2: Normalize by depth (Z)
x = X_cam[0] / X_cam[2]
y = X_cam[1] / X_cam[2]

# Step 3: Convert to pixel coordinates (Intrinsics)
u = fx * x + cx
v = fy * y + cy

# Final 2D projection
proj_2d = np.stack([u, v], axis=1)

Visualization:

![2D Projection Example](human3.6m-processing/dis.jpg)  


## 📝 Notes

From the Human3.6M website, the "Poses → D3 Positions" data and the videos were downloaded. Then, using the tools and structure inspired by [VideoPose3D by Facebook Research](https://github.com/facebookresearch/VideoPose3D/blob/main/DATASETS.md), the original dataset was converted to produce `data_3d_h36m.npz` and `data_2d_h36m_gt.npz`.

---



