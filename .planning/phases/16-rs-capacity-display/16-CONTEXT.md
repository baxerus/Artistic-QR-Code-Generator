# Phase 16: RS Capacity Display - Context

**Gathered:** 2026-02-18
**Status:** Ready for planning

<domain>
## Phase Boundary

Display correction count as "X of Y" capacity on result cards, where Y is the max capacity for the QR version at level H (computed from the jsQR VERSIONS table via `sum(ecBlocks × ecCodewordsPerBlock) / 2`).

</domain>

<decisions>
## Implementation Decisions

### Placement and label
- Replace existing "Corrections: X" text with "Corrections: X of Y" wherever that text appears (result card and any other views that show it).
- Keep the label as "Corrections".

### Text format and ordering
- Exact inline format: "Corrections: X of Y".
- Use integer values only (no decimals).
- Corrections appears before Pixels in the metrics order.

### Hover hint behavior
- Keep the existing hover hint and extend it to explain what the max value (Y) represents.
- Show the hover hint even when placeholders are shown.

### Edge/unknown states
- If X is missing: show placeholder (e.g., "Corrections: — of Y").
- If Y is unavailable or computes to 0/negative: treat as unavailable and show placeholder ("Corrections: X of —").

### Visual emphasis
- No warning cues or icons; keep the existing styling with no visual emphasis changes.

### Claude's Discretion
- Placeholder glyph choice and exact hover hint copy, as long as it explains the max value.

</decisions>

<specifics>
## Specific Ideas

- Extend the existing hover hint to explain that Y is the maximum correction capacity for the QR version.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 16-rs-capacity-display*
*Context gathered: 2026-02-18*
