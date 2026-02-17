# Phase 15: SVG Export - Context

**Gathered:** 2026-02-17
**Status:** Ready for planning

<domain>
## Phase Boundary

Add SVG download capability to result cards so users can export QR codes as scalable vector graphics for high-quality printing. The existing PNG download and Copy URL buttons remain unchanged. This phase does NOT modify PNG export behavior.

</domain>

<decisions>
## Implementation Decisions

### SVG module rendering
- Single `<path>` element with M/h/v commands for all black modules — most compact, standard QR SVG approach
- ViewBox at 1×1 unit per module (e.g., `0 0 29 29` for 29-module QR) — infinitely scalable by nature of SVG
- Include a white background `<rect>` — SVG is self-contained, always visible on any background
- Include 4-module quiet zone (white border) per QR spec — print-ready

### Button placement & styling
- Three buttons stacked vertically: "Download PNG", then "Download SVG", then "Copy URL"
- "Download SVG" button has identical styling to existing buttons — same look, same size
- Button text: "Download SVG" (parallel naming with "Download PNG")
- Disabled during search, same as existing buttons — behavior identical in all cases
- Click feedback matches whatever "Download PNG" does (or doesn't do)

### SVG metadata & sizing
- Include `<title>` element containing the encoded URL — accessible, shows on hover in some viewers
- Omit explicit `width`/`height` attributes, viewBox only — scales to fill container, user sets size in their design tool

### Claude's Discretion
- Data source for SVG generation (reuse `result.moduleData`/`result.moduleCount` or alternate approach)
- XML prologue inclusion (`<?xml?>` declaration vs bare `<svg>` tag)
- SVG `xmlns` handling

</decisions>

<specifics>
## Specific Ideas

- SVG should use black (#000) on white (#FFF) modules only, matching final QR output (per SVG-02 requirement)
- Filename follows same sanitization pattern as PNG export but with .svg extension (per SVG-03 requirement)
- Quiet zone is 4 modules wide, standard QR specification

</specifics>

<deferred>
## Deferred Ideas

- Add quiet zone to PNG export — currently PNG renders edge-to-edge without QR-spec quiet zone. Worth adding but separate from SVG work.

</deferred>

---

*Phase: 15-svg-export*
*Context gathered: 2026-02-17*
