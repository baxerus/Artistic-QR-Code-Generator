# Requirements: Artistic QR Code Generator

**Defined:** 2026-02-17
**Core Value:** The painted pattern is sacred and never changes. The tool finds URL variants that naturally align with the art.

## v1.5 Requirements

Requirements for v1.5 UX & Export Enhancements. Each maps to roadmap phases.

### RS Capacity Display

- [ ] **RSCAP-01**: Result card shows correction count as "X of Y" where Y is max capacity for QR version at level H
- [ ] **RSCAP-02**: Max capacity calculated from jsQR VERSIONS table using formula: `sum(ecBlocks × ecCodewordsPerBlock) / 2`

### SVG Export

- [ ] **SVG-01**: Result card has "Download SVG" button alongside existing PNG button
- [ ] **SVG-02**: SVG contains black/white modules only (final QR, not preview colors)
- [ ] **SVG-03**: Downloaded SVG filename follows same pattern as PNG export (different extension only)

### Paint Shortcut

- [ ] **PAINT-01**: Shift+click/drag (left or right button) paints "unset" regardless of selected color or button
- [ ] **PAINT-02**: Shift state captured at stroke start and held for entire drag

## Future Requirements

Deferred to future releases:

### UX Enhancements
- Save/load patterns to/from file (JSON export)
- Mobile/touch support

## Out of Scope

| Feature | Reason |
|---------|--------|
| Other error correction levels | H provides maximum error tolerance, complexity not warranted |
| Undo/redo | Clear/reset sufficient; Shift+paint adds quick erasing |
| Batch processing | Single URL workflow only |
| Server-side processing | Everything client-side |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| PAINT-01 | Phase 14 | Pending |
| PAINT-02 | Phase 14 | Pending |
| SVG-01 | Phase 15 | Pending |
| SVG-02 | Phase 15 | Pending |
| SVG-03 | Phase 15 | Pending |
| RSCAP-01 | Phase 16 | Pending |
| RSCAP-02 | Phase 16 | Pending |

**Coverage:**
- v1.5 requirements: 7 total
- Mapped to phases: 7
- Unmapped: 0 ✓

**Phase Order Rationale:** Risk-ascending (research recommendation)
- Phase 14: Shift+Paint (LOW risk) — standard pointer events
- Phase 15: SVG Export (LOW risk) — well-documented patterns
- Phase 16: RS Capacity (MEDIUM risk) — formula complexity

---
*Requirements defined: 2026-02-17*
*Last updated: 2026-02-17 after roadmap creation*
