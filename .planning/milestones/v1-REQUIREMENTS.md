# Requirements Archive: v1 MVP

**Archived:** 2026-02-09
**Status:** SHIPPED

This is the archived requirements specification for v1.
For current requirements, see `.planning/PROJECT.md`.

---

# Requirements: Artistic QR Code Generator

**Defined:** 2026-02-06
**Core Value:** The painted pattern is sacred and never changes. The tool finds URL variants that naturally align with the art.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Input & Configuration

- [x] **INPUT-01**: User can enter URL in input field with format validation
- [x] **INPUT-02**: System calculates minimum QR version based on URL length + error correction level H
- [x] **INPUT-03**: User can select QR version from valid range (minimum to Version 8)
- [x] **INPUT-04**: QR version selector shows only valid versions for entered URL

### Canvas & Painting

- [x] **CANVAS-01**: Painting canvas displays grid matching selected QR version dimensions (Version 2 = 25x25, Version 8 = 49x49)
- [x] **CANVAS-02**: Each pixel has three states: white (locked), black (locked), unset (QR decides)
- [x] **CANVAS-03**: User can click pixels to cycle states: unset → white → black → unset
- [x] **CANVAS-04**: All three pixel states are visually distinguishable on canvas
- [x] **CANVAS-05**: User can clear/reset all painted pixels to unset state

### Optimization Algorithm

- [x] **OPT-01**: User can start hash fragment optimization by clicking Generate button
- [x] **OPT-02**: System tries different URL hash fragments (#abc, #xyz, etc.) to minimize decoder errors
- [x] **OPT-03**: Locked pattern pixels never change during optimization
- [x] **OPT-04**: User can configure maximum search time in seconds
- [x] **OPT-05**: User can manually stop optimization search at any time
- [x] **OPT-06**: System displays live progress: attempts counter and elapsed time
- [x] **OPT-07**: System displays live preview of top 5 current best QR codes during search
- [x] **OPT-08**: System displays live error count for each of top 5 current best results

### QR Generation & Decoding

- [x] **QR-01**: System generates QR codes with error correction level H
- [x] **QR-02**: System overlays locked pattern pixels onto generated QR code (corrupting it)
- [x] **QR-03**: System decodes pattern-corrupted QR codes
- [x] **QR-04**: System counts decoder errors — *Implemented as pixel diff (painted pixel vs QR module alignment) rather than Reed-Solomon error counting. jsQR doesn't expose RS errors. Pixel diff is an effective proxy.*

### Results Display

- [x] **RESULT-01**: System displays top 5 results ranked by decoder error count (lowest first)
- [x] **RESULT-02**: Each result shows the QR code with embedded pattern
- [x] **RESULT-03**: Each result shows decoder error count
- [x] **RESULT-04**: Each result shows the hash fragment used

### Output & Export

- [x] **OUTPUT-01**: User can download QR code as PNG image for each result
- [x] **OUTPUT-02**: User can copy final URL (with hash fragment) to clipboard for each result
- [x] **OUTPUT-03**: System provides error visualization overlay showing: locked pattern pixels, QR data pixels, error/conflict pixels

### Technical Infrastructure

- [x] **TECH-01**: All JavaScript libraries are inlined in single HTML file
- [x] **TECH-02**: All CSS is inlined in single HTML file
- [x] **TECH-03**: HTML file works when served by web server (http://)
- [x] **TECH-04**: HTML file works when opened locally (file:// protocol)

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| INPUT-01 | Phase 1 | Complete |
| INPUT-02 | Phase 1 | Complete |
| INPUT-03 | Phase 1 | Complete |
| INPUT-04 | Phase 1 | Complete |
| CANVAS-01 | Phase 2 | Complete |
| CANVAS-02 | Phase 2 | Complete |
| CANVAS-03 | Phase 2 | Complete |
| CANVAS-04 | Phase 2 | Complete |
| CANVAS-05 | Phase 2 | Complete |
| OPT-01 | Phase 3 | Complete |
| OPT-02 | Phase 3 | Complete |
| OPT-03 | Phase 3 | Complete |
| OPT-04 | Phase 3 | Complete |
| OPT-05 | Phase 3 | Complete |
| OPT-06 | Phase 3 | Complete |
| OPT-07 | Phase 3 | Complete |
| OPT-08 | Phase 3 | Complete |
| QR-01 | Phase 1 | Complete |
| QR-02 | Phase 2 | Complete |
| QR-03 | Phase 2 | Complete |
| QR-04 | Phase 2 | Complete |
| RESULT-01 | Phase 4 | Complete |
| RESULT-02 | Phase 4 | Complete |
| RESULT-03 | Phase 4 | Complete |
| RESULT-04 | Phase 4 | Complete |
| OUTPUT-01 | Phase 4 | Complete |
| OUTPUT-02 | Phase 4 | Complete |
| OUTPUT-03 | Phase 4 | Complete |
| TECH-01 | Phase 1 | Complete |
| TECH-02 | Phase 1 | Complete |
| TECH-03 | Phase 1 | Complete |
| TECH-04 | Phase 1 | Complete |

**Coverage:** 31/31 requirements complete

---

## Milestone Summary

**Shipped:** 31 of 31 v1 requirements
**Adjusted:**
- QR-04: Originally specified "Reed-Solomon errors" but implemented as pixel diff (painted pixel vs QR module alignment). jsQR library doesn't expose RS error counts. Pixel diff serves as an effective practical proxy for alignment quality.

**Dropped:** None

---
*Archived: 2026-02-09 as part of v1 milestone completion*
