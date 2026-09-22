# 02 · Directory Layout and Naming

## 1. Hierarchy

```
subjects/
└── <subject_id>/
    ├── session_manifest.json
    ├── .staging/                       # staging area used during capture; empty on delivery
    └── angles/
        └── angle_<yaw>_<direction>/
            └── capture_<nn>_<UTC>/
                ├── capture_manifest.json
                ├── commit.json
                ├── camera_gemini_336l/
                │   └── <modality>/frame_<nn>.<ext>
                └── camera_realsense_d435i/
                    └── <modality>/frame_<nn>.<ext>
```

Fixed depth of 6 levels: subject → angle → capture → camera → modality → frame file. All directory names are ASCII.

## 2. Naming rules

| Level | Pattern | Notes |
| --- | --- | --- |
| Subject | `S` + 4 decimal digits | e.g. the form `S0005`; the actual ID list is not published |
| Angle directory | `angle_%03d_%s` | three-digit azimuth + English direction |
| Capture directory | `capture_%02d_%s` | two-digit attempt index + UTC timestamp (ISO 8601 basic format, microseconds + Z) |
| Camera directory | `camera_<model_slug>` | see §4 |
| Modality directory | `<modality>` | 8 kinds, see [03-file-formats-and-modalities.md](03-file-formats-and-modalities.md) |
| Frame file | `frame_%02d.<ext>` | frame_01 … frame_05 |
| Metadata | `session_manifest.json` / `capture_manifest.json` / `commit.json` | fixed names |

Example UTC timestamp shape: `20260906T103235960681Z` (YYYYMMDDThhmmssffffffZ).

## 3. The eight views

Convention: the **front view is 0°**, increasing 45° **clockwise** when seen from above.

| Directory | yaw_deg | Direction | Meaning |
| --- | --- | --- | --- |
| angle_000_front | 0 | front | front |
| angle_045_front_right | 45 | front_right | front-right |
| angle_090_right | 90 | right | right |
| angle_135_back_right | 135 | back_right | back-right |
| angle_180_back | 180 | back | back |
| angle_225_back_left | 225 | back_left | back-left |
| angle_270_left | 270 | left | left |
| angle_315_front_left | 315 | front_left | front-left |

All 8 views exist for every subject (83/83), each with exactly one successful attempt — there are no re-capture directories.

## 4. Camera directories and codes

| Directory | camera_code | Model | Role |
| --- | --- | --- | --- |
| camera_gemini_336l | C336L | Orbbec Gemini 336L | primary view, full body required (`full_body_required`) |
| camera_realsense_d435i | CD435I | Intel RealSense D435I | auxiliary view, FOV-limited and non-blocking (`auxiliary_fov_limited_non_blocking`) |

Within a session both cameras hold 5 frames, paired one-to-one as frame_01 … frame_05.

## 5. Space per directory

- Subject directory: 657 files, ≈ 509 MiB
- View directory: 82 files (80 data + 2 manifests), ≈ 63 MiB
- Modality directory: 5 files (frame_01 … frame_05)
