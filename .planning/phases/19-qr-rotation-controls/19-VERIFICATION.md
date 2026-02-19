---
phase: 19-qr-rotation-controls
verified: 2026-02-19T20:00:00Z
status: human_needed
score: 4/4 must-haves verified
human_verification:
  - test: "Rotate paint preview 0/90/180/270"
    expected: "QR base + protected overlay rotate while painted pixels remain fixed; decode metrics refresh after each rotation"
    why_human: "Requires visual confirmation of overlay alignment and decode behavior"
  - test: "Rotate and inspect result thumbnails + hover overlays"
    expected: "Result previews and overlays reflect current rotation and stay aligned"
    why_human: "Canvas alignment and overlay visuals are not verifiable via static analysis"
  - test: "Export PNG/SVG at different rotations"
    expected: "Exports match the rotated previews without rotating painted pattern"
    why_human: "File output correctness requires rendering/inspection"
  - test: "Enable Random rotation and run Generate"
    expected: "Search uses per-attempt 0/90/180/270 rotations; previews reflect per-result rotation without showing angle"
    why_human: "Requires runtime behavior verification across multiple attempts"
---

# Phase 19: QR Rotation Controls Verification Report

**Phase Goal:** Users can rotate QR presentation and search without rotating the painted pattern.
**Verified:** 2026-02-19T20:00:00Z
**Status:** human_needed
**Re-verification:** No — initial verification

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
| --- | --- | --- | --- |
| 1 | User can rotate the QR code and protected overlay by 90° in the Paint Pattern section while the painted pattern orientation remains fixed. | ✓ VERIFIED | Paint section includes rotate control and indicator (index.html ~964-975). `renderQRWithOverlay` applies `createRotationMapper(qrRotation, ...)` to base modules and protected overlay while painted pixels use `paintGrid.get(x, y)` unrotated (index.html ~14316-14376). `rotateQrPreview` advances 90° and calls `setQrRotation` (index.html ~13683-13688). |
| 2 | User sees result previews and hover overlays reflect the current QR rotation. | ✓ VERIFIED | Result rendering uses rotation-aware data: `buildRotatedResultModuleData` remaps with rotation mapper (index.html ~16645-16674). Overlays render with `renderErrorOverlay(..., rotationDegrees)` and pass rotation from `qrRotation` or per-result rotation (index.html ~17085-17095, ~17247-17258). |
| 3 | User exports PNG/SVG that match the current QR rotation while preserving the painted pattern orientation. | ✓ VERIFIED | Export buttons build `rotatedModuleData` via `buildRotatedResultModuleData` using `qrRotation` or `result.rotation`, then call `renderResultToCanvas` for PNG and `generateSVG` for SVG (index.html ~17169-17210). |
| 4 | User can enable random rotation for Generate so searches apply 0/90/180/270 rotation to the QR only, reflected in the resulting previews. | ✓ VERIFIED | Random rotation toggle exists (index.html ~1011-1017) and persisted/used (index.html ~13667-13679, ~17540-17550). Worker receives `randomRotation` and `baseRotation` (index.html ~16391-16404) and applies per-attempt random rotation (index.html ~15935-15938) while returning `rotation` in results (index.html ~16034). Previews use `result.moduleData`/`result.rotation` when random mode enabled (index.html ~17067-17095). |

**Score:** 4/4 truths verified

### Required Artifacts

| Artifact | Expected | Status | Details |
| --- | --- | --- | --- |
| `index.html` | Rotation state, persistence, preview/result rotation, exports, random rotation integration | ✓ VERIFIED | Rotation state/load/save/reset, paint preview rotation, result rendering/overlays, export rotation, and worker random rotation are implemented in this file (multiple sections, see evidence above). |

### Key Link Verification

