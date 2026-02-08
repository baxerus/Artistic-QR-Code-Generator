---
phase: 02-pixel-painting-corruption
plan: 02
subsystem: ui
tags: [qr-code, jsqr, canvas, decoder, pixel-manipulation, real-time-feedback]

# Dependency graph
requires:
  - phase: 02-01
    provides: Three-state pixel painting canvas with grid interaction
provides:
  - jsQR decoder integration for QR code validation
  - Function pattern masking system protecting QR structural elements
  - Real-time corruption overlay mapping painted pixels to QR modules
  - Decode status display showing success/failure feedback
  - Complete feedback loop from paint action to decode result
affects: [03-hash-search-optimization]

# Tech tracking
tech-stack:
  added: [jsQR v1.4.0]
  patterns:
    - Canvas-based QR rendering for pixel-level manipulation
    - Function pattern masking using version-specific coordinate tables
    - Real-time feedback loop on user interaction
    - ImageData manipulation for pixel-precise corruption

key-files:
  created: []
  modified: [index.html]

key-decisions:
  - "Inlined jsQR library for zero external dependencies"
  - "Forced Canvas rendering mode for pixel-level control"
  - "Protected all function patterns (finders, timing, alignment, dark module) from corruption"
  - "Binary decode status (success/fail) instead of granular error counts"
  - "Real-time re-decode on every paint click for instant feedback"

patterns-established:
  - "Function pattern mask as boolean array indexed by (y * size + x)"
  - "Overlay pipeline: generate clean QR → apply function mask → overlay painted pixels → decode → display status"
  - "Cell-to-pixel coordinate mapping between paint canvas and QR canvas"

# Metrics
duration: 9h 3min
completed: 2026-02-08
---

# Phase 02 Plan 02: QR Corruption Overlay & Decode Pipeline Summary

**jsQR decoder integration with function pattern masking, real-time corruption overlay, and instant decode feedback on every paint action**

## Performance

- **Duration:** 9h 3min
- **Started:** 2026-02-08T00:15:29+01:00
- **Completed:** 2026-02-08T19:18:48+01:00
- **Tasks:** 2 (1 auto + 1 checkpoint)
- **Files modified:** 1

## Accomplishments
- jsQR decoder inlined and integrated for real-time QR validation
- Function pattern masking protects QR structural elements (finders, timing, alignment) from corruption
- Pixel overlay system accurately maps painted grid cells to QR canvas modules
- Instant decode status feedback (Success/Failed) updates on every paint click
- Complete corruption pipeline closes the feedback loop: paint → overlay → decode → display

## Task Commits

Each task was committed atomically:

1. **Task 1: Inline jsQR decoder, implement function pattern masking and corruption overlay with decode status** - `f0ed465` (feat)
2. **Bug fix: Close jsQR script tag** - `408dc86` (fix)

## Files Created/Modified
- `index.html` - Added jsQR v1.4.0 inline, switched QR generation to Canvas mode, implemented function pattern masking, pixel overlay corruption system, decode status display, and real-time feedback loop

## Decisions Made

**1. Inlined jsQR library for zero dependencies**
- Fetched jsQR v1.4.0 minified source and embedded in script tag
- Maintains single-file architecture with no CDN dependencies
- Rationale: Portability and offline functionality

**2. Forced Canvas rendering instead of SVG**
- Extracted QR module grid from qrcodejs internal model
- Rendered to custom canvas element with `renderQRToCanvas` function
- Rationale: Pixel-level manipulation requires ImageData access

**3. Protected function patterns from corruption**
- Implemented version-specific masking for finders (9x9, 9x8, 8x9), timing patterns (row/col 6), alignment patterns (5x5 at coordinates), and dark module
- Overlay checks mask before applying painted pixels
- Rationale: Corrupting function patterns makes QR completely undecodable

**4. Binary decode status instead of granular error counts**
- Display shows "Decode: Success" or "Decode: Failed"
- jsQR library doesn't expose Reed-Solomon error counts (research finding)
- Rationale: Binary feedback sufficient for Phase 2, granular metrics deferred to Phase 3 enhancement

**5. Real-time re-decode on every paint click**
- Paint canvas pointerdown triggers full overlay and decode pipeline
- User sees immediate feedback on how pattern affects decodability
- Rationale: Instant feedback is core to user experience

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Unclosed script tag for jsQR**
- **Found during:** Task 1 verification (user reported syntax error)
- **Issue:** jsQR inline script tag was missing closing `</script>` tag, causing HTML parser errors
- **Fix:** Added closing `</script>` tag after jsQR library code
- **Files modified:** index.html
- **Verification:** Page loads without syntax errors, jsQR global available
- **Committed in:** 408dc86 (separate fix commit)

---

**Total deviations:** 1 auto-fixed (1 bug)
**Impact on plan:** Critical bug fix for functionality. No scope creep.

## Issues Encountered

None during planned work execution. Bug discovered during verification was fixed immediately.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

**Ready for Phase 3 (Hash Search & Optimization):**
- Complete feedback loop in place: paint → corrupt → decode → display
- Binary decode status provides success metric for optimization algorithm
- Function pattern mask ensures QR structural integrity during search
- Canvas-based rendering enables efficient pixel manipulation in search loop

**Known limitations for Phase 3:**
- Binary decode (success/fail) instead of granular error counts - optimization will use boolean metric
- Decoder performance not yet measured - Phase 3 will need to profile iterations/second for timeout recommendations
- No Web Worker integration yet - may be needed for search performance

**No blockers.** Phase 3 can begin hash search implementation.

## Self-Check: PASSED

All commits verified:
- f0ed465 ✓
- 408dc86 ✓

All files verified:
- index.html ✓

---
*Phase: 02-pixel-painting-corruption*
*Completed: 2026-02-08*
