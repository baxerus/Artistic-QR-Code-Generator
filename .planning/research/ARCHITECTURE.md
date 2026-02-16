# Architecture Research: RS Error Correction Integration

**Domain:** QR Code optimization with Reed-Solomon error correction metrics
**Researched:** 2026-02-16
**Confidence:** HIGH (based on direct code analysis of existing implementation)

## Context: Adding RS Correction Count to Existing Architecture

This document focuses specifically on **integrating Reed-Solomon error correction measurement** into the existing Artistic QR Code Generator. For general architecture patterns, see previous research from 2026-02-06.

## Current Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Main Thread (index.html)                           │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │ URL Input   │  │ Paint Canvas │  │ Results UI   │  │ Worker Manager  │  │
│  │ + Version   │  │ + Pattern    │  │ + Progress   │  │ (createWorker   │  │
│  │   Select    │  │   State      │  │   Display    │  │  Pool)          │  │
│  └──────┬──────┘  └──────┬───────┘  └──────▲───────┘  └───────┬─────────┘  │
│         │                │                 │                   │            │
│         └────────────────┴─────────────────┼───────────────────┘            │
│                                            │                                │
│         postMessage(START_OPTIMIZATION)    │  onmessage(PROGRESS/COMPLETE)  │
│                    ↓                       │           ↑                    │
├────────────────────┼───────────────────────┴───────────┼────────────────────┤
│                    │        Worker Pool (N workers)    │                    │
│  ┌─────────────────┴───────────────────────────────────┴─────────────────┐  │
│  │                        Web Worker (Blob URL)                          │  │
│  │  ┌────────────────┐  ┌────────────────┐  ┌─────────────────────────┐  │  │
│  │  │ QR Encoder     │  │ jsQR Decoder   │  │ Search Loop             │  │  │
│  │  │ (qrcodejs      │→ │ (full bundled  │→ │ - Generate hash         │  │  │
│  │  │  extracted)    │  │  library)      │  │ - Evaluate candidate    │  │  │
│  │  └────────────────┘  └────────────────┘  │ - Track top results     │  │  │
│  │                                          └─────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Component Locations

| Component | Responsibility | Location in Code |
|-----------|----------------|------------------|
| QR Encoder (qrcodejs) | Generate QR matrix from URL+hash | Lines 926-2076 (inlined library) |
| jsQR Decoder | Decode QR from image data, verify scannability | Lines 2080-14375 (inlined bundle) |
| RS Decoder | Reed-Solomon error correction (inside jsQR) | Lines 10646-10808 (module 9) |
| Worker Manager | Create/terminate worker pool, merge results | Lines 14548-15694 |
| Worker Code Factory | Generate worker script string | `createOptimizationWorker()` line 14396 |
| evaluateCandidate | Generate QR, decode, count pixel diff | Lines 15260-15316 (in worker) |

## Critical Discovery: jsQR Already Has RS Decoding

The existing jsQR bundle includes full Reed-Solomon error correction at **lines 10646-10808**. The `decode()` function performs RS correction and **knows the number of errors corrected** - it's the length of `errorLocations` array (line 10787). But this information is **discarded** by the current code.

### Where RS Correction Happens

```
jsQR decode flow (lines 3190-3258, module 5):
  1. Read version and format info
  2. Read codewords from matrix
  3. Split into data blocks
  4. For EACH data block:
     └── reedsolomon_1.decode(codewords, numEcCodewords)
         ├── If no errors: return original bytes
         ├── If correctable: return corrected bytes + (internally knows error count)
         └── If uncorrectable: return null (decode fails)
  5. Combine corrected blocks → final data
```

The RS correction count is available at line 10787-10806 as `errorLocations.length`, but it's never exposed.

## Integration Strategy: Modify jsQR Bundle (RECOMMENDED)

**Why this approach:** Minimal changes, no new dependencies, accurate count, preserves single-file architecture.

### Approach Overview

