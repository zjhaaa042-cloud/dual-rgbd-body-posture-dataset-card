# 05 · Synchronization and Calibration

## 1. Cross-camera synchronization

- Kind: **host-clock near sync** (`host_clock_near_sync`).
- Meaning: both cameras stream through their own SDK; the capture host pairs frames one-to-one in arrival order,
  recording both host timestamps and device timestamps.
- It is **not** hardware-triggered, so frame-to-frame offsets of a few to tens of milliseconds exist.
  Always consult the audit data before cross-camera fusion.

### 1.1 Timing audit fields

`capture_manifest.json` → `sync_audit`:

| Field | Notes |
| --- | --- |
| synchronization_kind | fixed `host_clock_near_sync` |
| pair_count | number of pairs, always 5 |
| max_host_timestamp_skew_ms | largest host-timestamp skew among the 5 pairs of the session |
| pairs[].host_timestamp_skew_ms | per-pair host-timestamp skew (ms) |
| pairs[].request_start_skew_ms | per-pair skew of the frame-request instant (ms) |
| pairs[].gemini_host_timestamp_ns / d435i_host_timestamp_ns | host receive timestamps of both cameras (ns) |
| pairs[].gemini_device_timestamp / d435i_device_timestamp | device timestamps of both cameras |
| pairs[].gemini_frame_number / d435i_frame_number | frame counters of both cameras |

The same information is summarized at subject level in
`angles.<Vxxx>.attempts[].max_host_timestamp_skew_ms`.

### 1.2 Measured skew distribution (664 sessions)

| Metric | Value |
| --- | --- |
| Pairs per session | always 5 (664/664) |
| Max host-timestamp skew · minimum | 9.00 ms |
| Max host-timestamp skew · mean | 19.40 ms |
| Max host-timestamp skew · maximum | 73.86 ms |
| Frame-request skew · range | 0 – 94 ms |

Conclusion: the two cameras stay within **tens of milliseconds** of each other — suitable for static body posture
and shape analysis. For fast motion, resample or align per pair.

## 2. Calibration

Calibration is stored **per frame** (the `per_frame_calibration_v1` storage feature), i.e. every frame carries the
intrinsics and extrinsics in force at capture time rather than relying on one global calibration file.

### 2.1 Intrinsics (three sets per frame)

| Camera | Color | Raw depth | Aligned depth |
| --- | --- | --- | --- |
| C336L | 1280×720, distortion model orbbec (8 coefficients) | 848×480, distortion model orbbec (8 coefficients) | 1280×720, same as color |
| CD435I | 1280×720, distortion.inverse_brown_conrady (5 coefficients) | 1280×720, distortion.brown_conrady (5 coefficients) | 1280×720, same as color |

Fields: fx, fy, cx, cy, width, height, distortion_model, coefficients[].
Note the two cameras use different distortion models and coefficient counts — undistortion implementations are not interchangeable.

### 2.2 Extrinsics

| Camera | Provided extrinsics | Translation unit |
| --- | --- | --- |
| C336L | depth_raw_to_color | **millimeters** |
| CD435I | depth_raw_to_color, color_to_depth_raw | **meters** |

**This is the most common pitfall**: the field named `translation` uses different units on the two cameras.
Normalize before fusion.

### 2.3 Alignment semantics

- `depth_aligned` lives in the **color camera** coordinate system and is pixel-aligned with the color image
  (aligned intrinsics equal the color intrinsics).
- `depth_raw` lives in the **native depth camera** coordinate system and is not aligned.
- Point clouds are always in **millimeters**, generated with one point every 4 pixels.

### 2.4 Depth scale

| Camera | depth_scale_mm_per_unit |
| --- | --- |
| C336L | 1.0 |
| CD435I | 1.0000000474974513 |

The value is identical across all 6,640 depth-file entries of the 664 sessions, but it should still be read per frame rather than hard-coded.

## 3. Intra-camera timing contract

`capture_manifest.json` → `frames.<camera_code>[i].camera_metadata.frame_contract`:

| Field | Meaning |
| --- | --- |
| spatial_alignment.depth_aligned_matches_rgb_pixels | aligned depth is pixel-aligned with color |
| spatial_alignment.aligned_intrinsics_match_rgb | aligned depth intrinsics equal the color intrinsics |
| spatial_alignment.raw_to_color_extrinsics_present | native-depth-to-color extrinsics are provided |
| temporal_alignment.source | single SDK frameset (CD435I streams are same-source) |
| temporal_alignment.stream_timestamp_skew_ms / maximum_allowed_ms | intra-camera stream skew and its bound |
| temporal_alignment.raw_aligned_depth_frame_number_equal | raw and aligned depth share the frame number |

## 4. Devices and stream profiles (model level, identical in 664/664 sessions)

| Item | C336L | CD435I |
| --- | --- | --- |
| Model | Orbbec Gemini 336L | Intel RealSense D435I |
| Interface | USB 2.1 | USB 3.2 |
| Firmware | 1.6.00 | 5.15.1.55 |
| SDK | pyorbbecsdk 2.1.1 | pyrealsense2 2.54.2.5684 |
| Color stream | 1280×720 @10fps, MJPG | 1280×720 @30fps, rgb8 |
| Raw depth stream | 848×480 @10fps, Y16 | 1280×720 @30fps, z16 |
| Aligned depth stream | aligned to color, 1280×720 | aligned to color, 1280×720 |
| Timestamp source | device stream timestamp | color stream as primary clock |
| IR stream | not provided (no fabricated modalities) | not captured |

The whole dataset was captured with **one single pair of physical devices** (serial numbers / UIDs are not published),
so firmware and SDK versions are uniform throughout — there is no cross-device version drift.
