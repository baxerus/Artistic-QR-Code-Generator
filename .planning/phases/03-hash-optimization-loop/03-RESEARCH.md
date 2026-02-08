# Phase 3: Hash Optimization Loop - Research

**Researched:** 2026-02-08
**Domain:** Web Workers, OffscreenCanvas, QR code optimization, hash search strategies
**Confidence:** MEDIUM

## Summary

Phase 3 implements an automated hash fragment search engine that runs in a Web Worker to avoid freezing the UI. The worker generates QR codes with different URL hash fragments, overlays the user's painted pattern, decodes the result, and measures error counts to find optimal variants. The research focused on Web Worker architecture, OffscreenCanvas capabilities, jsQR's Reed-Solomon error detection, and hash generation strategies.

**Key findings:**
- OffscreenCanvas with Web Workers is well-supported across all modern browsers since March 2023
- jsQR's Reed-Solomon decoder calculates error counts internally but doesn't expose them in the public API
- qrcodejs library requires DOM manipulation and cannot run directly in Web Workers
- Pure random search (Monte Carlo) is simpler and more reliable than hill climbing for this use case
- Transferable objects provide zero-copy transfer for ImageData between main thread and worker

**Primary recommendation:** Use Web Worker with manual QR generation (extract qrcodejs encoding logic), pure random hash search, and pixel difference counting as the error metric (Reed-Solomon error counts require jsQR source modification).

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Search configuration:**
- Only configurable setting: maximum search time in seconds
- Default timeout: 30 seconds
- Controls (timeout input + Generate button) placed below the canvas, continuing the vertical flow: URL > QR > paint > configure > generate
- Generate button disabled until at least one pixel is painted; show hint like "Paint pixels first"
- All other search parameters (hash length, character set, strategy) are automatic — Claude's discretion

**Live feedback layout:**
- Top 5 candidates displayed as horizontal row of small QR code previews, ranked left to right (best first)
- Stats shown during search: attempts counter + elapsed time (no rate display)
- Each QR preview shows error count only beneath it (hash fragment details deferred to Phase 4 results view)
- Subtle highlight (brief glow or color flash) when a new best result enters the top 5

**Error measurement:**
- Primary goal: measure actual Reed-Solomon error correction burden, not just pixel mismatch
  - The ideal result is a hash fragment where QR data naturally agrees with the painted pattern (pattern pixels are correct data, not errors)
  - Research whether RS correction counts can be extracted from jsQR or another browser decoder
- Fallback if RS counts aren't feasible: pixel diff count among decodable results
- Ranking: primary sort by decodable (yes/no), secondary sort by error count (lower is better)
- All results shown in top 5 regardless of decode status, but clearly marked
- Label format beneath each preview: decode badge + error count (e.g., "decodable - 3 errors" or "not decodable - 28 errors")

**Search completion:**
- On completion (timeout or manual stop): show summary line ("Done - 23,456 attempts, 4 decodable found") above frozen top 5 row
- Results stay in place — no automatic transition to a different view
- Re-running keeps existing top 5 and extends search (accumulates best results across runs)
- Single transforming button: "Generate" > "Stop" (during search) > "Run again" (after completion)
- Progress bar showing time remaining + elapsed time counter (e.g., visual bar with "18s / 30s")

### Claude's Discretion

- Hash fragment generation strategy (character set, length, randomization approach)
- Web Worker architecture and communication protocol
- QR generation + decode pipeline optimization within the worker
- Exact visual styling of progress bar, highlight animation, and QR preview sizing
- How to handle edge case where 0 results found after timeout

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope

</user_constraints>

## Standard Stack

The established technologies for this domain:

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Web Workers | Native API | Background processing | Standard browser API for offloading CPU-intensive work from main thread |
| OffscreenCanvas | Native API | Canvas rendering in workers | Available across browsers since March 2023, enables zero-copy canvas operations |
| qrcodejs (extract logic) | Current (in use) | QR encoding algorithm | Already in use for Phase 1-2, extract `_oQRCode` model generation logic |
| jsQR | Current (inlined) | QR decoding | Already inlined in Phase 2, pure JavaScript with no dependencies |
| Web Crypto API | Native API | Random hash generation | Hardware-backed randomness, available natively in all modern browsers |

