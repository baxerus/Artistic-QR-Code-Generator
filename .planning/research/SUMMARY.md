# Project Research Summary

**Project:** Artistic QR Code Generator
**Domain:** Browser-based QR code generation with pixel-level artistic control
**Researched:** 2026-02-06
**Confidence:** MEDIUM-HIGH

## Executive Summary

This project is a single-file HTML artistic QR code generator that enables pixel-level pattern control through a novel hash fragment optimization approach. Unlike AI-powered QR art tools (Stable Diffusion/ControlNet) or traditional logo-overlay generators, this tool treats user-painted patterns as sacred and immutable, then searches for URL hash fragments that naturally align with the artistic pattern while maintaining scannability.

The recommended approach uses Nayuki's QR Code generator (module-level access via `getModule(x,y)`) for encoding, @paulmillr/qr for decoding/error measurement, and native Canvas API for pixel manipulation. All libraries must be inlineable (~1500-2000 lines total) for single-file portability with file:// protocol compatibility. The core architecture requires Web Workers for the CPU-intensive optimization loop to prevent UI freezing, with a three-state pixel system (white/black/unset) where users lock specific pixels and the algorithm searches hash variants to minimize decoder errors.

Key risks center on computational complexity (hash search space grows exponentially), canvas-to-module mapping precision (off-by-one errors corrupt critical QR patterns), and unclear error measurement definitions (most decoders expose binary success/failure, not error correction metrics). Mitigation strategies include progressive search with early termination, explicit pixel-to-module mapping with visual debugging, and defining "decoder errors" clearly before optimization begins.

## Key Findings

### Recommended Stack

The 2026 stack for single-file artistic QR code generation requires three zero-dependency libraries: Nayuki's QR Code generator for encoding (provides `getModule(x,y)` pixel-level access), @paulmillr/qr (renamed to `qr` on npm, latest v0.5.3 Jan 2026) for decoding and error measurement, and native Canvas API for pixel manipulation. Critical constraint: all libraries must be inlineable without build tools.

