# 03 · File Formats and Modalities

Each camera directory holds exactly 8 modality directories, each with 5 files (frame_01 … frame_05).

## 1. Modality overview

| # | Directory | Ext | dtype | Shape (H×W[×C]) | Unit / color | Semantics |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | rgb_color | png | uint8 | 720×1280×3 | RGB / sRGB | color image |
| 2 | depth_raw_uint16 | png | uint16 | 480×848 (C336L) / 720×1280 (CD435I) | sensor depth units | raw depth, unaligned, lossless PNG |
| 3 | depth_raw_npy | npy | uint16 | same as depth_raw_uint16 | sensor depth units | raw array identical to the PNG |
| 4 | depth_raw_color | png | uint8 | 480×848×3 (C336L) / 720×1280×3 (CD435I) | RGB | pseudo-color preview of raw depth (viewing only) |
| 5 | depth_aligned_uint16 | png | uint16 | 720×1280 | sensor depth units | depth aligned to the color camera, lossless PNG |
| 6 | depth_aligned_npy | npy | uint16 | 720×1280 | sensor depth units | array identical to the aligned PNG |
| 7 | depth_aligned_color | png | uint8 | 720×1280×3 | RGB | pseudo-color preview of aligned depth (viewing only) |
| 8 | pointcloud_color_xyz_mm | ply | float32×3 + uint8×3 | ~5×10^4 vertices per frame | millimeter + RGB | colored point cloud |

"Identical" means: decoding the lossless PNG yields exactly the same uint16 array (values, shape, byte order) as loading the same-named .npy file.

## 2. Resolution differences

- C336L: raw depth is **848×480**, color and aligned depth are **1280×720** (the aligned depth is resampled to color resolution).
- CD435I: color, raw depth and aligned depth are **all 1280×720**.
- Never assume a fixed shape across cameras.

## 3. PNG details

| File | Color type | Bit depth | Notes |
| --- | --- | --- | --- |
| rgb_color/*.png | 2 (truecolor) | 8 | RGB channel order, file color space sRGB; the channel order returned by a PNG decoder is implementation-defined, convert explicitly |
| depth_raw_uint16 / depth_aligned_uint16 | 0 (grayscale) | 16 | must be read with the original bit depth preserved; reading as 8-bit truncates |
| depth_raw_color / depth_aligned_color | 2 (truecolor) | 8 | visualization only; the colormap carries no calibration information |

## 4. NPY details

- Standard NumPy 1.0 header: magic + version + dict (descr, fortran_order, shape), 128-byte aligned header.
- descr = `<u2` (little-endian uint16); fortran_order = False (C order); shape = (H, W).
- Pickle is disallowed (`allow_pickle = false`); mmap reading works directly.
- File size is deterministic: H × W × 2 + 128 bytes (e.g. 814,208 B for 848×480; 1,843,328 B for 1280×720).
- **NPY values are not premultiplied by the depth scale** and match the corresponding PNG exactly.

## 5. Point cloud details

- Format: binary_little_endian PLY 1.0, vertices only (no faces).
- Attributes: x float, y float, z float, red uchar, green uchar, blue uchar.
- Coordinate unit: **millimeters**; the frame follows the color camera of the respective sensor.
- Downsampling: **one point every 4 pixels** (`pixel_stride = 4`), so the vertex count is lower than the number of valid depth pixels.
- Color comes from the aligned color image in RGB order.

## 6. Depth semantics and millimeter conversion

1. Depth files store **sensor depth units** (`value_semantics = sensor_depth_units`), not millimeters.
2. Conversion:

```
depth_mm = depth_value * depth_scale_mm_per_unit
```

3. `depth_scale_mm_per_unit` is recorded **per frame**, consistently in two places:
   - `capture_manifest.json` → `frames.<camera_code>[i].depth_scale_mm_per_unit`
   - the corresponding `depth_*_npy` entry in `files[]`
4. Dataset-wide values (identical in all 664 sessions): C336L = 1.0; CD435I = 1.0000000474974513.
5. A value of 0 means no valid depth (no return / out of range) and should be masked out explicitly.
