# Architecture Research

**Domain:** Browser-based QR code generator with optimization
**Researched:** 2026-02-06
**Confidence:** HIGH

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      UI Layer                                │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌───────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Input   │  │  Canvas   │  │ Results  │  │  Output  │   │
│  │  Panel   │  │  Editor   │  │  Display │  │  Panel   │   │
│  └────┬─────┘  └─────┬─────┘  └────┬─────┘  └────┬─────┘   │
│       │              │              │             │          │
├───────┴──────────────┴──────────────┴─────────────┴─────────┤
│                    State Manager                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Current State: { url, version, pixels, results, ... }│   │
│  └──────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                   Core Logic Layer                           │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐        │
│  │ QR Generator│  │ Pixel Merger │  │  Validator  │        │
│  │   Module    │  │    Module    │  │   Module    │        │
│  └─────────────┘  └──────────────┘  └─────────────┘        │
├─────────────────────────────────────────────────────────────┤
│                  Web Worker Thread                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │           Optimization Engine                       │    │
│  │  ┌────────────┐  ┌────────────┐  ┌─────────────┐   │    │
│  │  │  Search    │  │  Decoder   │  │   Scorer    │   │    │
│  │  │  Manager   │  │  (Validate)│  │  (Rank)     │   │    │
│  │  └────────────┘  └────────────┘  └─────────────┘   │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| **Input Panel** | URL validation, QR version calculation | Direct DOM manipulation with event listeners |
| **Canvas Editor** | 3-state pixel painting interface | HTML5 Canvas with mouse event handlers |
| **QR Generator Module** | Generate base QR code with error correction | Wrapper around QR library (e.g., Nayuki's implementation) |
| **Pixel Merger Module** | Combine user pixels with QR code (locked pixels) | Canvas pixel manipulation or data array merge |
| **State Manager** | Centralized state with pub/sub for UI updates | Proxy-based reactive store or simple observer pattern |
| **Optimization Engine** | Try variations, decode, score, track top N | Runs in Web Worker with message passing |
| **Decoder/Validator** | Decode QR, count errors vs expected URL | jsQR or similar library |
| **Results Display** | Show top 5 candidates with error counts | Dynamic HTML generation |
| **Output Panel** | Download PNG, copy to clipboard, visualize errors | Canvas toBlob API, Clipboard API |

## Recommended Project Structure

Since this is a single HTML file:

```
index.html
├── <style>              # All CSS (scoped by component classes)
│   ├── .input-panel {}
│   ├── .canvas-editor {}
│   ├── .results-display {}
│   └── .output-panel {}
├── <div id="app">       # All HTML structure
│   ├── Input Panel
│   ├── Canvas Editor
│   ├── Results Display
│   └── Output Panel
├── <script>             # Main application logic
│   ├── // State Management (first)
│   ├── // QR Generation Module
│   ├── // Pixel Merger Module
│   ├── // Validation Module
│   ├── // UI Controllers (event handlers)
│   └── // Initialization
└── <script>             # Web Worker code (inline with Blob)
    ├── // Worker message handler
    ├── // Optimization loop
    ├── // Decoder integration
    └── // Result communication
```

### Structure Rationale

- **Single file constraint**: All components must be inline (CSS, HTML, JS, Worker)
- **Top-down organization**: State → Core Logic → UI Controllers → Worker
- **Component isolation**: Use CSS classes and function namespacing to separate concerns
- **Worker as blob**: Web Worker code embedded as `new Worker(URL.createObjectURL(new Blob([...])))` for file:// compatibility

## Architectural Patterns

### Pattern 1: Proxy-Based State Management

**What:** Use ES6 Proxy to create reactive state that automatically triggers UI updates

**When to use:** For managing global state (URL, QR version, user pixels, top results) in vanilla JS without frameworks

**Trade-offs:**
- ✅ Automatic UI updates when state changes
- ✅ Simple mental model (mutate state, UI updates)
- ✅ No dependencies
- ❌ Can be harder to debug (less explicit)
- ❌ Proxy overhead (negligible for this use case)

**Example:**
```javascript
const createStore = (initialState, watchers = {}) => {
  return new Proxy(initialState, {
    set(target, property, value) {
      const oldValue = target[property];
      target[property] = value;
      if (watchers[property]) {
        watchers[property](value, oldValue);
      }
      return true;
    }
  });
};

const state = createStore(
  { url: '', qrVersion: null, results: [] },
  {
    url: (newUrl) => updateUrlDisplay(newUrl),
    results: (newResults) => renderResultsTable(newResults)
  }
);

// Usage: state.url = 'https://example.com'; // Automatically updates UI
```

### Pattern 2: Message-Passing Worker Architecture

**What:** Offload optimization loop to Web Worker with structured message protocol

**When to use:** For CPU-intensive, long-running tasks that would freeze the UI (optimization search)

**Trade-offs:**
- ✅ Non-blocking UI during search
- ✅ Can run for minutes without freezing browser
- ✅ Parallel execution on multi-core systems
- ❌ Communication overhead (structured cloning)
- ❌ No direct DOM access from worker
- ❌ More complex debugging

**Example:**
```javascript
// Main thread
const workerCode = `
  self.onmessage = (e) => {
    const { type, payload } = e.data;
    if (type === 'START_OPTIMIZATION') {
      const { baseQR, userPixels, targetUrl } = payload;
      // Run optimization loop
      for (let i = 0; i < 1000000; i++) {
        const candidate = generateVariation(baseQR, userPixels, i);
        const decoded = decodeQR(candidate);
        const errors = countErrors(decoded, targetUrl);

        // Report progress every 1000 iterations
        if (i % 1000 === 0) {
          self.postMessage({ type: 'PROGRESS', payload: { iteration: i } });
        }

        // Report if top 5 candidate
        if (isTopFive(errors)) {
          self.postMessage({ type: 'NEW_RESULT', payload: { candidate, errors } });
        }
      }
      self.postMessage({ type: 'COMPLETE' });
    }
  };
`;

const blob = new Blob([workerCode], { type: 'application/javascript' });
const worker = new Worker(URL.createObjectURL(blob));

worker.onmessage = (e) => {
  const { type, payload } = e.data;
  switch(type) {
    case 'PROGRESS': updateProgressBar(payload.iteration); break;
    case 'NEW_RESULT': addToResults(payload); break;
    case 'COMPLETE': showComplete(); break;
  }
};
```

### Pattern 3: Canvas-Based Pixel Editor with State Layers

**What:** Maintain separate layers for user pixels and QR code, composite on demand

**When to use:** When users need to paint pixels that override or merge with generated QR code

**Trade-offs:**
- ✅ Clear separation of concerns (user data vs generated data)
- ✅ Easy to show/hide layers
- ✅ Simple undo/redo (just track user layer)
- ❌ Extra memory for layer storage
- ❌ Compositing step before optimization

**Example:**
```javascript
const canvasLayers = {
  user: new Uint8Array(size * size), // 0=empty, 1=white, 2=black
  qr: null, // Generated from QR library

  getPixel(x, y) {
    const idx = y * size + x;
    // User pixels override QR pixels
    return this.user[idx] !== 0 ? this.user[idx] : this.qr[idx];
  },

  composite() {
    const result = new Uint8Array(size * size);
    for (let i = 0; i < result.length; i++) {
      result[i] = this.user[i] !== 0 ? this.user[i] : this.qr[i];
    }
    return result;
  }
};
```

### Pattern 4: Priority Queue for Top-N Results

**What:** Maintain a min-heap of top N results by error count, efficiently replacing worst

**When to use:** When tracking "best N results" from thousands/millions of candidates

**Trade-offs:**
- ✅ O(log N) insertion for new candidates
- ✅ Constant-time access to worst of top N
- ✅ Memory efficient (only stores top N)
- ❌ Slightly more complex than sorted array
- ❌ Overkill if N is small (<10)

**Example:**
```javascript
class TopNResults {
  constructor(n) {
    this.n = n;
    this.results = [];
  }

  add(candidate, errors) {
    if (this.results.length < this.n) {
      this.results.push({ candidate, errors });
      this.results.sort((a, b) => a.errors - b.errors);
    } else if (errors < this.results[this.results.length - 1].errors) {
      this.results.pop();
      this.results.push({ candidate, errors });
      this.results.sort((a, b) => a.errors - b.errors);
    }
  }

  getAll() {
    return this.results.slice();
  }
}
```

### Pattern 5: Non-Blocking Progress Updates with Batching

**What:** Use requestAnimationFrame or setTimeout(0) to batch UI updates without freezing

**When to use:** When worker sends frequent progress messages (e.g., every 100ms)

**Trade-offs:**
- ✅ Smooth UI updates without jank
- ✅ Prevents flooding main thread with updates
- ✅ Better perceived performance
- ❌ Slight latency in progress display
- ❌ Need to track "pending update" state

**Example:**
```javascript
let pendingUpdate = null;

function scheduleProgressUpdate(iteration, results) {
  if (pendingUpdate === null) {
    pendingUpdate = requestAnimationFrame(() => {
      updateProgressBar(iteration);
      updateResultsTable(results);
      pendingUpdate = null;
    });
  }
  // If already scheduled, skip (batching)
}
```

## Data Flow

### Request Flow: User Action → QR Generation

```
[User enters URL]
    ↓
[Validate URL] → [Calculate min QR version]
    ↓
[Generate QR code with error correction]
    ↓
[Render to canvas layer]
    ↓
[Enable pixel editing]
```

### Request Flow: Optimization Loop

```
[User clicks "Optimize"]
    ↓
[Composite user pixels + QR pixels] → [Identify locked pixels]
    ↓
[Send to Web Worker]: { baseQR, userPixels, targetUrl, lockedPositions }
    ↓
[Worker: Generate variation] → [Try hash fragment #abc]
    ↓
[Worker: Decode QR] → [Get decoded URL]
    ↓
[Worker: Count errors] → [Compare to target URL]
    ↓
[Worker: Check if top 5] → [Send to main thread if yes]
    ↓
[Main thread: Update UI] → [Add to results table]
    ↓
[Repeat 1M times or until stopped]
    ↓
[Show final top 5 results]
```

### State Management Flow

```
[State Store]
    ↓ (subscribe)
[UI Components] ←→ [Actions] → [State Mutations] → [State Store]
                                                         ↓
                                                   [Watchers]
                                                         ↓
                                                   [Re-render UI]
```

### Key Data Flows

1. **URL → QR Version**: User inputs URL → Calculate encoded size → Determine minimum QR version (1-40) → Store in state
2. **QR + User Pixels → Composite**: Merge two arrays with user pixels taking priority → Used as optimization base
3. **Worker Results → UI**: Worker finds candidate → Post message to main thread → Batched update to results table
4. **Download Flow**: User selects result → Get canvas data → Convert to blob → Trigger download

## Scaling Considerations

| Scale | Architecture Adjustments |
|-------|--------------------------|
| 1-100 iterations | Run in main thread with setTimeout(0) yielding |
| 100-10k iterations | Use Web Worker, update UI every 100 iterations |
| 10k-1M iterations | Use Web Worker, batch UI updates with requestAnimationFrame |
| 1M+ iterations | Consider IndexedDB for storing candidates, pagination for results |

### Scaling Priorities

1. **First bottleneck: UI freezing during optimization**
   - Solution: Move optimization loop to Web Worker
   - Impact: Enables multi-minute searches without blocking UI

2. **Second bottleneck: Memory usage for storing candidates**
   - Solution: Only store top N results (N=5), discard others immediately
   - Impact: Constant memory usage regardless of iteration count

3. **Third bottleneck: QR decoding performance**
   - Solution: Use efficient decoder (jsQR or nimiq/qr-scanner)
   - Impact: 2-3x faster decoding vs older libraries

4. **Fourth bottleneck: Canvas rendering during pixel editing**
   - Solution: Use offscreen canvas, only render visible pixels
   - Impact: Smooth editing even for large QR codes (version 40)

## Anti-Patterns

### Anti-Pattern 1: Synchronous Optimization Loop in Main Thread

**What people do:** Run optimization loop directly in event handler with blocking for-loop

**Why it's wrong:** Browser freezes, user can't stop, terrible UX. Any task >50ms blocks UI.

**Do this instead:** Always use Web Worker for any loop >100 iterations. If <100 iterations, use setTimeout(0) or scheduler.yield() (2026+) between batches.

### Anti-Pattern 2: Decoding on Every Pixel Change

**What people do:** Re-decode QR code immediately when user paints pixel

**Why it's wrong:** Decoding is expensive (~10-50ms), creates lag during painting

**Do this instead:** Debounce decoding by 500ms, or trigger only on explicit "validate" button

### Anti-Pattern 3: Storing Full Image Data for Each Candidate

**What people do:** Store PNG blob or full ImageData for every candidate

**Why it's wrong:** Memory explodes (1MB per image × 1000 candidates = 1GB)

**Do this instead:** Store only minimal data (hash fragment, error count). Regenerate image on-demand when user selects result.

### Anti-Pattern 4: Tight Coupling Between QR Library and Decoder

**What people do:** Use same library for generation and decoding, assume symmetry

**Why it's wrong:** Generation ≠ validation. Different scanners have different behaviors. Testing against one decoder doesn't guarantee real-world scanning success.

**Do this instead:** Use best-in-class library for each task. Generation: Nayuki (accurate, flexible). Decoding: jsQR or nimiq/qr-scanner (fast, robust). Test generated codes with multiple decoders.

### Anti-Pattern 5: No Cancellation Mechanism

**What people do:** Start optimization, no way to stop until complete

**Why it's wrong:** User may realize they made mistake, or want to adjust parameters

**Do this instead:** Implement cancellation via worker.terminate() or message-based graceful shutdown. Always provide "Stop" button during search.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| QR Generation Library (Nayuki) | Import as inline JS or minimal build | Use `QrCode.encodeText()` for simple cases, `QrCode.encodeSegments()` for manual control |
| QR Decoder Library (jsQR) | Import as inline JS | Use `jsQR(imageData, width, height)` to decode canvas |
| None (all client-side) | File:// protocol compatible | No network requests, works offline |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Main Thread ↔ Web Worker | postMessage with structured types | Define clear message protocol: { type: 'ACTION', payload: {...} } |
| State Manager ↔ UI | Proxy watchers or PubSub | One-way data flow: state change → UI update |
| Canvas Editor ↔ State | Direct mutation + event dispatch | Mouse events → update state.userPixels → re-render |
| QR Generator ↔ Pixel Merger | Function call (sync) | Pass arrays, return composite array |

## Build Order and Dependencies

### Phase 1: Foundation (No dependencies)
1. **HTML Structure**: Create layout with all panels
2. **CSS Styling**: Make it usable (responsive not required)
3. **State Manager**: Implement Proxy-based store

### Phase 2: Basic QR Generation (Depends on: State Manager)
4. **Integrate QR Library**: Add Nayuki's generator
5. **URL Input Panel**: Validation, version calculation
6. **QR Display**: Render generated QR to canvas

### Phase 3: Pixel Editing (Depends on: QR Generation)
7. **Canvas Editor**: 3-state pixel painting
8. **Layer Management**: Separate user/QR layers
9. **Pixel Merger**: Combine layers with locked pixels

### Phase 4: Validation (Depends on: QR Generation)
10. **Integrate Decoder Library**: Add jsQR
11. **Validation Module**: Decode and compare URLs
12. **Error Counting**: Character-level diff

### Phase 5: Optimization Loop (Depends on: Validation, Pixel Merger)
13. **Web Worker Setup**: Inline worker with message protocol
14. **Search Strategy**: Generate variations (hash fragments)
15. **Top-N Tracker**: Priority queue for results
16. **Progress Updates**: Batched UI updates

### Phase 6: Output (Depends on: Optimization Loop)
17. **Results Display**: Table with error counts
18. **Download Feature**: Canvas to PNG blob
19. **Copy to Clipboard**: Use Clipboard API
20. **Error Visualization**: Highlight incorrect characters

### Dependency Graph

```
State Manager ────┬─────────────────────────┐
                  │                         │
         URL Input Panel                    │
                  │                         │
            QR Generator ───────────────────┼─────┐
                  │                         │     │
            QR Display              Canvas Editor │
                  │                         │     │
                  └──── Pixel Merger ───────┘     │
                            │                     │
                      QR Decoder ─────────────────┘
                            │
                   Validation Module
                            │
                   Optimization Loop (Worker)
                            │
                      Results Display
                            │
                   Output Panel (Download/Copy)
```

## Web Worker Considerations

### When to Use Web Worker

✅ **Use for:**
- Optimization loop (hundreds to millions of iterations)
- Any operation >100ms that runs repeatedly
- CPU-intensive decoding (if processing many candidates in batch)

❌ **Don't use for:**
- Single QR generation (too fast, overhead not worth it)
- UI updates (workers can't access DOM)
- Initial page load (complexity not justified)

### Worker Communication Protocol

```javascript
// Message Types (Main → Worker)
{
  type: 'START_OPTIMIZATION',
  payload: {
    baseQR: Uint8Array,
    userPixels: Uint8Array,
    lockedPixels: Array<[x, y]>,
    targetUrl: string,
    maxIterations: number
  }
}

{ type: 'STOP' }

// Message Types (Worker → Main)
{
  type: 'PROGRESS',
  payload: { iteration: number, elapsed: number }
}

{
  type: 'NEW_RESULT',
  payload: {
    variation: string, // e.g., "#abc"
    errors: number,
    decodedUrl: string,
    qrData: Uint8Array
  }
}

{ type: 'COMPLETE', payload: { totalIterations: number } }
{ type: 'ERROR', payload: { message: string } }
```

### Worker Performance Tips

1. **Minimize postMessage frequency**: Batch updates (e.g., every 1000 iterations)
2. **Use Transferable Objects**: For large arrays, transfer ownership instead of cloning
3. **Keep worker alive**: Don't create/destroy repeatedly, reuse single worker instance
4. **Graceful shutdown**: Respond to STOP message vs terminate() (preserves current results)

## State Management Patterns

### Centralized State Structure

```javascript
const appState = {
  // Input
  url: '',
  qrVersion: null,

  // Canvas
  userPixels: new Uint8Array(size * size), // 0=empty, 1=white, 2=black
  qrPixels: null,
  lockedPixels: [],

  // Optimization
  isOptimizing: false,
  currentIteration: 0,
  maxIterations: 1000000,

  // Results
  topResults: [], // [ { variation, errors, decodedUrl, qrData }, ... ]

  // UI
  selectedResult: null,
  showErrorVisualization: false
};
```

### Update Patterns

```javascript
// Pattern A: Direct mutation (if using Proxy)
state.url = 'https://example.com';

// Pattern B: Immutable updates (if using manual pub/sub)
setState({ ...state, url: 'https://example.com' });

// Pattern C: Action-based (Redux-style, overkill for this project)
dispatch({ type: 'SET_URL', payload: 'https://example.com' });
```

**Recommendation for this project:** Pattern A (Proxy-based) for simplicity in vanilla JS single file.

### Progress Tracking Pattern

```javascript
// In worker
let lastReportTime = Date.now();
for (let i = 0; i < maxIterations; i++) {
  // ... optimization logic ...

  // Throttle progress updates to every 100ms
  if (Date.now() - lastReportTime > 100) {
    self.postMessage({
      type: 'PROGRESS',
      payload: { iteration: i, percent: (i / maxIterations) * 100 }
    });
    lastReportTime = Date.now();
  }
}

// In main thread
worker.onmessage = (e) => {
  if (e.data.type === 'PROGRESS') {
    // Batch UI update
    scheduleProgressUpdate(e.data.payload);
  }
};
```

## Performance Benchmarks (Reference)

Based on research findings:

| Operation | Library/Method | Time |
|-----------|---------------|------|
| QR Generation (version 10) | Nayuki | ~5ms |
| QR Decoding | jsQR | ~10-50ms |
| QR Decoding | nimiq/qr-scanner | ~3-15ms (2-3x faster) |
| Canvas pixel read | getImageData | ~1ms |
| Canvas pixel write | putImageData | ~1-5ms |
| Worker postMessage (1KB) | Structured clone | <1ms |
| Worker postMessage (1MB) | Transfer | ~1ms |
| Worker postMessage (1MB) | Clone | ~50-100ms |

**Implications:**
- Decoding is the bottleneck (~10ms × 1M iterations = ~3 hours single-threaded)
- Use fastest decoder (nimiq/qr-scanner preferred)
- Transfer large arrays instead of cloning
- Target ~100 iterations/second as reasonable throughput

## Sources

**QR Code Libraries:**
- [Nayuki QR Code Generator](https://github.com/nayuki/QR-Code-generator) - High-quality multi-language QR library
- [QRious](https://github.com/neocotic/qrious) - Pure JavaScript QR generation
- [jsQR](https://github.com/cozmo/jsQR) - Pure JavaScript QR decoder
- [nimiq/qr-scanner](https://github.com/nimiq/qr-scanner) - High-performance QR scanner (2-3x faster)

**Web Workers & Performance:**
- [Web Worker Overview - web.dev](https://web.dev/learn/performance/web-worker-overview)
- [scheduler.yield() Deep Dive - Medium](https://medium.com/@tharunbalaji110/deep-dive-scheduler-yield-and-the-art-of-non-blocking-ui-updates-18b01241106a)
- [Optimize Long Tasks - web.dev](https://web.dev/articles/optimize-long-tasks)
- [Web Workers for Non-Blocking UI - DEV](https://dev.to/nikhilkumaran/web-workers-for-non-blocking-user-interface-i1a)

**Canvas Performance:**
- [Pixel Manipulation with Canvas - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Pixel_manipulation_with_canvas)
- [Optimize Image Processing - Sling Academy](https://www.slingacademy.com/article/optimize-image-processing-via-canvas-pixel-data-in-javascript/)

**State Management:**
- [State Management in Vanilla JS 2026 - Medium](https://medium.com/@chirag.dave/state-management-in-vanilla-js-2026-trends-f9baed7599de)
- [Build State Management with Vanilla JS - CSS-Tricks](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/)

**QR Code Theory:**
- [QR Code Error Correction - Scanova](https://scanova.io/qr-code-error-correction/)
- [QR Code Structure - Scanova](https://scanova.io/blog/qr-code-structure/)
- [System Design QR Code Generation - Medium](https://medium.com/@krutilin.sergey.ks/system-design-qr-code-generation-37c4262bbc55)

**UI Updates & Timing:**
- [requestAnimationFrame vs setTimeout - OpenReplay](https://blog.openreplay.com/requestanimationframe-settimeout-use/)
- [Optimizing JavaScript for High-Frequency Updates - Medium](https://javascript.plainenglish.io/what-i-learned-optimizing-javascript-for-high-frequency-ui-updates-42115e778e73)

---
*Architecture research for: Browser-based QR code generator with optimization*
*Researched: 2026-02-06*
