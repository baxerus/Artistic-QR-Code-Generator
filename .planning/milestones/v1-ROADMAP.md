# Milestone v1: MVP

**Status:** SHIPPED 2026-02-09
**Phases:** 1-4
**Total Plans:** 8

## Overview

Build a single-file HTML tool that lets users paint pixel patterns into QR codes by searching for URL hash fragments that naturally align with the art. The journey starts with basic QR generation infrastructure, adds three-state pixel painting and corruption logic, implements the hash optimization engine with live feedback, and finishes with results display and export capabilities.

## Phases

### Phase 1: Foundation & QR Generation

**Goal**: User can enter URL and see valid QR code generated in browser
**Depends on**: Nothing (first phase)
**Requirements**: INPUT-01, INPUT-02, INPUT-03, INPUT-04, QR-01, TECH-01, TECH-02, TECH-03, TECH-04
**Success Criteria** (what must be TRUE):
  1. User can enter URL in input field and see format validation feedback
  2. System automatically calculates and displays minimum QR version based on URL length
  3. User can select QR version from dropdown showing only valid range for entered URL
  4. Generated QR code displays on canvas with correct dimensions for selected version
  5. Single HTML file works when opened via file:// protocol (no external dependencies)

Plans:
- [x] 01-01-PLAN.md — HTML skeleton with inlined CSS and QR library
- [x] 01-02-PLAN.md — Application logic: validation, version calculation, QR generation

**Completed:** 2026-02-07

### Phase 2: Pixel Painting & Corruption

**Goal**: User can paint three-state patterns that lock onto QR codes and see decode status
**Depends on**: Phase 1
**Requirements**: CANVAS-01, CANVAS-02, CANVAS-03, CANVAS-04, CANVAS-05, QR-02, QR-03, QR-04
**Success Criteria** (what must be TRUE):
  1. User can click pixels on canvas to cycle through three states: unset → white → black → unset
  2. All three pixel states are visually distinguishable (different colors or patterns)
  3. User can clear/reset entire canvas to unset state with one button click
  4. Locked pattern pixels overlay onto generated QR code without changing during operations
  5. System decodes pattern-corrupted QR code and displays decode status (success/fail; granular error counting deferred to Phase 3)

Plans:
- [x] 02-01-PLAN.md — Three-state painting canvas with click-to-cycle and clear button
- [x] 02-02-PLAN.md — QR corruption overlay, function pattern masking, jsQR decode, and status display

**Completed:** 2026-02-07

### Phase 3: Hash Optimization Loop

**Goal**: User can run automated hash search that finds URL variants minimizing decoder errors
**Depends on**: Phase 2
**Requirements**: OPT-01, OPT-02, OPT-03, OPT-04, OPT-05, OPT-06, OPT-07, OPT-08
**Success Criteria** (what must be TRUE):
  1. User can click Generate button to start hash fragment optimization
  2. Search runs without freezing UI (users can see live progress)
  3. Live progress display shows attempts counter and elapsed time during search
  4. Live preview displays current top 5 best QR codes with their error counts
  5. User can manually stop search at any time before timeout expires
  6. User can configure maximum search time in seconds before starting
  7. Pattern pixels remain unchanged throughout entire optimization process

Plans:
- [x] 03-01-PLAN.md — Web Worker search engine with extracted QR encoding and hash evaluation pipeline
- [x] 03-02-PLAN.md — Optimization UI controls, live progress display, top 5 previews, and button state machine

**Completed:** 2026-02-09

### Phase 4: Results & Export

**Goal**: User can review ranked results and export chosen QR codes with their URLs
**Depends on**: Phase 3
**Requirements**: RESULT-01, RESULT-02, RESULT-03, RESULT-04, OUTPUT-01, OUTPUT-02, OUTPUT-03
**Success Criteria** (what must be TRUE):
  1. Top 5 results display in table ranked by decoder error count (lowest first)
  2. Each result shows QR code image, error count, and hash fragment used
  3. User can download any result as PNG image
  4. User can copy final URL (with hash fragment) to clipboard for any result
  5. Error visualization overlay clearly shows locked pattern pixels, QR data pixels, and conflict pixels

Plans:
- [x] 04-01-PLAN.md — Enhanced results display with Download PNG and Copy URL actions
- [x] 04-02-PLAN.md — Error visualization overlay with click-to-expand and color legend

**Completed:** 2026-02-09

## Progress

**Execution Order:** Phases 1 → 2 → 3 → 4

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation & QR Generation | 2/2 | Complete | 2026-02-07 |
| 2. Pixel Painting & Corruption | 2/2 | Complete | 2026-02-07 |
| 3. Hash Optimization Loop | 2/2 | Complete | 2026-02-09 |
| 4. Results & Export | 2/2 | Complete | 2026-02-09 |

## Milestone Summary

**Key Decisions:**
- Used qrcodejs library (inlined) for QR generation with patched typeNumber support
- Used jsQR library (inlined) for QR decoding
- Pixel diff as error metric (jsQR doesn't expose Reed-Solomon error counts)
- Web Worker via Blob URL for non-blocking search
- Batched worker loop (100 iterations + setTimeout) for clean stop behavior
- a-z0-9 charset, 6-char hash (~2.2B combinations)
- 1:1 module resolution PNG export (no scaling artifacts)
- Click-to-expand error overlay instead of toggle button
- Paint locking during optimization with stats reset on pattern change

**Issues Resolved:**
- qrcodejs ignoring typeNumber parameter — patched makeCode method
- jsQR not exposing Reed-Solomon errors — used pixel diff as practical proxy
- Worker synchronous loop blocking stop messages — converted to batched setTimeout
- Worker ImageData too small for jsQR — scaled to floor(400/moduleCount) px/module
- QR capacity overflow with hash — added HASH_OVERHEAD to version calculation

**Technical Debt:**
- Phase 3 missing formal VERIFICATION.md (verified via integration check)
- Console.log diagnostic statements throughout (appropriate, not stubs)

---
*Archived: 2026-02-09*
*For current project status, see .planning/PROJECT.md*
