# Pitfalls Research

**Domain:** Reed-Solomon Error Extraction for QR Code Measurement
**Researched:** 2026-02-16
**Confidence:** HIGH

## Critical Pitfalls

### Pitfall 1: jsQR Does Not Expose RS Error Counts

**What goes wrong:**
Developers assume they can get Reed-Solomon error correction counts from jsQR's return value. The library returns only `{ data, binaryData, chunks, version, location }` — zero information about how many codewords were corrected. The RS decode function internally calculates `errorLocations.length` (the exact metric we need) but discards it, only returning corrected bytes.

**Why it happens:**
jsQR is designed for decoding, not analysis. The RS decoder at line 10754 (`function decode(bytes, twoS)`) finds error locations but only uses them to fix the data — it never exposes the count. Developers don't realize library modification is required until after integration.

**How to avoid:**
- **Accept that jsQR source modification is required** — this is unavoidable for our use case
- Modify the RS `decode()` function (line 10754-10808) to return `{ correctedBytes, errorCount: errorLocations.length }` instead of just `correctedBytes`
- Propagate error count up through `decodeMatrix()` → `decode()` → final return value
- Alternatively, fork jsQR entirely and add error metrics throughout

**Warning signs:**
- Searching jsQR API for "error" or "correction" returns nothing useful
- Assuming "version" in return value contains error info (it doesn't — it's QR version number)
- Planning to "wrap" jsQR without modifying source

**Phase to address:**
Phase 1 (jsQR Modification) — The very first task must be modifying jsQR internals

---

### Pitfall 2: Confusing Module Errors with RS Codeword Errors

**What goes wrong:**
A developer corrupts 10 QR modules and expects `errorCount: 10`. But RS correction operates on **codewords** (8-bit bytes), not modules. One module flip might corrupt zero codewords (if the module was already wrong), one codeword, or affect multiple data blocks differently. The relationship is complex, not 1:1.

**Why it happens:**
QR codes are modular (black/white pixels) but RS correction operates on codewords extracted after:
1. Module reading (applying data mask)
2. De-interleaving across data blocks
3. Per-block RS decoding

A single corrupted module might affect one bit in one codeword, which RS can often correct, or it might destroy a codeword entirely, or the corruption might cancel out.

**How to avoid:**
- **Don't promise "modules corrupted = errors"** — measure and display separately
- Track both: `modulesCorrupted` (before decode) and `rsErrorsCorrected` (from decoder)
- Accept that the relationship is probabilistic, not deterministic
- UI should show: "X modules modified, Y RS corrections needed" — these are different metrics
- Document that RS errors are per-block: a QR may have multiple blocks, each with its own correction count

**Warning signs:**
- Tests assume `modulesCorrupted === rsErrors`
- Confusion when 5 modules flipped causes 0, 1, or 3 RS errors
- Not understanding why same pixel corruption pattern gives different error counts with different data

**Phase to address:**
Phase 2 (Metrics Integration) — Clear separation of module-level vs. codeword-level metrics

---

### Pitfall 3: Error Count Exceeds Correction Capacity But Decode "Succeeds"

**What goes wrong:**
The RS decoder reports N errors corrected, but N exceeds the theoretical correction capacity (t = ecCodewords / 2). This happens because:
1. The decode succeeded despite being at the edge of correctability
2. Decoder found fewer errors than expected (error locations collided)
3. **Critically:** A wrong decode occurred — the RS algorithm "corrected" to garbage

**Why it happens:**
Reed-Solomon can **detect** 2t errors but only **correct** t errors. When errors exceed t, RS may:
- Fail cleanly (return null — good)
- "Correct" to wrong data (silent failure — very bad)
- Correct correctly by luck (rare)

jsQR's RS decoder returns `null` when it detects uncorrectable errors (line 10784, 10788, 10800), but edge cases slip through.

**How to avoid:**
- **Always validate decoded data** — check URL format, expected content patterns
- Compare `rsErrorsCorrected` against theoretical max: `ecCodewordsPerBlock / 2` per block
- Flag warning if error count approaches capacity (>75% of max)
- Cross-check: encode the decoded result and compare codewords — any mismatch means wrong decode
- For optimization: prefer results with LOWER error counts, not just "decodable"

