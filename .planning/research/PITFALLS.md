# Pitfalls Research

**Domain:** QR Code Generation and Optimization
**Researched:** 2026-02-06
**Confidence:** MEDIUM

## Critical Pitfalls

### Pitfall 1: Misunderstanding Error Correction as Data Validation

**What goes wrong:**
Developers assume QR error correction will detect encoding errors or invalid data. Error correction only repairs physical damage (corrupted modules) but cannot fix logical errors. If a URL is entered incorrectly or the wrong information is encoded, the QR code will decode successfully but point to the wrong destination.

**Why it happens:**
The term "error correction" is misleading—it refers to Reed-Solomon codes that restore damaged modules, not semantic validation. Developers confuse physical corruption with data validation.

**How to avoid:**
- Implement separate validation layer for input data before encoding
- Verify decoded output matches expected format (URL structure, hash format, etc.)
- For optimization projects: validate that "decoder errors" specifically measure module-level corruption, not data correctness

**Warning signs:**
- Tests only verify that QR codes decode, not that they decode to correct values
- No input validation before encoding
- Confusion between "the code scans" and "the code contains correct data"

**Phase to address:**
Phase 1 (Core Generation) - Establish clear distinction between encoding validation and error correction capabilities

---

### Pitfall 2: Canvas Pixel-to-Module Mapping Misalignment

**What goes wrong:**
When locking/corrupting specific pixels for optimization, developers miscalculate the mapping between canvas coordinates and QR code modules. This causes:
- Wrong modules being corrupted (hitting finder patterns instead of data regions)
- Alignment patterns being accidentally destroyed
- Off-by-one errors that corrupt reserved format information areas

**Why it happens:**
QR codes have complex structure with function patterns (finder, timing, alignment) that must never be modified. The quiet zone, module size scaling, and coordinate system transformations create multiple opportunities for mapping errors.

**How to avoid:**
- Create explicit mapping function: `canvasCoords → moduleCoords` with unit tests
- Visualize locked regions on canvas with distinct colors before corruption
- Verify at least 4 pixels per module for reliable rendering (avoid sub-pixel issues)
- Never modify: finder patterns, timing patterns, separators, alignment patterns, format/version information areas
- Account for quiet zone (4 modules minimum) in coordinate calculations

**Warning signs:**
- QR codes become completely unscannable after single pixel corruption
- Different canvas sizes produce different corruption patterns for same module
- Locked pixels visually appear to overlap finder patterns

**Phase to address:**
Phase 2 (Corruption/Measurement) - Build and verify pixel-to-module mapping before implementing corruption

---

### Pitfall 3: Incorrect Mask Pattern Application During Optimization

**What goes wrong:**
When manually constructing or modifying QR codes, mask patterns are applied to function patterns (finder, timing, alignment) or skipped for data modules. This makes QR codes completely unscannable because decoders expect specific patterns to remain unmasked.

**Why it happens:**
The QR specification requires evaluating all 8 mask patterns and selecting the lowest penalty score. Developers either:
- Apply masks to all modules indiscriminately
- Skip mask evaluation and use pattern 0 (or random pattern)
- Miscalculate penalty scores leading to suboptimal mask selection

**How to avoid:**
- Masks ONLY apply to data and error correction modules
- Never mask: function patterns (finder, timing, separator, alignment) or reserved areas (format/version info)
- Implement all four penalty rules correctly:
  1. Consecutive modules (5+ same color in row/column)
  2. Block patterns (2×2 blocks of same color)
  3. Finder pattern similarity (avoid patterns resembling 1:1:3:1:1 ratio)
  4. Color balance (penalty if >50% dark or light)
- For optimization: if modifying existing QR codes, verify mask pattern remains valid after corruption

**Warning signs:**
- QR codes scan in some readers but not others
- Decoder libraries report "invalid format information"
- Visual inspection shows patterns in data region that look too regular

**Phase to address:**
Phase 1 (Core Generation) - If generating QR codes from scratch
Phase 2 (Corruption) - If modifying existing codes, verify mask validity after changes

---

### Pitfall 4: Measuring "Decoder Errors" Without Clear Definition

