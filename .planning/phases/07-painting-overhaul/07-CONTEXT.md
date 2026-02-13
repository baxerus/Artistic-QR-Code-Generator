# Phase 7: Painting Overhaul - Context

**Gathered:** 2026-02-13
**Status:** Ready for planning

<domain>
## Phase Boundary

Redesign the painting interface so users paint directly on the QR code preview with drag painting and clear visual feedback about protected areas. This phase replaces the separate painting canvas with an overlay-based approach where users paint on top of the QR code itself.

</domain>

<decisions>
## Implementation Decisions

### Drag Painting Interaction
- Click and hold to paint — user holds mouse button while dragging
- Grid detection sensitivity is tunable via a single constant (configurable from 100% pixel coverage down to small center threshold)
- Protected QR structural areas are "transparent" to drag — painting continues through them without effect, resumes on valid pixels
- Minimal visual feedback during painting — just the painted result appears, no cursor changes or pixel highlights

### Overlay Rendering
- Painted pixels render as **solid color overlay** (100% opaque) on top of QR code
- Painted pixels must have **distinguishable colors** from QR code modules:
  - Painted white pixels → light gray (not pure white)
  - Painted black pixels → dark gray (not pure black)
  - This ensures users can see what they painted vs. what's QR code content
- Unset pixels show **subtle grid overlay** — faint grid lines marking pixel boundaries
- QR code is **fully visible** underneath the overlay (not dimmed)
- Grid lines are **visible on hover only** — appear when cursor is over the canvas
- Pixels that happen to match QR modules underneath look **identical to other painted pixels** — no special visual distinction

### Protected Area Visualization
- Protected QR structural areas (finder patterns, timing patterns, alignment patterns) marked with **border outline**
- Border style/color is **Claude's discretion** — choose what looks good
- Protected areas **show actual QR modules** underneath — not blanked or patterned
- When user tries to paint on protected area: **cursor changes to blocked** (🚫 or not-allowed icon)
- **No legend needed** for protected areas — visual cues are self-explanatory
- **All protected areas look the same** — no distinction between finder/timing/alignment patterns

### Legend-Based Color Selection
- Color legend positioned **to the right** of the QR canvas
- Selected color indicated by **background change** on the legend button
- Legend shows three states with **literal colors**:
  - Black square (for black paint)
  - White square (for white paint)
  - Transparent/empty (for unset)
- Legend has **swatches with text labels**: "Black", "White", "Unset"

### Right-Click Behavior
- Right-click paints the **opposite color** of the currently selected legend color
- When "unset" is selected in legend: **right-click does nothing at all**
- Right-click has **full drag support** — hold right button and drag to paint opposite color continuously
- **No special visual feedback** for right-click mode — user knows by which button they're holding
- Right-click **always applies opposite color** — never toggles to unset, even on already-painted pixels

### Claude's Discretion
- Protected area border color and style
- Exact light gray / dark gray values for painted pixels
- Grid line appearance (color, thickness, style)
- Legend button styling and spacing
- Cursor icons for normal vs. blocked states
- Legend layout and sizing details

</decisions>

<specifics>
## Specific Ideas

- Paint detection sensitivity should be easily tunable with a single constant for testing different thresholds
- Protected areas should not disrupt the painting flow — user can drag through them seamlessly
- Users must be able to clearly distinguish what they've painted from what the QR code naturally shows

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 07-painting-overhaul*
*Context gathered: 2026-02-13*
