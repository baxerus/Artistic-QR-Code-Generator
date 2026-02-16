# Stack Research: RS Error Correction Count Extraction

**Domain:** QR Code decoding with Reed-Solomon error metrics  
**Researched:** 2026-02-16  
**Confidence:** HIGH  
**Scope:** Focused research for Milestone 2 - Does NOT supersede main STACK.md

## Executive Summary

**jsQR does NOT expose Reed-Solomon correction counts.** The underlying RS decoder (`reedsolomon_1.decode()`) computes `errorLocations.length` (the correction count) but the `decode()` function returns only `outputBytes`, discarding this data. This is not a configuration issue—it's a structural limitation in jsQR's architecture.

**The question asked:** "Does jsQR expose RS correction counts? What alternatives exist?"

**Answer:** 
1. jsQR: **NO** - correction count computed but discarded
2. zxing-wasm: **NO** - same architectural pattern
3. All other browser QR decoders: **NO** - none expose this metric

**Recommended approach:** Modify the inlined jsQR code to bubble up correction counts. This is the lowest-complexity, zero-dependency solution that maintains the single-file architecture.

## The Problem: Where Correction Count Gets Lost

### jsQR Code Flow (verified via source inspection)

```
jsQR() → scan() → extract() → decode() → decodeMatrix() 
    → reedsolomon_1.decode() → Returns only bytes (CORRECTION COUNT DISCARDED HERE)
```

### Specific Code Locations in index.html

| Function | Line Numbers | What It Does | The Problem |
|----------|--------------|--------------|-------------|
| `reedsolomon_1.decode()` | 10754-10807 | RS error correction | Returns `outputBytes`, discards `errorLocations.length` |
| `decodeMatrix()` | 3190-3239 | Aggregates data blocks | Calls RS decode in loop, no correction count aggregation |
| `decode()` | 3240-3259 | Entry point | Only returns decoded data |
| `jsQR()` | 2521-2552 | Main API | Returns `{binaryData, data, chunks, version, location}` - no RS info |

### The Discarded Value

In `reedsolomon_1.decode()` at line 10787-10806:
```javascript
var errorLocations = findErrorLocations(field, sigmaOmega[0]);
// ... errorLocations.length IS the correction count
for (var i = 0; i < errorLocations.length; i++) {
  // Apply corrections
}
return outputBytes;  // <-- errorLocations.length is lost here!
```

## Alternatives Analysis

### 1. zxing-wasm (v2.2.4)

**Checked:** 2026-02-16  
**Does it expose RS counts?** NO

The `ReadResult` interface from zxing-wasm includes:
- `ecLevel` (error correction LEVEL like "H")
- `isValid`, `error`, `format`, `bytes`, `text`
- `version`, `position`, `orientation`
- **NO correction count field**

ZXing-C++ source (`QRDecoder.cpp`) shows same pattern:
```cpp
static bool CorrectErrors(ByteArray& codewordBytes, int numDataCodewords) {
  // ...
  if (!ReedSolomonDecode(...))
    return false;  // Only returns bool, not count
  return true;
}
```

