# Requirements: Artistic QR Code Generator

**Defined:** 2026-02-10
**Core Value:** The painted pattern is sacred and never changes. The tool finds URL variants that naturally align with the art.

## v1.1 Requirements

Requirements for UX Overhaul & Optimization milestone. Each maps to roadmap phases.

### State Persistence

- [ ] **PERS-01**: App state (URL, painted pattern, QR version, settings) persists across page reloads via localStorage
- [ ] **PERS-02**: State restores correctly on page load, including re-rendering the QR code and pattern overlay

### Painting Overhaul

- [ ] **PAINT-01**: Paint pattern rendered as overlay directly on top of QR code preview (not a separate canvas)
- [ ] **PAINT-02**: Protected QR areas (finder patterns, timing patterns, alignment patterns) visually marked as non-editable
- [ ] **PAINT-03**: Color selection via legend (black/white/unset) replaces click-to-cycle behavior
- [ ] **PAINT-04**: Left-click and drag paints selected color across multiple pixels
- [ ] **PAINT-05**: Right-click and drag paints opposite color (white<->black); does nothing when unset is selected

### Generation Safety

- [ ] **SAFE-01**: URLs ending in a hash fragment are detected and rejected with clear error message
- [ ] **SAFE-02**: When pattern exists, version dropdown change does NOT auto-generate QR code
- [ ] **SAFE-03**: When pattern exists, only "Generate QR Code" button triggers generation
- [ ] **SAFE-04**: When generating with same QR version, painted pattern is preserved on new QR code
- [ ] **SAFE-05**: When generating with changed QR version (dropdown or URL-forced), user is warned about pattern loss and must confirm

### Optimization

- [ ] **OPT-01**: Optimization auto-stops when 5 results with 0 errors are found
- [ ] **OPT-02**: Multiple Web Workers run optimization in parallel for faster search throughput

### Configuration

- [ ] **CONF-01**: QR version range expanded from 2-8 to 2-10
- [ ] **CONF-02**: Maximum QR version is a named constant used everywhere, easy to change

## Future Requirements

Deferred to later milestones. Tracked but not in current roadmap.

### UX Enhancements

- **UX-01**: Undo/redo for pattern painting
- **UX-02**: Save/load patterns to/from file (JSON export)
- **UX-03**: Keyboard shortcuts for painting tools
- **UX-04**: Mobile/touch support for painting

### Output Enhancements

- **OUT-01**: SVG export format
- **OUT-02**: Color customization (foreground/background)
- **OUT-03**: High-resolution export (300+ DPI for print)

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| QR error correction levels other than H | H provides maximum error tolerance needed for artistic patterns |
| Batch processing multiple URLs | Single URL workflow only |
| Server-side processing | Everything client-side in browser |
| QR versions beyond 10 | Sufficient range; max version is a constant if adjustment needed |
| Real-time camera scan testing | Error count is sufficient proxy for scannability |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| CONF-01 | Phase 5 | Pending |
| CONF-02 | Phase 5 | Pending |
| PERS-01 | Phase 6 | Pending |
| PERS-02 | Phase 6 | Pending |
| PAINT-01 | Phase 7 | Pending |
| PAINT-02 | Phase 7 | Pending |
| PAINT-03 | Phase 7 | Pending |
| PAINT-04 | Phase 7 | Pending |
| PAINT-05 | Phase 7 | Pending |
| SAFE-01 | Phase 8 | Pending |
| SAFE-02 | Phase 8 | Pending |
| SAFE-03 | Phase 8 | Pending |
| SAFE-04 | Phase 8 | Pending |
| SAFE-05 | Phase 8 | Pending |
| OPT-01 | Phase 9 | Pending |
| OPT-02 | Phase 9 | Pending |

**Coverage:**
- v1.1 requirements: 16 total
- Mapped to phases: 16
- Unmapped: 0

---
*Requirements defined: 2026-02-10*
*Last updated: 2026-02-10 after roadmap creation*
