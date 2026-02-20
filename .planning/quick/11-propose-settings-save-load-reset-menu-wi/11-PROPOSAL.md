# Proposal: Settings Save/Load/Reset Menu

## Goal

Add a compact header control to save all current settings to a JSON file, load them back, and reset to defaults with confirmation. This proposal documents UX, schema, storage scope, and behaviors before implementation.

## UX Placement and Behavior

- **Location:** Header, aligned to the right of the headline.
- **Control:** Small icon-only button (disk glyph). Clicking opens a dropdown menu.
- **Menu options (exact copy):**
  - "Save settings to file"
  - "Load settings from file"
  - "Reset settings"
- **Reset confirmation:** Modal confirm dialog with copy "Reset settings and clear stored pattern?" (default: Cancel).
- **Blocking rule:** Reset is blocked while `searchState === "running"` (mirror existing clear/optimize guards).
- **Interaction notes:** Menu closes after selection; clicking outside closes menu.

## Data Scope and JSON Schema

The saved file mirrors all values currently persisted in localStorage plus paint pattern data.

**localStorage keys covered:**

- `qr-art.url`
- `qr-art.version`
- `qr-art.rotation`
- `qr-art.random-rotation`
- `qr-art.color`
- `qr-art.pattern`

Proposed JSON structure (exported file):

```
{
  "url": "https://example.com",
  "version": 4,
  "rotation": 0,
  "randomRotation": false,
  "selectedColor": "black",
  "pattern": {
    "version": 4,
    "size": 33,
    "cells": [0, 1, 2, ...]
  }
}
```

Notes:

- `pattern.cells` uses the same serialization as localStorage (flattened grid array).
- Missing fields: keep existing runtime values unchanged (best-effort import).
- Extra fields: ignore.
- Version mismatch: allow, but only apply pattern if size matches computed grid size for that version.

## Export Flow

1. Build a snapshot from current runtime state and localStorage equivalents.
2. Create a JSON Blob and trigger download.
3. Filename: `qr-art-settings.json` (content type: `application/json`).

## Import Flow

1. Open file picker (accept `.json`, `application/json`).
2. Parse JSON; if invalid, show alert and do not modify state.
3. Apply safe fields in this order:
   - `url` (update input + validation UI + saveURL)
   - `version` (saveVersion + update select + initPaintCanvas)
   - `rotation` + `randomRotation` (use existing setters, avoid rerender while applying)
   - `selectedColor` (setSelectedColor)
   - `pattern` (if present and size matches version grid; replace paintGrid cells, savePattern)
4. Re-sync previews and results (runCorruptionPipeline, resetOptimizationResults).
5. Error handling: invalid JSON or schema mismatch → show alert/toast, leave current state intact.

## Reset Flow

1. Confirm dialog.
2. Clear all localStorage keys for URL, version, rotation, random rotation, selected color, pattern.
3. Reset runtime state to defaults and clear UI inputs.
4. Keep paint grid empty but initialized for the default version; persist empty pattern.
5. Error handling: if localStorage write fails, still reset runtime state and log warning.

## Touch Points (index.html)

- Header markup: add disk-icon button and dropdown menu near the headline.
- Styles: small icon button, dropdown panel, hover/focus states.
- JS handlers:
  - `exportSettingsToJson()`
  - `importSettingsFromJson()`
  - `resetSettingsWithConfirm()`

## Open Questions / Assumptions

1. Confirm the exact list of storage keys to include (STORAGE_KEYS.\*) and whether any ephemeral UI state should be excluded.
2. Confirm the disk icon glyph (SVG inline vs. existing icon font or emoji).
3. Confirm reset copy and whether it should mention URL/rotation explicitly.
4. Confirm whether import should prompt before overwriting an existing painted pattern.
5. Confirm export file name schema.