1. Patch `reedsolomon.decode()` (module 9) to return `{bytes, corrections}` instead of just `bytes`
2. Patch `decodeMatrix()` (module 5) to accumulate corrections across all RS blocks
3. Patch `decode()` (module 5) to include `rsCorrections` in return value
4. Patch `scan()` (module 3) to pass `rsCorrections` to final result
5. Update worker's `evaluateCandidate()` to extract and report this value
6. Update message protocol to include `rsCorrections`

### Alternatives Considered

| Approach | Pros | Cons | Verdict |
|----------|------|------|---------|
| **Patch jsQR (selected)** | Minimal code, accurate, single file | Maintenance on bundled code | RECOMMENDED |
| Standalone RS Counter | Cleaner separation | Duplicates GF(256) math, 2x decode time | NOT RECOMMENDED |
| Fork jsQR | Clean long-term | Overkill complexity | NOT RECOMMENDED |
| ZXing-js | More features | 100KB+ bundle, different output format | NOT RECOMMENDED |

## Detailed Integration Points

### 1. Patch reedsolomon.js (module 9, lines 10754-10808)

**Current code (line 10754-10808):**
```javascript
function decode(bytes, twoS) {
  // ... RS algorithm ...
  if (!error) {
    return outputBytes;  // <-- Just returns bytes
  }
  // ... error correction ...
  return outputBytes;  // <-- Just returns bytes
}
```

**Modified code:**
```javascript
function decode(bytes, twoS) {
  // ... RS algorithm ...
  if (!error) {
    return { bytes: outputBytes, corrections: 0 };  // CHANGED
  }
  // ... error correction ...
  // errorLocations.length is the count we need
  return { 
    bytes: outputBytes, 
    corrections: errorLocations.length  // NEW: expose error count
  };
}
```

### 2. Patch decoder.js decodeMatrix() (module 5, lines 3190-3239)

**Current code (around line 3214-3230):**
```javascript
for (var _i = 0, dataBlocks_3 = dataBlocks; _i < dataBlocks_3.length; _i++) {
  var dataBlock = dataBlocks_3[_i];
  var correctedBytes = reedsolomon_1.decode(
    dataBlock.codewords,
    dataBlock.codewords.length - dataBlock.numDataCodewords
  );
  if (!correctedBytes) {
    return null;
  }
  for (var i = 0; i < dataBlock.numDataCodewords; i++) {
    resultBytes[resultIndex++] = correctedBytes[i];
  }
}
```

**Modified code:**
```javascript
var totalCorrections = 0;  // NEW: Track RS corrections
for (var _i = 0, dataBlocks_3 = dataBlocks; _i < dataBlocks_3.length; _i++) {
  var dataBlock = dataBlocks_3[_i];
  var rsResult = reedsolomon_1.decode(  // CHANGED: rename variable
    dataBlock.codewords,
    dataBlock.codewords.length - dataBlock.numDataCodewords
  );
  if (!rsResult || !rsResult.bytes) {  // CHANGED: check rsResult.bytes
    return null;
  }
  totalCorrections += rsResult.corrections;  // NEW: accumulate
  for (var i = 0; i < dataBlock.numDataCodewords; i++) {
    resultBytes[resultIndex++] = rsResult.bytes[i];  // CHANGED
  }
}
// Then include totalCorrections in the return object
```

### 3. Patch decoder.js decode() return (module 5, around line 3232-3238)

**Current code:**
```javascript
try {
  return decodeData_1.decode(resultBytes, version.versionNumber);
} catch (_a) {
  return null;
}
```

**Modified code:**
```javascript
try {
  var dataResult = decodeData_1.decode(resultBytes, version.versionNumber);
  if (dataResult) {
    dataResult.rsCorrections = totalCorrections;  // NEW: add to result
  }
  return dataResult;
} catch (_a) {
  return null;
}
```

### 4. Patch jsQR main scan() (module 3, lines 2487-2513)

**Current code (around line 2487-2513):**
```javascript
return {
  binaryData: decoded.bytes,
  data: decoded.text,
  chunks: decoded.chunks,
  version: decoded.version,
  location: { /* ... */ }
};
```

