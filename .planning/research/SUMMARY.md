# Project Research Summary

**Project:** Artistic QR Code Generator — RS Error Correction Exposure (v1.4)
**Domain:** QR Code optimization with Reed-Solomon error metrics
**Researched:** 2026-02-16
**Confidence:** HIGH

## Executive Summary

This milestone adds Reed-Solomon error correction count display to an existing Artistic QR Code Generator. The critical discovery is that **jsQR does NOT expose RS correction counts** — the library calculates `errorLocations.length` internally but discards it. Modification of the inlined jsQR source is required; no new dependencies are needed. The fix involves patching 4 functions in jsQR to bubble up error counts from the RS decoder through to the public API.

The recommended approach is a 4-phase build order: (1) patch jsQR's RS decoder to return correction counts, (2) propagate counts through jsQR's decode chain, (3) update the Web Worker to include counts in messages, (4) update the UI to display RS corrections alongside existing pixel diff metrics. This order follows strict dependencies — each phase's output is required by the next.

Key risks include confusing module-level pixel differences with codeword-level RS corrections (they are fundamentally different metrics), and failing to verify that the modified jsQR propagates to workers (which extract jsQR source dynamically). Both risks are mitigated by understanding the architecture: pixel diff measures visual conflicts, RS corrections measure scannability burden; workers automatically get modified jsQR because they extract it from the same `<script>` tag.

## Key Findings

### Recommended Stack

**No new dependencies required.** The existing stack (Nayuki QR encoder, inlined jsQR decoder, Canvas API, Web Workers) is sufficient. The solution is purely architectural — modifying inlined jsQR to expose what it already calculates.

**Core technologies (existing, unchanged):**
- **Nayuki QR Generator**: QR encoding with module-level access — already inlined
- **jsQR Decoder**: QR decoding with RS error correction — needs modification, not replacement
- **Canvas API**: Pixel manipulation and QR rendering — native browser API
- **Web Workers**: Parallel hash optimization — existing architecture preserved

### Expected Features

**Must have (table stakes):**
- RS correction count displayed on result cards — core promised feature
- Clear labeling distinguishing RS corrections from pixel diff — user understanding
- Sorting by RS corrections (primary), pixel diff (secondary) — quality ranking
- Updated auto-stop: RS=0 AND pixelDiff=0 for "perfect" — correctness

**Should have (differentiators):**
- RS capacity indicator ("X of Y corrections used") — shows headroom remaining
- Visual RS health bar — quick assessment
- Explanation tooltip for RS vs pixel diff — user education

**Defer (v2+):**
- Per-block RS breakdown display — adds UI complexity, total count is sufficient
- RS correction trend graph — overkill for artistic QR use case

### Architecture Approach

The integration follows a 4-function modification pattern within the existing jsQR bundle. The RS decoder at lines 10754-10808 calculates error counts but discards them. Changes propagate from RS decoder → decodeMatrix → decode → scan → jsQR public API. The worker architecture automatically gets modified jsQR because it extracts source from the same `<script>` tag.

**Major components (modification points):**
1. **reedsolomon.decode()** (line 10754-10808) — return `{bytes, corrections}` instead of just `bytes`
2. **decodeMatrix()** (line 3190-3239) — accumulate corrections across all RS blocks
3. **decode()/scan()** (lines 3240-3258, 2487-2513) — include `rsCorrections` in return value
4. **Worker evaluateCandidate()** (line 15260-15316) — extract and report `rsCorrections`

### Critical Pitfalls

1. **jsQR doesn't expose RS counts** — Accept modification is required; patch the RS `decode()` function to return `{bytes, corrections: errorLocations.length}`

2. **Module diff ≠ RS corrections** — Track both separately. One pixel change may cause 0, 1, or multiple RS corrections depending on codeword boundaries. Display both metrics; use RS as primary ranking.

3. **Worker gets modified source automatically** — The worker extracts jsQR from `document.querySelectorAll("script")[1]`. Modifying the inlined script propagates automatically. Verify by logging extracted source.

4. **Per-block RS tracking matters** — QR codes have multiple RS blocks. One block at capacity = failure even if total looks fine. Track per-block metrics: `{blocks: [{count, capacity}...]}`.

5. **Only success paths return error counts** — Modify RS decoder to always return metrics, even on failure. Failure cases need "how close?" data for optimization ranking.

## Implications for Roadmap

Based on research, suggested phase structure follows strict build-order dependencies:

### Phase 1: RS Decoder Modification
**Rationale:** This is the foundation — all other phases depend on RS counts being available
**Delivers:** Modified jsQR RS decoder that returns `{bytes, corrections}` instead of just `bytes`
**Addresses:** Core data extraction — the `errorLocations.length` value that exists but is discarded
**Avoids:** Trying to add separate RS decoder (duplicates work, different counts possible)
**Location:** Lines 10754-10808 in index.html (jsQR bundle, module 9)

### Phase 2: jsQR Decode Chain Propagation
**Rationale:** RS counts must flow from inner decoder to public API before worker can access them
**Delivers:** jsQR() public API returning `rsCorrections` field alongside existing return values
**Uses:** Modified RS decoder from Phase 1
**Implements:** 3 function modifications: decodeMatrix, decode, scan
**Avoids:** Incomplete call chain (modifying only inner function, counts never reach API)
**Location:** Lines 3190-3258 (module 5) and 2487-2513 (module 3)

### Phase 3: Worker Integration
**Rationale:** Optimization runs in workers — RS counts must flow through worker message protocol
**Delivers:** Worker messages include `rsCorrections` field; results sorted by RS (primary), pixel diff (secondary)
**Uses:** Modified jsQR from Phase 2 (workers auto-extract modified source)
**Implements:** evaluateCandidate modification, message protocol update, sorting logic update
**Avoids:** Message serialization losing fields (use flat structure)
**Location:** Lines 15260-15430 (worker code string), 15458-15498 (message handler)

### Phase 4: UI Display
**Rationale:** Final phase — display what we now have access to
**Delivers:** Result cards showing RS corrections, updated auto-stop condition (RS=0 AND pixelDiff=0)
**Addresses:** Table stakes features: RS count displayed, clear labeling, sorting
**Avoids:** Confusing users by mixing RS corrections with pixel diff (display both clearly)
**Location:** Main thread UI code

### Phase Ordering Rationale

- **Strict dependency chain:** RS decoder → jsQR API → Worker → UI. Each phase's output is required input for the next.
- **Single-file architecture preserved:** All changes in index.html, no new files or dependencies.
- **Worker auto-propagation:** No separate worker modification needed for jsQR — it extracts from the same script tag.
- **Testing at each phase:** Phase 1 can be tested with console.log in RS decoder; Phase 2 with direct jsQR() call; Phase 3 with worker message inspection; Phase 4 with visual verification.

### Research Flags

Phases with well-documented patterns (skip research-phase):
- **Phase 1-4:** All phases have clear implementation paths from direct code analysis. The jsQR source is available, modification points are identified with line numbers, and the architecture is fully understood. No additional research needed.

Validation during implementation:
- **Phase 1:** Verify RS counts are accurate for known-error QR codes
- **Phase 2:** Verify multi-block QRs correctly sum corrections across blocks
- **Phase 3:** Verify worker messages include rsCorrections field (log both sides)
- **Phase 4:** Verify auto-stop triggers only when both RS=0 AND pixelDiff=0

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | No new dependencies; modification of existing inlined code only |
| Features | HIGH | Clear table stakes from jsQR source analysis; MVP scope well-defined |
| Architecture | HIGH | Direct code analysis with line numbers; call chain fully traced |
| Pitfalls | HIGH | Based on deep understanding of RS theory and jsQR implementation |

**Overall confidence:** HIGH

### Gaps to Address

- **Per-block capacity calculation:** RS capacity varies by QR version and block. For v1.4, show raw count. Capacity indicator ("X of Y") deferred — requires lookup table mapping (version, EC level) → max corrections per block.

- **Failure case metrics:** Current plan returns -1 for rsCorrections when decode fails. Future enhancement: return `{success: false, detectedErrors, capacity}` to rank failed attempts by "how close."

## Sources

### Primary (HIGH confidence)
- **jsQR source code** (index.html lines 2080-14375) — Direct analysis of RS decoder, decode chain, return values
- **index.html architecture** — Worker extraction, message protocol, existing metrics
- **ISO/IEC 18004:2015** — QR code RS specification (correction capacity = ecCodewords / 2)

### Secondary (MEDIUM confidence)
- **Thonky QR Code Tutorial** — Error correction coding explanation
- **Reed-Solomon Wikipedia** — Theory validation

### Note on API Availability
Neither jsQR nor any alternative decoder (ZXing-js, @paulmillr/qr) exposes RS correction counts in their public APIs. Source modification is the only path for this feature.

---
*Research completed: 2026-02-16*
*Ready for roadmap: yes*
