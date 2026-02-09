---
phase: 03-hash-optimization-loop
plan: 01
subsystem: optimization
tags: [web-worker, qr-encoding, jsqr, hash-generation, image-processing, blob-url]

# Dependency graph
requires:
  - phase: 02-pixel-painting-corruption
    provides: Paint grid, QR corruption pipeline, function pattern masking, jsQR decoder
provides:
  - Inline Web Worker with QR encoding and jsQR decoding
  - Hash optimization search loop with random generation
  - Pixel diff calculation for painted pattern alignment
  - Worker message protocol (START, PROGRESS, STOP, COMPLETE)
  - Top 5 result tracking and reporting
affects: [03-02-ui-controls]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Web Worker from Blob URL for inline threading"
    - "QR encoding extraction for worker-compatible use"
    - "Runtime script injection into worker code"

key-files:
  created: []
  modified:
    - index.html

key-decisions:
  - "Extract QR encoding core from qrcodejs for worker use (DOM-independent)"
  - "Include jsQR via runtime script extraction and concatenation"
  - "Random hash generation with crypto.getRandomValues"
  - "Top 5 tracking sorted by decodability then pixel diff"
  - "Progress reporting every 100 attempts or 500ms"

patterns-established:
  - "Blob URL worker pattern for single-file architecture"
  - "Worker message protocol with START/PROGRESS/STOP/COMPLETE"
  - "Module-level pixel comparison for painted pattern alignment"

# Metrics
duration: 5min
completed: 2026-02-09
---

# Phase 03 Plan 01: Web Worker Search Engine Summary

**Inline Web Worker with extracted QR encoding, jsQR decoding, and hash optimization search loop generating random URL fragments and measuring painted pattern alignment**

## Performance

- **Duration:** 5 minutes
- **Started:** 2026-02-09T18:18:45Z
- **Completed:** 2026-02-09T18:23:48Z
- **Tasks:** 1
- **Files modified:** 1

## Accomplishments
- Extracted QR encoding core from minified qrcodejs library for worker use
- Built complete QRCodeModel with pattern generation, timing, masking, and Reed-Solomon error correction
- Integrated jsQR decoder into worker via runtime script extraction
- Implemented hash optimization search loop with random generation and candidate evaluation
- Created pixel diff calculation comparing painted pixels with QR module states
- Built worker message protocol with START_OPTIMIZATION, PROGRESS, STOP, COMPLETE messages
- Implemented top 5 result tracking sorted by decodability and pixel difference
- Added configurable timeout and progress reporting (every 100 attempts or 500ms)

## Task Commits

Each task was committed atomically:

1. **Task 1: Extract QR encoding logic and create inline Web Worker with search loop** - `e238a93` (feat)

## Files Created/Modified
- `index.html` - Added 1070 lines: Web Worker with QR encoding, jsQR decoding, hash search loop, worker lifecycle functions

## Decisions Made

**1. QR encoding extraction approach**
- Extracted core QR classes (QRCodeModel, QR8BitByte, QRPolynomial, QRRSBlock, QRUtil, QRMath, QRBitBuffer) from minified qrcodejs
- Stripped all DOM-dependent rendering code (Canvas, SVG, Table drawing classes)
- Created pure `generateQRModel(text, version, errorCorrectionLevel)` function
- This enables worker-compatible QR generation without DOM access

**2. jsQR integration method**
- Used runtime script extraction: `document.querySelectorAll('script')[1].textContent`
- Concatenated jsQR code into worker string: `` ` + jsQRScript + ` ``
- This maintains single-file architecture while enabling worker-side decoding

**3. Random hash generation**
- 8-character hashes from URL-safe charset: `ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_`
- Used `crypto.getRandomValues()` for cryptographically strong randomness
- Provides 64^8 = 281 trillion possible combinations

**4. Pixel diff calculation**
- Module-level comparison (not pixel-level) between painted state and QR module dark/light
- Only count painted pixels (state 1 or 2) that are NOT in function pattern mask
- Lower pixel diff = better alignment with painted pattern

**5. Top 5 result tracking**
- Sort by decodable (true first), then by pixelDiff (ascending)
- Keep only top 5 to limit memory usage
- Accumulate across multiple runs via existingTopResults parameter

**6. Progress reporting frequency**
- Send PROGRESS every 100 attempts OR every 500ms (whichever triggers first)
- Prevents main thread flooding while ensuring responsive UI updates
- Final COMPLETE message sent on timeout or stop

**7. Global function exposure**
- Exposed `createOptimizationWorker`, `startOptimization`, `stopOptimization`, `terminateOptimizationWorker`, `getLatestOptimizationResults` to window
- Enables console testing and debugging during development
- Plan 02 will wire these to UI controls

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None. QR encoding extraction from minified code was complex but completed successfully. jsQR runtime injection worked as designed.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

**Ready for Plan 02 (UI Controls):**
- Worker creation function available: `createOptimizationWorker()`
- Start function available: `startOptimization(timeout)`
- Stop function available: `stopOptimization()`
- Results accessor available: `getLatestOptimizationResults()`
- Worker sends PROGRESS messages with topResults array
- Worker sends COMPLETE message with final statistics

**Worker capabilities verified:**
- Generates QR codes matching main thread qrcodejs output
- Runs hash search loop without freezing UI
- Reports progress with attempts count and top results
- Responds to STOP message cleanly

**Next phase needs:**
- UI button to trigger `startOptimization()`
- UI button to trigger `stopOptimization()`
- Display for progress messages (attempts, elapsed time)
- Display for top 5 results table
- Optional timeout configuration input

---
*Phase: 03-hash-optimization-loop*
*Completed: 2026-02-09*

## Self-Check: PASSED
