# Phase 19: QR Rotation Controls - Context

**Gathered:** 2026-02-19
**Status:** Ready for planning

<domain>
## Phase Boundary

Add QR rotation controls so users can rotate QR presentation and search without rotating the painted pattern. Rotation must apply to preview, results, and exports, and allow optional random rotation during Generate.

</domain>

<decisions>
## Implementation Decisions

### Rotation controls UI
- Rotation control lives near the paint tools in the Paint Pattern section.
- Use a single rotate button that advances 90 degrees per click and shows current state via icon-only indicator.
- No separate reset button; reset via selecting 0 degrees through the control flow.

### Rotation state rules
- Default to the last used angle (persist across sessions).
- Reset rotation to 0 degrees when QR version changes.
- Reset rotation to 0 degrees when Clear Pattern is used.
- Rotation persists across Generate runs unless changed by the user.

### Random rotation behavior
- Random rotation changes per attempt within a Generate run.
- Do not display the chosen random angle to the user.
- No lock option for random angle.
- Random mode overrides manual selection until random is turned off.

### Result/overlay alignment
- Apply rotation to paint preview, results (thumbnails and expanded), and PNG/SVG exports.
- Rotate hover overlays along with the rotated QR.
- Painted pattern grid stays unrotated in all views.
- Decode metrics should auto-update when rotation changes.

### Claude's Discretion
- Exact iconography and visual styling of the rotate button.

</decisions>

<specifics>
## Specific Ideas

- Rotate control should be near the paint tools with icon-only angle indication.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 19-qr-rotation-controls*
*Context gathered: 2026-02-19*