### Supporting

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| Structured Clone Algorithm | Native | Data transfer | Default for postMessage(), suitable for small objects (<100KB) |
| Transferable Objects | Native | Zero-copy transfer | Large binary data (ImageData, ArrayBuffer) to avoid copying overhead |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Pure random search | Hill climbing / genetic algorithms | Hill climbing gets trapped in local optima; random search is simpler and more reliable for this problem |
| Pixel difference count | Reed-Solomon error count | RS error counts require jsQR source modification (not exposed in API); pixel diff is sufficient fallback |
| Extract qrcodejs logic | Port different QR library | qrcodejs already in use and model extraction pattern proven in Phase 2 |

**Installation:**

No installation needed — all technologies are native browser APIs or already included in the single-file HTML.

## Architecture Patterns

### Recommended Project Structure

Since this is a single-file HTML application, organization is done through script sections and module-level functions:

```
<script>
  // ===== WEB WORKER DEFINITION =====
  const workerCode = `
    // Worker-level state
    // QR generation logic (extracted from qrcodejs)
    // Hash generation functions
    // QR + overlay + decode pipeline
    // Message handlers
  `;

  // ===== MAIN THREAD: WORKER MANAGEMENT =====
  let optimizationWorker = null;
  let workerState = { running: false, startTime: null };

  // ===== MAIN THREAD: UI UPDATE HANDLERS =====
  // Progress updates
  // Top 5 display
  // Button state management

  // ===== MAIN THREAD: USER INTERACTION =====
  // Generate button click
  // Stop button click
  // Timeout configuration
</script>
```

### Pattern 1: Inline Worker Creation

**What:** Create Web Worker from inline JavaScript string using Blob URL
**When to use:** Single-file HTML applications where external worker files would break portability
**Example:**

```javascript
// Source: MDN Web Workers + single-file pattern
const workerCode = `
  // All worker logic here
  self.onmessage = function(e) {
    // Handle messages
  };
`;

const blob = new Blob([workerCode], { type: 'application/javascript' });
const workerUrl = URL.createObjectURL(blob);
const worker = new Worker(workerUrl);
```

**Why:** Maintains single-file architecture while enabling background processing.

### Pattern 2: Worker Communication Protocol

**What:** Structured message passing with type-based routing
**When to use:** Workers with multiple operation types
**Example:**

```javascript
// Source: MDN Web Workers structured messaging
// Main thread sends:
worker.postMessage({
  type: 'START_OPTIMIZATION',
  baseURL: 'https://example.com',
  paintedPixels: [[0,0,'black'], [1,1,'white']],
  functionMask: maskData,
  timeout: 30000
});

// Worker responds:
self.postMessage({
  type: 'PROGRESS_UPDATE',
  attempts: 1234,
  elapsed: 5000,
  topResults: [/* top 5 candidates */]
});
```

### Pattern 3: Transferable ImageData

**What:** Use transferable objects for ImageData to avoid copy overhead
**When to use:** Passing canvas image data between main thread and worker
**Example:**

```javascript
// Source: MDN Transferable Objects
// Create ImageData in worker
const imageData = new ImageData(width, height);
// ... populate imageData

// Transfer ownership to main thread (zero-copy)
self.postMessage({
  type: 'RESULT_CANDIDATE',
  imageData: imageData
}, [imageData.data.buffer]);
// imageData.data.buffer is now unusable in worker
```

**Performance:** Transferring a 32MB ArrayBuffer takes 6.6ms with transferable objects vs 302ms with structured cloning.

### Pattern 4: Pure Random Hash Search

**What:** Generate random hash fragments and evaluate independently
**When to use:** Search space exploration where local optima are a concern
**Example:**