**What goes wrong:**
The concept of "decoder errors" in corrupted QR codes is ambiguous. Projects measure wrong things:
- Counting "failed scans" (binary: works/doesn't work) instead of error correction usage
- Measuring decoding time instead of module errors corrected
- Missing distinction between detectable/correctable errors vs. undetectable failures

**Why it happens:**
QR decoder libraries don't expose internal error correction metrics consistently. Developers must infer corruption level from external behavior, leading to inconsistent measurements across decoders.

**How to avoid:**
- Define exactly what "error" means for your project:
  - Number of corrupted modules? (direct measurement)
  - Reed-Solomon error correction codewords used? (requires decoder internals)
  - Scanning success rate across N attempts? (statistical approach)
  - Decoding confidence score? (decoder-dependent)
- Document which QR decoder library is canonical for error measurement
- Verify measurement methodology: does corruption of 1 module = 1 "error"?
- Account for error correction limits: L=7%, M=15%, Q=25%, H=30% of codewords, not modules

**Warning signs:**
- "Error measurement" code only checks if decode succeeds (boolean)
- No correlation between number of locked pixels and measured error count
- Different decoders report vastly different error counts for same QR code
- Measurement fails to distinguish between "corrected successfully" and "too damaged to decode"

**Phase to address:**
Phase 2 (Corruption/Measurement) - Define measurement methodology before optimization begins

---

### Pitfall 5: Brute-Force Hash Search Without Search Space Reduction

**What goes wrong:**
Naive brute-force search of hash fragments grows exponentially. For a 6-character hex hash fragment (24 bits), there are 16.7 million combinations. Without strategic search space reduction, optimization becomes computationally infeasible even for modest hash lengths.

**Why it happens:**
Developers underestimate how fast search space explodes:
- 4 hex chars: 65K combinations
- 6 hex chars: 16.7M combinations
- 8 hex chars: 4.3B combinations

In single-threaded JavaScript without optimization, this means hours to days of computation.

**How to avoid:**
- Start with shorter hash fragments (4 chars max) to validate methodology
- Implement progressive search: optimize 1 char at a time, accumulating results
- Use pattern-based pruning: eliminate hashes that don't meet QR code structure constraints
- Consider heuristic search instead of exhaustive brute-force
- For browser: use Web Workers for parallel search, monitor performance impact
- Set realistic expectations: 8+ character optimization may be impractical without GPU acceleration

**Warning signs:**
- Initial time estimates don't account for exponential growth
- No early termination criteria (searches forever hoping to find perfect hash)
- UI freezes during optimization (running synchronously on main thread)
- Memory usage grows unbounded as search space expands

**Phase to address:**
Phase 3 (Optimization) - Design search strategy before implementation, with explicit complexity analysis

---

### Pitfall 6: Canvas Operations Causing UI Freezing in Browser

**What goes wrong:**
Long-running QR code optimization with repeated canvas `getImageData()`/`putImageData()` calls blocks the browser main thread, causing UI freezing, unresponsive tabs, and browser warnings about "slow script."

**Why it happens:**
Canvas pixel operations involve GPU-to-CPU transfer overhead (several ms per call). When optimization requires thousands of iterations:
- Each `getImageData()` copies entire canvas from GPU to system RAM
- Setting `willReadFrequently: true` disables GPU acceleration entirely (slower writes)
- Not setting it means slow reads (default assumption: infrequent access)

**How to avoid:**
- Use Web Workers with OffscreenCanvas API for optimization computation
- Transfer canvas control to worker via `transferControlToOffscreen()`
- Batch canvas operations: minimize `getImageData()/putImageData()` frequency
- Set `willReadFrequently: true` only if reads dominate over writes
- Implement progress reporting: update UI periodically (every 100ms, not every iteration)
- Add "cancel" mechanism for long-running searches

**Warning signs:**
- Browser shows "page unresponsive" warnings during optimization
- No user interaction possible while optimization runs
- DevTools performance profiling shows main thread constantly busy
- Memory usage steadily increases during optimization

**Phase to address:**
Phase 3 (Optimization) - Architecture decision about threading model before implementation

---

### Pitfall 7: File:// Protocol Breaking Module Imports

**What goes wrong:**
Single HTML file tools fail when trying to use ES6 module imports (`import/export` syntax) from file:// protocol due to CORS restrictions. This breaks modern code organization and makes library integration difficult.

**Why it happens:**
Browsers treat file:// URLs with strict security policies. Module scripts require CORS headers, which don't exist for local files. This is a fundamental browser security restriction, not a bug.

**How to avoid:**
- For single HTML file: inline all JavaScript (no module imports)
- Use old-school script concatenation instead of ES6 modules
- Bundle libraries directly into HTML using `<script>` tags with inline code
- Alternative: use UMD or standalone builds of libraries (not ES6 module builds)
- Consider build step: bundle modules during development, output single HTML file
- For development: use local HTTP server (`python -m http.server`), NOT file://

**Warning signs:**
- Browser console shows CORS errors for local file imports
- Module imports work in development but fail when distributed as single HTML file
- Library documentation shows ES6 imports but doesn't provide UMD/standalone build

**Phase to address:**
Phase 1 (Core Generation) - Architecture decision about single-file approach and library integration

---

### Pitfall 8: Error Correction Level Selection Without Context

**What goes wrong:**
Choosing wrong error correction level for use case:
- Level H (30% recovery) for clean digital displays wastes space, makes QR code larger/denser
- Level L (7% recovery) for physical prints fails when QR code gets dirty/damaged
- Level Q/H when adding logos without accounting for actual logo coverage percentage

**Why it happens:**
Developers pick highest level thinking "more is better" or lowest level for "smaller codes" without considering deployment environment and corruption expectations.

**How to avoid:**
- Match error correction to environment:
  - L (7%): Clean digital screens, pristine environments, maximum data density
  - M (15%): General purpose, moderate reliability needs
  - Q (25%): Factory environments, outdoor signage, moderate wear expected
  - H (30%): Logos (up to 30% obstruction), heavy damage expected
- For optimization project: select level based on how much intentional corruption you'll add
- Account for size trade-offs: same URL at Level L=25×25 pixels, Level H=29×29 pixels
- Don't use H by default "just in case"—measure actual corruption needs

**Warning signs:**
- All QR codes use same error correction level regardless of use case
- QR codes become unscannable at small sizes (over-specified error correction)
- Physical prints fail frequently (under-specified error correction)
- Logo placement exceeds error correction capacity

**Phase to address:**
Phase 1 (Core Generation) - Error correction level should be configurable parameter, not hardcoded

---

## Technical Debt Patterns

Shortcuts that seem reasonable but create long-term problems.

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Hardcoding error correction to Level M | Simpler API, one less decision | Can't optimize for specific use cases, limits max corruption | MVP/prototype only, not production |
| Using only first mask pattern (0) without evaluation | Faster generation, skip penalty calculation | Produces suboptimal QR codes that scan slower/less reliably | Never acceptable—violates QR spec |
| Synchronous optimization on main thread | Simpler code, no worker complexity | UI freezes, poor UX, browser warnings | Only for very short hash fragments (<4 chars) |
| Testing with only one QR decoder library | Faster test execution | May miss decoder-specific compatibility issues | Acceptable if target decoder is known/fixed |
| Skipping quiet zone in canvas rendering | Slightly smaller output image | QR codes fail to scan in some conditions | Never acceptable for production use |
| Using `willReadFrequently` globally | Consistent performance | Disables GPU acceleration for all canvas operations | Only if reads dominate (>80% getImageData vs putImageData) |

## Integration Gotchas

Common mistakes when connecting to external services.

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| QR decoder libraries | Assuming all libraries expose error correction metrics | Check library API documentation—most only return decoded data, not correction details |
| Canvas 2D context | Not specifying `willReadFrequently` attribute | Set explicitly based on read/write ratio: `canvas.getContext('2d', {willReadFrequently: true})` if reads dominate |
| Web Workers | Trying to pass canvas directly to worker | Use `OffscreenCanvas` via `canvas.transferControlToOffscreen()` |
| URL encoding in QR | Encoding unescaped URLs with special characters | Use proper URL encoding before QR generation to avoid decoder interpretation issues |
| Library bundling | Including ES6 module builds in single HTML file | Use UMD/standalone builds or inline transpiled code |

## Performance Traps

Patterns that work at small scale but fail as usage grows.

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Calling `getImageData()` in tight loop | UI becomes sluggish, frame rate drops | Batch pixel operations, use OffscreenCanvas in worker | >100 calls per second, canvas >500×500px |
| Exhaustive brute-force without pruning | Optimization takes hours/never completes | Implement progressive search, early termination, heuristics | Hash fragments >6 characters |
| Creating new canvas for each QR code | Memory usage grows, garbage collection pauses | Reuse single canvas, clear between generations | Generating >1000 QR codes in session |
| Storing all intermediate results in memory | Browser tab crashes due to memory limit | Stream results to UI/download, don't accumulate in RAM | Optimization iterations >10,000 |
| Synchronous pixel manipulation | Browser "slow script" warnings | Use Web Workers for computation-heavy operations | Optimization runtime >2 seconds |

## Security Mistakes

Domain-specific security issues beyond general web security.

| Mistake | Risk | Prevention |
|---------|------|------------|
| Displaying unvalidated decoded QR content | XSS if QR contains malicious URLs/scripts | Validate and sanitize all decoded data before rendering |
| Not validating hash fragment format | Corrupted hashes may contain injection payloads | Enforce strict hex character validation, length limits |
| Trusting error correction to prevent tampering | Attackers can intentionally modify modules within correction capacity | Error correction enables tampering, not prevents it—verify with signatures/checksums |
| Generating predictable QR codes for sensitive data | Brute-force attacks can recover data | Use proper encryption before encoding, not just obfuscation |
| Exposing internal decoder states in UI | Information leakage about QR structure/algorithms | Only display user-relevant information (decoded data, success/failure) |

## UX Pitfalls

Common user experience mistakes in this domain.

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| No progress indication during long optimization | Users think browser crashed, abandon tool | Show progress bar, iteration count, estimated time remaining |
| No way to cancel running optimization | Must close tab/browser to stop, lose all work | Implement cancel button, save partial results |
| QR codes too dense to scan on mobile | Users frustrated, can't use generated codes | Warn when data length requires dense QR code, suggest URL shortening |
| Insufficient contrast between QR and background | Scan failures, especially in poor lighting | Validate contrast ratio, show warning if <3:1 |
| No visual feedback which pixels are locked | Users don't understand what optimization is doing | Highlight locked/corrupted pixels in distinct color during process |
| Exporting QR without quiet zone | Codes scan unreliably, users blame tool | Always include 4-module quiet zone in exported images |

## "Looks Done But Isn't" Checklist

Things that appear complete but are missing critical pieces.

- [ ] **QR code generation:** Often missing quiet zone rendering — verify exported images include 4-module white border
- [ ] **Error correction selection:** Often hardcoded to Level M — verify all four levels (L/M/Q/H) are configurable
- [ ] **Mask pattern selection:** Often uses first pattern only — verify all 8 patterns evaluated with penalty scoring
- [ ] **Pixel corruption:** Often corrupts function patterns by mistake — verify only data/error correction modules are corrupted
- [ ] **Decoder error measurement:** Often measures "scan success" not "errors corrected" — verify meaningful error metrics from decoder
- [ ] **Canvas pixel mapping:** Often off-by-one errors — verify module coordinates match canvas pixels across different QR sizes
- [ ] **Long-running optimization:** Often blocks main thread — verify Web Worker implementation with progress reporting
- [ ] **File:// compatibility:** Often breaks with module imports — verify single HTML file works without HTTP server
- [ ] **Hash search termination:** Often searches forever — verify early termination conditions and timeout logic
- [ ] **Result validation:** Often assumes decode success = correct data — verify decoded output matches expected format

## Recovery Strategies

When pitfalls occur despite prevention, how to recover.

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Wrong error correction level hardcoded | LOW | Add configuration parameter, regenerate all QR codes with correct level |
| Canvas pixel mapping off by one | MEDIUM | Create visual debugging mode showing module boundaries, fix coordinate calculations, re-verify all corruption logic |
| Optimization blocks UI thread | MEDIUM | Refactor to Web Worker architecture, may require significant code restructuring |
| ES6 imports break in single file | LOW | Switch to UMD library builds, inline all scripts in HTML |
| Mask pattern applied to function modules | HIGH | Complete rewrite of masking logic, comprehensive testing across all QR versions/sizes |
| No progress indication for long operations | LOW | Add progress callback hooks, update UI incrementally |
| Search space too large to complete | MEDIUM | Implement progressive search strategy, may require algorithm redesign |
| Decoder error measurement ill-defined | MEDIUM | Clarify requirements, may need different decoder library with better introspection |

## Pitfall-to-Phase Mapping

How roadmap phases should address these pitfalls.

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Error correction as data validation | Phase 1 (Core Generation) | Unit tests verify validation separate from encoding |
| Canvas pixel-to-module mapping | Phase 2 (Corruption/Measurement) | Visual debugging shows locked pixels align with intended modules |
| Incorrect mask pattern application | Phase 1 (Core Generation) | QR codes scan across multiple decoder libraries |
| Unclear decoder error measurement | Phase 2 (Corruption/Measurement) | Error metrics correlate with number of corrupted modules |
| Brute-force without search reduction | Phase 3 (Optimization) | Time complexity analysis, performance benchmarks for different hash lengths |
| Canvas operations freezing UI | Phase 3 (Optimization) | UI remains responsive during optimization, progress updates every 100ms |
| File:// protocol module imports | Phase 1 (Core Generation) | Single HTML file opens and functions from file:// URL |
| Wrong error correction level | Phase 1 (Core Generation) | All four levels selectable, appropriate defaults for use cases |

## Sources

**QR Code Error Correction:**
- [QR Code Error Correction Explained in 2026](https://scanova.io/blog/qr-code-error-correction/)
- [Error correction feature | DENSO WAVE](https://www.qrcode.com/en/about/error_correction.html)
- [Reed-Solomon codes for coders - Wikiversity](https://en.wikiversity.org/wiki/Reed%E2%80%93Solomon_codes_for_coders)

**QR Code Structure and Module Placement:**
- [What is QR Code Structure and How Does It Work? | 2026 Guide](https://scanova.io/blog/qr-code-structure/)
- [Module Placement in Matrix - QR Code Tutorial](https://www.thonky.com/qr-code-tutorial/module-placement-matrix)
- [What is a QR Code Quiet Zone and Why Does It Matter?](https://www.the-qrcode-generator.com/blog/qr-code-quiet-zone)

**Canvas and Performance:**
- [OffscreenCanvas—speed up your canvas operations with a web worker](https://web.dev/articles/offscreen-canvas)
- [Slow HTML Canvas Performance? Understanding Chrome's willReadFrequently Attribute](https://www.schiener.io/2024-08-02/canvas-willreadfrequently)
- [Mozilla Bug 1001069 - Slow getImageData() and putImageData() performance](https://bugzilla.mozilla.org/show_bug.cgi?id=1001069)

**Mask Pattern Selection:**
- [Data Masking - QR Code Tutorial](https://www.thonky.com/qr-code-tutorial/data-masking)
- [Let's develop a QR Code Generator, part VI: mask optimization](https://dev.to/maxart2501/let-s-develop-a-qr-code-generator-part-vi-mask-optimization-20gj)

**File Protocol Limitations:**
- [Suggestion: You should be able to load module-scripts in the file:// protocol](https://github.com/whatwg/html/issues/8121)
- [Useful patterns for building HTML tools](https://simonwillison.net/2025/Dec/10/html-tools/)

**JavaScript QR Libraries:**
- [QR Code generator library](https://www.nayuki.io/page/qr-code-generator-library)
- [qrcode.js](https://davidshimjs.github.io/qrcodejs/)
- [QRious - Pure JavaScript library for QR code generation](https://github.com/neocotic/qrious)

---
*Pitfalls research for: QR Code Generation and Optimization*
*Researched: 2026-02-06*
