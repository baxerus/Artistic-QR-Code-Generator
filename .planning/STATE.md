# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-02-06)

**Core value:** The painted pattern is sacred and never changes. The tool finds URL variants that naturally align with the art.
**Current focus:** Phase 2 in progress - Building pixel painting and corruption logic.

## Current Position

Phase: 2 of 4 (Pixel Painting & Corruption)
Plan: 2 of 3 in phase
Status: In progress
Last activity: 2026-02-08 — Completed 02-02-PLAN.md (QR Corruption Overlay & Decode Pipeline)

Progress: [████████░░] 67% (2/3 plans complete in current phase)

## Performance Metrics

**Velocity:**
- Total plans completed: 4
- Average duration: 143 minutes
- Total execution time: 9.52 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 1 | 2/2 | 20min | 10min |
| 2 | 2/3 | 545min | 272min |

**Recent Trend:**
- Last plan: 02-02 (543 minutes / 9h 3min)
- Trend: Phase 2 plan 02-02 took significantly longer due to complex decoder integration

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

### Pending Todos

None.

### Blockers/Concerns

**Phase 2 Planning:**
- ✓ RESOLVED: Binary decode (success/fail) implemented, granular error counts deferred to Phase 3 enhancement
- ✓ RESOLVED: Canvas-to-module mapping implemented with function pattern masking
- ✓ RESOLVED: Function pattern locations implemented using version-specific coordinate tables

**Phase 3 Planning:**
- Hash search performance ceiling unknown (iterations/second affects realistic timeout recommendations)
- Web Worker + OffscreenCanvas integration needs verification for 2026 browser support
- May need research phase for optimization algorithm and worker architecture

## Session Continuity

Last session: 2026-02-08T19:18:48Z
Stopped at: Completed 02-02-PLAN.md
Resume file: None

---
*Next step: Execute 02-03-PLAN.md (Phase 2 remaining plan) or begin Phase 3 planning*