```javascript
// Source: Research on Monte Carlo vs Hill Climbing
function generateRandomHash(length, charset) {
  const randomBytes = new Uint8Array(length);
  crypto.getRandomValues(randomBytes);

  let hash = '';
  for (let i = 0; i < length; i++) {
    hash += charset[randomBytes[i] % charset.length];
  }
  return hash;
}

// Evaluation loop
while (Date.now() - startTime < timeout) {
  const hash = generateRandomHash(8, URL_SAFE_CHARS);
  const score = evaluateCandidate(baseURL + '#' + hash);
  updateTopResults(hash, score);
}
```

**Why:** Simpler than hill climbing, avoids local optima, and performs well for independent evaluations.

### Pattern 5: QR Model Extraction and Manual Rendering

**What:** Extract QR code model from qrcodejs and manually render to canvas
**When to use:** Need QR generation without DOM manipulation (for Web Worker)
**Example:**

```javascript
// Source: Current implementation in index.html (lines 11011-11022)
// Main thread: Extract model
const tempDiv = document.createElement('div');
const qrInstance = new QRCode(tempDiv, {
  text: url,
  width: 400,
  height: 400,
  typeNumber: version,
  correctLevel: QRCode.CorrectLevel.H
});
const qrModel = qrInstance._oQRCode;

// Worker: Manually render model to ImageData
function renderQRModelToImageData(moduleCount, isDarkFunc) {
  const cellSize = 1; // 1 pixel per module for compact representation
  const size = moduleCount * cellSize;
  const imageData = new ImageData(size, size);

  for (let row = 0; row < moduleCount; row++) {
    for (let col = 0; col < moduleCount; col++) {
      const isDark = isDarkFunc(row, col);
      const idx = (row * size + col) * 4;
      const color = isDark ? 0 : 255;
      imageData.data[idx] = color;     // R
      imageData.data[idx+1] = color;   // G
      imageData.data[idx+2] = color;   // B
      imageData.data[idx+3] = 255;     // A
    }
  }

  return imageData;
}
```

**Critical:** qrcodejs's QR generation logic must be extracted and ported to worker code. The `_oQRCode` object contains the module data structure.

### Anti-Patterns to Avoid

- **Running QR generation in main thread:** Freezes UI for 2-3 seconds during optimization
- **Copying ImageData with structured clone:** 50x slower than transferable objects for large data
- **Using hill climbing search:** Gets trapped in local optima; QR code bit flips create discontinuous search space
- **Attempting to modify jsQR source:** Reed-Solomon error counts aren't exposed; pixel diff is sufficient fallback
- **Creating new worker per run:** Worker initialization overhead; reuse single worker instance

## Don't Hand-Roll

Problems that look simple but have existing solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Random number generation | `Math.random()` with character picking | `crypto.getRandomValues()` with modulo | Hardware-backed cryptographic randomness, uniform distribution |
| QR code encoding | Parse QR spec and implement Reed-Solomon | Extract qrcodejs `_oQRCode` logic | QR encoding is complex (version detection, error correction, masking); proven library exists |
| Worker-main communication | Custom string parsing protocol | Structured Clone Algorithm with typed messages | Native, automatic serialization of complex objects |
| Canvas in worker | Polyfill or emulation | OffscreenCanvas native API | Well-supported since 2023, zero-copy operations |
| Search algorithm tuning | Complex genetic algorithm or simulated annealing | Pure random search (Monte Carlo) | QR space is discontinuous; random search avoids local optima and is simpler |

**Key insight:** QR code optimization is fundamentally different from continuous optimization problems. Flipping a single bit in the hash can dramatically change QR module positions due to encoding and error correction. This makes gradient-based or local search methods ineffective. Pure random sampling with tracking of best results is the most reliable approach.

## Common Pitfalls

### Pitfall 1: Worker Cannot Access qrcodejs Directly

**What goes wrong:** Attempting to use `new QRCode()` inside Web Worker fails with "document is not defined"

**Why it happens:** qrcodejs is DOM-dependent; it creates DOM elements to render QR codes. Workers have no DOM access.

**How to avoid:** Extract the QR encoding logic from qrcodejs into worker code:
1. Copy the QR encoding algorithm (module placement, masking, error correction)
2. Port the model generation to pure JavaScript (no DOM)
3. Test that generated module matrix matches main thread QRCode instance `_oQRCode`

