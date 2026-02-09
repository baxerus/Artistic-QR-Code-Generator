# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-02-06)

**Core value:** The painted pattern is sacred and never changes. The tool finds URL variants that naturally align with the art.
**Current focus:** Phase 3 complete. Ready for Phase 4 planning.

## Current Position

Phase: 3 of 4 (Hash Optimization Loop) — COMPLETE
Plan: 2 of 2 in phase
Status: Complete
Last activity: 2026-02-09 — Completed 03-02-PLAN.md

Progress: [██████████████] 100% (6 of 6 plans complete, Phases 1-3)

## Performance Metrics

**Velocity:**
- Total plans completed: 6
- Average duration: 105 minutes
- Total execution time: ~10.4 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 1 | 2/2 | 20min | 10min |
| 2 | 2/2 | 545min | 272min |
| 3 | 2/2 | 50min | 25min |

**Recent Trend:**
- Last plan: 03-02 (~45 minutes including iterative bug fixes during checkpoint)
- Trend: Phase 3 completed efficiently despite multiple checkpoint fixes

*Updated after each plan completion*

## Accumulated Context

### Decisions

Decisions are logged in PROJECT.md Key Decisions table.
Recent decisions affecting current work:

- Hash fragments for URL variants — Clean approach that doesn't affect destination URL
- Three-state pixels (white/black/unset) — Pattern pixels locked, others free for QR algorithm
- Top 5 results instead of single best — Aesthetic choice matters beyond pure error metrics
- Error correction level H — Maximum error tolerance for artistic corruption
- Everything in single HTML file — Simplicity, portability, works via file://
- Modern browsers only — Simplifies development, avoid legacy compatibility
- Used qrcodejs library from CDN — Pre-built browser bundle, no build tooling needed (01-01)
- Light gray background (#f5f5f5) — Subtle contrast with white content areas (01-01)
- System font stack — Zero loading overhead, native OS appearance (01-01)
- Patched qrcodejs makeCode to respect typeNumber — Library ignores version by default (01-02)
- URL validation requires hostname dot — URL constructor alone too permissive (01-02)
- Real-time invalid state — Red border during typing, not just on blur (01-02)
- fillRect for grid lines instead of strokeRect — Avoids sub-pixel anti-aliasing blur (02-01)
- Subtle border on white pixels — Distinguishes white from canvas background (02-01)
- pointerdown event for pixel cycling — Better responsiveness than click event (02-01)
- Inlined jsQR library for zero external dependencies — Maintains single-file architecture (02-02)
- Forced Canvas rendering mode for pixel-level control — ImageData access needed for overlay (02-02)
- Protected all function patterns from corruption — Finders, timing, alignment cannot be modified (02-02)
- Binary decode status instead of granular error counts — jsQR limitation, sufficient for Phase 2 (02-02)
- Real-time re-decode on every paint click for instant feedback — Core user experience (02-02)
- Extract QR encoding core for worker use — DOM-independent QR generation (03-01)
- Runtime script extraction for jsQR in worker — Maintains single-file architecture (03-01)
- Random hash with crypto.getRandomValues — a-z0-9 charset, 6 chars (03-01/03-02)
- Top 5 tracking sorted by decodability then pixel diff — Best alignment results (03-01)
- Batched search loop (100 iterations + setTimeout(0)) — Allows stop messages (03-02)
- Cumulative stats across runs — totalAttempts/totalDecodableCount persist (03-02)
- Lock painting during optimization, reset on paint change — Prevents stale results (03-02)
- HASH_OVERHEAD in version calculation — Ensures QR capacity for URL + hash (03-02)

### Pending Todos

None.

### Blockers/Concerns

**Phase 2 Planning:**
- ✓ RESOLVED: Binary decode (success/fail) implemented, granular error counts deferred to Phase 3 enhancement
- ✓ RESOLVED: Canvas-to-module mapping implemented with function pattern masking
- ✓ RESOLVED: Function pattern locations implemented using version-specific coordinate tables

**Phase 3 Execution:**
- ✓ RESOLVED: Web Worker architecture verified - Blob URL pattern works with inline code
- ✓ RESOLVED: QR encoding extraction successful - Core classes isolated from DOM rendering
- ✓ RESOLVED: jsQR integration via runtime script concatenation working
- ✓ RESOLVED: Worker ImageData scaling needed for jsQR decode (floor(400/moduleCount) px/module)
- ✓ RESOLVED: Synchronous worker loop blocked stop messages - fixed with batched setTimeout
- ✓ RESOLVED: Result accumulation across runs with cumulative stats

## Session Continuity

Last session: 2026-02-09
Stopped at: Completed Phase 3 (Hash Optimization Loop)
Resume file: None

---
*Next step: Plan Phase 4 (Results & Export)*
