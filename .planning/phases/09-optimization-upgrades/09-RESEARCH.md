# Phase 9: Optimization Upgrades - Research

**Researched:** 2026-02-14
**Domain:** Multi-Worker Parallelization, Auto-Stop Conditions, Web Worker Coordination
**Confidence:** HIGH

## Summary

Phase 9 adds two optimization enhancements: (1) auto-stop when 5 results with 0 pixel errors are found (OPT-01), and (2) parallel multi-worker optimization for faster search throughput (OPT-02). The existing single-worker architecture (inline blob worker with START/STOP/PROGRESS/COMPLETE message protocol) provides a solid foundation. The auto-stop feature is straightforward — check `pixelDiff === 0` count in `updateTopResults()` and trigger stop when threshold reached. Multi-worker parallelization requires spawning N worker instances, distributing search work, coordinating stop signals, and merging results from all workers.

The current implementation already handles result accumulation across runs (`currentTopResults`, `existingTopResults`) and has proper sorting/deduplication logic. The key architectural decision is whether to use a worker pool with a coordinator pattern or simple parallel workers with main-thread coordination. Given the single-file constraint and existing architecture, **main-thread coordination is simpler and sufficient**.

**Primary recommendation:** Add auto-stop condition to main thread (check 5 perfect results after each PROGRESS), then convert single worker to worker pool (2-4 workers based on `navigator.hardwareConcurrency`). Main thread coordinates all workers: sends same config to all, merges results from all PROGRESS messages, broadcasts STOP to all when auto-stop or timeout triggers.

## Standard Stack

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Web Workers API | Native | Parallel background processing | Standard browser API, already in use |
| `navigator.hardwareConcurrency` | Native | Detect CPU core count | Reliable way to size worker pool, returns 4-16 on typical devices |
| Structured Clone Algorithm | Native | Worker message passing | Already in use for START_OPTIMIZATION messages |

### Supporting

| Technique | Purpose | When to Use |
|-----------|---------|-------------|
| Worker pool pattern | Manage multiple worker instances | OPT-02: Parallel optimization |
| Hash-based deduplication | Prevent duplicate results from workers | Result merging |
| Broadcast stop signal | Terminate all workers simultaneously | Auto-stop or user stop |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Main-thread coordination | SharedWorker coordinator | Adds complexity, single-file constraint makes SharedWorker difficult to inline |
| Fixed worker count (4) | Dynamic based on hardwareConcurrency | Dynamic is better — respects device capabilities |
| Hash-based deduplication | Time-based result windows | Hash comparison is simpler and exact |
| Multiple blob workers | Single worker with internal parallelism | Multiple workers is cleaner, existing worker code unchanged |

## Architecture Patterns

### Recommended Project Structure

```
<script>
  // ===== CONSTANTS =====
  const PERFECT_RESULT_THRESHOLD = 5;  // Auto-stop when this many 0-error results found
  
  // ===== WORKER POOL STATE =====
  let optimizationWorkers = [];        // Array of Worker instances
  let workerResultBuffers = [];        // Per-worker result arrays (for merging)
  const WORKER_COUNT = Math.min(navigator.hardwareConcurrency || 4, 8);
  
  // ===== EXISTING WORKER CODE =====
  const workerCode = `...`;  // Unchanged from current implementation
  
  // ===== COORDINATION FUNCTIONS =====
  function createWorkerPool() { ... }
  function startParallelOptimization() { ... }
  function mergeWorkerResults() { ... }
  function stopAllWorkers() { ... }
  function checkAutoStopCondition() { ... }
</script>
```

### Pattern 1: Auto-Stop Condition Check

**What:** Stop optimization when 5 results with 0 pixel errors are found
**When to use:** After each PROGRESS message updates results
**Example:**