**Warning signs:**
- Error counts reported near or above correction capacity
- Decoded URLs that are slightly corrupted versions of the original
- Different decoders giving different decode results for same QR

**Phase to address:**
Phase 2 (Metrics Integration) — Add validation layer after decode

---

### Pitfall 4: Modifying Inlined jsQR Breaks Worker

**What goes wrong:**
Developer modifies the main-page jsQR to return error counts. But the Worker has its own COPY of jsQR (extracted from `document.querySelectorAll("script")[1].textContent` at line 14398). The worker's jsQR remains unmodified, so:
- Main thread decode shows error counts
- Worker decode shows nothing (old API)
- Metrics disagree between main thread and workers

**Why it happens:**
The architecture copies jsQR source as a string into each worker via Blob URL. Modifying the inlined script tag doesn't automatically propagate — the worker re-extracts the same source code position, but now the line numbers/structure may have shifted.

**How to avoid:**
- **Understand the architecture:** jsQR source lives in script tag #1, worker extracts it dynamically
- Modify jsQR in ONE place (the `<script>` tag) — the worker will get the modified version automatically
- Verify after modification: console.log the first 200 chars of extracted jsQR in worker to confirm
- Test worker decode path separately from main thread decode
- If restructuring jsQR significantly, update the extraction selector (`document.querySelectorAll("script")[1]`)

**Warning signs:**
- Main thread shows RS errors, worker results don't
- Worker throws "undefined" errors on new fields you added
- `createOptimizationWorker()` still references old API

**Phase to address:**
Phase 1 (jsQR Modification) — Verify worker gets modified source

---

### Pitfall 5: Returning Error Count Only on Success, Not Failure

**What goes wrong:**
Developer modifies RS decode to return error count, but only when decode succeeds. When decode fails (too many errors), they return `null` — losing valuable information. For optimization, we need to know "how close was this to working?" even for failures.

**Why it happens:**
The existing jsQR RS decoder returns `null` on failure, so developers follow the same pattern. But for optimization ranking, we want: "This pattern needed 15 corrections but capacity is 13" — that's better than one needing 50 corrections.

**How to avoid:**
- **Return rich objects, not just bytes:** `{ success: boolean, correctedBytes, errorCount, capacity, reason }`
- On failure, still return `{ success: false, errorCount: detectedErrors, capacity: t, reason: 'exceeded' }`
- Track failure reasons: "too many errors", "error locator failed", "invalid position"
- For ranking: failed decodes with errorCount close to capacity beat those far over capacity

**Warning signs:**
- All failed decodes reported as "errorCount: undefined" or "errorCount: 0"
- Can't distinguish between "1 error over capacity" and "100 errors over capacity"
- Optimization can't learn which direction moves toward success

**Phase to address:**
Phase 1 (jsQR Modification) — Design rich return type from the start

---

### Pitfall 6: Per-Block Error Counts Lost in Aggregation

**What goes wrong:**
A QR code has multiple RS blocks (e.g., Version 5-H has 4 blocks). Developer sums error counts: "12 errors total." But one block had 8 errors (over its 7-error capacity = failed) while others had 4 total (fine). The aggregate "12" hides that this QR is fundamentally broken.

**Why it happens:**
Line 3214-3229 in jsQR loops through `dataBlocks` calling `reedsolomon_1.decode()` per block. Developers sum errors across blocks, losing per-block information critical for understanding failure modes.

**How to avoid:**
- **Track per-block metrics:** `{ blocks: [{ errorCount, capacity }...], totalErrors, minBlockCapacity }`
- Check EACH block's error count against its capacity
- Display "Block 1: 3/7, Block 2: 5/7, Block 3: 8/7 (FAILED)"
- For optimization: a QR with all blocks under 50% capacity is better than one with one block at 99%

