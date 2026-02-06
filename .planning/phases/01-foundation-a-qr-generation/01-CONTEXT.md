# Phase 1: Foundation & QR Generation - Context

**Gathered:** 2026-02-06
**Status:** Ready for planning

<domain>
## Phase Boundary

Single-file HTML tool that accepts URL input and generates valid QR codes on canvas. Users can enter URLs, see validation feedback, select appropriate QR versions, and view generated QR codes. This phase establishes the foundational UI and core QR generation capability.

</domain>

<decisions>
## Implementation Decisions

### Page Structure
- Content centered on page
- Minimal title/header at top (tool name only, no description)
- Footer with minimal credit/version info
- Page background: light gray or subtle color (not pure white)

### Input Layout
- URL input positioned above canvas (traditional form-then-result flow)
- Controls organized in single compact form
- Version dropdown on separate row below URL input
- Generate button positioned below all controls (separate action row)

### Canvas Presentation
- Responsive to viewport size
- Maintains square aspect ratio always (1:1, no stretching)
- Clear border and shadow around canvas for visual containment
- Canvas prominently displayed in main content area

### Spacing & Density
- Balanced overall spacing (not too tight, not too loose)
- Medium vertical gaps between major sections (20-30px)
- Sections feel connected but distinct

### Validation & Feedback
- Inline validation with icon and color indicators
- Visual feedback at the input field (red border, icon) - subtle approach
- Minimum version info: only restrict dropdown options (don't show explicit text)

### Claude's Discretion
- Mobile/small screen responsive behavior (handle breakpoints appropriately)
- Exact shadow and border styling for canvas
- Specific color values for background, borders, and validation states
- Typography choices (font family, sizes, weights)
- Loading state presentation during QR generation
- Exact spacing values within the 20-30px guideline

</decisions>

<specifics>
## Specific Ideas

No specific product references provided - standard single-file HTML tool aesthetic desired.

Key preferences:
- "Balanced" density throughout
- Canvas should stand out visually (border + shadow + background contrast)
- Form controls grouped but with clear visual hierarchy (separate rows)

</specifics>

<deferred>
## Deferred Ideas

None - discussion stayed within phase scope.

</deferred>

---

*Phase: 01-foundation-a-qr-generation*
*Context gathered: 2026-02-06*