| From | To | Via | Status | Details |
| --- | --- | --- | --- | --- |
| `index.html` | `runCorruptionPipeline` | rotation change handler | ✓ WIRED | `setQrRotation` calls `runCorruptionPipeline()` when rotation changes (index.html ~13643-13661). |
| `index.html` | `saveRotation` | rotation state updates | ✓ WIRED | `setQrRotation` calls `saveRotation` when not random rotation (index.html ~13656-13657). |
| `index.html` | `renderResultToCanvas` | rotation-aware module mapping | ✓ WIRED | PNG export uses `buildRotatedResultModuleData` + `renderResultToCanvas` (index.html ~17169-17176). |
| `index.html` | `renderErrorOverlay` | rotation-aware overlay mapping | ✓ WIRED | Overlays render with `renderErrorOverlay(..., rotation)` in results and collapse/expand flows (index.html ~17085-17095, ~17247-17258). |
| `index.html` | `generateSVG` | rotation-aware SVG path | ✓ WIRED | SVG export uses rotated module data before `generateSVG` (index.html ~17200-17208, ~16853-16879). |
| `index.html` | `startOptimization` | rotation settings in worker message | ✓ WIRED | `startOptimization` sends `baseRotation` and `randomRotation` in START_OPTIMIZATION (index.html ~16391-16404) and worker reads them (index.html ~16164-16168). |

### Requirements Coverage

| Requirement | Source Plan | Description | Status | Evidence |
| --- | --- | --- | --- | --- |
| ROT-01 | 19-01-PLAN | Paint Pattern rotate 90° control rotates QR + protected overlay only | ✓ SATISFIED | Rotate control UI + rotation-mapped base/protected overlay while paint stays fixed (index.html ~964-975, ~14316-14376). |
| ROT-02 | 19-02-PLAN | Results previews and hover overlays respect current QR rotation | ✓ SATISFIED | Result preview module mapping + overlay rotation usage (index.html ~16645-16674, ~17085-17095). |
| ROT-03 | 19-02-PLAN | PNG/SVG exports reflect current QR rotation while preserving painted pattern | ✓ SATISFIED | PNG/SVG export uses rotated module data with `buildRotatedResultModuleData` (index.html ~17169-17210). |
| ROT-04 | 19-03-PLAN | Generate random rotation 0/90/180/270 applied to QR only during search | ✓ SATISFIED | Random rotation toggle, START_OPTIMIZATION includes `randomRotation`, worker applies per-attempt rotation and returns rotation (index.html ~1011-1017, ~16391-16404, ~15935-16034). |

**Orphaned requirements:** None detected for Phase 19 (ROT-01..ROT-04 all mapped in plans).

### Anti-Patterns Found

None.

### Human Verification Required

### 1. Rotate paint preview 0/90/180/270

**Test:** Click rotate control through all angles. Paint a pattern and observe preview.
**Expected:** QR base + protected overlay rotate while painted pixels remain fixed; decode metrics refresh after each rotation.
**Why human:** Visual alignment and decode correctness require runtime inspection.

### 2. Rotate and inspect result thumbnails + hover overlays

**Test:** Generate results, change rotation, and hover/expand cards.
**Expected:** Thumbnails and overlays rotate/align with the current rotation or per-result rotation in random mode.
**Why human:** Canvas rendering/overlay alignment is not provable statically.

### 3. Export PNG/SVG at different rotations

**Test:** Export PNG/SVG at multiple rotations.
**Expected:** Exports match previews and keep painted pixels orientation fixed.
**Why human:** File output correctness requires inspection.

### 4. Enable Random rotation and run Generate

**Test:** Enable Random rotation and run multiple Generate searches.
**Expected:** Attempts use 0/90/180/270 rotations; previews reflect per-result rotation; UI does not show angle.
**Why human:** Randomized runtime behavior cannot be verified statically.

### Gaps Summary

No code-level gaps detected. Runtime/visual verification is required to confirm the rotation behavior, overlays, and exports align with expectations.

---

_Verified: 2026-02-19T20:00:00Z_
_Verifier: Claude (gsd-verifier)_
