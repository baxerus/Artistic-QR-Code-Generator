# Requirements: Artistic QR Code Generator

**Defined:** 2026-02-15
**Core Value:** The painted pattern is sacred and never changes. The tool finds URL variants that naturally align with the art.

## v1.2 Requirements

Requirements for v1.2 Visual Consistency & Result Inspection.

### Visual Consistency

- [ ] **VIS-01**: Protected area borders on paint canvas use blue instead of red
- [ ] **VIS-02**: Scaled results show blue dashed borders around protected areas on hover

### Result Inspection

- [ ] **INSP-01**: Scaled results display plain QR code by default (no overlay colors)
- [ ] **INSP-02**: Scaled results show error visualization overlay on hover

## Future Requirements

Deferred to future releases.

### UX Enhancements

- **UX-01**: Undo/redo for pattern painting
- **UX-02**: Save/load patterns to/from file (JSON export)
- **UX-03**: Keyboard shortcuts for painting tools
- **UX-04**: Mobile/touch support

### Output Enhancements

- **OUT-01**: SVG export format
- **OUT-02**: Color customization (foreground/background)
- **OUT-03**: High-resolution export (300+ DPI for print)
- **OUT-04**: Transparent background PNG export

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| QR error correction levels other than H | H provides maximum error tolerance |
| Undo/redo | Deferred to future — clear/reset sufficient for now |
| Saving/loading patterns to file | Deferred to future — localStorage sufficient for now |
| Mobile touch optimization | Desktop-first |
| SVG export | Deferred to future |
| Batch processing | Single URL workflow only |
| Server-side processing | Everything client-side |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| VIS-01 | — | Pending |
| VIS-02 | — | Pending |
| INSP-01 | — | Pending |
| INSP-02 | — | Pending |

**Coverage:**
- v1.2 requirements: 4 total
- Mapped to phases: 0
- Unmapped: 4 (pending roadmap)

---
*Requirements defined: 2026-02-15*
*Last updated: 2026-02-15 after initial definition*
