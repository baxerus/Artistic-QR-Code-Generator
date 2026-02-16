# Feature Research: RS Error Correction Exposure

**Domain:** QR Error Measurement for Artistic QR Code Generator
**Researched:** 2026-02-16
**Confidence:** HIGH (based on jsQR source code analysis + QR standard documentation)

## Context

This is milestone-specific research for adding Reed-Solomon error correction count display to an existing Artistic QR Code Generator. The app already has:
- Hash optimization search across multiple workers
- Pixel diff error counting (comparing painted vs generated QR modules)
- Top 10 results display with error visualization
- Auto-stop when 5 perfect results found (currently: 0 pixel errors)

**Goal:** Measure actual RS correction burden so users know how much error correction headroom remains.

## Feature Landscape

### Table Stakes (Users Expect These)

Features users assume exist when a tool claims to measure RS errors.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| RS correction count displayed | Core promised feature; "how many bytes did the decoder correct?" | MEDIUM | Requires modifying jsQR decode function to expose `errorLocations.length` |
| RS count visible on result cards | Consistency with existing pixel diff display | LOW | Add to existing result card UI pattern |
| Clear labeling ("RS corrections" or "Codewords corrected") | Users need to understand what the number means | LOW | Terminology should match QR spec ("codewords") or be user-friendly |
| Sortable/ranked by RS corrections | Users want best results first; RS is primary quality metric | LOW | Already have sorting logic; add RS as primary key |
| Zero RS = truly perfect | Perfect result means both pixel diff=0 AND RS corrections=0 | LOW | Update auto-stop condition to check both metrics |

### Differentiators (Competitive Advantage)

Features that add value beyond basic RS exposure.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| RS capacity indicator | "X of Y possible corrections used" shows headroom remaining | MEDIUM | Requires lookup table: EC level + version -> max corrections per block |
| Per-block RS breakdown | Show corrections per data block (QR uses multiple RS blocks) | HIGH | Would need to aggregate from each `dataBlock` decode; adds complexity |
| Visual RS health bar | Progress bar showing 0% (perfect) to 100% (at capacity) | LOW | Simple UI addition once we have count and max |
| RS vs pixel diff comparison | Show both metrics side-by-side for education | LOW | Already planned; helps users understand pixel!=codeword |
| Explanation tooltip | "RS corrections measure codewords fixed, not pixels changed" | LOW | Educational; reduces confusion |

### Anti-Features (Commonly Requested, Often Problematic)

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| "Exact RS error locations" visualization | "Show me exactly which bytes were corrected" | RS works at codeword level, not pixel level; byte positions don't map cleanly to visual pixels | Show pixel diff for visual conflicts; RS count for scannability margin |
| Per-module RS attribution | "Which painted pixel caused which RS correction?" | RS corrects bytes, not modules; no 1:1 correspondence; would be misleading | Keep pixel diff for "which pixels mismatch"; RS for overall health |
| RS success probability | "What's the chance this will scan?" | Depends on scanner implementation, viewing conditions, print quality—too many variables | RS count relative to capacity is the honest metric |
| Force decode mode | "Decode even if RS fails" | If RS can't recover, data is corrupted; returning garbage isn't useful | Show "decode failed" clearly; that's valuable information |
| Alternative EC level support | "Let me choose L/M/Q for higher capacity" | Artistic corruption needs H (30% tolerance); lower levels fail immediately | Document why H is required; don't offer false flexibility |

## Feature Dependencies

```
[Existing Pixel Diff System]
    └──integrates-with──> [RS Correction Count]
                              └──requires──> [Modified jsQR decode function]
                                                 └──exposes──> [errorLocations.length per block]

[RS Correction Count]
    └──enables──> [Dual-metric ranking (RS primary, pixel diff secondary)]
                      └──enables──> [Updated auto-stop condition (RS=0 AND pixelDiff=0)]

[RS Correction Count]
    └──enables──> [RS Capacity Indicator] (optional enhancement)
                      └──requires──> [EC capacity lookup table]
```

