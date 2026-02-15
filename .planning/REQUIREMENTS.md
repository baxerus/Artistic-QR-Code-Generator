# Requirements: Artistic QR Code Generator

**Defined:** 2026-02-15
**Core Value:** The painted pattern is sacred and never changes. The tool finds URL variants that naturally align with the art.

## v1.3 Requirements

Requirements for v1.3 Hash Capacity Optimization.

### Hash Optimization

- [ ] **HASH-01**: Hash length dynamically calculated based on QR version capacity minus URL length
- [ ] **HASH-02**: MIN_HASH_LENGTH constant replaces HASH_LENGTH (clarifies minimum, not fixed)
- [ ] **HASH-03**: Hash generation fills remaining byte capacity of selected QR version

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
| HASH-01 | Phase 11 | Pending |
| HASH-02 | Phase 11 | Pending |
| HASH-03 | Phase 11 | Pending |

**Coverage:**
- v1.3 requirements: 3 total
- Mapped to phases: 3 ✓
- Unmapped: 0

---
*Requirements defined: 2026-02-15*
*Last updated: 2026-02-15 after initial definition*
