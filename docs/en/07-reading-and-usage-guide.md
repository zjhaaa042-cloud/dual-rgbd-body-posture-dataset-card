# 07 · Reading and Usage Guide

## 1. Recommended read order

1. Read the subject manifest `session_manifest.json`: confirm all 8 views and get each session path plus its manifest hash.
2. Read the session manifest `capture_manifest.json`: inspect `sync_audit` (cross-camera timing), `metadata.distance_mm`, per-frame intrinsics and depth scale.
3. Verify files against `files[]` (`size_bytes` + `sha256`).
4. Only then load the modality files you need.

## 2. Depth loading and millimeter conversion

```python
import json
import numpy as np
import cv2

cap = "subjects/<subject_id>/angles/angle_000_front/capture_01_<UTC>"
manifest = json.load(open(f"{cap}/capture_manifest.json", encoding="utf-8"))

cam = "C336L"
frame_idx = 0                       # 0..4 corresponds to frame_01..frame_05
scale = manifest["frames"][cam][frame_idx]["depth_scale_mm_per_unit"]

# Option A: lossless uint16 PNG (original bit depth must be preserved)
png = cv2.imread(f"{cap}/camera_gemini_336l/depth_raw_uint16/frame_01.png",
                 cv2.IMREAD_UNCHANGED)
assert png.dtype == np.uint16, png.dtype

# Option B: equivalent NumPy array (mmap, no premultiplied scale)
npy = np.load(f"{cap}/camera_gemini_336l/depth_raw_npy/frame_01.npy", mmap_mode="r")

assert np.array_equal(png, npy)     # pixel-identical

depth_mm = npy.astype(np.float32) * scale
depth_mm[npy == 0] = np.nan         # 0 = invalid
```

## 3. Color and point clouds

```python
# Color: RGB order, convert explicitly if needed
rgb = cv2.cvtColor(cv2.imread(".../rgb_color/frame_01.png"), cv2.COLOR_BGR2RGB)

# Point cloud: binary_little_endian PLY, x/y/z float32 + rgb uint8, unit = millimeter
import open3d as o3d
pcd = o3d.io.read_point_cloud(".../pointcloud_color_xyz_mm/frame_01.ply")
```

Point clouds can also be parsed with plyfile, or by hand: read the ASCII header up to `end_header`, then read
15 bytes per vertex (3×float32 + 3×uint8).

## 4. Pre-fusion checklist

- [ ] Read `sync_audit.max_host_timestamp_skew_ms` of the session and confirm it is acceptable (dataset range: 9.00–73.86 ms).
- [ ] For per-pair timing use `sync_audit.pairs[]` and pair with `host_timestamp_ns` (nearest or interpolated).
- [ ] Normalize extrinsics translation units (C336L in millimeters, CD435I in meters).
- [ ] Normalize depth units first (per-frame `depth_scale_mm_per_unit`; never hard-code).
- [ ] Undistort with the correct `distortion_model` and coefficient count per camera.

## 5. Common pitfalls

| Pitfall | Notes |
| --- | --- |
| Reading depth as 8-bit | 16-bit PNGs are truncated by default readers; request the original bit depth |
| Assuming a uniform resolution | C336L raw depth is 848×480 while color/aligned depth is 1280×720; CD435I is 1280×720 everywhere |
| Treating previews as depth | `depth_*_color` are 8-bit visualizations with no quantitative meaning |
| Forgetting the depth scale | Neither PNG nor NPY is premultiplied; the CD435I factor is not exactly 1.0 |
| Treating 0 as 0 mm | 0 means invalid depth and should be masked |
| Mixed extrinsics units | millimeters on C336L, meters on CD435I |
| Expecting IR | C336L provides no IR stream and missing modalities are never fabricated |
| Using CD435I as the primary view | Its FOV is limited (`auxiliary_fov_limited_non_blocking`); the body may be cropped |
| Assuming hardware sync | The cameras are host-clock near-synced; align explicitly for fast motion |
| Hard-coding field presence | `depth_scale_mm_per_unit`, `logical_modality` and the color fields only appear for specific modalities |