**Modified code:**
```javascript
return {
  binaryData: decoded.bytes,
  data: decoded.text,
  chunks: decoded.chunks,
  version: decoded.version,
  rsCorrections: decoded.rsCorrections || 0,  // NEW: pass through
  location: { /* ... */ }
};
```

### 5. Update Worker evaluateCandidate() (lines 15260-15316)

**Current code (around line 15280-15285):**
```javascript
const decodeResult = jsQR(imageData.data, size, size);
const decodable = decodeResult !== null && decodeResult.data === expectedUrl;

if (decodable) {
  decodableCount++;
}
```

**Modified code:**
```javascript
const decodeResult = jsQR(imageData.data, size, size);
const decodable = decodeResult !== null && decodeResult.data === expectedUrl;
const rsCorrections = decodeResult ? (decodeResult.rsCorrections || 0) : -1;  // NEW

if (decodable) {
  decodableCount++;
}
```

**And update return object (around line 15316):**
```javascript
return { 
  hash, 
  pixelDiff, 
  decodable, 
  rsCorrections,  // NEW
  moduleData, 
  qrModuleData, 
  moduleCount 
};
```

### 6. Update Worker Message Protocol

**Current PROGRESS/COMPLETE message (lines 15340-15353, 15416-15430):**
```javascript
self.postMessage({
  type: 'PROGRESS',
  topResults: topResults.map(r => ({
    hash: r.hash,
    pixelDiff: r.pixelDiff,
    decodable: r.decodable,
    moduleData: Array.from(r.moduleData),
    qrModuleData: Array.from(r.qrModuleData),
    moduleCount: r.moduleCount
  })),
  // ...
});
```

**Updated message:**
```javascript
self.postMessage({
  type: 'PROGRESS',
  topResults: topResults.map(r => ({
    hash: r.hash,
    pixelDiff: r.pixelDiff,
    decodable: r.decodable,
    rsCorrections: r.rsCorrections,  // NEW
    moduleData: Array.from(r.moduleData),
    qrModuleData: Array.from(r.qrModuleData),
    moduleCount: r.moduleCount
  })),
  // ...
});
```

### 7. Update Results Sorting (lines 15319-15334)

**Current sorting:**
```javascript
topResults.sort((a, b) => {
  if (a.decodable !== b.decodable) {
    return b.decodable ? 1 : -1;
  }
  return a.pixelDiff - b.pixelDiff;
});
```

**Updated sorting:**
```javascript
topResults.sort((a, b) => {
  // Decodable first
  if (a.decodable !== b.decodable) {
    return b.decodable ? 1 : -1;
  }
  // Among decodable: fewer RS corrections better (more robust)
  if (a.decodable && b.decodable) {
    if (a.rsCorrections !== b.rsCorrections) {
      return a.rsCorrections - b.rsCorrections;
    }
  }
  // Then by pixel diff
  return a.pixelDiff - b.pixelDiff;
});
```

## Build Order (Dependencies)

```
Phase 1: Patch RS decoder to expose error count
  ├── Modify reedsolomon decode() to return {bytes, corrections}
  ├── Lines: 10754-10808 in index.html
  └── Test: decode still works, corrections are accurate

Phase 2: Patch jsQR decoder to propagate count
  ├── Modify decodeMatrix() to accumulate corrections
  ├── Modify decode() to include rsCorrections in result
  ├── Modify scan() to pass rsCorrections through
  ├── Lines: 3190-3258 and 2487-2513
  └── Test: jsQR(imageData) now returns rsCorrections field

Phase 3: Update worker code
  ├── Modify evaluateCandidate() to extract rsCorrections
  ├── Update result object to include rsCorrections
  ├── Update sendProgress()/sendComplete() message format
  ├── Lines: 15260-15430 (in worker code string)
  └── Test: worker messages include rsCorrections

Phase 4: Update main thread
  ├── Modify createWorkerMessageHandler() to handle rsCorrections
  ├── Update mergeWorkerResults() to preserve rsCorrections
  ├── Update sorting logic to use rsCorrections
  ├── Update UI to display rsCorrections
  └── Test: full pipeline works, UI shows RS correction counts
```

