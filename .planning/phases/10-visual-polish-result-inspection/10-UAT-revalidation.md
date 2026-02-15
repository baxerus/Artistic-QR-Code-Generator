---
status: diagnosed
phase: 10-visual-polish-result-inspection
source: [10-02-SUMMARY.md]
started: 2026-02-15T18:00:00Z
updated: 2026-02-15T18:10:00Z
---

## Current Test

[testing complete]

## Tests

### 1. Blue Overlay Visibility (Fix Verification)
expected: Open the app and generate a QR code. The blue protected area overlay on the paint canvas should now be clearly visible — more opaque than before (opacity increased from 0.15 to 0.25).
result: pass

### 2. Immediate Overlay Transition (Fix Verification)
expected: Hover over a small result in the Top 5 list. The overlay should appear immediately (0.1s transition) without any noticeable delay. It should feel responsive, not sluggish.
result: pass

### 3. No Legend on Small Result Hover (Fix Verification)
expected: Hover over a small (non-expanded) result in the Top 5 list. The overlay appears but NO legend text should appear (no 'Match', 'Conflict', 'Function' labels). The result card should NOT change height or cause the page to visually jump.
result: pass

### 4. Expanded Result Hover Behavior (Fix Verification)
expected: Click a result to expand it. When expanded, the overlay should NOT be visible by default. Only when you hover over the expanded result should the overlay (with legend) appear. Moving mouse away hides the overlay.
result: issue
reported: "The overlay is shown only on hover, that is correct. BUT the blue overlay for function parts is not shown over to correct parts. Or better it is bleeding into parts that are not functional parts. It looks like blur because of maybe scaling? In general there shouldn't be different shades of blue overlay (what is the case in the moment). Also the blue functional parts should have a dashed outline as the overlay on top of the paint pattern, but there is no dashed outline at all"
severity: major

### 5. Visible Dashed Borders on Hover (Fix Verification)
expected: Hover over any result (small or expanded). The blue protected areas should show dashed border outlines that are clearly visible — not too thin to see.
result: issue
reported: "No there is no dashed blue line (in neither small or expanded)"
severity: major

## Summary

total: 5
passed: 3
issues: 2
pending: 0
skipped: 0

## Gaps

- truth: "Blue overlay covers exactly the protected/functional areas with dashed outline borders"
  status: failed
  reason: "User reported: The overlay is shown only on hover, that is correct. BUT the blue overlay for function parts is not shown over to correct parts. Or better it is bleeding into parts that are not functional parts. It looks like blur because of maybe scaling? In general there shouldn't be different shades of blue overlay (what is the case in the moment). Also the blue functional parts should have a dashed outline as the overlay on top of the paint pattern, but there is no dashed outline at all"
  severity: major
  test: 4
  root_cause: "renderErrorOverlay missing ctx.imageSmoothingEnabled = false (line 15855). Canvas anti-aliasing causes blue fill to blur across module boundaries when CSS-scaled from 21px to 120px+."
  artifacts:
    - path: "index.html"
      issue: "Missing imageSmoothingEnabled = false in renderErrorOverlay (line 15855)"
  missing:
    - "Add ctx.imageSmoothingEnabled = false after getting context in renderErrorOverlay"
  debug_session: ".planning/debug/blue-overlay-bleeding.md"

- truth: "Blue dashed border outlines visible around protected areas on hover"
  status: failed
  reason: "User reported: No there is no dashed blue line (in neither small or expanded)"
  severity: major
  test: 5
  root_cause: "renderProtectedBordersOverlay uses lineWidth 0.5 and dash [2,1] on module-scale canvas (~21px). Sub-pixel strokes get anti-aliased to invisibility when CSS-scaled to 120px."
  artifacts:
    - path: "index.html"
      issue: "lineWidth 0.5 too small (line 15896)"
    - path: "index.html"
      issue: "setLineDash([2, 1]) too small for module edges (line 15897)"
  missing:
    - "Increase lineWidth to 1-2 pixels (visible after scaling)"
    - "Adjust dash pattern proportional to module size or use smaller fractions like [0.3, 0.15]"
    - "Alternative: render overlay at higher resolution matching display size"
  debug_session: ".planning/debug/missing-dashed-borders.md"
