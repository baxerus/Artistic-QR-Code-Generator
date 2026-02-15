---
status: diagnosed
trigger: "Missing dashed blue borders around protected areas on hover"
created: 2026-02-15T21:00:00Z
updated: 2026-02-15T21:15:00Z
---

## Current Focus

hypothesis: Line width and dash pattern are too small for the canvas-to-display scaling
test: Confirmed by code analysis
expecting: lineWidth and dash pattern need to scale with display size
next_action: Return diagnosis - ROOT CAUSE FOUND

## Symptoms

expected: Blue dashed border outlines visible around protected areas on hover
actual: "No there is no dashed blue line (in neither small or expanded)"
errors: None reported
reproduction: Test 5 in UAT - Hover over any result (small or expanded)
started: Current implementation

## Eliminated

(none - root cause found on first hypothesis)

## Evidence

- timestamp: 2026-02-15T21:05:00Z
  checked: Overlay canvas creation (lines 16087-16091)
  found: Canvas created with dimensions `moduleCount x moduleCount` (e.g., 21x21 pixels)
  implication: Each QR module = 1 pixel in canvas coordinate space

- timestamp: 2026-02-15T21:07:00Z
  checked: CSS canvas scaling (lines 703-711)
  found: Overlay canvas has `width: 100%` inside 120px container, with `image-rendering: pixelated`
  implication: Canvas is scaled up ~5-6x (from ~21px to 120px) via CSS

- timestamp: 2026-02-15T21:10:00Z
  checked: renderProtectedBordersOverlay function (lines 15893-15936)
  found: `ctx.lineWidth = 0.5` and `ctx.setLineDash([2, 1])`
  implication: Drawing 0.5px lines with 2px dash / 1px gap on a 21px canvas

- timestamp: 2026-02-15T21:12:00Z
  checked: Line drawing coordinates
  found: `ctx.moveTo(x, y)` to `ctx.lineTo(x + 1, y)` - 1 pixel lines in canvas space
  implication: Lines are sub-pixel (0.5px wide) and dash pattern spans only ~3 modules total

- timestamp: 2026-02-15T21:14:00Z
  checked: PROTECTED_OUTLINE_COLOR constant (line 13582)
  found: `const PROTECTED_OUTLINE_COLOR = "#3b82f6";` (blue)
  implication: Color is correct, but lines are not visible due to scale issues

## Resolution

root_cause: The `renderProtectedBordersOverlay` function draws lines with `lineWidth: 0.5` and `setLineDash([2, 1])` on a canvas that is only `moduleCount` pixels (e.g., 21px for version 1 QR code). When this canvas is scaled up to 120px via CSS, the 0.5px lines become ~2.5px when scaled, but more critically, the dash pattern of [2, 1] means only 2 pixels of dash followed by 1 pixel of gap - at the tiny canvas scale, this creates dashes that are effectively invisible sub-pixel segments. The dashes get anti-aliased into near-transparency when the CSS scales up the canvas with `image-rendering: pixelated`.

fix: (not applied - goal is find_root_cause_only)
verification: (not applied)
files_changed: []