**Warning signs:** `ReferenceError: document is not defined` in worker console

### Pitfall 2: jsQR Reed-Solomon Error Counts Not Accessible

**What goes wrong:** Attempting to extract `errorsCorrected` or similar property from jsQR decode result returns undefined

**Why it happens:** jsQR's Reed-Solomon decoder (lines 8661-8697) calculates `errorLocations.length` internally but doesn't expose it in the returned decode result object. The public API only returns `{data, binaryData, chunks, version, location}`.

**How to avoid:** Use pixel difference count as error metric instead:
1. Generate QR code with hash fragment
2. Overlay painted pattern pixels
3. Decode with jsQR to get decodable status
4. Count how many pixels differ between overlay and original QR
5. Rank by: primary = decodable (yes/no), secondary = pixel diff count

**Warning signs:** Decode result has no `errors`, `errorCount`, or similar property

### Pitfall 3: Structured Clone Performance with Large ImageData

**What goes wrong:** Optimization runs slowly, taking 200-300ms per iteration with visible UI lag

**Why it happens:** ImageData for QR codes (400x400 = 640KB) is large. Structured cloning copies the entire buffer each time, which takes hundreds of milliseconds.

**How to avoid:** Use transferable objects for ImageData transfer:
```javascript
// Worker sends result
self.postMessage({
  type: 'CANDIDATE',
  imageData: imageData
}, [imageData.data.buffer]); // Transfer ownership
```

**Performance impact:** 6.6ms vs 302ms for 32MB buffer; QR ImageData is smaller but still significant savings

**Warning signs:** High CPU usage on main thread, janky UI during optimization, DevTools profiler shows time in `postMessage`

### Pitfall 4: Worker State Accumulation

**What goes wrong:** After multiple optimization runs, worker uses excessive memory or slows down

**Why it happens:** Worker accumulates state (candidates array, ImageData buffers) without cleanup between runs

**How to avoid:**
- Clear candidate arrays at start of new run
- Don't keep references to transferred ImageData buffers
- Consider terminating and recreating worker if memory grows excessively
- Use Chrome DevTools Task Manager to monitor worker memory

**Warning signs:** Browser tab memory usage grows with each run, eventual slowdown or crash

### Pitfall 5: Re-running Overwrites Previous Results

**What goes wrong:** User expects results to accumulate across runs, but starting new search clears previous top 5

**Why it happens:** Worker resets top results array at start of optimization

**How to avoid:**
- Send previous top results to worker when starting new run
- Worker merges new candidates with existing top 5
- Main thread maintains authoritative top 5 state
- Worker only proposes updates, main thread decides whether to replace

**Warning signs:** User confusion, lost good results from previous runs

## Code Examples

Verified patterns from research and existing implementation:

### Worker Creation (Single-File Pattern)

```javascript
// Source: MDN + single-file architecture requirement
const workerCode = `
  // Import jsQR if needed (inline entire library or use importScripts with data URL)

  self.onmessage = function(e) {
    const { type, ...data } = e.data;

    switch(type) {
      case 'START_OPTIMIZATION':
        startOptimization(data);
        break;
      case 'STOP_OPTIMIZATION':
        stopOptimization();
        break;
    }
  };

  function startOptimization({ baseURL, paintedPixels, functionMask, timeout }) {
    const startTime = Date.now();
    let attempts = 0;

    while (Date.now() - startTime < timeout) {
      // Generate, test, report
      attempts++;

      if (attempts % 100 === 0) {
        self.postMessage({
          type: 'PROGRESS',
          attempts,
          elapsed: Date.now() - startTime
        });
      }
    }

    self.postMessage({ type: 'COMPLETE', attempts });
  }
`;

const blob = new Blob([workerCode], { type: 'application/javascript' });
const workerUrl = URL.createObjectURL(blob);
const worker = new Worker(workerUrl);

worker.onmessage = function(e) {
  const { type, ...data } = e.data;
  // Handle worker responses
};
```

### Random Hash Generation (URL-Safe)

