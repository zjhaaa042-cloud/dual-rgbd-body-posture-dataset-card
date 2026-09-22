# 06 · Quality Assurance and Integrity

## 1. Verification chain (three-way anchoring)

| Level | Content | Purpose |
| --- | --- | --- |
| File | `files[]` in capture_manifest.json records SHA-256 and size per file | per-file content verification |
| Manifest | `angles.<Vxxx>.attempts[].capture_manifest_sha256` in the subject manifest | prevents manifest substitution |
| Commit | a separate `commit.json` per session (status = COMMITTED) with the manifest hash and a full file-list copy | cross-checked double write |

Supporting storage features:

| Feature | Meaning |
| --- | --- |
| depth_npy_uint16_v1 | depth is provided both as lossless uint16 PNG and as an equivalent NumPy array |
| per_frame_calibration_v1 | intrinsics/extrinsics are written per frame, no global calibration file needed |
| durable_commit_v1 | session-level commit record; committing anchors the session |

## 2. Integrity results (dataset-wide)

| Check | Result |
| --- | --- |
| integrity.status = OK | 83 / 83 |
| integrity.errors empty | 83 / 83 |
| reconciliation_required = false | 83 / 83 |
| recovery_report = null | 83 / 83 |
| completion.blockers empty | 83 / 83 |
| Files per session = 80 | 664 / 664 |
| Frames per camera = 5 | 664 / 664 |
| Sync pairs per session = 5 | 664 / 664 |
| Leftover files in .staging | 0 (all 83 directories empty) |
| Unexpected file types | 0 |

The delivery is **sealed**: every session is committed, with no pending staging, no missing frames and no re-captures.

## 3. Capture admission controls

The following evidence is written before/while capturing, so "why is this frame usable" can be traced back:

| Field | Location | Notes |
| --- | --- | --- |
| metadata.ready_confirmed_at | session manifest | confirmation that subject and rig were ready |
| metadata.framing_policy | session manifest | C336L requires the full body in frame; CD435I is FOV-limited but non-blocking |
| metadata.distance_mm | session manifest | actual capture distance (mm) |
| camera_metadata.device.stream_preflight | session manifest | stream preflight (required modalities present, timeout) |
| camera_metadata.temporal_alignment.maximum_allowed_ms | frame contract | intra-camera stream-skew bound |
| camera_metadata.runtime_controls | session manifest | runtime controls (exposure/gain/laser …) and their read status |

## 4. Anthropometric QC (five-measurement-qc-v1)

Policy: up to 3 readings per measurement; the difference between the first two readings plus a threshold decides
whether a third reading is needed.

| Rule | Notes |
| --- | --- |
| third_measurement_threshold | difference between the first two readings above this threshold (cm) triggers the 3rd reading |
| qc_status = PASS_2 | first two readings already converged; two readings accepted |
| qc_status = PASS_3 | a third reading was required and completed |
| reduction_rule = MEAN_FIRST_TWO | default: mean of the first two readings |
| reduction_rule = MEAN_CLOSEST_PAIR | mean of the closest pair among three readings |

Measured items (5 per subject, always complete):

| measurement_id | field_name | Semantics |
| --- | --- | --- |
| M01 | height_cm | standing height (cm) |
| M02 | biacromial_breadth_cm | biacromial breadth (cm) |
| M03 | nipple_chest_circumference_cm | chest circumference (cm) |
| M04 | midpoint_waist_circumference_cm | waist circumference (cm) |
| M05 | max_hip_circumference_cm | hip circumference (cm) |

Dataset-wide QC outcome distribution (no values published):

| Outcome | Count |
| --- | --- |
| Subjects with all items PASS_2 | 82 |
| Subjects with some PASS_3 items | 1 |
| Subjects using MEAN_CLOSEST_PAIR | 1 |
| Missing required items | 0 |
| Items pending review | 0 |

## 5. Cross-session consistency

Comparing all 664 sessions, the following dimensions are **completely uniform**, which serves as evidence of a
single shared configuration:

- session manifest version (`dual-capture-v1.1`) and the storage_features set
- files per session (80) and frames per camera (5)
- resolution combination per modality (C336L 848×480 / 1280×720; CD435I 1280×720 everywhere)
- intrinsics layout (distortion model and resolution of the color / raw depth / aligned depth sets)
- extrinsics entries and their units
- depth scale values, point cloud format and downsample stride
- framing policy and synchronization kind
