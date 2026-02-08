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

- [x] **CANVAS-01**: Painting canvas displays grid matching selected QR version dimensions (Version 2 = 25×25, Version 8 = 49×49)
- [x] **CANVAS-02**: Each pixel has three states: white (locked), black (locked), unset (QR decides)
- [x] **CANVAS-03**: User can click pixels to cycle states: unset → white → black → unset
- [x] **CANVAS-04**: All three pixel states are visually distinguishable on canvas
- [x] **CANVAS-05**: User can clear/reset all painted pixels to unset state

### Optimization Algorithm

- [ ] **OPT-01**: User can start hash fragment optimization by clicking Generate button
- [ ] **OPT-02**: System tries different URL hash fragments (#abc, #xyz, etc.) to minimize decoder errors
- [ ] **OPT-03**: Locked pattern pixels never change during optimization
- [ ] **OPT-04**: User can configure maximum search time in seconds
- [ ] **OPT-05**: User can manually stop optimization search at any time
- [ ] **OPT-06**: System displays live progress: attempts counter and elapsed time
- [ ] **OPT-07**: System displays live preview of top 5 current best QR codes during search
- [ ] **OPT-08**: System displays live error count for each of top 5 current best results

### QR Generation & Decoding

- [x] **QR-01**: System generates QR codes with error correction level H
- [x] **QR-02**: System overlays locked pattern pixels onto generated QR code (corrupting it)
- [x] **QR-03**: System decodes pattern-corrupted QR codes
- [x] **QR-04**: System counts decoder errors (Reed-Solomon errors, not binary success/fail)

### Results Display

- [ ] **RESULT-01**: System displays top 5 results ranked by decoder error count (lowest first)
- [ ] **RESULT-02**: Each result shows the QR code with embedded pattern
- [ ] **RESULT-03**: Each result shows decoder error count
- [ ] **RESULT-04**: Each result shows the hash fragment used

### Output & Export

- [ ] **OUTPUT-01**: User can download QR code as PNG image for each result
- [ ] **OUTPUT-02**: User can copy final URL (with hash fragment) to clipboard for each result
- [ ] **OUTPUT-03**: System provides error visualization overlay showing: locked pattern pixels, QR data pixels, error/conflict pixels

### Technical Infrastructure

- [x] **TECH-01**: All JavaScript libraries are inlined in single HTML file
- [x] **TECH-02**: All CSS is inlined in single HTML file
- [x] **TECH-03**: HTML file works when served by web server (http://)
- [x] **TECH-04**: HTML file works when opened locally (file:// protocol)

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Enhanced Export

- **EXPORT-01**: User can configure PNG export resolution (DPI settings)
- **EXPORT-02**: User can customize QR code colors (foreground/background)

### Workflow Improvements

- **WORKFLOW-01**: User can save painted pattern as JSON file
- **WORKFLOW-02**: User can load previously saved pattern from JSON file
- **WORKFLOW-03**: User can toggle grid overlay visibility on canvas
- **WORKFLOW-04**: User can use keyboard shortcuts for painting (spacebar, number keys)

### Advanced Features

- **ADV-01**: User can access help/tutorial overlay explaining three-state pixels and optimization
- **ADV-02**: System provides side-by-side comparison view of top 5 results
- **ADV-03**: System shows current hash fragment being tested during search (algorithm transparency)

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| QR error correction levels L/M/Q | Level H is required for artistic corruption; lower levels would fail immediately |
| Brush tools or drawing modes | Click-to-cycle pixel state is sufficient for pixel art; complexity not worth UX burden |
| Undo/redo history | Clear/reset is sufficient; artistic workflow is exploratory (try-and-reset) |
| SVG export | PNG is sufficient initially; SVG is nice-to-have for v2+ |
| Batch processing multiple URLs | Artistic workflow is one-off creative process; different use case than business bulk generation |
| Mobile/touch optimization | Pixel painting on small touchscreens impractical; desktop suits precision work |
| Real-time camera scan testing | Requires camera API, device testing infrastructure; error count is sufficient proxy for scannability |
| AI-generated artistic QR codes | Contradicts core value of manual pixel control and transparency |
| Social media sharing platform | Tool value is in generation, not sharing; users handle distribution themselves |
| Pattern templates/gallery | Encourages exploration without crutches; templates would limit creativity |
| QR version range beyond 2-8 | Versions 2-8 cover most artistic use cases; larger versions add complexity |
| Dynamic QR codes with analytics | Requires hosting, tracking infrastructure; against privacy-first portability philosophy |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

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
| OPT-01 | Phase 3 | Pending |
| OPT-02 | Phase 3 | Pending |
| OPT-03 | Phase 3 | Pending |
| OPT-04 | Phase 3 | Pending |
| OPT-05 | Phase 3 | Pending |
| OPT-06 | Phase 3 | Pending |
| OPT-07 | Phase 3 | Pending |
| OPT-08 | Phase 3 | Pending |
| QR-01 | Phase 1 | Complete |
| QR-02 | Phase 2 | Complete |
| QR-03 | Phase 2 | Complete |
| QR-04 | Phase 2 | Complete |
| RESULT-01 | Phase 4 | Pending |
| RESULT-02 | Phase 4 | Pending |
| RESULT-03 | Phase 4 | Pending |
| RESULT-04 | Phase 4 | Pending |
| OUTPUT-01 | Phase 4 | Pending |
| OUTPUT-02 | Phase 4 | Pending |
| OUTPUT-03 | Phase 4 | Pending |
| TECH-01 | Phase 1 | Complete |
| TECH-02 | Phase 1 | Complete |
| TECH-03 | Phase 1 | Complete |
| TECH-04 | Phase 1 | Complete |

**Coverage:**
- v1 requirements: 31 total
- Mapped to phases: 31/31 ✓
- Unmapped: 0

**Phase Distribution:**
- Phase 1: 9 requirements (INPUT + QR-01 + TECH)
- Phase 2: 9 requirements (CANVAS + QR-02/03/04)
- Phase 3: 8 requirements (OPT)
- Phase 4: 5 requirements (RESULT + OUTPUT)

---
*Requirements defined: 2026-02-06*
*Last updated: 2026-02-06 after roadmap creation*
