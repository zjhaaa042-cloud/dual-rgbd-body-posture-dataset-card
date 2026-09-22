# Dual-Camera 8-View Body-Posture RGB-D Dataset · Data Card

> This repository **documents the dataset only**. It contains **no raw data**
> — no images, no depth maps, no point clouds, no anthropometric values, no device identifiers.

[中文说明](README.md) · [Documentation index](#3-documentation-index)

---

## 1. What this is

A multi-view RGB-D dataset for body-posture and body-shape research. The subject stands still while
a **pair of depth cameras** (Orbbec Gemini 336L and Intel RealSense D435I) capture the body from
**8 azimuth angles** (one every 45°, a full turn), recording **5 RGB-D frames** per view.

- Capture mode: two cameras at a fixed station, view rotated between captures (not a camera array, not hardware-triggered)
- Synchronization: **host-clock near sync** (`host_clock_near_sync`), with per-pair timing audit
- Per frame: color image, raw depth, aligned depth, pseudo-color preview, point cloud, plus intrinsics/extrinsics/device metadata
- Integrity: every session carries SHA-256 manifests and a commit record, verifiable offline

## 2. Scale at a glance

| Item | Value |
| --- | --- |
| Subjects | 83 |
| Views per subject | 8 (0° / 45° / … / 315°) |
| Capture sessions | 664 = 83 × 8 |
| Cameras per session | 2 |
| Frames per camera | 5 |
| Modalities | 8 per camera |
| Data files | 53,120 |
| Metadata files | 1,411 (83 subject-level + 664 × 2 session-level) |
| Total files | 54,532 |
| Total size | 44,280,472,033 B ≈ 41.24 GiB |
| Per subject | 657 files / ≈ 509 MiB |
| Per session | 82 files / ≈ 63 MiB |
| Capture window | 2026-09-06 — 2026-09-18 (7 capture days) |
| Completeness | 83/83 subjects complete, 8/8 views present, 664/664 sessions intact |

Detailed statistics: [docs/en/01-data-scale-and-statistics.md](docs/en/01-data-scale-and-statistics.md).

## 3. Documentation index

| Document | Content |
| --- | --- |
| [docs/en/01-data-scale-and-statistics.md](docs/en/01-data-scale-and-statistics.md) | File/size distribution, completeness, capture window and distance settings |
| [docs/en/02-directory-layout-and-naming.md](docs/en/02-directory-layout-and-naming.md) | Directory depth, the 8 view names and 2 camera codes |
| [docs/en/03-file-formats-and-modalities.md](docs/en/03-file-formats-and-modalities.md) | Format, dtype, shape, unit and semantics of all 8 modalities |
| [docs/en/04-metadata-field-dictionary.md](docs/en/04-metadata-field-dictionary.md) | Field-by-field dictionary of the three JSON manifests |
| [docs/en/05-synchronization-and-calibration.md](docs/en/05-synchronization-and-calibration.md) | Sync mechanism, timing audit, intrinsics/extrinsics, alignment |
| [docs/en/06-quality-assurance-and-integrity.md](docs/en/06-quality-assurance-and-integrity.md) | Verification chain, capture admission, anthropometric QC policy |
| [docs/en/07-reading-and-usage-guide.md](docs/en/07-reading-and-usage-guide.md) | Read order, code samples, mm conversion, pitfalls |
| [docs/en/08-publication-scope-and-privacy.md](docs/en/08-publication-scope-and-privacy.md) | What is / is not published, anonymization notes |
| [assets/directory-tree.txt](assets/directory-tree.txt) | Directory tree with placeholders |

中文版文档见 [README.md](README.md) 与 `docs/`。

## 4. Schema versions

| Object | Field | Value |
| --- | --- | --- |
| Subject manifest | schema_version / layout_version | dual-rgbd-v2.2 / readable-v1 |
| Capture manifest | schema_version | dual-capture-v1.1 |
| Commit record | schema_version | dual-commit-v1.0 |
| Frame contract | frame_contract.schema_version | rgbd-frame-contract-v1 |
| Anthropometry policy | anthropometry.policy_version | five-measurement-qc-v1 |
| Storage features | storage_features | depth_npy_uint16_v1, per_frame_calibration_v1, durable_commit_v1 |

## 5. Capture hardware (model level)

| Code | Model | Color stream | Raw depth stream | Aligned depth stream | Primary clock |
| --- | --- | --- | --- | --- | --- |
| C336L | Orbbec Gemini 336L | 1280×720 @10fps (MJPG) | 848×480 @10fps (Y16) | 1280×720 @10fps | device stream timestamp |
| CD435I | Intel RealSense D435I | 1280×720 @30fps (rgb8) | 1280×720 @30fps (z16) | 1280×720 @30fps (z16) | color stream |

C336L is the **primary** view (full body required in frame); CD435I is the **auxiliary** view
(limited field of view, non-blocking). Serial numbers and UIDs are **not published**.

## 6. Prerequisites and limitations

- The two cameras are **host-clock near-synced**, not hardware-triggered. Read `sync_audit` before any cross-camera fusion.
- Depth values are in **sensor units**; multiply by the per-frame `depth_scale_mm_per_unit` to get millimeters.
- C336L provides no IR stream; the acquisition layer never fabricates a missing modality.
- For research and algorithm validation only. Confirm compliance with your institution's ethics and data-governance rules before use.

## 7. License

MIT — see [LICENSE](LICENSE).
