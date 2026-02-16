# Phase 12: RS Measurement Integration - Context

**Gathered:** 2026-02-16
**Status:** Ready for planning

<domain>
## Phase Boundary

Workers extract and return Reed-Solomon correction count for each candidate QR during optimization search. Each valid QR candidate decoded during the search will have its RS correction count captured and returned alongside pixel diff data.

</domain>

<decisions>
## Implementation Decisions

### Decoder Strategy
- **Primary approach:** Use jsQR if it exposes RS correction data natively
- **Fallback:** If jsQR doesn't expose RS data, patch or fork jsQR to expose internal RS correction count during decoding
- **Single decoder:** Use one decoder pass that provides both valid/invalid status AND RS corrections — avoid double-decoding
- **Priority:** RS data accuracy takes precedence over decoder speed — choose the most accurate decoder even if slower

### Data Flow & Structure
- **Result structure:** Extend existing result object to include RS field — `{url, pixel_diff, rs_corrections}`
- **Collection logic:** RS data influences which candidates are kept — workers should prefer/return candidates with lower RS corrections
- **Inline extraction:** Extract RS for every candidate during the main optimization loop (not post-processing)
- **RS data points:** Single RS correction count per candidate QR (determined at implementation)

### Fallback Behavior
- **Missing RS data:** If a decoder can't provide RS corrections for a candidate, treat it as a failure — don't return that candidate
- **Complete failure:** If RS extraction isn't possible at all, the phase fails — need working RS extraction before proceeding
- **Perfect results:** RS=0 is just a numeric value — no special flag needed, just the count
- **Thresholds:** Don't predefine "acceptable" RS levels — Phase 13 will determine rankings and thresholds

### Performance Considerations
- **Coverage:** Extract RS for every single valid QR candidate (not sampled, not just finalists)
- **Speed tradeoff:** RS data is valuable — up to 2x slower search speed is acceptable for accurate RS metrics
- **Caching:** No caching needed — decode fresh each time (QRs are different candidates anyway)
- **Messaging:** Include RS data in existing batched worker messages — no separate RS-specific message overhead

### Claude's Discretion
- Specific jsQR patching approach if needed
- Exact decoder library selection if jsQR doesn't work
- Implementation of "prefer lower RS" collection logic
- Exact performance optimizations

</decisions>

<specifics>
## Specific Ideas

- "Use jsQR if it exposes RS data" — check current jsQR API first
- "Patch or fork jsQR" — willing to modify library internals if needed
- "Accuracy first" — precise RS correction count is essential
- RS data should influence which candidates workers keep, not just be metadata

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 12-rs-measurement*
*Context gathered: 2026-02-16*