```javascript
// Source: Native JavaScript, simple condition check
const PERFECT_RESULT_THRESHOLD = 5;

function checkAutoStopCondition(topResults) {
  if (!topResults || topResults.length === 0) return false;
  
  // Count results with 0 pixel errors
  const perfectCount = topResults.filter(r => r.pixelDiff === 0).length;
  
  return perfectCount >= PERFECT_RESULT_THRESHOLD;
}

// In worker message handler:
optimizationWorker.onmessage = function(e) {
  if (e.data.type === 'PROGRESS') {
    // ... existing UI updates ...
    
    // OPT-01: Check auto-stop condition
    if (checkAutoStopCondition(currentTopResults)) {
      console.log('Auto-stop: 5 perfect results found');
      stopOptimization();
    }
  }
};
```

### Pattern 2: Worker Pool Creation

**What:** Create multiple worker instances from same inline code
**When to use:** Initializing parallel optimization
**Example:**

```javascript
// Source: MDN Web Workers + existing blob pattern
const WORKER_COUNT = Math.min(navigator.hardwareConcurrency || 4, 8);

function createWorkerPool() {
  // Terminate any existing workers
  terminateWorkerPool();
  
  // Extract jsQR script (existing pattern)
  const jsQRScript = document.querySelectorAll("script")[1].textContent;
  const workerCode = createWorkerCode(jsQRScript);  // Existing function
  
  const blob = new Blob([workerCode], { type: 'application/javascript' });
  const workerUrl = URL.createObjectURL(blob);
  
  optimizationWorkers = [];
  for (let i = 0; i < WORKER_COUNT; i++) {
    const worker = new Worker(workerUrl);
    worker.workerId = i;  // Tag for debugging
    
    worker.onmessage = createWorkerMessageHandler(i);
    worker.onerror = (err) => console.error(`Worker ${i} error:`, err);
    
    optimizationWorkers.push(worker);
  }
  
  // Clean up blob URL after all workers created
  URL.revokeObjectURL(workerUrl);
  
  return optimizationWorkers;
}

function terminateWorkerPool() {
  optimizationWorkers.forEach(w => w.terminate());
  optimizationWorkers = [];
}
```

### Pattern 3: Result Merging from Multiple Workers

**What:** Combine results from all workers into single top-5 ranking
**When to use:** After receiving PROGRESS from any worker
**Example:**

```javascript
// Source: Adaptation of existing updateTopResults logic
function mergeWorkerResults(allWorkerResults) {
  // Flatten all worker results
  const combined = allWorkerResults.flat();
  
  // Deduplicate by hash (same hash = same QR configuration)
  const hashSet = new Set();
  const unique = combined.filter(r => {
    if (hashSet.has(r.hash)) return false;
    hashSet.add(r.hash);
    return true;
  });
  
  // Sort: decodable first, then by pixelDiff ascending (existing logic)
  unique.sort((a, b) => {
    if (a.decodable !== b.decodable) {
      return b.decodable ? 1 : -1;
    }
    return a.pixelDiff - b.pixelDiff;
  });
  
  // Return top 5
  return unique.slice(0, 5);
}

// Per-worker buffers for accumulation
let workerResultBuffers = [];

function createWorkerMessageHandler(workerId) {
  return function(e) {
    if (e.data.type === 'PROGRESS') {
      // Update this worker's buffer
      workerResultBuffers[workerId] = e.data.topResults;
      
      // Merge all worker results
      currentTopResults = mergeWorkerResults(workerResultBuffers);
      
      // Update accumulated stats
      totalAttempts = workerResultBuffers.reduce((sum, buf, idx) => {
        // Need to track per-worker attempt counts
        return sum + (workerAttemptCounts[idx] || 0);
      }, 0);
      
      // Update UI with merged results
      updateTopResults(currentTopResults);
      
      // Check auto-stop
      if (checkAutoStopCondition(currentTopResults)) {
        stopAllWorkers();
      }
    }
    // ... COMPLETE handling ...
  };
}
```

### Pattern 4: Broadcast Stop to All Workers

**What:** Send STOP_OPTIMIZATION to all workers simultaneously
**When to use:** Auto-stop condition met, timeout reached, or user clicks Stop
**Example:**

