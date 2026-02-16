---
phase: quick-4
plan: 01
subsystem: paint-canvas
tags: [painting, functional-patterns, qr-code]
dependency_graph:
  requires: []
  provides: [functional-pattern-painting]
  affects: [paint-canvas, optimization-worker, qr-display]
tech_stack:
  added: []
  patterns: []
key_files:
  created: []
  modified: [index.html]
decisions:
  - Removed all protected area restrictions from painting
  - Functional patterns can now be painted over like any other cell
metrics:
  duration: 57s
  completed: 2026-02-16
---

# Quick Task 4: Allow Painting Into Functional Patterns Summary

Enabled painting into QR functional patterns (finder, timing, alignment) for full artistic control over all pixels.

## What Changed

### Task 1: Remove painting restrictions (0737b1d)

Removed six protective checks that blocked painting in functional areas:

1. **`paintAt()`**: Removed early return that skipped painting on protected cells
2. **`updateCursorForPosition()`**: Removed blocked cursor feedback, always shows paintable
3. **`runCorruptionPipeline()`**: Removed `!isProtected` conditions from decode test logic
4. **`renderQRWithOverlay()` Layer 1**: Removed mask skip so overlay renders on functional areas
5. **`drawPaintedBorders()`**: Removed mask skip and mask conditions from edge detection

### Task 2: Update worker optimization (f8842f4)

Updated `evaluateCandidate()` in the web worker:

1. **Pixel diff calculation**: Removed `|| isFunctionPattern` from skip condition
2. **QR render for decode**: Removed `!isFunctionPattern &&` from painted pixel application

## Technical Impact

- **Before**: Functional patterns were read-only "protected zones" with red overlay
- **After**: All grid cells are paintable; functional patterns still have visual indicator but painting works

This allows users to incorporate finder patterns, timing patterns, and alignment patterns into their artistic designs. The decodability calculation now accurately reflects conflicts in functional areas.

## Deviations from Plan

None - plan executed exactly as written.

## Verification

All success criteria met:
- Painting allowed on ALL grid cells (no functional pattern restrictions)
- Painted overlay visible on functional patterns
- Pixel diff calculations include functional area conflicts
- QR decode testing applies painted pixels over functional patterns

## Self-Check: PASSED

- [x] index.html modified with all 6 changes
- [x] Commit 0737b1d exists
- [x] Commit f8842f4 exists
