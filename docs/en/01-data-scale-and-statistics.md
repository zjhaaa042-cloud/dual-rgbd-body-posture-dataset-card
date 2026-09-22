# 01 · Data Scale and Statistics

All figures on this page are **dataset-wide aggregates**. No per-subject data content is included.

## 1. Totals

| Metric | Value |
| --- | --- |
| Subjects | 83 |
| Subject IDs | one contiguous numeric block, 83 IDs (the ID list is not published) |
| Views per subject | 8 |
| Capture sessions | 664 |
| Cameras per session | 2 |
| Frames per camera per session | 5 |
| Data files | 53,120 |
| Metadata files | 1,411 |
| Total files | 54,532 |
| Total bytes | 44,280,472,033 B |
| Total size | ≈ 41.24 GiB (≈ 44.28 GB) |

Data files = 664 sessions × 2 cameras × 5 frames × 8 modalities = 53,120.
Metadata files = 83 subject-level manifests + 664 × 2 session-level manifests (`capture_manifest.json`, `commit.json`) = 1,411.

## 2. By file type

| Extension | Files | Bytes | Size |
| --- | --- | --- | --- |
| .npy | 13,280 | 21,062,717,440 | 19.62 GiB |
| .png | 33,200 | 18,005,289,067 | 16.77 GiB |
| .ply | 6,640 | 5,106,238,745 | 4.76 GiB |
| .json | 1,411 | 106,225,601 | 101.3 MiB |
| .txt | 1 | 1,180 | 1.2 KiB |

NPY dominates in size (uncompressed uint16 arrays); PNG dominates in count (lossless 16-bit depth maps plus 8-bit color and preview images).

## 3. By modality directory (6,640 files each = 664 × 2 cameras × 5 frames)

| Directory | Files | Size |
| --- | --- | --- |
| depth_aligned_npy | 6,640 | 11.40 GiB |
| depth_raw_npy | 6,640 | 8.22 GiB |
| rgb_color | 6,640 | 5.53 GiB |
| pointcloud_color_xyz_mm | 6,640 | 4.76 GiB |
| depth_aligned_uint16 | 6,640 | 3.23 GiB |
| depth_raw_uint16 | 6,640 | 2.94 GiB |
| depth_aligned_color | 6,640 | 2.62 GiB |
| depth_raw_color | 6,640 | 2.44 GiB |
| **Total** | **53,120** | **≈ 41.14 GiB** |

Aligned depth (1280×720) costs more than raw depth (848×480 on C336L). Pseudo-color previews are for viewing only and carry no quantitative meaning.

## 4. Per-subject distribution (83 subjects)

| Metric | Value |
| --- | --- |
| Files | exactly 657 per subject (8 sessions × 80 data files + 8 × 2 manifests + 1 subject manifest) |
| Average size | ≈ 509 MiB |
| Minimum size | 507,025,658 B (≈ 483.5 MiB) |
| Maximum size | 547,802,897 B (≈ 522.4 MiB) |

The spread comes from image content (lossless PNG compression varies with texture) and from the number of valid points, not from missing frames.

## 5. Per-session composition (identical across all 664 sessions)

| Item | Value |
| --- | --- |
| Data files | 80 = 2 cameras × 5 frames × 8 modalities |
| Metadata files | 2 (`capture_manifest.json`, `commit.json`) |
| Average size | ≈ 63 MiB |
| Cameras | C336L and CD435I, 5 frames each |
| Sync pairs | 5 (one pair per frame) |

## 6. Capture window

| Item | Value |
| --- | --- |
| Time base | UTC |
| Earliest session | 2026-09-06T10:32:35Z |
| Latest session | 2026-09-18T10:17:15Z |
| Capture days | 2026-09-06 / 09-10 / 09-11 / 09-12 / 09-16 / 09-17 / 09-18 (7 days) |

## 7. Capture settings

| Parameter | Value | Subjects |
| --- | --- | --- |
| target_distance_mm | 2500 | 78 |
| target_distance_mm | 2300 | 5 |
| View step | 45° | 83 |
| Frames per view | 5 | 83 |

Distance settings differ by batch; the per-session value is stored in `metadata.distance_mm`.

## 8. Completeness

| Check | Result |
| --- | --- |
| Subject status = COMPLETE | 83 / 83 |
| All 8 views CAPTURED | 83 / 83 |
| Exactly 1 attempt per view | 83 / 83 |
| integrity.status = OK | 83 / 83 |
| anthropometry.status = COMPLETE | 83 / 83 |
| Missing required measurements | 0 |
| Items pending review | 0 |
| reconciliation_required = true | 0 |
| recovery_report present | 0 |
| Sessions with 80 files | 664 / 664 |
| Sessions with 5 frames per camera | 664 / 664 |
| Consistent versions / resolutions / calibration layout | 664 / 664 |

In short: a complete dataset with **no missing views, no missing frames and no re-captures**.
