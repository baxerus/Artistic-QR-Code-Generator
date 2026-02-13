# Phase 7: Painting Overhaul - Research

**Researched:** 2026-02-13
**Domain:** HTML5 Canvas API, Mouse Interaction, Overlay Rendering
**Confidence:** HIGH

## Summary

This phase transforms the painting interface from a separate canvas with click-to-cycle behavior into an overlay-based painting system on top of the QR code itself. The research covers Canvas API techniques for drag painting, coordinate transformation, overlay compositing, and cursor management.

The existing codebase already has a solid foundation: QR code generation via qrcodejs, function pattern masking (`createFunctionPatternMask`), and a `PixelGrid` class for state management. The overhaul replaces the separate `paint-canvas` with direct painting on the QR display canvas using overlay rendering.

**Primary recommendation:** Use a single canvas with layered rendering (QR base → painted overlay → grid lines → protected area outlines) and pointer event handlers for drag painting with coordinate transformation.

## User Constraints (from CONTEXT.md)

### Locked Decisions

**Drag Painting Interaction**
- Click and hold to paint — user holds mouse button while dragging
- Grid detection sensitivity is tunable via a single constant (configurable from 100% pixel coverage down to small center threshold)
- Protected QR structural areas are "transparent" to drag — painting continues through them without effect, resumes on valid pixels
- Minimal visual feedback during painting — just the painted result appears, no cursor changes or pixel highlights

**Overlay Rendering**
- Painted pixels render as **solid color overlay** (100% opaque) on top of QR code
- Painted pixels must have **distinguishable colors** from QR code modules:
  - Painted white pixels → light gray (not pure white)
  - Painted black pixels → dark gray (not pure black)
- Unset pixels show **subtle grid overlay** — faint grid lines marking pixel boundaries
- QR code is **fully visible** underneath the overlay (not dimmed)
- Grid lines are **visible on hover only** — appear when cursor is over the canvas
- Pixels that happen to match QR modules underneath look **identical to other painted pixels** — no special visual distinction

**Protected Area Visualization**
- Protected QR structural areas (finder patterns, timing patterns, alignment patterns) marked with **border outline**
- Border style/color is **Claude's discretion** — choose what looks good
- Protected areas **show actual QR modules** underneath — not blanked or patterned
- When user tries to paint on protected area: **cursor changes to blocked** (🚫 or not-allowed icon)
- **No legend needed** for protected areas — visual cues are self-explanatory
- **All protected areas look the same** — no distinction between finder/timing/alignment patterns

**Legend-Based Color Selection**
- Color legend positioned **to the right** of the QR canvas
- Selected color indicated by **background change** on the legend button
- Legend shows three states with **literal colors**:
  - Black square (for black paint)
  - White square (for white paint)
  - Transparent/empty (for unset)
- Legend has **swatches with text labels**: "Black", "White", "Unset"

**Right-Click Behavior**
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

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope

## Standard Stack

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| HTML5 Canvas API | Native | Rendering painted overlay, grid lines, protected area outlines | Native browser API, pixel-perfect control needed |
| Pointer Events | Native | Mouse/touch interaction for drag painting | Modern replacement for mouse events, handles all pointer types |

### Supporting

| Technique | Purpose | When to Use |
|-----------|---------|-------------|
| `getBoundingClientRect()` | Coordinate transformation from screen to canvas | Converting mouse coordinates to grid coordinates |
| `globalCompositeOperation` | Controlling how overlay blends with QR | Not needed for opaque overlay; use default `source-over` |
| CSS `cursor` property | Changing cursor on hover (normal vs blocked) | Protected area detection feedback |
| CSS `pointer-events` | Controlling click-through behavior | Not applicable — canvas handles all interactions |

## Architecture Patterns

### Recommended Rendering Pipeline

The QR canvas should use a layered rendering approach within a single canvas:

```
Layer 0 (Bottom): QR code modules (black/white from qrcodejs)
Layer 1: Painted pixel overlay (dark gray/light gray, 100% opaque)
Layer 2: Grid lines (on hover only)
Layer 3: Protected area outlines (always visible)
```

### Pattern 1: Coordinate Transformation