**Warning signs:**
- Only seeing aggregate error count
- "12 errors total" when max capacity is 28 (seems fine, but one block is over)
- Can't explain why QR with "fewer total errors" fails while one with "more" succeeds

**Phase to address:**
Phase 2 (Metrics Integration) — Preserve block-level granularity

---

### Pitfall 7: Minified jsQR Breaks Modification

**What goes wrong:**
The inlined jsQR is a webpack bundle with preserved structure (modules 0-12), but modifying one section causes cascading issues:
- Variable names are minified (`__webpack_require__`, `exports.default`)
- Module boundaries must be preserved
- Adding new fields to return objects breaks downstream consumers

**Why it happens:**
jsQR was webpack-bundled before inlining. The source is readable but fragile. Developers modify `decode()` return value but forget to update callers in other modules.

**How to avoid:**
- **Trace the full call chain before modifying:**
  1. `reedsolomon_1.decode()` (module 9, line 10754) — returns correctedBytes
  2. `decodeMatrix()` (module 5, line 3190) — calls RS decode per block
  3. `decode(matrix)` (module 5, line 3240) — returns result or null
  4. `scan()` (module 3, line 2474) — returns final result object
  5. `jsQR()` (module 3, line 2521) — public API
- Update EVERY level to propagate new fields
- Test module boundaries: each `__webpack_require__` is a separate module
- Consider adding new fields as OPTIONAL to avoid breaking existing code

**Warning signs:**
- "TypeError: Cannot read property 'errorCount' of undefined"
- Only innermost function returns error count, outer layers don't
- `jsQR()` return value unchanged despite internal modifications

**Phase to address:**
Phase 1 (jsQR Modification) — Map complete call chain first

---

### Pitfall 8: Worker Message Serialization Loses Error Details

**What goes wrong:**
Worker calculates RS error counts, but when posting results via `postMessage()`, complex objects get flattened or lost. Developer expects `result.rsErrors` in main thread, gets `undefined`.

**Why it happens:**
`postMessage()` uses structured clone algorithm. Most objects survive, but:
- Function properties are stripped
- Symbol keys are lost
- Circular references fail
- Uint8ClampedArray serializes fine, but if nested in complex objects, issues arise

Current worker message format (line 15373+) posts simple objects like `{ type, data, decodable, pixelDiff }`. Adding nested error info may require format changes.

**How to avoid:**
- **Use flat message structures:** `{ rsErrors: 5, rsCapacity: 10, blockErrors: [3, 2] }`
- Avoid nested objects deeper than 2 levels in messages
- Convert Uint8ClampedArray to regular Array before posting if issues arise
- Log message on both sides during development to verify serialization
- Update message handler (`createWorkerMessageHandler`, line 15458) to expect new fields

**Warning signs:**
- Worker console shows correct error count, main thread shows undefined
- `postMessage` silently drops fields
- "DataCloneError" in console

**Phase to address:**
Phase 3 (Worker Integration) — Design message format before implementation

---

### Pitfall 9: Syndrome Calculation Shows "0 Errors" for Corrupted QR

**What goes wrong:**
A corrupted QR decodes successfully, RS reports 0 errors corrected. Developer assumes "0 errors = no corruption." But the syndrome check (line 10761-10769) only detects errors that cause syndrome non-zero. Some corruption patterns affect modules but not codewords (cancel out via masking), or the syndrome happens to evaluate to zero anyway.

**Why it happens:**
RS syndrome is calculated as polynomial evaluation. If `poly.evaluateAt(generator^s) == 0` for all s, no error is detected — even if modules were physically changed. This happens when:
- Changed modules were in function patterns (not data)
- Changes canceled out in codeword extraction
- By pure mathematical coincidence (extremely rare for large changes)

**How to avoid:**
- **Track module-level changes separately** — don't rely solely on RS error count
- UI: "X modules changed, Y RS errors detected" — both numbers matter
- For optimization: use module diff as primary metric, RS errors as secondary validation
- Remember: RS error count of 0 doesn't mean "no corruption," it means "no codeword-level impact"