```javascript
// Source: Web Crypto API best practices + URL-safe character research
const URL_SAFE_CHARS = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_';

function generateRandomHash(length = 8) {
  const randomBytes = new Uint8Array(length);
  crypto.getRandomValues(randomBytes);

  let hash = '';
  for (let i = 0; i < length; i++) {
    // Use modulo to map random byte to charset (acceptable for non-cryptographic use)
    hash += URL_SAFE_CHARS[randomBytes[i] % URL_SAFE_CHARS.length];
  }

  return hash;
}

// Example: generateRandomHash(8) -> "aB3-xY9_"
```

**Note:** For pure cryptographic applications, modulo introduces slight bias. For URL hash generation, this is acceptable.

### Pixel Difference Calculation

```javascript
// Source: Existing implementation pattern + ImageData comparison
function calculatePixelDifference(originalImageData, overlayImageData) {
  if (originalImageData.width !== overlayImageData.width ||
      originalImageData.height !== overlayImageData.height) {
    throw new Error('ImageData dimensions must match');
  }

  let diffCount = 0;
  const length = originalImageData.data.length;

  // Compare R channel only (grayscale QR: R === G === B)
  for (let i = 0; i < length; i += 4) {
    const orig = originalImageData.data[i];
    const overlay = overlayImageData.data[i];

    // Threshold for black/white difference
    if (Math.abs(orig - overlay) > 128) {
      diffCount++;
    }
  }

  return diffCount;
}
```

### Top Results Management (Main Thread)

```javascript
// Source: User requirements for accumulating results across runs
class TopResultsManager {
  constructor(maxResults = 5) {
    this.maxResults = maxResults;
    this.results = [];
  }

  addCandidate(hash, imageData, decodable, errorCount) {
    const candidate = { hash, imageData, decodable, errorCount };

    // Insert into sorted position
    this.results.push(candidate);
    this.results.sort((a, b) => {
      // Primary: decodable status (decodable first)
      if (a.decodable !== b.decodable) {
        return b.decodable ? 1 : -1;
      }
      // Secondary: error count (lower first)
      return a.errorCount - b.errorCount;
    });

    // Keep only top N
    if (this.results.length > this.maxResults) {
      this.results = this.results.slice(0, this.maxResults);
    }
  }

  shouldAddCandidate(decodable, errorCount) {
    if (this.results.length < this.maxResults) return true;

    const worst = this.results[this.results.length - 1];

    // Better if decodable when worst is not
    if (decodable && !worst.decodable) return true;

    // Better if both decodable and lower error count
    if (decodable === worst.decodable && errorCount < worst.errorCount) {
      return true;
    }

    return false;
  }

  clear() {
    this.results = [];
  }

  // Keep results across runs (don't clear)
  getResults() {
    return this.results;
  }
}
```

### Progress Bar Update