**What:** Convert mouse coordinates to grid coordinates accounting for CSS scaling

**When to use:** All mouse/pointer events on the canvas

**Example:**
```javascript
// Source: Current codebase pattern + MDN Canvas docs
function getGridCoordinates(canvas, event, cellSize) {
  const rect = canvas.getBoundingClientRect();
  
  // Account for CSS scaling vs actual canvas resolution
  const scaleX = canvas.width / rect.width;
  const scaleY = canvas.height / rect.height;
  
  // Mouse position relative to canvas
  const canvasX = (event.clientX - rect.left) * scaleX;
  const canvasY = (event.clientY - rect.top) * scaleY;
  
  // Convert to grid coordinates
  const gridX = Math.floor(canvasX / cellSize);
  const gridY = Math.floor(canvasY / cellSize);
  
  return { x: gridX, y: gridY };
}
```

### Pattern 2: Drag Painting State Machine

**What:** Track painting state across pointer events for continuous drag painting

**When to use:** Implementing click-and-drag painting

**Example:**
```javascript
// Source: Eloquent JavaScript paint program + StackOverflow patterns
let isPainting = false;
let lastPaintedCell = null;
let currentTool = 'black'; // 'black' | 'white' | 'unset'

function handlePointerDown(e) {
  isPainting = true;
  paintAt(e);
}

function handlePointerMove(e) {
  if (!isPainting) return;
  paintAt(e);
}

function handlePointerUp(e) {
  isPainting = false;
  lastPaintedCell = null;
}

function paintAt(e) {
  const coords = getGridCoordinates(canvas, e, cellSize);
  const cellKey = `${coords.x},${coords.y}`;
  
  // Debounce: don't repaint same cell during drag
  if (lastPaintedCell === cellKey) return;
  
  // Check if protected area (finder/timing/alignment)
  if (isProtected(coords.x, coords.y)) {
    // Cursor changes to blocked via CSS, painting continues transparently
    return;
  }
  
  // Apply paint
  paintGrid.set(coords.x, coords.y, getPaintValue(e.button));
  lastPaintedCell = cellKey;
  requestAnimationFrame(render);
}
```

### Pattern 3: Tunable Grid Detection Sensitivity

**What:** Configurable threshold for determining which grid cell a mouse position maps to

**When to use:** Fine-tuning painting precision

**Example:**
```javascript
// Source: Adapted from common game development patterns
const GRID_SENSITIVITY = 0.5; // 0.0 = center only, 1.0 = full cell

function getGridCoordinatesWithSensitivity(canvas, event, cellSize) {
  const rect = canvas.getBoundingClientRect();
  const scaleX = canvas.width / rect.width;
  const scaleY = canvas.height / rect.height;
  
  const canvasX = (event.clientX - rect.left) * scaleX;
  const canvasY = (event.clientY - rect.top) * scaleY;
  
  // Calculate grid position with sensitivity threshold
  const rawX = canvasX / cellSize;
  const rawY = canvasY / cellSize;
  
  const gridX = Math.floor(rawX + (1 - GRID_SENSITIVITY) * 0.5);
  const gridY = Math.floor(rawY + (1 - GRID_SENSITIVITY) * 0.5);
  
  return { x: gridX, y: gridY };
}
```

### Pattern 4: Overlay Rendering Strategy

**What:** Render painted pixels as opaque overlay on top of QR code

**When to use:** The core overlay painting requirement

**Example:**
```javascript
// Source: MDN Canvas Compositing Tutorial + verified patterns
function render() {
  const ctx = canvas.getContext('2d');
  
  // Clear and draw base QR
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawQRCode(ctx, qrModel, cellSize);
  
  // Draw painted overlay (source-over = draw on top)
  ctx.globalCompositeOperation = 'source-over';
  for (const [x, y, value] of paintGrid.entries()) {
    if (value === 0) continue; // unset
    
    // Use distinguishable grays
    ctx.fillStyle = value === 1 ? '#d0d0d0' : '#404040'; // light gray / dark gray
    ctx.fillRect(x * cellSize, y * cellSize, cellSize, cellSize);
  }
  
  // Draw protected area outlines
  drawProtectedAreaOutlines(ctx, functionMask, cellSize);
  
  // Draw grid lines (on hover only - controlled by CSS class)
  if (isHovering) {
    drawGridLines(ctx, gridSize, cellSize);
  }
}
```