```javascript
// Source: Simple iteration pattern
function stopAllWorkers() {
  optimizationWorkers.forEach(worker => {
    worker.postMessage({ type: 'STOP_OPTIMIZATION' });
  });
}

// Modify existing stopOptimization:
function stopOptimization() {
  stopAllWorkers();
  // ... rest of cleanup ...
}
```

### Pattern 5: Parallel Start with Same Configuration

**What:** Send identical START_OPTIMIZATION to all workers
**When to use:** Beginning parallel optimization run
**Example:**

```javascript
// Source: Adaptation of existing startOptimization
function startParallelOptimization(timeout = 10000) {
  if (!currentURL || !currentVersion || !paintGrid) {
    console.error('Cannot start optimization: missing URL, version, or paint grid');
    return;
  }
  
  createWorkerPool();
  
  // Reset per-worker buffers
  workerResultBuffers = new Array(WORKER_COUNT).fill([]);
  workerAttemptCounts = new Array(WORKER_COUNT).fill(0);
  
  const message = {
    type: 'START_OPTIMIZATION',
    baseURL: currentURL,
    version: currentVersion,
    paintedPixels: Array.from(paintGrid.cells),
    functionMask: Array.from(createFunctionPatternMask(currentVersion)),
    timeout: timeout,
    existingTopResults: currentTopResults,  // Each worker starts with existing results
  };
  
  // Send to all workers
  optimizationWorkers.forEach(worker => {
    worker.postMessage(message);
  });
}
```

### Pattern 6: Completion Coordination

**What:** Wait for all workers to complete before finalizing
**When to use:** Handling COMPLETE messages from multiple workers
**Example:**

```javascript
// Source: Coordination pattern for parallel workers
let completedWorkerCount = 0;
let totalWorkerAttempts = 0;
let totalWorkerDecodable = 0;

function createWorkerMessageHandler(workerId) {
  return function(e) {
    if (e.data.type === 'PROGRESS') {
      // ... as above ...
    } else if (e.data.type === 'COMPLETE') {
      completedWorkerCount++;
      totalWorkerAttempts += e.data.attempts;
      totalWorkerDecodable += e.data.decodableCount;
      
      // Final merge from this worker
      workerResultBuffers[workerId] = e.data.topResults;
      currentTopResults = mergeWorkerResults(workerResultBuffers);
      
      // All workers done?
      if (completedWorkerCount === WORKER_COUNT) {
        handleAllWorkersComplete({
          attempts: totalWorkerAttempts,
          decodableCount: totalWorkerDecodable,
          topResults: currentTopResults
        });
      }
    }
  };
}

function handleAllWorkersComplete(aggregatedData) {
  // Existing handleSearchComplete logic with aggregated data
  handleSearchComplete(aggregatedData);
}
```

### Anti-Patterns to Avoid

- **Creating new workers per run without termination:** Memory leak; always terminate previous pool before creating new one
- **Using SharedArrayBuffer for result sharing:** Requires cross-origin isolation headers; overkill for this use case
- **Trying to coordinate workers via worker-to-worker communication:** Complex and unnecessary; main thread coordination is simpler
- **Ignoring `navigator.hardwareConcurrency`:** Spawning too many workers degrades performance; respect device capabilities
- **Not deduplicating results:** Same hash can be found by multiple workers; merge must filter duplicates

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Worker count detection | Fixed magic number | `navigator.hardwareConcurrency` | Respects device capabilities (2-16 cores) |
| Result deduplication | Complex merge sort | Simple hash Set filter | Hash uniquely identifies QR configuration |
| Stop coordination | Custom lock mechanisms | Simple broadcast + flag | Workers already check `shouldStop` flag |
| Progress aggregation | Complex state machine | Sum of per-worker counts | Workers are independent, simple accumulation |

**Key insight:** The existing single-worker architecture is already well-designed. Multi-worker is mostly about creating N instances and merging their outputs. The worker code itself (`createOptimizationWorker`) doesn't need modification — the same code runs in parallel.

## Common Pitfalls

### Pitfall 1: Race Condition in Result Merging

**What goes wrong:** Results appear out of order or duplicates slip through during rapid PROGRESS updates