## File Locations for Changes (All in index.html)

| Change | Approximate Lines | Section |
|--------|-------------------|---------|
| Patch reedsolomon decode() | 10754-10808 | jsQR bundle, module 9 |
| Patch decodeMatrix() | 3190-3239 | jsQR bundle, module 5 |
| Patch decode() wrapper | 3240-3258 | jsQR bundle, module 5 |
| Patch scan() return | 2487-2513 | jsQR bundle, module 3 |
| Worker evaluateCandidate() | 15260-15316 | Worker code string |
| Worker result objects | 15316, 15348, 15422 | Worker code string |
| Worker sorting logic | 15319-15334 | Worker code string |
| createWorkerMessageHandler | 15458-15498 | Main thread |
| UI updates | TBD | Main thread display code |

## Updated Message Protocol

**Main → Worker (unchanged):**
```javascript
{
  type: 'START_OPTIMIZATION',
  url: string,
  version: number,
  ecLevel: number,
  paintedPattern: number[],
  functionMask: boolean[],
  hashLength: number,
  timeout: number,
  topResultsCount: number,
  progressInterval: number,
  existingTopResults: []
}
```

**Worker → Main (updated):**
```javascript
{
  type: 'PROGRESS' | 'COMPLETE',
  attempts: number,
  elapsed: number,
  topResults: [{
    hash: string,
    pixelDiff: number,
    decodable: boolean,
    rsCorrections: number,  // NEW: -1 if not decodable, 0+ if decodable
    moduleData: number[],
    qrModuleData: number[],
    moduleCount: number
  }],
  decodableCount: number
}
```

## RS Correction Count Semantics

| Value | Meaning |
|-------|---------|
| `rsCorrections = -1` | QR failed to decode (too many errors or malformed) |
| `rsCorrections = 0` | QR decoded perfectly, no errors in any data block |
| `rsCorrections = N > 0` | QR had N codeword errors across all blocks that were corrected |

**User benefit:** Users can see which hash variants produce more **robust** QRs (lower correction count = more margin for additional damage).

For EC Level H (highest), each RS block can correct up to `ecCodewordsPerBlock / 2` errors. More corrections used = QR is closer to failure threshold.

## Anti-Patterns to Avoid

### Anti-Pattern 1: Separate RS Decoding

**What:** Add standalone RS decoder alongside jsQR
**Why wrong:** Duplicates GF(256) math, may count differently, doubles decode time
**Instead:** Patch existing jsQR to expose its internal count

### Anti-Pattern 2: Post-hoc RS Analysis

**What:** After jsQR succeeds, re-read codewords and count syndromes
**Why wrong:** jsQR already did this work, requires extracting BitMatrix etc.
**Instead:** Modify jsQR to return what it already computed

### Anti-Pattern 3: Breaking Single-File Architecture

**What:** Split into separate .js files for cleaner patching
**Why wrong:** Breaks constraint, complicates deployment
**Instead:** Patch inline, use clear comments marking modifications

## Validation Checklist

- [ ] RS count is accurate for known-error QR codes
- [ ] rsCorrections = -1 when decode fails
- [ ] rsCorrections = 0 when QR is perfect
- [ ] Multiple RS blocks correctly sum their corrections
- [ ] No measurable decode performance regression
- [ ] Sorting correctly prioritizes lower rsCorrections
- [ ] UI displays rsCorrections for each result

## Sources

- Direct analysis of `/workspace/index.html` (existing codebase)
- jsQR library structure inlined at lines 2080-14375
- Reed-Solomon implementation at lines 10646-10808
- Worker architecture at lines 14380-15694
- QR code RS specification (ISO/IEC 18004:2015)

---
*Architecture research for: RS error correction count integration*
*Researched: 2026-02-16*
