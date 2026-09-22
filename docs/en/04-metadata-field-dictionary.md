# 04 · Metadata Field Dictionary

This page describes the **fields, types and value domains** of the three JSON manifests. Concrete values
(anthropometric measurements, device serial numbers, local paths) are outside the publication scope —
see [08-publication-scope-and-privacy.md](08-publication-scope-and-privacy.md).

## 1. session_manifest.json (subject level, one per subject)

| Field | Type | Notes |
| --- | --- | --- |
| schema_version | string | subject manifest version, fixed `dual-rgbd-v2.2` |
| layout_version | string | directory layout version, fixed `readable-v1` |
| storage_features | string[] | enabled storage features |
| subject_id | string | subject identifier (S + 4 digits) |
| status | string | subject status, see §4 |
| created_at / completed_at | string | record creation / completion time (UTC, with offset) |
| output_directory / output_root | string | capture-side output paths (**not published**) |
| clothing_note | string | clothing note (empty string dataset-wide) |
| target_distance_mm | number | target capture distance in millimeters |
| angles | object | progress keyed by view (V000, V045 … V315) |
| anthropometry | object | measurements and QC outcome |
| integrity | object | integrity check result |
| reconciliation_required | boolean | whether reconciliation is required |
| recovery_report | object \| null | recovery report |
| completion | object | completion decision |

### 1.1 angles.<Vxxx>

| Field | Type | Notes |
| --- | --- | --- |
| yaw_deg | number | azimuth of the view (0/45/…/315) |
| status | string | view status, value `CAPTURED` |
| attempts | array | capture attempts for this view |

attempts[] element:

| Field | Type | Notes |
| --- | --- | --- |
| attempt_id | string | attempt identifier, equal to the session directory name `capture_<nn>_<UTC>` |
| path | string | relative path angles/angle_xxx_yyy/capture_... |
| captured_at | string | capture time (UTC) |
| capture_manifest_sha256 | string | SHA-256 of that session's capture_manifest.json (tamper anchor) |
| max_host_timestamp_skew_ms | number | maximum cross-camera host-timestamp skew of that session (ms) |

### 1.2 anthropometry

| Field | Type | Notes |
| --- | --- | --- |
| status / complete | string / boolean | measurement completion state |
| saved_at | string | time the measurements were saved |
| policy_version | string | policy version, fixed `five-measurement-qc-v1` |
| records | array | per-measurement records (5 items) |
| missing_required / review_required | string[] | missing required / pending review items (empty dataset-wide) |

records[] element:

| Field | Type | Notes |
| --- | --- | --- |
| measurement_id | string | item id, e.g. M01 |
| field_name | string | measurement field name (see §4) |
| m1 / m2 | number | 1st and 2nd readings (**values not published**) |
| first_two_difference | number | difference between the first two readings |
| third_measurement_threshold | number | threshold that triggers a 3rd reading (cm) |
| third_measurement_required | boolean | whether a 3rd reading was required |
| selected_trial_indices | number[] | indices of the accepted readings |
| selected_difference / closest_pair_difference | number | difference of accepted readings / of the closest pair |
| reduction_rule | string | reduction rule, see §4 |
| final_value | number | final value (**not published**) |
| qc_status | string | QC outcome for the item, see §4 |

### 1.3 integrity / completion

| Field | Type | Notes |
| --- | --- | --- |
| integrity.status | string | `OK` means the integrity check passed |
| integrity.errors | string[] | error list (empty dataset-wide) |
| integrity.checked_at | string | check time |
| completion.can_complete / completed | boolean | completable / completed |
| completion.completed_at / status | string | completion time and status |
| completion.blockers | string[] | blockers (empty dataset-wide) |

## 2. capture_manifest.json (session level, one per session)

| Field | Type | Notes |
| --- | --- | --- |
| schema_version | string | `dual-capture-v1.1` |
| storage_features | string[] | enabled storage features |
| attempt_id / subject_id / group_id | string | session id / subject / view key |
| yaw_deg / angle_name | number / string | azimuth and direction name |
| captured_at | string | capture time (UTC) |
| cameras | string[] | camera codes, `["C336L","CD435I"]` |
| frames | object | per-frame records grouped by camera code (5 entries each) |
| metadata | object | capture conditions |
| sync_audit | object | timing audit |
| pointcloud | object | point cloud conventions |
| files | array | all 80 files with per-file SHA-256 |

### 2.1 frames.<camera_code>[i]

