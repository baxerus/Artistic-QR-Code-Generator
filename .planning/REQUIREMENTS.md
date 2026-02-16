# Requirements: Artistic QR Code Generator v1.4

**Defined:** 2026-02-16
**Core Value:** The painted pattern is sacred and never changes. The tool finds URL variants that naturally align with the art.

## v1.4 Requirements

### RS Error Measurement

- [ ] **RS-01**: Worker extracts Reed-Solomon correction count from each candidate QR during optimization search
- [ ] **RS-02**: RS extraction works with jsQR library or alternative browser decoder if jsQR doesn't expose RS data
- [ ] **RS-03**: RS correction count is returned alongside pixel diff for each candidate result

### Results Display & Ranking

- [ ] **RSLT-01**: Result cards display both RS corrections and pixel diff metrics
- [ ] **RSLT-02**: Results are ranked by RS corrections (primary, ascending), pixel diff (secondary, ascending)
- [ ] **RSLT-03**: "Perfect result" is redefined as RS=0 (regardless of pixel diff)
- [ ] **RSLT-04**: Auto-stop triggers when TOP_RESULTS_COUNT results with RS=0 are found

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| RS-01 | Phase 12 | Pending |
| RS-02 | Phase 12 | Pending |
| RS-03 | Phase 12 | Pending |
| RSLT-01 | Phase 13 | Pending |
| RSLT-02 | Phase 13 | Pending |
| RSLT-03 | Phase 13 | Pending |
| RSLT-04 | Phase 13 | Pending |

**Coverage:**
- v1.4 requirements: 7 total
- Mapped to phases: 7
- Unmapped: 0 ✓

---
*Requirements defined: 2026-02-16*
*Last updated: 2026-02-16 after roadmap creation*