### Pattern 5: Protected Area Detection

**What:** Check if a grid coordinate is in a protected (function pattern) area

**When to use:** Before applying paint, for cursor feedback

**Example:**
```javascript
// Source: Existing codebase (createFunctionPatternMask)
// The function mask already exists in the codebase

function isProtected(x, y) {
  const idx = y * gridSize + x;
  return functionMask[idx];
}

// Cursor feedback via CSS class on canvas container
function updateCursor(x, y) {
  if (isProtected(x, y)) {
    canvasContainer.style.cursor = 'not-allowed';
  } else {
    canvasContainer.style.cursor = 'crosshair';
  }
}
```

### Pattern 6: Right-Click Handling

**What:** Handle right-click to paint opposite color

**When to use:** Implementing right-click drag painting

**Example:**
```javascript
// Source: MDN MouseEvent.button docs + current codebase patterns
function getPaintValue(button) {
  const selectedTool = getSelectedLegendColor(); // 'black' | 'white' | 'unset'
  
  if (button === 0) {
    // Left click: paint selected color
    return selectedTool === 'black' ? 2 : selectedTool === 'white' ? 1 : 0;
  } else if (button === 2) {
    // Right click: paint opposite color
    if (selectedTool === 'unset') return null; // Does nothing
    return selectedTool === 'black' ? 1 : 2; // Opposite
  }
  return null;
}

// Prevent context menu on right-click
canvas.addEventListener('contextmenu', e => e.preventDefault());
```

### Anti-Patterns to Avoid

- **Using two separate canvases:** Current implementation uses separate paint-canvas. Phase 7 consolidates to single canvas with overlay.
- **Repainting entire canvas on every mouse move:** Use debouncing/cell-tracking to avoid redundant renders.
- **Using mouse events instead of pointer events:** Pointer events unify mouse/touch handling.
- **Storing cursor state in JavaScript:** Use CSS classes for cursor state — it's more performant.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Coordinate scaling | Custom math | `getBoundingClientRect()` + scale ratio | Handles CSS transforms, device pixel ratio, subpixel rendering |
| Drag state management | Boolean flags | Pointer capture API (`setPointerCapture`) | Handles pointer leaving element while dragging |
| Debounced painting | `setTimeout` | Cell-based deduplication (track last painted cell) | More precise for grid-based painting |
| Right-click detection | `which` or `button` inconsistently | `event.button` (0=left, 2=right) | Standard, cross-browser |

**Key insight:** The Canvas API is already optimized for pixel manipulation. Don't add abstraction layers — use direct pixel/grid coordinate math.

## Common Pitfalls

### Pitfall 1: Coordinate Drift with CSS Scaling

**What goes wrong:** Canvas displayed at different size than internal resolution causes mouse coordinates to map to wrong grid cells.

**Why it happens:** Canvas has internal width/height (pixels) and CSS display size — they can differ.

**How to avoid:** Always calculate scale ratio:
```javascript
const scaleX = canvas.width / canvas.getBoundingClientRect().width;
const scaleY = canvas.height / canvas.getBoundingClientRect().height;
```

**Warning signs:** Painting appears offset from cursor position, especially at different zoom levels or responsive layouts.

### Pitfall 2: Rapid Event Firing Causing Jank

**What goes wrong:** `pointermove` fires very rapidly during drag, causing too many renders.

**Why it happens:** Browsers fire pointer events at 60-120Hz, which can exceed render budget.

**How to avoid:** Track last painted cell and skip redundant paints:
```javascript
if (lastCell === currentCell) return; // Skip if same cell
```

### Pitfall 3: Context Menu on Right-Click

**What goes wrong:** Browser context menu appears on right-click instead of painting.

**Why it happens:** Default browser behavior for right-click is context menu.

**How to avoid:** Prevent default on `contextmenu` event:
```javascript
canvas.addEventListener('contextmenu', e => e.preventDefault());
```