**Why it happens:** Multiple workers send PROGRESS at similar times; merge function called concurrently

**How to avoid:** 
- Store per-worker results in indexed array (not one shared array)
- Always re-merge from all worker buffers (idempotent operation)
- UI update is triggered after merge completes

**Warning signs:** Same hash appearing twice in top-5, results flickering

### Pitfall 2: Memory Accumulation with Worker Pool

**What goes wrong:** Memory usage grows significantly with each optimization run

**Why it happens:** Workers accumulate internal state; Blob URLs not revoked; ImageData buffers retained

**How to avoid:**
- Call `URL.revokeObjectURL(workerUrl)` after all workers created
- Call `terminateWorkerPool()` before creating new pool
- Clear `workerResultBuffers` at start of each run
- Don't store full ImageData in long-term state (extract only needed display data)

**Warning signs:** Tab memory in DevTools Task Manager grows with each "Run again" click

### Pitfall 3: Workers Not All Stopping on Auto-Stop

**What goes wrong:** Some workers continue running after auto-stop condition met

**Why it happens:** STOP message sent but worker is between batches; `shouldStop` flag not checked

**How to avoid:** The existing worker design already handles this correctly:
```javascript
// Existing worker code (line 15265):
if (shouldStop || Date.now() >= timeoutEnd) {
  sendComplete();
  return;
}
```
The `setTimeout(..., 0)` yield between batches ensures `shouldStop` is checked regularly.

**Warning signs:** Attempt counter continues incrementing after "Done" displayed

### Pitfall 4: Inconsistent Total Attempt Count

**What goes wrong:** Displayed attempt count doesn't match sum of worker attempts

**Why it happens:** Attempt count updated on PROGRESS but reset incorrectly on COMPLETE

**How to avoid:**
- Track `totalAttempts` as cumulative sum across all workers
- On PROGRESS: recalculate from all worker counts
- On COMPLETE: add final worker count to cumulative total
- Don't reset on each PROGRESS (existing bug pattern)

**Warning signs:** Attempt count jumps or resets during search

### Pitfall 5: Not Accounting for `existingTopResults` Correctly

**What goes wrong:** Results from previous runs lost when starting parallel optimization

**Why it happens:** Each worker receives `existingTopResults` but main thread doesn't preserve them in merge

**How to avoid:**
- Include `currentTopResults` (which already has previous run results) in each worker's `existingTopResults`
- On merge, include main thread's `currentTopResults` in the merge input
- Workers already handle this correctly internally

**Warning signs:** Running "Run again" clears good results from previous runs

### Pitfall 6: Auto-Stop Triggering Too Early

**What goes wrong:** Auto-stop fires before results are actually displayed/confirmed

**Why it happens:** Check happens on PROGRESS before UI has rendered merged results

**How to avoid:**
- Check auto-stop AFTER merging and updating `currentTopResults`
- Use the merged results for the check, not per-worker results
- The 5 perfect results must be in the merged top-5, not distributed across workers

**Warning signs:** Search stops but displayed results show fewer than 5 perfect results

## Code Examples

### Complete Auto-Stop Implementation

```javascript
// Source: Native JavaScript, derived from requirements
const PERFECT_RESULT_THRESHOLD = 5;

/**
 * Check if auto-stop condition is met
 * @param {Array} topResults - Current merged top results
 * @returns {boolean} True if 5 or more results have 0 pixel errors
 */
function checkAutoStopCondition(topResults) {
  if (!topResults || topResults.length < PERFECT_RESULT_THRESHOLD) {
    return false;
  }
  
  const perfectCount = topResults.filter(r => 
    r.decodable && r.pixelDiff === 0
  ).length;
  
  return perfectCount >= PERFECT_RESULT_THRESHOLD;
}

// Integration point - in worker message handler:
if (e.data.type === 'PROGRESS') {
  // ... existing code ...
  
  // Merge results from all workers
  currentTopResults = mergeWorkerResults(workerResultBuffers);
  updateTopResults(currentTopResults);
  
  // OPT-01: Check auto-stop
  if (checkAutoStopCondition(currentTopResults)) {
    console.log('Auto-stop triggered: 5 perfect results found');
    stopAllWorkers();
    // Will trigger COMPLETE from each worker
  }
}
```