**Warning signs:**
- Corrupting 50 modules shows "0 RS errors" because they were all function patterns
- Assuming RS errors = pixels changed
- Not understanding why "working" QRs show 0 errors even with visible changes

**Phase to address:**
Phase 2 (Metrics Integration) — Keep module diff metric alongside RS errors

---

### Pitfall 10: Testing Only Happy Path (Decode Success)

**What goes wrong:**
All tests verify RS error extraction when decode succeeds. Edge cases untested:
- Decode fails (too many errors) — does error count propagate?
- Decode returns null early (malformed QR) — no error count available
- Single-block vs multi-block QRs behave differently

**Why it happens:**
"It works" testing: developers test one QR version/size/corruption level, confirm it works, move on. RS edge cases require testing near capacity limits.

**How to avoid:**
- **Test matrix:**
  | Scenario | Expected Behavior |
  |----------|-------------------|
  | Perfect QR | errorCount: 0 |
  | 1 error | errorCount: 1, success |
  | 50% capacity errors | errorCount: N, success |
  | 99% capacity errors | errorCount: N, success |
  | 101% capacity errors | errorCount: N, success: false |
  | Far over capacity | success: false, reason: 'exceeded' |
  | Invalid QR structure | null (no RS phase reached) |
- Test multiple QR versions (single-block v1-v2, multi-block v3+)
- Test all error correction levels (L/M/Q/H have different capacities)

**Warning signs:**
- Only testing with QR Version 1 (single block, simple case)
- Never testing failure cases
- "Works in demo" but fails with real-world varied QRs

**Phase to address:**
Phase 1 (jsQR Modification) — Build test suite covering edge cases

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Hardcoding error extraction for single block | Faster initial implementation | Breaks on QR version 3+ (multi-block) | Never — multi-block support is required |
| Returning aggregate error count only | Simpler API | Loses per-block detail needed for optimization | MVP only if single-block QRs guaranteed |
| Modifying jsQR without upstream tracking | No fork maintenance | Can't upgrade jsQR, bugs harder to track | Acceptable if jsQR is "frozen" dependency |
| Using console.log for error tracking | Fast debugging | Production code with debug noise, breaks workers | Development only |
| Skipping failure case metrics | Less code | Can't rank failed attempts for optimization | Never — failure ranking is critical for search |
| Copying jsQR extraction code instead of abstracting | Works immediately | Two places to update when jsQR changes | Never — creates sync bugs |

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| jsQR RS decoder modification | Only modifying `decode()`, not updating callers | Trace full chain: RS decode → decodeMatrix → decode → scan → jsQR |
| Worker jsQR extraction | Assuming worker automatically gets changes | Verify: log extracted source in worker to confirm modifications present |
| postMessage with error metrics | Nesting error data deeply | Use flat structure: `{ rsErrors, rsCapacity, blockDetails: [...] }` |
| Multi-block QR handling | Summing errors without checking per-block capacity | Track per-block: `blocks: [{ count, capacity, success }]` |
| Return type modification | Adding fields to existing returns | Wrap in new structure: `{ originalReturn, errorMetrics }` to avoid breaking existing code |

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Logging error details on every decode | Worker slow, console flooded | Log only on RESULT messages or summary, not per-attempt | >100 decodes/second |
| Creating new error tracking objects per decode | Memory growth, GC pauses | Reuse tracking structure, reset values | >10,000 optimization iterations |
| Deep cloning error arrays for postMessage | High serialization overhead | Send primitive counts, not full arrays | Large multi-block QRs (version 20+) |
| Storing full error history | Memory exhaustion | Store only top N results, discard rest | Long-running optimization (>30 seconds) |

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Exposing internal RS state in UI | Information leakage about QR structure | Only show user-relevant metrics (error count, capacity percentage) |
| Not validating decoded data after RS | RS may "correct" to garbage | Validate decoded content format matches expected pattern |
| Trusting worker-reported error counts | Worker could be manipulated | Not a real risk for browser-local tool, but don't trust for server-side |

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Showing raw error count without context | "5 errors" means nothing without knowing capacity | Show "5/10 errors used (50%)" |
| Not indicating multi-block breakdown | User doesn't understand why "low total errors" QR fails | Show per-block health bars or breakdown |
| RS errors only on success | User can't understand failed attempts | Show "Failed: 15 errors detected, capacity is 10" |
| Hiding module diff vs RS error distinction | User confuses two different concepts | Clearly label: "Pixels changed: 50" vs "RS corrections: 3" |