**Core technologies:**
- **Nayuki QR Code Generator (TypeScript/JS)**: QR generation with error correction level H and module matrix access — only generator with clean `getModule(x,y)` API, ~970 lines, zero dependencies, actively maintained
- **@paulmillr/qr (npm: `qr`)**: QR decoding with Reed-Solomon error correction — zero dependencies, ~162 ops/sec performance, recently updated Jan 2026, exposes error correction internals
- **Canvas API (native)**: Pixel manipulation via ImageData and Uint8ClampedArray — native browser API, works on file:// protocol, no external dependencies
- **ES6 Modules (inline)**: Code organization within single HTML file — use `<script type="module">` inline blocks, not external imports (file:// CORS restrictions)

**Version requirements:**
- QR versions 2-8 (25×25 to 49×49 grids) for artistic use cases
- Error correction level H hardcoded (30% tolerance necessary for pixel corruption)
- ES6+ browsers only (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)

### Expected Features

Research shows this tool competes in two domains: AI-powered artistic QR generators and traditional custom QR tools. Differentiation comes from deterministic pixel-level control (vs. AI black boxes) and sacred pattern philosophy (vs. logo-in-safe-zone overlays).

**Must have (table stakes):**
- URL input with validation and QR version auto-calculation — users expect this in all QR generators
- QR code preview and download as PNG — universal expectation
- Clear/reset functionality — standard UI pattern
- Visual distinction of pixel states (white/black/unset) — users need to see what they're painting
- Error correction level selection — QR spec feature, though we hardcode H with clear rationale
- Size/resolution control — scalable output for various use cases

**Should have (competitive differentiators):**
- **Pixel-level pattern control with three-state painting** — core innovation, no competitor offers this
- **Hash fragment optimization algorithm** — novel approach to align URL variants with art patterns
- **Live error counting during search** — real-time Reed-Solomon error metrics, transparency competitors lack
- **Top 5 results ranking** — choice beyond pure metrics (mathematical best ≠ aesthetic best)
- **Error visualization overlay** — shows pattern vs QR data vs conflicts, educational and practical
- **Single-file portability** — works via file://, zero deployment complexity
- **Manual stop control** — user stops when satisfied, doesn't wait for timeout
- **Live search progress** — attempts counter, elapsed time, current best preview

**Defer (v2+):**
- High-resolution export settings (300+ DPI for print) — add when users request print-quality
- Color customization (foreground/background) — nice-to-have, black/white sufficient initially
- Pattern save/load (JSON format) — iteration support, add after core validates
- Keyboard shortcuts for painting — power user feature, not MVP
- SVG export — PNG sufficient, vector is luxury
- Undo/redo history — clear/reset works, undo is UX refinement
- Mobile/responsive layout — pixel painting on touchscreens impractical, desktop-first
- QR versions beyond 8 (up to 40) — covers most cases, larger adds complexity
- Batch URL processing — different use case (business vs artistic)
- Real-time camera scan testing — requires camera API, error count is sufficient proxy

### Architecture Approach

Standard browser-based QR generator architecture with Web Worker offloading for optimization. Key pattern: separate layers for user pixels (locked) and QR-generated pixels, composite on demand. Use Proxy-based reactive state management for vanilla JS simplicity, message-passing worker architecture for non-blocking UI, and canvas-based pixel editor with state layers.

**Major components:**
1. **Input Panel** — URL validation and QR version calculation; direct DOM manipulation with event listeners
2. **Canvas Editor** — Three-state pixel painting interface; HTML5 Canvas with mouse event handlers for white/black/unset cycling
3. **State Manager** — Centralized reactive state (Proxy-based); manages URL, QR version, user pixels, locked positions, top results
4. **QR Generator Module** — Wrapper around Nayuki library; generates base QR code with error correction level H
5. **Pixel Merger Module** — Combines user pixels (locked) with QR pixels; user pixels override QR data in conflict areas
6. **Optimization Engine (Web Worker)** — Generates hash variations, decodes QR, scores by error count, tracks top 5; runs in separate thread to prevent UI freezing
7. **Decoder/Validator Module** — jsQR or @paulmillr/qr; decodes QR and counts Reed-Solomon errors vs expected URL
8. **Results Display** — Top 5 candidates table with error counts, download PNG per result, copy URL per result
9. **Output Panel** — Canvas toBlob API for PNG download, Clipboard API for URL copy, error visualization overlay

**Critical architectural decisions:**
- **Web Worker required:** Optimization loop may run thousands/millions of iterations; main thread would freeze (see Pitfall 6)
- **Single-file constraint:** All JavaScript inlined in HTML (no external imports); ES6 modules work inline but NOT via file:// imports
- **Layer separation:** User pixels (locked) and QR pixels (generated) stored separately, composited only during optimization
- **Top-N tracking:** Priority queue maintains only top 5 results; constant memory regardless of iteration count

### Critical Pitfalls

Research identified 8 critical and 40+ general pitfalls. Top 5 by severity and likelihood:

1. **Canvas pixel-to-module mapping misalignment** — Miscalculation causes corruption of finder/timing/alignment patterns instead of data regions, making QR codes completely unscannable. Prevention: explicit mapping function with unit tests, visualize locked regions before corruption, verify 4 pixels per module minimum, never modify function patterns. *Address in Phase 2 (Pixel Painting & Corruption).*

2. **Brute-force hash search without search space reduction** — 6-char hex hash = 16.7M combinations; 8-char = 4.3B combinations. Naive brute-force becomes computationally infeasible. Prevention: start with 4 chars max, implement progressive search (optimize 1 char at a time), use pattern-based pruning, set realistic expectations (8+ chars may need GPU acceleration). *Address in Phase 3 (Optimization).*

3. **Canvas operations causing UI freezing** — Repeated `getImageData()`/`putImageData()` blocks main thread (each call = several ms GPU-to-CPU transfer). Prevention: use Web Workers with OffscreenCanvas API, batch canvas operations, implement progress reporting (every 100ms, not every iteration), add cancel mechanism. *Address in Phase 3 (Optimization).*

4. **Measuring "decoder errors" without clear definition** — "Decoder errors" is ambiguous: corrupted modules? Reed-Solomon codewords? Scanning success rate? Most decoders only expose binary success/failure. Prevention: define exactly what "error" means (prefer: number of corrupted modules), document canonical decoder library, verify measurement methodology correlates with corruption level. *Address in Phase 2 (Pixel Painting & Corruption).*

5. **File:// protocol breaking module imports** — ES6 `import/export` syntax fails on file:// URLs due to CORS restrictions. Prevention: inline all JavaScript (no external imports), use UMD/standalone builds (not ES6 module builds), bundle libraries directly in HTML `<script>` tags, use local HTTP server for development (NOT file://). *Address in Phase 1 (Foundation & QR Generation).*

**Additional critical pitfalls:**
- **Misunderstanding error correction as data validation** — Reed-Solomon corrects physical damage (corrupted modules), not logical errors (wrong URL encoded). Must validate inputs separately.
- **Incorrect mask pattern application** — Masks apply ONLY to data/error modules, NEVER to function patterns (finder, timing, alignment) or reserved areas. Violating this makes codes unscannable.
- **Error correction level selection without context** — Level H wastes space on clean digital displays; Level L fails on physical prints. Match level to environment and expected corruption.

## Implications for Roadmap

Based on research dependencies and risk mitigation, suggested 4-phase structure:

### Phase 1: Foundation & QR Generation
**Rationale:** Establish single-file architecture and core QR generation before adding complexity. Must resolve file:// protocol constraints and library integration immediately (Pitfall 7). QR generation is foundation for all subsequent phases.

**Delivers:**
- Single HTML file with inlined Nayuki QR generator (~970 lines)
- URL input panel with validation
- Automatic QR version calculation (minimum version based on URL length)
- Basic QR code rendering to canvas
- Error correction level H hardcoded with documented rationale

**Addresses features:**
- URL input with validation (table stakes)
- QR code preview (table stakes)
- QR version auto-calculation (differentiator)

**Avoids pitfalls:**
- Pitfall 7: File:// protocol breaking module imports — inline all libraries as UMD/standalone
- Pitfall 8: Wrong error correction level — hardcode H, document why, make future-configurable
- Pitfall 1: Error correction as data validation — implement input validation separate from encoding

**Research flag:** Standard patterns, skip phase research. QR generation is well-documented domain (Nayuki docs, ISO/IEC 18004 spec).

---

### Phase 2: Pixel Painting & Corruption
**Rationale:** Three-state pixel system is core differentiator and most architecturally novel component. Must establish pixel-to-module mapping correctly before optimization (Pitfall 2) and define error measurement methodology (Pitfall 4).

**Delivers:**
- Canvas editor with three-state pixel painting (white/black/unset)
- Click to cycle pixel states
- Visual distinction between states (color-coded)
- Clear/reset button
- Pixel merger module (locks user pixels, overlays on QR grid)
- QR decoder integration (@paulmillr/qr)
- Error counting mechanism with clear definition

**Addresses features:**
- Three-state pixel painting (core differentiator)
- Visual distinction of pixel states (table stakes)
- Clear/reset functionality (table stakes)

**Avoids pitfalls:**
- Pitfall 2: Canvas pixel-to-module mapping misalignment — explicit mapping function, visual debugging mode
- Pitfall 4: Unclear decoder error measurement — define "error" as corrupted modules, verify correlation
- Pitfall 3: Incorrect mask pattern application — verify locked pixels don't corrupt function patterns

**Research flag:** NEEDS RESEARCH. Three-state pixel system and pixel-to-module mapping are project-specific. Use `/gsd:research-phase` for:
- Canvas coordinate transformations (canvas pixels → QR modules)
- QR code function pattern locations (finder, timing, alignment by version)
- Decoder error measurement internals (@paulmillr/qr source code inspection)

---

### Phase 3: Hash Optimization Loop
**Rationale:** Optimization is highest-risk component (computational complexity, UI freezing). Requires Web Worker architecture (Pitfall 6) and search space reduction strategy (Pitfall 5). Depends on Phase 2 error measurement.

**Delivers:**
- Web Worker implementation with message protocol
- Hash fragment search strategy (start 4 chars, progressive expansion)
- Top 5 results tracking (priority queue)
- Live progress display (attempts, elapsed time)
- Live error count and preview of current best
- Manual stop button
- Configurable max search time

**Addresses features:**
- Hash fragment optimization (core differentiator)
- Live error counting (differentiator)
- Live search progress (differentiator)
- Manual stop control (differentiator)
- Top 5 results display (differentiator)

**Avoids pitfalls:**
- Pitfall 6: Canvas operations freezing UI — Web Worker with OffscreenCanvas, batched progress updates
- Pitfall 5: Brute-force without search reduction — start 4-char hashes, progressive search, early termination
- Performance trap: Exhaustive brute-force — implement iteration limits, time-based cutoffs

**Research flag:** NEEDS RESEARCH. Optimization algorithm and Web Worker architecture require deeper investigation. Use `/gsd:research-phase` for:
- Web Worker communication protocols (structured cloning vs transferables)
- OffscreenCanvas API and browser compatibility (2026 support)
- Hash search heuristics (pattern-based pruning strategies)
- Performance benchmarks (iterations/second for different QR versions)

---

### Phase 4: Results & Export
**Rationale:** Output features depend on optimization results. Straightforward implementation using standard browser APIs.

**Delivers:**
- Results table showing top 5 candidates with error counts
- Download PNG per result (Canvas toBlob API)
- Copy URL with hash fragment per result (Clipboard API)
- Error visualization overlay (highlight conflicts: pattern vs QR data vs errors)

**Addresses features:**
- Download PNG (table stakes)
- Copy URL to clipboard (table stakes)
- Error visualization overlay (core differentiator)

**Avoids pitfalls:**
- UX pitfall: No visual feedback on locked pixels — error overlay shows exactly which pixels conflict
- UX pitfall: Exporting without quiet zone — ensure 4-module white border in all exports

**Research flag:** Standard patterns, skip phase research. Canvas toBlob and Clipboard API are well-documented browser APIs.

---

### Phase Ordering Rationale

**Dependency chain:**
1. Phase 1 establishes single-file architecture → necessary for all subsequent phases (file:// constraint)
2. Phase 2 requires Phase 1 QR generation → can't paint pixels without base QR grid
3. Phase 3 requires Phase 2 error measurement → can't optimize without decoder/error counting
4. Phase 4 requires Phase 3 results → can't export without optimization output

**Risk mitigation order:**
- Phase 1 addresses Pitfall 7 (file:// imports) immediately — affects all phases
- Phase 2 addresses Pitfall 2 (pixel mapping) before optimization — prevents wasted effort optimizing broken mappings
- Phase 3 addresses Pitfalls 5-6 (search complexity, UI freezing) when they occur — optimization is where these manifest

**Feature grouping logic:**
- Phase 1: Input and basic generation (standard QR generator baseline)
- Phase 2: Artistic features (three-state pixels, pattern locking)
- Phase 3: Optimization intelligence (hash search, error minimization)
- Phase 4: Output and visualization (results, export, error overlay)

### Research Flags

**Phases needing `/gsd:research-phase` during planning:**

- **Phase 2 (Pixel Painting & Corruption):**
  - Canvas-to-module coordinate mapping (project-specific, niche)
  - QR code function pattern locations by version (requires ISO/IEC 18004 spec parsing)
  - @paulmillr/qr decoder internals for error counting (source code inspection needed)

- **Phase 3 (Hash Optimization Loop):**
  - Web Worker + OffscreenCanvas integration patterns (2026-specific APIs, evolving)
  - Hash search heuristics and pruning strategies (domain-specific optimization)
  - Performance benchmarks across QR versions (empirical testing required)

**Phases with standard patterns (skip research-phase):**

- **Phase 1 (Foundation & QR Generation):**
  - QR generation well-documented (Nayuki official docs, ISO spec)
  - Single-file HTML patterns established (inline scripts, UMD bundles)
  - URL validation is standard web development

- **Phase 4 (Results & Export):**
  - Canvas toBlob API documented on MDN
  - Clipboard API standard browser feature
  - Results table rendering is basic DOM manipulation

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Nayuki QR generator verified via official docs, actively maintained. @paulmillr/qr updated Jan 2026, zero dependencies. Canvas API is native browser standard. All libraries confirmed inlineable and file:// compatible. |
| Features | MEDIUM-HIGH | Competitive analysis comprehensive (AI tools, traditional generators, specialized tools). Three-state pixel system validated as novel differentiator. MVP feature set aligns with table stakes expectations. Defer list matches v2+ refinements. |
| Architecture | HIGH | Web Worker for optimization is well-established pattern. Proxy-based state management standard for vanilla JS. Canvas layer separation proven approach. Single-file constraint addressed via inline scripts. |
| Pitfalls | MEDIUM | Critical pitfalls identified from research (8 critical, 40+ general). Pixel-to-module mapping and error measurement are project-specific risks needing validation during implementation. Hash search complexity well-understood mathematically but performance needs empirical testing. |

**Overall confidence:** MEDIUM-HIGH

Research covers standard QR generation (high confidence) and novel optimization approach (medium confidence). Uncertainty centers on:
1. Decoder error measurement precision (most libraries expose binary success/failure, not error counts)
2. Hash search performance at scale (theoretical complexity clear, practical throughput needs benchmarking)
3. Canvas-to-module mapping edge cases (different QR versions, scaling factors, quiet zones)

### Gaps to Address

**Gap 1: Decoder error measurement internals**
- **Issue:** @paulmillr/qr and most decoders return decoded data or null, not error correction metrics
- **Impact:** May need to count corrupted modules directly (compare generated QR to corrupted QR pixel-by-pixel) instead of Reed-Solomon error counts
- **Resolution:** During Phase 2 planning, inspect @paulmillr/qr source code for error correction hooks. If unavailable, define "error" as pixel differences between perfect QR and corrupted QR.

**Gap 2: Hash search performance ceiling**
- **Issue:** Unclear how many iterations/second achievable in browser Web Worker for QR decode + error count cycle
- **Impact:** Affects max hash length recommendations and user expectations for search duration
- **Resolution:** During Phase 3 planning, benchmark simple QR decode loop (no optimization) to establish baseline throughput. Use benchmark to calculate realistic iteration counts for 30s/60s/120s timeouts.

**Gap 3: QR version capacity vs URL length**
- **Issue:** Need precise capacity table (QR version × error level H → max bytes) to auto-calculate minimum version
- **Impact:** Version selector may show invalid ranges or fail to encode long URLs
- **Resolution:** During Phase 1 implementation, extract capacity table from Nayuki library documentation or source code. Hardcode table for versions 2-8 with Level H.

**Gap 4: Browser compatibility for OffscreenCanvas**
- **Issue:** OffscreenCanvas API relatively new (standardized ~2018, wide support ~2020+); may have edge cases in 2026
- **Impact:** Web Worker canvas operations may fail in older browsers
- **Resolution:** During Phase 3 planning, verify OffscreenCanvas support in target browsers (Chrome 90+, Firefox 88+, Safari 14+). Document minimum browser versions in README.

**Gap 5: Mask pattern interaction with locked pixels**
- **Issue:** Locking arbitrary pixels may interfere with QR mask pattern selection (Nayuki auto-selects best mask)
- **Impact:** Locked pixels could force suboptimal mask selection, reducing scannability
- **Resolution:** During Phase 2 implementation, test whether Nayuki's mask selection works correctly when modules are pre-locked. May need to manually evaluate all 8 masks and select lowest penalty.

## Sources

### Primary (HIGH confidence)

**STACK.md sources:**
- [QR Code generator library (Nayuki)](https://www.nayuki.io/page/qr-code-generator-library) — QR generation with module access
- [GitHub: nayuki/QR-Code-generator](https://github.com/nayuki/QR-Code-generator) — Source code verification
- [npm: qr (formerly @paulmillr/qr)](https://www.npmjs.com/package/qr) — Decoder library, v0.5.3 Jan 2026
- [MDN: Canvas API Pixel Manipulation](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Pixel_manipulation_with_canvas) — Canvas ImageData API
- [DENSO WAVE: Error Correction Feature](https://www.qrcode.com/en/about/error_correction.html) — Official QR spec documentation

**FEATURES.md sources:**
- [Quick QR Art](https://quickqr.art/) — AI-powered artistic QR competitor analysis
- [OpenArt AI QRCode](https://openart.ai/apps/ai_qrcode) — AI QR generator competitor
- [QRCode Monkey](https://www.qrcode-monkey.com/) — Traditional custom QR tool
- [Canva QR Generator](https://www.canva.com/qr-code-generator/) — Template-based QR tool

**ARCHITECTURE.md sources:**
- [Web Worker Overview - web.dev](https://web.dev/learn/performance/web-worker-overview) — Web Worker architecture patterns
- [jsQR (cozmo)](https://github.com/cozmo/jsQR) — QR decoder library
- [nimiq/qr-scanner](https://github.com/nimiq/qr-scanner) — High-performance decoder alternative

**PITFALLS.md sources:**
- [QR Code Structure (Scanova)](https://scanova.io/blog/qr-code-structure/) — Module placement and function patterns
- [OffscreenCanvas - web.dev](https://web.dev/articles/offscreen-canvas) — Canvas performance in workers
- [willReadFrequently attribute (Schiener)](https://www.schiener.io/2024-08-02/canvas-willreadfrequently) — Canvas performance pitfall
- [Data Masking - QR Code Tutorial](https://www.thonky.com/qr-code-tutorial/data-masking) — Mask pattern implementation

### Secondary (MEDIUM confidence)

- [Scanova: AI Art QR Code Guide 2026](https://scanova.io/blog/ai-art-qr-code-guide/) — Competitor landscape analysis
- [State Management in Vanilla JS 2026 (Medium)](https://medium.com/@chirag.dave/state-management-in-vanilla-js-2026-trends-f9baed7599de) — State management patterns
- [scheduler.yield() Deep Dive (Medium)](https://medium.com/@tharunbalaji110/deep-dive-scheduler-yield-and-the-art-of-non-blocking-ui-updates-18b01241106a) — UI performance optimization
- [Reed-Solomon codes for coders (Wikiversity)](https://en.wikiversity.org/wiki/Reed%E2%80%93Solomon_codes_for_coders) — Error correction theory

### Tertiary (LOW confidence, needs validation)

- [ScienceDirect: Aesthetic QR Code Solution](https://www.sciencedirect.com/science/article/abs/pii/S016412121500148X) — Academic research on artistic QR (paywalled, abstract only)
- [Wiley: Artistic QR Code Embellishment](https://onlinelibrary.wiley.com/doi/10.1111/cgf.12221) — Academic research (paywalled)
- Various QR code generator comparison blogs (2026 rankings, marketing-focused)

---

*Research completed: 2026-02-06*
*Ready for roadmap: yes*