### Complete Worker Pool Implementation

```javascript
// Source: MDN Web Workers + existing architecture
let optimizationWorkers = [];
let workerResultBuffers = [];
let workerAttemptCounts = [];
let completedWorkerCount = 0;

const WORKER_COUNT = Math.min(navigator.hardwareConcurrency || 4, 8);

function createWorkerPool() {
  terminateWorkerPool();
  
  const jsQRScript = document.querySelectorAll("script")[1].textContent;
  const workerCode = buildWorkerCode(jsQRScript);  // Existing createOptimizationWorker logic
  
  const blob = new Blob([workerCode], { type: 'application/javascript' });
  const workerUrl = URL.createObjectURL(blob);
  
  optimizationWorkers = [];
  workerResultBuffers = [];
  workerAttemptCounts = [];
  
  for (let i = 0; i < WORKER_COUNT; i++) {
    const worker = new Worker(workerUrl);
    worker.workerId = i;
    
    worker.onmessage = createWorkerMessageHandler(i);
    worker.onerror = (err) => console.error(`Worker ${i} error:`, err);
    
    optimizationWorkers.push(worker);
    workerResultBuffers.push([]);
    workerAttemptCounts.push(0);
  }
  
  URL.revokeObjectURL(workerUrl);
  return optimizationWorkers;
}

function terminateWorkerPool() {
  optimizationWorkers.forEach(w => w.terminate());
  optimizationWorkers = [];
  workerResultBuffers = [];
  workerAttemptCounts = [];
}

function createWorkerMessageHandler(workerId) {
  return function(e) {
    if (e.data.type === 'PROGRESS') {
      workerResultBuffers[workerId] = e.data.topResults;
      workerAttemptCounts[workerId] = e.data.attempts;
      
      // Merge and update
      currentTopResults = mergeWorkerResults(workerResultBuffers);
      const totalAttempts = workerAttemptCounts.reduce((a, b) => a + b, 0);
      
      updateSearchStats(totalAttempts, e.data.elapsed);
      updateTopResults(currentTopResults);
      
      // Check auto-stop
      if (checkAutoStopCondition(currentTopResults)) {
        stopAllWorkers();
      }
      
    } else if (e.data.type === 'COMPLETE') {
      completedWorkerCount++;
      workerResultBuffers[workerId] = e.data.topResults;
      workerAttemptCounts[workerId] = e.data.attempts;
      
      if (completedWorkerCount >= WORKER_COUNT) {
        finalizeParallelOptimization();
      }
    }
  };
}

function mergeWorkerResults(buffers) {
  const combined = buffers.flat();
  
  // Deduplicate by hash
  const seen = new Set();
  const unique = combined.filter(r => {
    if (seen.has(r.hash)) return false;
    seen.add(r.hash);
    return true;
  });
  
  // Sort: decodable first, then pixelDiff ascending
  unique.sort((a, b) => {
    if (a.decodable !== b.decodable) return b.decodable ? 1 : -1;
    return a.pixelDiff - b.pixelDiff;
  });
  
  return unique.slice(0, 5);
}

function stopAllWorkers() {
  optimizationWorkers.forEach(w => {
    w.postMessage({ type: 'STOP_OPTIMIZATION' });
  });
}

function startParallelOptimization(timeout = 10000) {
  if (!currentURL || !currentVersion || !paintGrid) {
    console.error('Cannot start: missing URL, version, or paintGrid');
    return;
  }
  
  createWorkerPool();
  completedWorkerCount = 0;
  
  const message = {
    type: 'START_OPTIMIZATION',
    baseURL: currentURL,
    version: currentVersion,
    paintedPixels: Array.from(paintGrid.cells),
    functionMask: Array.from(createFunctionPatternMask(currentVersion)),
    timeout: timeout,
    existingTopResults: currentTopResults,
  };
  
  optimizationWorkers.forEach(w => w.postMessage(message));
}

function finalizeParallelOptimization() {
  const totalAttempts = workerAttemptCounts.reduce((a, b) => a + b, 0);
  currentTopResults = mergeWorkerResults(workerResultBuffers);
  
  handleSearchComplete({
    attempts: totalAttempts,
    topResults: currentTopResults,
    // decodableCount would need similar aggregation
  });
}
```