### Pitfall 4: Cursor Lag on Protected Areas

**What goes wrong:** Cursor changes lag behind mouse movement when entering/leaving protected areas.

**Why it happens:** Updating cursor via JavaScript on every `pointermove` is expensive.

**How to avoid:** Use CSS `:hover` with classes or efficient state tracking. Consider throttling cursor updates.

### Pitfall 5: Lost Pointer Events When Leaving Canvas

**What goes wrong:** User drags off canvas, releases mouse, returns — painting state is stuck.

**Why it happens:** `pointerup` doesn't fire if pointer leaves element before release.

**How to avoid:** Use `setPointerCapture` on `pointerdown` to receive events even outside element:
```javascript
canvas.addEventListener('pointerdown', e => {
  canvas.setPointerCapture(e.pointerId);
  // ...
});
```

## Code Examples

### Complete Drag Painting Setup

```javascript
// Source: Adapted from Eloquent JavaScript + MDN docs + verified patterns
class PaintController {
  constructor(canvas, gridSize, functionMask) {
    this.canvas = canvas;
    this.gridSize = gridSize;
    this.functionMask = functionMask;
    this.cellSize = canvas.width / gridSize;
    this.isPainting = false;
    this.lastCell = null;
    this.selectedColor = 'black'; // 'black' | 'white' | 'unset'
    this.paintGrid = new PixelGrid(gridSize);
    
    this.setupEventListeners();
  }
  
  setupEventListeners() {
    this.canvas.addEventListener('pointerdown', this.handlePointerDown.bind(this));
    this.canvas.addEventListener('pointermove', this.handlePointerMove.bind(this));
    this.canvas.addEventListener('pointerup', this.handlePointerUp.bind(this));
    this.canvas.addEventListener('contextmenu', e => e.preventDefault());
    
    // Hover for cursor feedback
    this.canvas.addEventListener('pointermove', this.handleHover.bind(this));
  }
  
  getGridCoords(event) {
    const rect = this.canvas.getBoundingClientRect();
    const scaleX = this.canvas.width / rect.width;
    const scaleY = this.canvas.height / rect.height;
    
    const x = Math.floor(((event.clientX - rect.left) * scaleX) / this.cellSize);
    const y = Math.floor(((event.clientY - rect.top) * scaleY) / this.cellSize);
    
    return { x, y };
  }
  
  isValidCell(x, y) {
    return x >= 0 && x < this.gridSize && y >= 0 && y < this.gridSize;
  }
  
  isProtected(x, y) {
    return this.functionMask[y * this.gridSize + x];
  }
  
  handlePointerDown(e) {
    if (e.button !== 0 && e.button !== 2) return;
    
    this.isPainting = true;
    this.canvas.setPointerCapture(e.pointerId);
    this.paintAt(e);
  }
  
  handlePointerMove(e) {
    if (!this.isPainting) return;
    this.paintAt(e);
  }
  
  handlePointerUp(e) {
    this.isPainting = false;
    this.lastCell = null;
    this.canvas.releasePointerCapture(e.pointerId);
  }
  
  handleHover(e) {
    const { x, y } = this.getGridCoords(e);
    if (!this.isValidCell(x, y)) return;
    
    this.canvas.style.cursor = this.isProtected(x, y) ? 'not-allowed' : 'crosshair';
  }
  
  paintAt(e) {
    const { x, y } = this.getGridCoords(e);
    if (!this.isValidCell(x, y)) return;
    
    const cellKey = `${x},${y}`;
    if (this.lastCell === cellKey) return; // Debounce
    
    // Protected areas are transparent to drag — continue painting flow
    if (this.isProtected(x, y)) {
      this.lastCell = cellKey;
      return;
    }
    
    const paintValue = this.getPaintValue(e.button);
    if (paintValue === null) return;
    
    this.paintGrid.set(x, y, paintValue);
    this.lastCell = cellKey;
    
    requestAnimationFrame(() => this.render());
  }
  
  getPaintValue(button) {
    if (button === 0) {
      // Left click: selected color
      return this.selectedColor === 'black' ? 2 : 
             this.selectedColor === 'white' ? 1 : 0;
    } else if (button === 2) {
      // Right click: opposite (or nothing if unset)
      if (this.selectedColor === 'unset') return null;
      return this.selectedColor === 'black' ? 1 : 2;
    }
    return null;
  }
  
  render() {
    // Delegates to main render pipeline
    window.renderQRWithOverlay(this.paintGrid);
  }
}
```

