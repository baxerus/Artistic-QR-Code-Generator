# Phase 18: Move Pattern Tool - Context

**Gathered:** 2026-02-18
**Status:** Ready for planning

<domain>
## Phase Boundary

Add a Move Pattern tool that lets users reposition the painted pattern without changing any painted pixel values.

</domain>

<decisions>
## Implementation Decisions

### Move interaction model
- Move happens only when the Move tool is selected.
- Drag to move; live movement during drag; release commits.
- Click without drag is a no-op.

### Movement constraints
- Movement snaps to the QR module grid (whole-module shifts).
- Diagonal movement allowed (free 2D drag).
- Movement wraps around at edges (toroidal wrap).
- Provide arrow-key nudges for single-step moves.

### Tool UI + feedback
- Move Pattern tool appears alongside Black/White/Unset in the same selector row.
- Cursor is grab when Move tool selected; grabbing while dragging.
- No hint text; rely on label/cursor/behavior.
- Other paint buttons remain available (inactive only).

### State behavior
- After move completes, decode status auto-recalculates.
- Move only the painted pixels (protected patterns do not move).
- If moved pixels overlap existing painted pixels, the moved pixel wins.
- Clear Pattern resets the move offset.

### Claude's Discretion
- None specified.

</decisions>

<specifics>
## Specific Ideas

No specific requirements — open to standard approaches.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 18-move-pattern-tool*
*Context gathered: 2026-02-18*
