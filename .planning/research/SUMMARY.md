# Research Summary: v1.5 UX & Export Enhancements

**Synthesized:** 2026-02-17
**Research Files:** STACK.md, FEATURES.md, ARCHITECTURE.md, PITFALLS.md
**Confidence:** HIGH
**Scope:** Corrections capacity display (X of Y), SVG export button, Shift+paint shortcut

---

## Executive Summary

The v1.5 milestone adds three focused UX improvements to the existing Artistic QR Code Generator: displaying RS correction capacity alongside the current count, adding SVG export alongside PNG, and implementing Shift+click as an alternative to right-click for opposite-color painting.

**All three features require zero external dependencies.** The existing codebase already contains the RS capacity data (in the VERSIONS table within jsQR), the module data needed for SVG generation, and pointer events that expose `shiftKey`. Each feature integrates at a single, well-defined point with ~30-60 lines of new code.

**Key risk:** The RS capacity formula requires dividing by 2 (RS corrects t errors using 2t parity bytes). The pitfalls document identifies this as the most likely implementation error. The other features are straightforward with well-documented patterns.

---

## Key Findings

### From STACK.md

No new dependencies needed for v1.5:

| Technology | Status | Notes |
|------------|--------|-------|
| Canvas API | Already present | Used for existing rendering |
| SVG generation | Native browser | Use DOM/string generation, no library |
| Pointer events | Already present | `shiftKey` property available |
| jsQR VERSIONS table | Already embedded | Contains RS capacity data |

**Stack additions needed:** Zero. All features use native APIs and existing embedded libraries.

### From FEATURES.md

| Feature | Complexity | Estimated Lines | Priority |
|---------|------------|-----------------|----------|
| RS Capacity Display | LOW | ~30 | Must-have |
| SVG Export Button | LOW | ~60 | Must-have |
| Shift+Paint Shortcut | LOW | ~30 | Must-have |

**Total estimated changes:** ~120 lines

**Display format recommendation:** `RS: X of Y` - clear, matches existing label style

**SVG approach:** Generate from `moduleData` directly using path elements with move commands (compact output)

**Shift behavior:** Treat `Shift+left-click` equivalent to `right-click` (paints opposite color)

### From ARCHITECTURE.md

**Integration points are clean and isolated:**

| Feature | Integration Point | Modification Scope |
|---------|-------------------|-------------------|
| RS Capacity | `updateTopResults()` + new `getMaxCorrectionsCapacity()` | 1 function modified, 1 added |
| SVG Export | Result card actions + new `renderResultToSVG()` | 2 functions added, 1 location modified |
| Shift+Paint | `getPaintValue()`, `paintAt()`, `setupPaintingEvents()` | 3 functions modified |

**Critical discovery:** The VERSIONS table in jsQR (lines 10856-12800) contains the authoritative RS capacity data. Must expose it via `jsQR._VERSIONS` for capacity lookup.

**Capacity formula:**
```javascript
maxCapacity = sum(ecBlocks.map(b => b.numBlocks * ecCodewordsPerBlock)) / 2
```

### From PITFALLS.md

**Critical pitfalls (must address):**

| Pitfall | Risk | Mitigation |
|---------|------|------------|
| Wrong RS capacity formula | HIGH | Divide EC codewords by 2, test against spec table |
| Version table off-by-one | MEDIUM | Use `(version - 1)` for 0-indexed lookup, test version 2+ |
| SVG colors mismatch | MEDIUM | Export final QR state (black/white), not preview colors |
| Shift key stale state | LOW | Use `event.shiftKey` from pointer event, not separate listener |

**Minor pitfalls (low risk):**
- SVG viewBox/size mismatch (test in multiple apps)
- Touch devices can't use Shift (document as keyboard shortcut)
- File naming clutter (include hash or timestamp)

---

## Implications for Roadmap

### Suggested Phase Structure

All three features have **zero dependencies on each other** and can be built in parallel. However, a sequential approach by risk level is recommended:

#### Phase 1: Shift+Paint Shortcut
**Rationale:** Lowest risk, purely UI-level change, impossible to break existing functionality

**Delivers:**
- `Shift+left-click` paints opposite color (equivalent to right-click)
- Accessibility benefit for trackpad users without right-click

**Implementation:**
- Modify `getPaintValue()` to accept `shiftKey` parameter
- Modify `paintAt()` to pass `shiftKey`
- Modify pointer event handlers to capture `shiftKey`

**Pitfalls to avoid:**
- Don't use separate `keydown`/`keyup` listeners (stale state risk)
- Capture shift state at stroke start, not during drag

#### Phase 2: SVG Export Button
**Rationale:** Low risk, additive feature parallel to existing PNG export

**Delivers:**
- "Download SVG" button on result cards alongside PNG
- Vector-crisp output using path elements

**Implementation:**
- Add `renderResultToSVG(moduleData, moduleCount)` function
- Add `downloadAsFile()` helper (reusable for future formats)
- Add button to result card actions

**Pitfalls to avoid:**
- Export final QR colors (black/white), not preview grays
- Use Blob URL for Safari compatibility
- Include meaningful filename with hash or timestamp

#### Phase 3: RS Capacity Display
**Rationale:** Highest risk due to formula complexity, but isolated to display layer

**Delivers:**
- "RS: X of Y" format showing current corrections and maximum capacity
- Clearer feedback on how much headroom remains

**Implementation:**
- Expose `jsQR._VERSIONS` at end of jsQR section
- Add `getMaxCorrectionsCapacity(version)` function using formula
- Modify label creation in `updateTopResults()`

**Pitfalls to avoid:**
- **CRITICAL:** Divide EC codewords by 2 to get correction capacity
- Account for multiple RS blocks (sum across all)
- Use `(version - 1)` for 0-indexed table lookup

### Research Flags

| Phase | Research Needed? | Notes |
|-------|------------------|-------|
| Phase 1 (Shift+Paint) | NO | Standard pointer event pattern |
| Phase 2 (SVG Export) | NO | Well-documented SVG generation |
| Phase 3 (RS Capacity) | LIGHT | Verify formula against QR spec for one version |

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Zero new dependencies, all native APIs |
| Features | HIGH | Clear requirements, simple implementations |
| Architecture | HIGH | Direct codebase analysis, integration points identified |
| Pitfalls | HIGH | Formula validated against QR spec, patterns documented |

### Gaps to Address

1. **Formula verification:** Should validate capacity formula against official QR spec for at least one version during Phase 3 implementation
2. **Safari SVG download:** May need testing on actual Safari to confirm Blob URL behavior
3. **Firefox Shift+right-click:** May need testing, though current `contextmenu` prevention should work

---

## Sources

### High Confidence (Official Documentation)
- MDN: SVG Tutorial, Blob API, PointerEvent.shiftKey
- QR Code spec (ISO/IEC 18004) for RS capacity tables
- Direct codebase analysis of `/workspace/index.html`

### Medium Confidence (Validated Libraries)
- Nayuki QR Code Generator documentation (existing embedded library)
- jsQR VERSIONS table structure analysis

### Low Confidence (Requires Validation)
- Safari-specific SVG download behavior (needs testing)
- Firefox Shift+right-click context menu bypass (needs testing)

---

## Summary

**v1.5 is a low-risk, high-value milestone.** All three features:
- Require zero external dependencies
- Have clear, isolated integration points
- Can be implemented in ~120 lines total
- Have well-documented pitfalls with clear mitigations

**Primary risk:** Getting the RS capacity formula wrong (dividing by 2 is required). This is easily caught by testing against known spec values.

**Recommended approach:** Build in order of increasing risk (Shift+Paint > SVG Export > RS Capacity), though parallel development is also viable given the complete isolation between features.

---

*Research synthesis for: v1.5 UX & Export Enhancements*
*Synthesized: 2026-02-17*