## "Looks Done But Isn't" Checklist

- [ ] **RS error extraction:** Often only extracts on success — verify failure cases return error count too
- [ ] **Multi-block handling:** Often tested only on Version 1 — verify Version 5+ QRs work correctly
- [ ] **Worker propagation:** Often main thread works but worker doesn't — test worker decode path explicitly
- [ ] **Call chain complete:** Often only innermost function modified — verify jsQR() public API returns errors
- [ ] **Message serialization:** Often fields lost in postMessage — log both sides to confirm
- [ ] **Capacity calculation:** Often hardcoded — verify capacity is dynamic based on QR version/level
- [ ] **Error count validation:** Often trusts decoder completely — verify against theoretical capacity

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| jsQR modification incomplete (missing call chain) | LOW | Map call chain, update remaining functions, re-test |
| Worker not getting modified jsQR | LOW | Fix extraction selector, verify with console log |
| Aggregate-only error count (no per-block) | MEDIUM | Refactor return type throughout chain, update callers |
| Message format doesn't include errors | LOW | Update message structure, update handler |
| Only success cases return error count | MEDIUM | Refactor RS decode to always return metrics, handle in callers |
| Tests missing edge cases | LOW | Add test cases, may find existing bugs |

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| jsQR doesn't expose RS errors | Phase 1 (jsQR Modification) | jsQR() return includes errorCount field |
| Module vs codeword confusion | Phase 2 (Metrics Integration) | UI shows both metrics separately |
| Error count exceeds capacity | Phase 2 (Metrics Integration) | Validation warns when errors approach limit |
| Worker doesn't get modified jsQR | Phase 1 (jsQR Modification) | Worker decode returns error count |
| Only success returns error count | Phase 1 (jsQR Modification) | Failed decodes include error metrics |
| Per-block info lost | Phase 2 (Metrics Integration) | Block-level breakdown available |
| Minified jsQR breaks | Phase 1 (jsQR Modification) | All call chain points updated |
| Message serialization loses data | Phase 3 (Worker Integration) | postMessage includes error metrics |
| Syndrome shows 0 for changed QR | Phase 2 (Metrics Integration) | Module diff tracked separately |
| Only testing happy path | Phase 1 (jsQR Modification) | Test suite covers failure cases |

## Sources

**jsQR Source Analysis:**
- Inlined jsQR library in `/workspace/index.html` lines 2079-13147
- RS decoder function: lines 10754-10808
- `decodeMatrix()`: lines 3190-3238
- `decode(matrix)`: lines 3240-3258
- Public `jsQR()`: lines 2521-2552
- Worker extraction: lines 14396-14400

**Reed-Solomon Theory:**
- [Reed-Solomon codes for coders - Wikiversity](https://en.wikiversity.org/wiki/Reed%E2%80%93Solomon_codes_for_coders)
- [Error detection and correction - Wikipedia](https://en.wikipedia.org/wiki/Error_detection_and_correction)
- Correction capacity: t = ecCodewordsPerBlock / 2 (can correct t codewords)

**QR Code Structure:**
- QR version info structure: lines 10817-13147
- Error correction levels: L=0, M=1, Q=2, H=3 (mapped in FORMAT_INFO_TABLE)
- Multi-block structure: Version 3+ uses multiple RS blocks

**Worker Architecture:**
- Worker code generation: `createOptimizationWorker()` line 14396
- Blob URL creation: line 15556-15557
- Message handler: `createWorkerMessageHandler()` line 15458

---
*Pitfalls research for: Reed-Solomon Error Extraction for QR Code Measurement*
*Researched: 2026-02-16*
