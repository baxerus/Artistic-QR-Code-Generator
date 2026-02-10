# Phase 6: State Persistence - Context

**Gathered:** 2026-02-10
**Status:** Ready for planning

<domain>
## Phase Boundary

Save and restore the full app state across page reloads so users never lose their work. Covers URL input, QR version selection, and painted pattern. Optimization results and future features (multi-session, export/import) are out of scope.

</domain>

<decisions>
## Implementation Decisions

### Restore experience
- Auto-regenerate QR code on page load from restored inputs (not a static preview)
- No loading indicator needed — generation is fast enough to appear instant
- Silent fallback on corrupt/missing data — use defaults for anything that can't be restored, no error messages
- Optimization results are NOT restored — only inputs and painted pattern

### Save triggers & timing
- Claude's discretion on exact trigger strategy (debounced on change vs on significant actions)
- Paint pattern saves at end of stroke (mouse/touch release), not every pixel
- No beforeunload safety net needed
- Multiple tabs are independent — last tab to save wins, no cross-tab sync

### State scope
- Persist the basics: URL input text, QR version dropdown, painted pattern grid
- Pattern storage format: Claude's discretion (full grid vs sparse)
- Pattern is tied to its QR version — version + pattern saved together, mismatched sizes rejected on restore
- Separate localStorage keys per concern (not a single JSON blob)

### Reset & clear behavior
- No explicit "Clear / Start Fresh" button in the UI
- Pattern stays in localStorage even if URL is cleared — user keeps their painting
- No expiration — state persists forever until browser data is cleared
- Graceful merge for state shape changes — merge saved state with defaults for new fields, no explicit schema versioning

### Claude's Discretion
- Exact save trigger strategy (debounced vs event-based)
- Pattern storage format (full grid array vs sparse coordinates)
- localStorage key naming convention
- Any internal implementation details

</decisions>

<specifics>
## Specific Ideas

No specific requirements — open to standard approaches

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 06-state-persistence*
*Context gathered: 2026-02-10*
