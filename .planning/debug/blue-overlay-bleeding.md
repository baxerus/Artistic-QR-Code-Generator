---
status: diagnosed
trigger: "Blue overlay bleeding/blur on protected areas - from UAT Test 4"
created: 2026-02-15T00:00:00Z
updated: 2026-02-15T00:00:00Z
---

## Current Focus

hypothesis: Canvas anti-aliasing causes sub-pixel rendering blur when drawing at module-level coordinates, and line stroke parameters are too small to be visible when CSS-scaled
test: Examined canvas rendering code and CSS scaling approach
expecting: Found issues with anti-aliasing in fillRect and stroke operations at sub-pixel dimensions
next_action: Document root cause for plan-phase --gaps

## Symptoms

expected: Blue overlay covers exactly the protected/functional areas with dashed outline borders
actual: "The overlay is shown only on hover, that is correct. BUT the blue overlay for function parts is not shown over to correct parts. Or better it is bleeding into parts that are not functional parts. It looks like blur because of maybe scaling? In general there shouldn't be different shades of blue overlay (what is the case in the moment). Also the blue functional parts should have a dashed outline as the overlay on top of the paint pattern, but there is no dashed outline at all"
errors: None reported
reproduction: Test 4 in UAT - Click a result to expand it, hover over expanded result
started: Feature implementation

## Eliminated

(none - first investigation)

## Evidence

- timestamp: 2026-02-15T00:01:00Z
  checked: Canvas dimensions and CSS scaling
  found: Overlay canvas created at moduleCount dimensions (e.g., 21x21 pixels for version 1 QR), then CSS scales to 120px or 100% width via `width: 100%; height: 100%;` with `image-rendering: pixelated`
  implication: Small canvas dimensions cause sub-pixel rendering issues

- timestamp: 2026-02-15T00:02:00Z
  checked: renderErrorOverlay function (lines 15844-15885)
  found: Uses `ctx.fillRect(col, row, 1, 1)` to fill 1x1 pixel squares for protected areas with `rgba(0, 100, 255, 0.35)`
  implication: Canvas anti-aliasing enabled by default causes blur/bleeding on edges of fillRect calls

- timestamp: 2026-02-15T00:03:00Z
  checked: renderProtectedBordersOverlay function (lines 15893-15936)
  found: Uses `ctx.lineWidth = 0.5` and `ctx.setLineDash([2, 1])` to draw borders, with strokes at integer coordinates (x, y, x+1, y+1)
  implication: lineWidth=0.5 is sub-pixel, may not render visibly. Dash pattern [2,1] assumes pixel units but canvas is only ~21-33 pixels wide, so dashes won't be visible when CSS-scaled

- timestamp: 2026-02-15T00:04:00Z
  checked: Canvas context initialization
  found: No `ctx.imageSmoothingEnabled = false` set on overlay canvas context, unlike in renderQRToCanvas (line 13068) which explicitly disables it
  implication: Anti-aliasing is ON for the overlay canvas, causing blur/bleeding between modules

- timestamp: 2026-02-15T00:05:00Z
  checked: Canvas stroke coordinate alignment
  found: Strokes at integer coordinates (x, y) but canvas strokes are centered on the path. At lineWidth=0.5, a stroke from (0,0) to (1,0) will render from y=-0.25 to y=0.25, causing anti-aliased blending
  implication: Need half-pixel offset (e.g., 0.5) for crisp strokes, or use a larger canvas resolution

## Resolution

root_cause: |
  TWO DISTINCT ISSUES:
  
  1. **Blue fill bleeding/blur (different shades)**: The overlay canvas context has anti-aliasing ENABLED (imageSmoothingEnabled defaults to true). When drawing 1x1 pixel fillRect operations at integer coordinates on a 21x21 canvas that gets CSS-scaled to 120px+, the canvas anti-aliasing causes sub-pixel blending at module boundaries, creating the "blur" and "different shades of blue" effect.
  
  2. **Dashed borders not visible**: The stroke parameters are too small for the canvas resolution:
     - `lineWidth = 0.5` is a sub-pixel line width, which may not render visibly
     - `setLineDash([2, 1])` creates a 2-pixel dash with 1-pixel gap, but on a 21x21 canvas this creates at most ~3 dashes per finder pattern edge
     - Combined with anti-aliasing, these thin strokes get blurred into near-invisibility when CSS-scaled
  
  FILES INVOLVED:
  - index.html lines 15851-15885 (renderErrorOverlay): Missing `ctx.imageSmoothingEnabled = false`
  - index.html lines 15893-15936 (renderProtectedBordersOverlay): lineWidth and dash pattern too small for module-level resolution

fix: (not applied - goal: find_root_cause_only)
verification: (not applied)
files_changed: []