| Field | Type | Notes |
| --- | --- | --- |
| frame_index | number | frame index 1–5 |
| host_timestamp_ns | number | host receive timestamp (ns) |
| device_timestamp | number | device timestamp (unit given by timestamp_unit) |
| frame_number | number | device frame counter |
| depth_scale_mm_per_unit | number | per-frame depth-unit → millimeter factor |
| stream_timestamps | object | per-stream timestamps (color / depth_raw / depth_aligned) |
| stream_frame_numbers | object | per-stream frame counters |
| intrinsics | object | three sets: color, depth_raw, depth_aligned |
| extrinsics | object | depth_raw_to_color (CD435I also exposes color_to_depth_raw) |
| camera_metadata | object | device info, stream profiles, runtime controls, frame contract |

Intrinsics entry fields: fx, fy, cx, cy, width, height, distortion_model, coefficients[].
Extrinsics entry fields: source, target, rotation[9], translation[3], translation_unit.

### 2.2 metadata / sync_audit / pointcloud

| Field | Notes |
| --- | --- |
| metadata.distance_mm | actual capture distance (mm) |
| metadata.ready_confirmed_at | readiness confirmation time |
| metadata.framing_policy | framing policy per camera (keys C336L / CD435I) |
| sync_audit.synchronization_kind | fixed `host_clock_near_sync` |
| sync_audit.pair_count | number of paired frames (always 5) |
| sync_audit.max_host_timestamp_skew_ms | maximum host-timestamp skew of the session (ms) |
| sync_audit.pairs[] | per pair: request_start_skew_ms, host_timestamp_skew_ms, host/device timestamps and frame counters of both cameras |
| pointcloud.format | fixed `binary_little_endian_ply` |
| pointcloud.coordinate_unit | fixed `millimeter` |
| pointcloud.color_order | fixed `RGB` |
| pointcloud.pixel_stride | fixed 4 (one point every 4 pixels) |

### 2.3 files[]

Field sets differ slightly by modality:

| Field | Modalities | Notes |
| --- | --- | --- |
| camera_code / camera_name | all | camera code and model name |
| frame | all | frame_01 … frame_05 |
| modality | all | modality key (8 kinds) |
| logical_modality | *_npy only | folds back to depth_raw / depth_aligned |
| modality_name | all | human-readable modality name |
| path | all | path relative to the session directory |
| format / dtype / shape | all except point cloud | format, data type, shape |
| order / allow_pickle | *.npy only | C order / pickle disallowed |
| size_bytes / sha256 | all | file size and content hash |
| depth_scale_mm_per_unit / value_semantics | *_npy only | depth factor and value semantics |
| array_channel_order / file_color_space / png_decoder_channel_order | rgb_color only | channel order / file color space / decoder note |

## 3. commit.json (session-level commit record, one per session)

| Field | Type | Notes |
| --- | --- | --- |
| schema_version | string | `dual-commit-v1.0` |
| status | string | `COMMITTED` |
| attempt_id / subject_id / group_id / yaw_deg | — | same source as the session manifest |
| committed_at | string | commit time (UTC) |
| capture_manifest | object | path and SHA-256 of the committed manifest |
| file_count | number | number of files (always 80) |
| files[] | array | copy of the file list, same shape as capture_manifest.files |

commit.json provides **double-write anchoring**: the manifest hash is recorded in the subject manifest and the
file list is stored again in a commit record, so either side tampering is detectable.

## 4. Enumerations and fixed values

| Field | Value | Dataset-wide distribution |
| --- | --- | --- |
| status (subject / completion) | COMPLETE | 83 |
| angles.*.status | CAPTURED | 664 |
| integrity.status | OK | 83 |
| anthropometry.status | COMPLETE | 83 |
| qc_status | PASS_2 | 82 subjects (all items) |
| qc_status | PASS_3 | 1 subject (some items) |
| reduction_rule | MEAN_FIRST_TWO | 82 subjects |
| reduction_rule | MEAN_CLOSEST_PAIR | 1 subject |
| measurement field_name | height_cm, biacromial_breadth_cm, nipple_chest_circumference_cm, midpoint_waist_circumference_cm, max_hip_circumference_cm | 5 items per subject |
| third_measurement_threshold | 1 (cm) | identical for all 5 items |
| storage_features | depth_npy_uint16_v1, per_frame_calibration_v1, durable_commit_v1 | 664 |
| framing_policy | full_body_required (C336L), auxiliary_fov_limited_non_blocking (CD435I) | 664 |
| pointcloud.format / coordinate_unit / color_order / pixel_stride | binary_little_endian_ply / millimeter / RGB / 4 | 664 |
| synchronization_kind | host_clock_near_sync | 664 |
| modality (keys) | rgb, depth_raw, depth_raw_npy, depth_raw_color, depth_aligned, depth_aligned_npy, depth_aligned_color, pointcloud_color_xyz_mm | 3,320 each |
