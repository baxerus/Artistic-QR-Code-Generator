# Phase 17: Decode Metrics Alignment - Context

**Gathered:** 2026-02-18
**Status:** Ready for planning

<domain>
## Phase Boundary

Paint Pattern decode test shows corrections and pixel errors consistent with result cards, with a stable layout (no height jump).

</domain>

<decisions>
## Implementation Decisions

### Status row layout
- Text-only status (no badges or icons), uses same Success/Fail/Neutral colors as result cards.
- Fixed two-line height: label line + metrics line; no animation on state changes.

### Metrics display
- Use the exact result-card labels: "Corrections: X of Y" and "Pixel errors: Z".
- Corrections first, pixel errors second.
- Use the same max-capacity tooltip behavior as result cards.
- On decode failure, show dashes: "Corrections: — of —" and "Pixel errors: —".

### Decode trigger behavior
- Auto-run decode when the Paint Pattern section is expanded.
- While expanded, auto re-run after paint changes with a 300–500ms debounce.
- No manual re-run button.
- Decode can run during optimization search.

### Neutral state behavior
- Neutral only when the pattern is empty; label is "No pattern applied".
- Any non-empty pattern should auto-decode and show Success or Fail.
- Decode errors (cannot run/throws) show Neutral.
- Metrics line hidden in Neutral state.

### Claude's Discretion
- Exact spacing/typography for the two-line row within existing layout constraints.

</decisions>

<specifics>
## Specific Ideas

No specific references beyond matching existing result-card metrics and colors.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 17-decode-metrics-alignment*
*Context gathered: 2026-02-18*
