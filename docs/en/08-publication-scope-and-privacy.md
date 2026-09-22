# 08 · Publication Scope and Privacy

## 1. What this repository publishes

| Published | Content |
| --- | --- |
| ✅ | Directory layout, naming rules, field dictionary |
| ✅ | File formats, bit depth, shapes, units and semantics |
| ✅ | Aggregate scale statistics (file counts, sizes, completeness) and quality outcome distributions |
| ✅ | Synchronization mechanism, skew distribution, calibration conventions and pitfalls |
| ✅ | Reading instructions and usage notes |

## 2. What this repository does not publish

| Not published | Reason |
| --- | --- |
| ❌ Raw RGB / depth / point cloud files | Contain subject appearance information |
| ❌ Pseudo-color previews | Same, and easily recognizable visually |
| ❌ Anthropometric values (m1/m2/final_value …) | Personal body measurements |
| ❌ The actual subject ID list, its start/end range, and per-person mappings | Enables cross-dataset re-identification |
| ❌ Device serial numbers, UIDs, unique identifiers | Device fingerprint |
| ❌ `output_directory` / `output_root` and other absolute local paths | Exposes the capture environment |
| ❌ Per-session UTC timestamps and per-session skew details | Weak identifiers that narrow the search space |
| ❌ Verbatim copies of the three JSON manifests | Carrier of the fields above |

## 3. Where the sensitive fields live (self-audit list for the data owner)

If the raw manifests are ever shared, at minimum remove or hash the following:

| File | Fields |
| --- | --- |
| session_manifest.json | output_directory, output_root, anthropometry.records[].m1 / m2 / final_value, subject_id |
| capture_manifest.json | frames[].camera_metadata.device.serial_number / uid / id, attempt_id, captured_at |
| commit.json | subject_id, attempt_id, committed_at |

The repository `.gitignore` already blocks the `subjects/` tree, the three manifest file names and common data
file types, preventing accidental commits.

## 4. If illustrative material must be published

- Publish **texture-free renderings** only (point cloud or skeleton), never color-textured results;
- Crop the background and any identifiable environment;
- Avoid the face region, or blur/crop it;
- Do not attach subject IDs or timestamps to individual assets.

## 5. Compliance note

The data is intended for research and algorithm validation. Confirm compliance with your institution's ethics
review and data-governance requirements, and do not use it for individual identification or commercial
redistribution (see LICENSE for the governing terms).
