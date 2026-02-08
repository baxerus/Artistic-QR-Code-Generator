# Phase 3: Hash Optimization Loop - Context

**Gathered:** 2026-02-08
**Status:** Ready for planning

<domain>
## Phase Boundary

Automated search engine that tries URL hash fragment variants, generates QR codes with the user's painted pattern overlaid, and tracks which variants produce the fewest decoder errors. Users configure timeout, start/stop search, and see live top 5 results. Results display (PNG export, URL copy, error visualization) belongs to Phase 4.

</domain>

<decisions>
## Implementation Decisions

### Search configuration
- Only configurable setting: maximum search time in seconds
- Default timeout: 30 seconds
- Controls (timeout input + Generate button) placed below the canvas, continuing the vertical flow: URL > QR > paint > configure > generate
- Generate button disabled until at least one pixel is painted; show hint like "Paint pixels first"
- All other search parameters (hash length, character set, strategy) are automatic — Claude's discretion

### Live feedback layout
- Top 5 candidates displayed as horizontal row of small QR code previews, ranked left to right (best first)
- Stats shown during search: attempts counter + elapsed time (no rate display)
- Each QR preview shows error count only beneath it (hash fragment details deferred to Phase 4 results view)
- Subtle highlight (brief glow or color flash) when a new best result enters the top 5

### Error measurement
- Primary goal: measure actual Reed-Solomon error correction burden, not just pixel mismatch
  - The ideal result is a hash fragment where QR data naturally agrees with the painted pattern (pattern pixels are correct data, not errors)
  - Research whether RS correction counts can be extracted from jsQR or another browser decoder
- Fallback if RS counts aren't feasible: pixel diff count among decodable results
- Ranking: primary sort by decodable (yes/no), secondary sort by error count (lower is better)
- All results shown in top 5 regardless of decode status, but clearly marked
- Label format beneath each preview: decode badge + error count (e.g., "decodable - 3 errors" or "not decodable - 28 errors")

### Search completion
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

</decisions>

<specifics>
## Specific Ideas

- The core insight: some hash fragments might produce QR data that naturally aligns with the painted pattern, so the pattern pixels are actually useful data rather than errors the decoder must correct. The error metric should capture this distinction — it's not just about pixel mismatch, it's about decoder burden.
- Re-running should accumulate results, not restart — lets users do multiple short runs instead of one long one.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 03-hash-optimization-loop*
*Context gathered: 2026-02-08*
