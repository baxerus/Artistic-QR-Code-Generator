# Requirements: Artistic QR Code Generator

**Defined:** 2026-02-18
**Core Value:** The painted pattern is sacred and never changes. The tool finds URL variants that naturally align with the art.

## v1.6 Requirements

Requirements for v1.6 QR Code Rotation & Pattern Tools. Each maps to roadmap phases.

### Decode Metrics

- [ ] **DECODE-01**: Paint Pattern decode test displays corrections as "X of Y" using the same capacity calculation as result cards
- [ ] **DECODE-02**: Paint Pattern decode test displays pixel error count alongside corrections
- [ ] **DECODE-03**: Decode status rows (Success/Fail/Neutral) use a fixed height with no layout jump when state changes

### Pattern Tools

- [ ] **PATTERN-01**: Paint tool selector includes "Move Pattern" alongside Black/White/Unset
- [ ] **PATTERN-02**: Move Pattern drag shifts the entire painted pattern without changing any pixel values

### QR Rotation

- [ ] **ROT-01**: Paint Pattern section includes a "Rotate 90°" control that rotates the QR code and protected overlay only (pattern stays fixed)
- [ ] **ROT-02**: Results QR previews and hover overlays respect the current QR rotation
- [ ] **ROT-03**: PNG and SVG exports reflect the current QR rotation while preserving the painted pattern orientation
- [ ] **ROT-04**: Generate control includes "Random rotation" option (0/90/180/270) applied to QR only during search

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
| DECODE-01 | Phase 17 | Pending |
| DECODE-02 | Phase 17 | Pending |
| DECODE-03 | Phase 17 | Pending |
| PATTERN-01 | Phase 18 | Pending |
| PATTERN-02 | Phase 18 | Pending |
| ROT-01 | Phase 19 | Pending |
| ROT-02 | Phase 19 | Pending |
| ROT-03 | Phase 19 | Pending |
| ROT-04 | Phase 19 | Pending |

**Coverage:**
- v1.6 requirements: 9 total
- Mapped to phases: 9
- Unmapped: 0 ✓

---
*Requirements defined: 2026-02-18*
*Last updated: 2026-02-18 after requirements definition*