### Worker Count Detection

```javascript
// Source: MDN Navigator.hardwareConcurrency
/**
 * Get optimal worker count based on device capabilities
 * @returns {number} Number of workers to spawn (2-8)
 */
function getOptimalWorkerCount() {
  const cores = navigator.hardwareConcurrency || 4;
  
  // Use cores - 1 to leave headroom for main thread, minimum 2
  // Cap at 8 to avoid diminishing returns
  return Math.min(Math.max(cores - 1, 2), 8);
}

// Usage:
const WORKER_COUNT = getOptimalWorkerCount();
console.log(`Using ${WORKER_COUNT} optimization workers`);
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Single worker | Worker pool based on core count | Phase 9 | Linear throughput scaling with cores |
| Manual stop only | Auto-stop on 5 perfect results | Phase 9 | Faster iteration when optimal solutions exist |
| Fixed 4 workers | Dynamic via hardwareConcurrency | 2016+ (API available) | Respects device capabilities |

**Deprecated/outdated:**
- `navigator.hardwareConcurrency` fallback of 4 is conservative but safe
- SharedArrayBuffer-based coordination: requires COOP/COEP headers, not needed here

## Open Questions

1. **Optimal worker count formula**
   - What we know: `hardwareConcurrency - 1` leaves room for main thread
   - What's unclear: Whether all cores should be used for CPU-bound QR generation
   - Recommendation: Start with `min(cores - 1, 8)` and benchmark; can tune later

2. **Progress update frequency with multiple workers**
   - What we know: Each worker sends PROGRESS every batch (~25 iterations)
   - What's unclear: Will N workers sending N PROGRESS messages cause UI jank?
   - Recommendation: Current batch size of 25 should be fine; if jank observed, increase batch size or throttle UI updates

3. **Should perfect result count include non-decodable?**
   - What we know: Requirement says "5 results with 0 pixel errors"
   - What's unclear: Does "0 pixel errors" imply decodable?
   - Recommendation: Yes, require `decodable && pixelDiff === 0` — a non-decodable result isn't useful even with 0 errors

4. **Memory overhead of worker pool**
   - What we know: Each worker has ~1000 lines of QR code plus jsQR
   - What's unclear: Is 8x duplication of code significant?
   - Recommendation: Worker code is <100KB; 8x = <1MB; acceptable on modern devices

## Sources

### Primary (HIGH confidence)

- MDN Web Workers API - Worker creation, message passing, termination
- MDN `navigator.hardwareConcurrency` - CPU core detection
- Existing codebase implementation:
  - `createOptimizationWorker()` (line 14288-15357) - Worker creation pattern
  - `optimizationWorker.onmessage` (line 15322-15350) - Message handling
  - `updateTopResults()` (line 15701-15878) - Result display logic
  - `handleSearchComplete()` (line 15884-15917) - Completion handling
- Phase 3 RESEARCH.md - Worker architecture decisions, communication protocol

### Secondary (MEDIUM confidence)

- Chrome DevTools documentation - Worker memory monitoring
- web.dev Web Worker performance articles - Pool patterns

### Tertiary (LOW confidence)

- None - all patterns are well-established browser APIs and existing codebase patterns

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - Native browser APIs, no external dependencies
- Architecture: HIGH - Simple extension of existing worker pattern
- Pitfalls: HIGH - Based on existing implementation review and common concurrency issues
- Auto-stop logic: HIGH - Simple condition check, straightforward implementation

**Research date:** 2026-02-14
**Valid until:** 2026-03-14 (30 days - stable browser APIs)