**Additional problems with zxing-wasm:**
- Requires external .wasm file (~919KB for reader-only)
- Breaks single-file HTML architecture
- Requires fetch() to load WASM (incompatible with file:// protocol without extra work)
- Would need to inline base64 WASM (~1.2MB base64) to maintain single-file

### 2. Other Browser QR Decoders

| Library | Exposes RS Count? | Notes |
|---------|-------------------|-------|
| qr-scanner | NO | Wrapper, doesn't add metrics |
| @nuintun/qrcode | NO | Same decode pattern |
| @aspect-dev/jsqr | Not found | No longer exists on npm |

### 3. Why No Library Exposes This

QR decoders are designed for **"decode or fail"** semantics. The RS correction count is:

1. **Computed** during decoding (in `findErrorLocations()`)
2. **Used** to correct errors
3. **Discarded** because consumers typically only care: "Did it decode?"

The Artistic QR Generator is unusual in wanting this metric for quality assessment during optimization search.

## Recommended Solution: Modify Inlined jsQR

### Why This Is The Right Approach

| Approach | Pros | Cons |
|----------|------|------|
| **Modify inlined jsQR** | Zero new dependencies, maintains single-file, minimal changes | Must maintain custom changes |
| Fork jsQR on GitHub | Upstream contribution | Overkill for internal use |
| Use zxing-wasm | More features | Breaks single-file, still no RS count |
| Build custom decoder | Full control | Massive effort, reinventing wheel |

### Required Modifications (~20 lines total)

#### 1. Modify `reedsolomon_1.decode()` (2 lines)

**Current** (line ~10807):
```javascript
return outputBytes;
```

**Modified:**
```javascript
return { bytes: outputBytes, correctionCount: errorLocations ? errorLocations.length : 0 };
```

Also handle the early return for no errors (line ~10772):
```javascript
if (!error) {
  return { bytes: outputBytes, correctionCount: 0 };
}
```

#### 2. Modify `decodeMatrix()` (5 lines)

**Current** (lines 3219-3229):
```javascript
var correctedBytes = reedsolomon_1.decode(dataBlock.codewords, ...);
if (!correctedBytes) {
  return null;
}
for (var i = 0; i < dataBlock.numDataCodewords; i++) {
  resultBytes[resultIndex++] = correctedBytes[i];
}
```

**Modified:**
```javascript
var rsResult = reedsolomon_1.decode(dataBlock.codewords, ...);
if (!rsResult) {
  return null;
}
totalCorrectionCount += rsResult.correctionCount;
for (var i = 0; i < dataBlock.numDataCodewords; i++) {
  resultBytes[resultIndex++] = rsResult.bytes[i];
}
```

Add at function start: `var totalCorrectionCount = 0;`

#### 3. Modify `decode()` return (2 lines)

Pass `totalCorrectionCount` through to final return object.

#### 4. Modify `jsQR()` return (1 line)

Add `correctionCount` to the returned `QRCode` object.

### New Return Structure

```javascript
// Current jsQR return
{
  binaryData: [...],
  data: "...",
  chunks: [...],
  version: 3,
  location: { ... }
}

// After modification
{
  binaryData: [...],
  data: "...",
  chunks: [...],
  version: 3,
  location: { ... },
  correctionCount: 3  // NEW: Total RS corrections across all data blocks
}
```

### Integration with Existing Worker Code

In the worker (line ~15284):
```javascript
// Current
const code = jsQR(imageData.data, imageData.width, imageData.height);
decodable = (code !== null);

// After modification - can use correction count for quality metric
const code = jsQR(imageData.data, imageData.width, imageData.height);
decodable = (code !== null);
const correctionCount = code ? code.correctionCount : Infinity;
// Can now report correctionCount alongside decodable status
```

## What NOT to Add

| Avoid | Why | Impact |
|-------|-----|--------|
| **zxing-wasm** | External .wasm file breaks single-file; still doesn't expose RS counts | +900KB dependency, deployment change |
| **Any npm package** | Project is single-file HTML, no build system | Architectural mismatch |
| **WASM compilation** | Overkill for extracting one number | Massive complexity increase |
| **Upstream jsQR fork** | Library is effectively unmaintained since 2021 | PR unlikely to be merged |

## Verification Protocol

### Confidence Assessment

| Claim | Confidence | Verification |
|-------|------------|--------------|
| jsQR doesn't expose RS counts | HIGH | Direct source inspection of jsQR master branch |
| zxing-wasm doesn't expose RS counts | HIGH | ReadResult interface inspection + ZXing-C++ source |
| Modification approach is feasible | HIGH | Code flow traced, ~20 lines identified |
| No browser library exposes this | MEDIUM | Checked major libraries; rare metric need |

### Sources

| Source | What Was Verified | Confidence |
|--------|-------------------|------------|
| https://github.com/cozmo/jsQR/blob/master/src/decoder/reedsolomon/index.ts | RS decode returns only bytes | HIGH |
| https://github.com/cozmo/jsQR/blob/master/src/decoder/decoder.ts | decodeMatrix() discards count | HIGH |
| https://github.com/zxing-cpp/zxing-cpp/blob/master/core/src/qrcode/QRDecoder.cpp | CorrectErrors() returns bool | HIGH |
| https://github.com/Sec-ant/zxing-wasm/blob/main/src/bindings/readResult.ts | ReadResult has no correctionCount | HIGH |
| npm registry jsqr | v1.4.0, last modified 2025-11-13 | HIGH |
| npm registry zxing-wasm | v2.2.4, last modified 2025-11-18 | HIGH |

## Summary for Roadmap

### Key Finding
The RS correction count IS computed by jsQR but immediately discarded. A small modification (~20 lines) to the inlined jsQR code can expose this metric without adding any dependencies.

### Implementation Complexity
**LOW** - The changes are:
1. Additive (don't change existing behavior)
2. Localized (4 functions)
3. Small (20 lines total)
4. Testable (compare modified vs unmodified decode results)

### Risk Assessment
| Risk | Likelihood | Mitigation |
|------|------------|------------|
| Changes break decoding | LOW | Test unchanged behavior first |
| Performance impact | NEGLIGIBLE | One integer addition per block |
| Future jsQR updates | N/A | Library unmaintained since 2021 |

---

*Research for: Milestone 2 - RS Error Correction Count Extraction*  
*Researched: 2026-02-16*  
*Supplements but does not replace: .planning/research/STACK.md*