### Dependency Notes

- **RS count requires jsQR modification:** The inlined jsQR library returns corrected bytes but discards `errorLocations.length`. We need to modify the RS decoder to return this count alongside the data.
- **Dual-metric ranking depends on RS count:** Can't rank by RS until we have it. Existing pixel diff remains useful as tiebreaker.
- **Auto-stop condition depends on both metrics:** "Perfect" result now means RS=0 AND pixelDiff=0, not just pixelDiff=0.
- **RS capacity indicator is optional:** Nice to have but not required for core milestone.

## MVP Definition (This Milestone)

### Launch With (v1.4)

- [x] Modify jsQR RS decoder to return correction count per block — **Essential**
- [x] Aggregate total RS corrections across all blocks — **Essential**
- [x] Display RS correction count on result cards — **Essential**
- [x] Sort results by RS corrections (primary), pixel diff (secondary) — **Essential**
- [x] Update auto-stop to require RS=0 AND pixelDiff=0 — **Essential**
- [ ] Clear labeling distinguishing RS corrections from pixel diff — **Essential**

### Add After Validation (v1.4.x)

- [ ] RS capacity indicator ("X of Y corrections used") — **Trigger**: Users ask "is this close to failing?"
- [ ] Visual health bar for RS margin — **Trigger**: Users want quick visual assessment
- [ ] Explanation tooltip for RS vs pixel diff — **Trigger**: Users confused by two different numbers

### Future Consideration (v2+)

- [ ] Per-block RS breakdown display — **Why defer**: Adds UI complexity; total count is sufficient
- [ ] RS correction trend graph — **Why defer**: Overkill for artistic QR use case

## Technical Details

### How QR RS Error Correction Works

**Key insight:** RS correction happens at the *codeword* (byte) level, not pixel level.

1. QR data is encoded as 8-bit codewords (bytes)
2. RS redundancy codewords are appended for each data block
3. Decoder calculates syndromes to detect errors
4. If syndromes are non-zero, RS algorithm locates and corrects errors
5. **Number of corrections = `errorLocations.length`** in jsQR code

**Correction capacity by EC level:**
- Level L: ~7% of codewords correctable
- Level M: ~15% of codewords correctable
- Level Q: ~25% of codewords correctable
- Level H: ~30% of codewords correctable (our hardcoded level)

### jsQR RS Decoder Location

In inlined jsQR (index.html lines ~10754-10808):

```javascript
function decode(bytes, twoS) {
  // ... syndrome calculation ...
  if (!error) {
    return outputBytes;  // No errors, returns without count
  }
  // ... error location finding ...
  var errorLocations = findErrorLocations(field, sigmaOmega[0]);
  // errorLocations.length IS THE RS CORRECTION COUNT
  // ... error magnitude calculation and correction ...
  return outputBytes;  // Currently only returns bytes, not count
}
```

**Modification needed:** Return `{ bytes: outputBytes, correctionCount: errorLocations ? errorLocations.length : 0 }` instead of just `outputBytes`.

### Pixel Diff vs RS Corrections

| Metric | What It Measures | Why It Matters |
|--------|------------------|----------------|
| **Pixel diff** | Modules where painted pattern disagrees with QR encoding | Visual conflict count; shows where art interferes with QR |
| **RS corrections** | Codewords the decoder had to fix | Actual scannability burden; measures error correction usage |

**They're not the same because:**
- One painted pixel might cause 0 RS corrections (if it happens to match what RS would produce anyway)
- One painted pixel might cause multiple RS corrections (if it corrupts data spanning byte boundaries)
- Multiple painted pixels might cause 1 RS correction (if they all corrupt the same codeword)

**Display strategy:** Show both metrics; RS is primary for ranking, pixel diff is supplementary for understanding.

### Result Card Display Example