### Protected Area Outline Rendering

```javascript
// Source: MDN Canvas Tutorial + verified patterns
function drawProtectedAreaOutlines(ctx, functionMask, gridSize, cellSize) {
  ctx.strokeStyle = '#ff6b6b'; // Claude's discretion: red border
  ctx.lineWidth = 2;
  ctx.setLineDash([4, 2]); // Dashed line
  
  // Find connected protected regions and draw bounding boxes
  // Or simply draw per-cell borders for protected areas
  for (let y = 0; y < gridSize; y++) {
    for (let x = 0; x < gridSize; x++) {
      if (functionMask[y * gridSize + x]) {
        // Draw border around protected cell
        ctx.strokeRect(x * cellSize, y * cellSize, cellSize, cellSize);
      }
    }
  }
  
  ctx.setLineDash([]); // Reset
}
```

### Grid Lines on Hover

```javascript
// Grid visibility controlled via CSS class on container
// .canvas-container:hover .grid-lines { opacity: 1; }

function drawGridLines(ctx, gridSize, cellSize) {
  ctx.strokeStyle = '#e0e0e0';
  ctx.lineWidth = 1;
  
  ctx.beginPath();
  for (let i = 0; i <= gridSize; i++) {
    const pos = i * cellSize;
    ctx.moveTo(pos, 0);
    ctx.lineTo(pos, gridSize * cellSize);
    ctx.moveTo(0, pos);
    ctx.lineTo(gridSize * cellSize, pos);
  }
  ctx.stroke();
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Separate paint canvas | Single canvas with overlay | Phase 7 | Unified visual experience, simplified DOM |
| Click-to-cycle | Drag painting with legend | Phase 7 | More intuitive, faster painting |
| No protected area visualization | Border outlines + cursor feedback | Phase 7 | Clearer UX, prevents accidental corruption |

**Deprecated/outdated:**
- Mouse-specific events (`mousedown`, `mousemove`, `mouseup`): Use pointer events for unified handling
- Separate canvas element for painting layer: Use single canvas with compositing

## Open Questions

1. **Performance on large QR versions (10+)?**
   - What we know: Canvas rendering scales linearly with pixel count
   - What's unclear: Whether protected area outline rendering causes jank
   - Recommendation: Implement and profile; use `requestAnimationFrame` batching

2. **Mobile/touch device support?**
   - What we know: Pointer events handle touch
   - What's unclear: Whether drag painting works well on touch devices
   - Recommendation: Test on mobile; consider touch-specific interactions if needed

3. **Legend positioning responsive design?**
   - What we know: Legend should be to the right of canvas
   - What's unclear: Exact layout on narrow viewports
   - Recommendation: Use flexbox, stack vertically on mobile breakpoints

## Sources

### Primary (HIGH confidence)

- MDN Canvas API Documentation - `getBoundingClientRect()`, `globalCompositeOperation`, pointer events
- MDN CSS `cursor` property - cursor values (`crosshair`, `not-allowed`)
- Current codebase - `createFunctionPatternMask()` implementation (lines 12879-12945 in index.html)
- Current codebase - `PixelGrid` class implementation (lines 12828-12858 in index.html)
- Eloquent JavaScript Chapter 19 - Paint program architecture patterns

### Secondary (MEDIUM confidence)

- StackOverflow verified answers on Canvas coordinate transformation
- HTML5 Canvas tutorials on compositing operations
- CodeSearch results on drag painting implementation patterns

### Tertiary (LOW confidence)

- Web research on QR code alignment pattern positioning (may need validation against spec)

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - Native Canvas API, well-documented
- Architecture: HIGH - Clear layering pattern, existing codebase foundation
- Pitfalls: MEDIUM-HIGH - Common issues well-documented in community

**Research date:** 2026-02-13
**Valid until:** 2026-03-13 (30 days for stable Canvas API)
