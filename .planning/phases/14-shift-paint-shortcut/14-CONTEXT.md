# Phase 14: Shift+Paint Shortcut - Context

**Gathered:** 2026-02-17
**Status:** Ready for planning

<domain>
## Phase Boundary

Users can quickly paint "unset" to erase painted pixels by holding Shift, without changing their legend selection. This is a modifier-key shortcut for the existing painting system.

</domain>

<decisions>
## Implementation Decisions

### Visual feedback
- No cursor change when Shift is held — user relies on key feel
- Legend highlights "unset" option when Shift is held
- Highlight uses same visual style as normal color selection (not a distinct temporary style)
- Only "unset" appears selected while Shift held — original selection indicator hidden temporarily

### Shift behavior (from requirements)
- Shift+click/drag (left or right button) paints "unset" regardless of selected color
- Shift state captured at stroke start and held for entire drag operation
- Releasing Shift mid-drag does not change behavior — stroke completes as "unset"
- Normal painting behavior unchanged when Shift is not held

### Claude's Discretion
- Edge case: what happens if Shift pressed mid-stroke (likely ignore until next stroke)
- Implementation approach for tracking Shift state
- How to handle legend state restoration when Shift released

</decisions>

<specifics>
## Specific Ideas

No specific requirements — standard modifier key behavior expected.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 14-shift-paint-shortcut*
*Context gathered: 2026-02-17*