```javascript
// Source: User requirements for time remaining + elapsed counter
function updateProgressBar(elapsed, timeout) {
  const progressBar = document.getElementById('progress-bar');
  const progressText = document.getElementById('progress-text');

  const percentComplete = Math.min(100, (elapsed / timeout) * 100);
  progressBar.style.width = percentComplete + '%';

  const elapsedSeconds = Math.floor(elapsed / 1000);
  const totalSeconds = Math.floor(timeout / 1000);
  progressText.textContent = `${elapsedSeconds}s / ${totalSeconds}s`;
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Math.random() for hash generation | crypto.getRandomValues() | 2015+ (Web Crypto API) | Hardware-backed randomness, uniform distribution |
| Canvas operations on main thread | OffscreenCanvas in Web Worker | March 2023 (wide browser support) | UI remains responsive during heavy rendering |
| MessageChannel string passing | Structured Clone Algorithm | Always available but underutilized | Automatic serialization, easier to use |
| Copying ArrayBuffer | Transferable objects | 2012+ but adoption slow | 50x faster for large binary data |

**Deprecated/outdated:**
- Polyfills for OffscreenCanvas: No longer needed, native support is universal
- Complex search algorithms for QR optimization: Random search outperforms due to discontinuous search space

## Open Questions

Things that couldn't be fully resolved:

1. **Actual iterations/second achievable**
   - What we know: QR generation + overlay + decode pipeline must complete per iteration
   - What's unclear: Without benchmarking, don't know if 30 seconds yields 100 or 10,000 attempts
   - Recommendation: Implement basic pipeline first, benchmark on target hardware, adjust default timeout if needed

2. **Reed-Solomon error count extraction**
   - What we know: jsQR calculates `errorLocations.length` internally but doesn't expose it
   - What's unclear: Whether modifying jsQR source is worth the maintenance burden
   - Recommendation: Start with pixel difference metric; revisit RS counts in Phase 4 if users demand more accurate error measurement

3. **Hash length optimal range**
   - What we know: Longer hashes = more search space, shorter = fewer collisions with existing content
   - What's unclear: What length provides best balance of search performance vs URL aesthetics
   - Recommendation: Start with 8 characters (64^8 = 281 trillion combinations), provide way to adjust in future if needed

4. **QRCodeJS extraction complexity**
   - What we know: `_oQRCode` object exists and has `isDark(row, col)` method
   - What's unclear: Whether full QR generation can be ported to worker or if model must be generated in main thread
   - Recommendation: First attempt to extract encoding logic; if too complex, generate models in main thread and transfer to worker (slower but simpler)

5. **Memory usage at scale**
   - What we know: Keeping ImageData for top 5 results in memory
   - What's unclear: Whether multiple runs cause memory bloat
   - Recommendation: Monitor memory in DevTools during testing; implement cleanup if needed

## Sources

### Primary (HIGH confidence)

- [OffscreenCanvas - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/OffscreenCanvas) - API capabilities and browser support
- [Using Web Workers - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers) - Communication patterns and best practices
- [Worker: postMessage() method - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Worker/postMessage) - Structured clone and transferable objects
- [The structured clone algorithm - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm) - Data transfer mechanics
- Current implementation in `/workspace/index.html` (lines 10561-11155) - QR model extraction pattern

### Secondary (MEDIUM confidence)

- [Transferable objects - Lightning fast | Chrome Developers](https://developer.chrome.com/blog/transferable-objects-lightning-fast) - Performance benchmarks (32MB: 6.6ms vs 302ms)
- [Is postMessage slow? - surma.dev](https://surma.dev/things/is-postmessage-slow/) - Message size budgets (10KiB safe, 100KiB acceptable)
- [OffscreenCanvas—speed up your canvas operations with a web worker | web.dev](https://web.dev/articles/offscreen-canvas) - OffscreenCanvas usage patterns
- [Building 60 FPS QR Scanner for the Mobile Web | Tokopedia Engineering](https://medium.com/tokopedia-engineering/building-60-fps-qr-scanner-for-the-mobile-web-eb0deddce099) - Web Worker QR performance insights (6ms per decode)

### Tertiary (LOW confidence)

- [Comparative Analysis of Random Search Algorithms - OMSCS](https://sites.gatech.edu/omscs7641/2024/02/19/comparative-analysis-of-random-search-algorithms/) - Random restart dominates Monte Carlo
- [QRCode Canvas - JSFiddle](https://jsfiddle.net/paulftw/R3Q4Q/) - Community QR generation patterns
- [JavaScript Random String Generator - CopyProgramming](https://copyprogramming.com/howto/javascript-program-to-generate-random-string) - Web Crypto API usage patterns
- WebSearch results about genetic algorithms and simulated annealing - Theoretical context for search strategy decision

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - Web Workers, OffscreenCanvas, and Web Crypto API are well-documented native APIs
- Architecture: MEDIUM - Worker patterns verified via MDN, but QRCodeJS extraction not yet proven in worker context
- Pitfalls: MEDIUM - jsQR internals verified via source code, but worker-specific issues based on research not direct testing
- Performance: LOW - No direct benchmarks of QR generation+decode pipeline; estimates based on related work

**Research date:** 2026-02-08
**Valid until:** 2026-03-08 (30 days - stable native APIs, but QR library landscape could change)
