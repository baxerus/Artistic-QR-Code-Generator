---
phase: 01-foundation-a-qr-generation
plan: 02
subsystem: frontend-logic
tags: [javascript, url-validation, qr-generation, event-wiring, qrcodejs]

requires:
  - phase: 01-foundation-a-qr-generation
    plan: 01
    provides: HTML skeleton with CSS and inlined QRCode library
provides:
  - URL validation with dot-in-hostname check
  - QR version capacity table and minimum version calculation
  - Version dropdown management (filters to valid range)
  - QR code generation using qrcodejs with explicit version support
  - Complete event wiring (input, click, keypress, change)
affects:
  - phase: 02-pixel-painting-corruption (builds on this foundation)

tech-stack:
  added: []
  patterns:
    - URL validation via native URL constructor + hostname dot check
    - Byte-accurate URL length via Blob().size
    - qrcodejs patched to respect typeNumber in makeCode

key-files:
  created: []
  modified:
    - index.html

key-decisions:
  - "Patched qrcodejs makeCode to use typeNumber option instead of always auto-detecting"
  - "URL validation requires dot in hostname (not first/last char) beyond just URL constructor"
  - "Real-time validation on input event (not just blur) for both valid and invalid states"
  - "Container uses overflow:hidden + max-width/max-height on child elements for square aspect"

patterns-established:
  - "QR generation: clear container innerHTML, create new QRCode instance each time"
  - "Validation flow: input event → isValidURL → updateVersionDropdown → enable/disable button"

duration: 15min
completed: 2026-02-07
---

# Phase 1 Plan 02: Application Logic Summary

**URL validation, QR version calculation, version dropdown management, QR generation, and event wiring**

## Performance
- **Duration:** ~15 minutes (including checkpoint verification and bug fixes)
- **Started:** 2026-02-07
- **Completed:** 2026-02-07
- **Tasks:** 2 (1 auto + 1 checkpoint)
- **Files modified:** 1

## Accomplishments
- Implemented URL validation using native URL constructor with hostname dot requirement
- Created QR capacity lookup table for error correction level H byte mode (versions 1-8)
- Implemented minimum version calculation using Blob-based byte length
- Built version dropdown management that filters to valid range for entered URL
- Implemented QR code generation using qrcodejs constructor API with explicit version
- Patched qrcodejs library to respect typeNumber option in makeCode method
- Wired all event handlers: input validation, generate click, Enter key, version change
- Fixed container CSS for proper square aspect ratio with overflow hidden
- Verified via human checkpoint: validation, generation, version switching all work

## Task Commits
1. **Task 1: Implement URL validation, version calculation, and QR generation** - `c607fd5` (feat)
   - isValidURL with URL constructor
   - CAPACITIES_H_BYTE lookup table
   - calculateMinimumVersion with Blob byte count
   - updateVersionDropdown, generateQR, updateValidationUI
   - Full event wiring in DOMContentLoaded
2. **Bug fix: QR container id and validation blur state** - `837a247` (fix)
   - Added missing id="canvas-container" to div
   - Fixed blur handler to also add invalid class to input element
   - Clear container innerHTML before generating QR
3. **Bug fix: Validation timing, URL strictness, container shape, version selection** - `4644bfb` (fix)
   - Show invalid state while typing (not just on blur)
   - Require dot in hostname for URL validation
   - Fix square container with overflow:hidden + child constraints
   - Patch qrcodejs makeCode to respect typeNumber option
4. **Task 2: Human checkpoint verification** - Approved by user

## Files Created/Modified
- `index.html` - Complete working application with validation, version calculation, and QR generation

## Decisions Made
- **qrcodejs typeNumber patch:** The library's makeCode method always auto-detects minimum version, ignoring the typeNumber config. Patched to use `this._htOption.typeNumber || r(a, ...)` so explicit version selection works
- **Real-time invalid state:** Show red border + X icon while typing invalid URLs (not just on blur), matching the green border behavior for valid URLs
- **Hostname dot check:** `new URL("http://x")` parses successfully but isn't a real URL. Added requirement for at least one dot in hostname, not at first or last position
- **Square container fix:** QRCode library creates 400x400 canvas that overflows 360px content area. Fixed with overflow:hidden on container + max-width/max-height constraints on child elements

## Deviations from Plan
### Auto-fixed Issues

**1. [Rule 3 - Blocking] qrcodejs API differs from plan examples**
- **Found during:** Task 1
- **Issue:** Plan referenced `QRCode.toCanvas()` API from npm qrcode package, but the inlined library is qrcodejs which uses `new QRCode(element, options)` constructor API
- **Fix:** Used qrcodejs constructor API instead. Functionally equivalent.
- **Commit:** c607fd5

**2. [Rule 3 - Blocking] Missing container id for QR generation**
- **Found during:** Checkpoint
- **Issue:** HTML had `class="canvas-container"` but no `id`, causing `getElementById` to return null
- **Fix:** Added `id="canvas-container"` to the div
- **Commit:** 837a247

**3. [Rule 3 - Blocking] qrcodejs ignores typeNumber option**
- **Found during:** Checkpoint
- **Issue:** Library's makeCode always calls auto-detect function r(), ignoring configured typeNumber
- **Fix:** Patched minified library to use `this._htOption.typeNumber || r(a, correctLevel)`
- **Commit:** 4644bfb

## Issues Encountered
- Blur-only invalid state was confusing (inconsistent with real-time valid state)
- URL validation accepted `http://x` as valid (URL constructor too permissive)
- Container aspect-ratio broken by library-generated 400px canvas overflowing padding

## Self-Check: PASSED

All claims verified:
- File exists: index.html
- Commits exist: c607fd5, 837a247, 4644bfb
- isValidURL function present in index.html
- CAPACITIES_H_BYTE table present in index.html
- addEventListener calls present in index.html
- QRCode generation working with explicit version

---
*Phase: 01-foundation-a-qr-generation*
*Completed: 2026-02-07*
