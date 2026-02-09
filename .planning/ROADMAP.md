# Roadmap: Artistic QR Code Generator

## Overview

Build a single-file HTML tool that lets users paint pixel patterns into QR codes by searching for URL hash fragments that naturally align with the art. The journey starts with basic QR generation infrastructure, adds three-state pixel painting and corruption logic, implements the hash optimization engine with live feedback, and finishes with results display and export capabilities.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Foundation & QR Generation** - Single-file architecture with URL input and basic QR code generation
- [x] **Phase 2: Pixel Painting & Corruption** - Three-state canvas editor with pattern locking and error measurement
- [x] **Phase 3: Hash Optimization Loop** - Web Worker search algorithm with live progress and top 5 tracking
- [x] **Phase 4: Results & Export** - Results display with PNG download, URL copy, and error visualization

## Phase Details

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
**Plans**: 2 plans

Plans:
- [x] 01-01-PLAN.md — HTML skeleton with inlined CSS and QR library
- [x] 01-02-PLAN.md — Application logic: validation, version calculation, QR generation

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
**Plans**: 2 plans

Plans:
- [x] 02-01-PLAN.md — Three-state painting canvas with click-to-cycle and clear button
- [x] 02-02-PLAN.md — QR corruption overlay, function pattern masking, jsQR decode, and status display

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
**Plans**: 2 plans

Plans:
- [x] 03-01-PLAN.md — Web Worker search engine with extracted QR encoding and hash evaluation pipeline
- [x] 03-02-PLAN.md — Optimization UI controls, live progress display, top 5 previews, and button state machine

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
**Plans**: 2 plans

Plans:
- [x] 04-01-PLAN.md — Enhanced results display with Download PNG and Copy URL actions
- [x] 04-02-PLAN.md — Error visualization overlay with click-to-expand and color legend

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation & QR Generation | 2/2 | Complete | 2026-02-07 |
| 2. Pixel Painting & Corruption | 2/2 | Complete | 2026-02-07 |
| 3. Hash Optimization Loop | 2/2 | Complete | 2026-02-09 |
| 4. Results & Export | 2/2 | Complete | 2026-02-09 |

---
*Roadmap created: 2026-02-06*
*Last updated: 2026-02-09*