```
#abc123                    [Download] [Copy URL]
RS corrections: 3          (out of max ~30 for version 4-H)
Pixel mismatches: 12       (painted pixels that differ from QR)
[QR Preview]               [Error Visualization]
```

### Ranking Logic

```javascript
// Sort by RS corrections (lower is better), then pixel diff (lower is better)
results.sort((a, b) => {
  if (a.rsCorrections !== b.rsCorrections) {
    return a.rsCorrections - b.rsCorrections;
  }
  return a.pixelDiff - b.pixelDiff;
});
```

### Auto-Stop Condition Update

```javascript
// Current: stop when 5 results have pixelDiff === 0
// New: stop when 5 results have BOTH rsCorrections === 0 AND pixelDiff === 0
const perfectResults = results.filter(r => r.rsCorrections === 0 && r.pixelDiff === 0);
if (perfectResults.length >= 5) {
  stopSearch();
}
```

## Pitfalls to Avoid

### Common Mistake: Assuming Pixel Diff = RS Corrections

**Wrong:** "12 pixel mismatches means 12 RS corrections"
**Right:** They're different metrics measuring different things

### Common Mistake: Reporting "% Error Correction Used"

**Careful:** RS capacity varies by block. QR codes have multiple data blocks, each with its own RS capacity. Simply dividing total corrections by total capacity may be misleading if corrections cluster in one block.

**Recommendation for v1.4:** Just show raw count. Capacity indicator can come later with proper per-block accounting.

### Common Mistake: Assuming RS=0 Always Possible

**Reality:** Some painted patterns may be impossible to align with any URL hash—every variant might need corrections.
**Recommendation:** Don't treat RS>0 as "failure"; it's useful information. RS=3 is perfectly scannable.

### Common Mistake: Modifying jsQR in Multiple Places

**Risk:** The RS decoder is called from multiple code paths (normal decode, mirrored decode).
**Recommendation:** Modify the core `decode` function in reedsolomon module to return count. Changes propagate automatically.

## Sources

### Primary (HIGH confidence)

- **jsQR source code analysis** (in index.html lines 10754-10808): Direct examination of Reed-Solomon decoder implementation
- **Thonky QR Code Tutorial - Error Correction Coding**: https://www.thonky.com/qr-code-tutorial/error-correction-coding
- **ISO/IEC 18004:2015** (QR Code standard): Defines RS error correction for QR codes

### Secondary (MEDIUM confidence)

- **jsQR GitHub Repository**: https://github.com/cozmo/jsQR - Confirms jsQR doesn't expose RS count in public API
- **ZXing-js Library**: https://github.com/zxing-js/library - Alternative decoder, also doesn't expose RS count by default
- **DENSO WAVE Error Correction Feature**: https://www.qrcode.com/en/about/error_correction.html

### Note on API Availability

**Neither jsQR nor ZXing-js expose RS correction counts in their public APIs.** Both libraries return only:
- Decoded data (string/bytes)
- Location coordinates
- Version number
- Binary data

To get RS correction count, we must modify the inlined jsQR library. This is acceptable because:
1. jsQR is already inlined (bundled into index.html)
2. Modification is localized to RS decoder
3. No external dependency to break

## Confidence Assessment

| Area | Confidence | Reason |
|------|------------|--------|
| RS count is `errorLocations.length` | HIGH | Direct source code analysis of jsQR |
| jsQR modification approach | HIGH | Localized change, well-understood code path |
| Pixel diff vs RS relationship | HIGH | Follows from QR standard (byte vs module) |
| Ranking logic (RS primary) | HIGH | Matches user goal (scannability margin) |
| Auto-stop condition | HIGH | Logical extension of existing behavior |
| RS capacity calculation | MEDIUM | Per-block accounting adds complexity; defer to future |

---
*Feature research for: RS Error Correction Exposure*
*Milestone: v1.4 RS Error Measurement*
*Researched: 2026-02-16*
