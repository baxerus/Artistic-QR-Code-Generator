# Phase 02: Pixel Painting & Corruption - Research

**Researched:** 2026-02-07
**Domain:** Canvas pixel manipulation, QR code structure, JavaScript QR decoding
**Confidence:** MEDIUM

## Summary

Phase 02 enables interactive pixel painting on a canvas grid matching QR code dimensions, with three-state pixels (white/black/unset) that overlay onto QR codes to corrupt them artistically. The implementation requires HTML5 Canvas for pixel-perfect grid rendering, event handling for click-to-paint interactions, QR function pattern awareness to avoid corrupting critical areas, and QR decoding capabilities to measure error impact.

The standard approach uses vanilla JavaScript with HTML5 Canvas API for grid rendering and mouse/pointer event handling. Canvas provides native pixel manipulation via ImageData objects and coordinate mapping from click events to grid cells. QR code structure is well-defined with function patterns (finder, timing, alignment) that occupy fixed positions per version—these areas must be avoided when overlaying painted pixels.

The critical challenge is decoder error measurement. Standard JavaScript QR decoders (jsQR, html5-qrcode/zxing-js) expose binary success/failure, not Reed-Solomon error counts. Most return only decoded data and location information. True error counting requires either low-level Reed-Solomon decoder integration or building custom error analysis. QRazyBox demonstrates this is technically feasible but requires significant Reed-Solomon implementation work.

**Primary recommendation:** Build canvas grid with click-to-paint three-state cycling, calculate QR function pattern masks per version to prevent corruption of critical areas, and START with binary decode success/failure metrics while flagging error counting as a known gap requiring custom decoder work.

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| HTML5 Canvas API | Native | Pixel grid rendering and manipulation | Built into all modern browsers, no dependencies |
| Pointer Events API | Native | Unified touch/mouse interaction | Modern standard replacing separate mouse/touch handlers |
| ImageData | Native | Direct pixel buffer access | Standard Canvas API for pixel-level manipulation |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| jsQR | Latest | QR code decoding from canvas | Pure JavaScript, works standalone, returns location data |
| qrcodejs | Current (inlined) | QR generation | Already integrated in Phase 1 |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Canvas API | SVG with grid elements | SVG easier for click detection but slower for large grids, harder to extract pixel data for QR overlay |
| jsQR | html5-qrcode (wraps zxing-js) | html5-qrcode adds camera streaming features not needed, heavier bundle |
| Pointer Events | Separate mouse/touch handlers | Pointer Events unified, simpler code, better touch support |

**Installation:**
No installation needed—use native browser APIs. For jsQR decoder (if using):
```html
<!-- Inline from CDN or bundle -->
<script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
```

## Architecture Patterns

### Recommended Data Structure
```javascript
// Three-state pixel grid
class PixelGrid {
  constructor(size) {
    this.size = size;  // e.g., 25 for Version 2
    // Each cell: 0 = unset, 1 = white (locked), 2 = black (locked)
    this.cells = new Array(size * size).fill(0);
  }

  get(x, y) {
    return this.cells[y * this.size + x];
  }

  set(x, y, state) {
    this.cells[y * this.size + x] = state;
  }

  cycle(x, y) {
    const current = this.get(x, y);
    this.set(x, y, (current + 1) % 3);  // 0 → 1 → 2 → 0
  }

  clear() {
    this.cells.fill(0);
  }
}
```

### Pattern 1: Canvas Grid Rendering
**What:** Render pixel grid with visual distinction for three states
**When to use:** Initial render and after state changes
**Example:**
```javascript
// Source: MDN Canvas API + Eloquent JavaScript patterns
function renderGrid(ctx, grid, cellSize) {
  const size = grid.size;

  // Disable anti-aliasing for crisp pixels
  ctx.imageSmoothingEnabled = false;

  for (let y = 0; y < size; y++) {
    for (let x = 0; x < size; x++) {
      const state = grid.get(x, y);

      // Visual encoding for three states
      if (state === 0) {
        // Unset: light gray background
        ctx.fillStyle = '#f0f0f0';
        ctx.fillRect(x * cellSize, y * cellSize, cellSize, cellSize);
      } else if (state === 1) {
        // White locked: solid white
        ctx.fillStyle = '#ffffff';
        ctx.fillRect(x * cellSize, y * cellSize, cellSize, cellSize);
      } else if (state === 2) {
        // Black locked: solid black
        ctx.fillStyle = '#000000';
        ctx.fillRect(x * cellSize, y * cellSize, cellSize, cellSize);
      }

      // Grid lines for visual separation
      ctx.strokeStyle = '#cccccc';
      ctx.lineWidth = 1;
      ctx.strokeRect(x * cellSize, y * cellSize, cellSize, cellSize);
    }
  }
}
```

### Pattern 2: Click-to-Grid Coordinate Mapping
**What:** Convert mouse/touch events to grid cell coordinates
**When to use:** All pointer interaction handlers
**Example:**
```javascript
// Source: MDN getBoundingClientRect + Canvas tutorial patterns
function getGridCoordinates(canvas, event, cellSize) {
  const rect = canvas.getBoundingClientRect();

  // Browser coords to canvas coords
  const canvasX = event.clientX - rect.left;
  const canvasY = event.clientY - rect.top;

  // Canvas coords to grid coords
  const gridX = Math.floor(canvasX / cellSize);
  const gridY = Math.floor(canvasY / cellSize);

  return { x: gridX, y: gridY };
}

canvas.addEventListener('click', (event) => {
  const { x, y } = getGridCoordinates(canvas, event, cellSize);

  // Bounds check
  if (x >= 0 && x < grid.size && y >= 0 && y < grid.size) {
    grid.cycle(x, y);
    renderGrid(ctx, grid, cellSize);
  }
});
```

### Pattern 3: QR Function Pattern Masking
**What:** Calculate which grid cells contain QR function patterns (off-limits for painting)
**When to use:** Canvas initialization when QR version changes
**Example:**
```javascript
// Source: thonky.com QR code tutorial structure
function createFunctionPatternMask(version) {
  const size = 17 + (version * 4);
  const mask = new Array(size * size).fill(false);

  function markRect(x, y, width, height) {
    for (let dy = 0; dy < height; dy++) {
      for (let dx = 0; dx < width; dx++) {
        if (x + dx < size && y + dy < size && x + dx >= 0 && y + dy >= 0) {
          mask[(y + dy) * size + (x + dx)] = true;
        }
      }
    }
  }

  // Finder patterns (7×7) at three corners
  markRect(0, 0, 9, 9);  // Top-left (includes separator)
  markRect(size - 8, 0, 8, 9);  // Top-right
  markRect(0, size - 8, 9, 8);  // Bottom-left

  // Timing patterns (row 6 and column 6)
  for (let i = 0; i < size; i++) {
    mask[6 * size + i] = true;  // Horizontal timing
    mask[i * size + 6] = true;  // Vertical timing
  }

  // Alignment patterns (version-specific)
  const alignmentCoords = getAlignmentPatternCoords(version);
  for (const row of alignmentCoords) {
    for (const col of alignmentCoords) {
      // Skip if overlaps finder pattern
      if (!((row < 9 && col < 9) ||
            (row < 9 && col >= size - 8) ||
            (row >= size - 8 && col < 9))) {
        markRect(col - 2, row - 2, 5, 5);  // 5×5 alignment pattern
      }
    }
  }

  // Dark module (always at specific position)
  mask[(4 * version + 9) * size + 8] = true;

  // Format information areas (around separators)
  // Top-left horizontal
  for (let i = 0; i < 9; i++) {
    mask[8 * size + i] = true;
  }
  // Top-left vertical
  for (let i = 0; i < 8; i++) {
    mask[i * size + 8] = true;
  }
  // Top-right and bottom-left format areas
  for (let i = 0; i < 8; i++) {
    mask[8 * size + (size - 8 + i)] = true;  // Top-right horizontal
    mask[(size - 7 + i) * size + 8] = true;  // Bottom-left vertical
  }

  return mask;
}

function getAlignmentPatternCoords(version) {
  const table = {
    1: [],
    2: [6, 18],
    3: [6, 22],
    4: [6, 26],
    5: [6, 30],
    6: [6, 34],
    7: [6, 22, 38],
    8: [6, 24, 42]
  };
  return table[version] || [];
}
```

### Pattern 4: Canvas-to-QR Overlay
**What:** Apply painted pixels to QR code by extracting QR's ImageData and overwriting locked pixels
**When to use:** After QR generation, before display/decode
**Example:**
```javascript
// Source: MDN Canvas pixel manipulation
function overlayPaintedPixels(qrCanvas, paintedGrid, functionMask) {
  const ctx = qrCanvas.getContext('2d');
  const size = paintedGrid.size;

  // Get QR's pixel data
  const imageData = ctx.getImageData(0, 0, qrCanvas.width, qrCanvas.height);
  const cellSize = qrCanvas.width / size;

  for (let y = 0; y < size; y++) {
    for (let x = 0; x < size; x++) {
      const state = paintedGrid.get(x, y);

      // Skip unset pixels and function pattern areas
      if (state === 0 || functionMask[y * size + x]) continue;

      // Calculate pixel range for this grid cell
      const px = Math.floor(x * cellSize);
      const py = Math.floor(y * cellSize);
      const pw = Math.ceil(cellSize);
      const ph = Math.ceil(cellSize);

      // Fill with locked color
      const color = state === 1 ? 255 : 0;  // white : black

      for (let dy = 0; dy < ph; dy++) {
        for (let dx = 0; dx < pw; dx++) {
          const pixelX = px + dx;
          const pixelY = py + dy;

          if (pixelX < imageData.width && pixelY < imageData.height) {
            const index = (pixelY * imageData.width + pixelX) * 4;
            imageData.data[index] = color;      // R
            imageData.data[index + 1] = color;  // G
            imageData.data[index + 2] = color;  // B
            imageData.data[index + 3] = 255;    // A
          }
        }
      }
    }
  }

  ctx.putImageData(imageData, 0, 0);
}
```

### Pattern 5: QR Decoding from Canvas
**What:** Extract corrupted QR image data and attempt decode
**When to use:** After overlay, to measure decode success and error tolerance
**Example:**
```javascript
// Source: jsQR library usage pattern
function decodeQRFromCanvas(canvas) {
  const ctx = canvas.getContext('2d');
  const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);

  // jsQR expects ImageData: { data: Uint8ClampedArray, width, height }
  const code = jsQR(imageData.data, imageData.width, imageData.height);

  if (code) {
    return {
      success: true,
      data: code.data,
      version: code.version,
      location: code.location,
      // Note: jsQR does NOT expose error count
      errorCount: null  // Not available from standard decoders
    };
  } else {
    return {
      success: false,
      data: null,
      errorCount: null
    };
  }
}
```

### Anti-Patterns to Avoid
- **DOM-per-pixel:** Creating a DOM element for each grid cell—use Canvas instead for performance
- **Fractional coordinates without rounding:** Always use Math.floor/ceil when mapping clicks to grid cells
- **Painting over function patterns:** Always check function pattern mask before allowing paint
- **Assuming error counts are available:** Standard decoders return binary success, not error counts
- **SVG QR manipulation:** QR renders as SVG from qrcodejs, but convert to Canvas for pixel-level corruption

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| QR code decoding | Custom QR decoder | jsQR or zxing-js | QR decoding involves complex Reed-Solomon math, error correction, mask detection—jsQR is battle-tested |
| Pointer event unification | Separate mouse/touch handlers | Pointer Events API | Native API handles mouse, touch, pen with single event model |
| Pixel coordinate math | Manual offset calculations | getBoundingClientRect() | Handles scroll, zoom, transforms automatically |
| QR function pattern positions | Hardcoded coordinates | Calculated from QR spec formula | Version-dependent, error-prone to hardcode |
| Reed-Solomon error correction | Custom RS implementation | Existing RS libraries OR accept binary decode only | RS math is complex, existing implementations are robust |

**Key insight:** QR code structure and decoding are mathematically complex with many edge cases. Use existing libraries for encoding/decoding. Canvas API provides all needed pixel manipulation primitives—use them directly rather than abstraction layers.

## Common Pitfalls

### Pitfall 1: Canvas Coordinate Blurriness
**What goes wrong:** Grid lines appear blurry or anti-aliased at certain zoom levels
**Why it happens:** Canvas coordinates at whole numbers (0, 1, 2) mark pixel *edges*, not centers. Lines drawn between whole coordinates span two pixels.
**How to avoid:**
- Set `ctx.imageSmoothingEnabled = false` for pixel-perfect rendering
- Use integer pixel sizes for grid cells
- For crisp 1px lines, draw at half-pixel offsets (0.5, 1.5, etc.) OR use fillRect instead of strokeRect
**Warning signs:** Grid appears fuzzy or inconsistent thickness across zoom levels

### Pitfall 2: Click Detection on Scaled Canvas
**What goes wrong:** Clicks register at wrong grid cells when canvas is CSS-scaled
**Why it happens:** `getBoundingClientRect()` returns CSS size, but click coordinates may need adjustment if canvas internal size differs from display size
**How to avoid:**
- Keep canvas internal dimensions (width/height attributes) proportional to display size
- Always calculate cell size as `canvas.width / gridSize` (internal, not CSS size)
- Use `event.clientX - rect.left` for canvas-relative coordinates
**Warning signs:** Click offset increases toward edges of canvas, misalignment on mobile

### Pitfall 3: Corrupting QR Function Patterns
**What goes wrong:** QR becomes undecodable even with minimal corruption
**Why it happens:** Finder patterns, timing patterns, alignment patterns are essential for QR detection—corrupting them breaks the decoder before it reaches data area
**How to avoid:**
- Calculate function pattern mask when version changes
- Always check mask before applying painted pixel
- Test with minimal corruption to verify mask is correct
**Warning signs:** QR fails to decode with only a few painted pixels, decoder can't locate QR at all

### Pitfall 4: Assuming Error Counts are Available
**What goes wrong:** Plan includes "show Reed-Solomon error count" but decoders only return success/failure
**Why it happens:** Most JavaScript QR libraries wrap high-level decoders that don't expose internal error correction details
**How to avoid:**
- Check decoder API documentation for what data is returned
- Plan for binary decode success initially
- Flag error counting as advanced feature requiring custom decoder work
- Consider jsQR source code modification OR separate RS implementation
**Warning signs:** No decoder documentation mentions error counts, returned object only has data/location fields

### Pitfall 5: State Cycling Without Bounds
**What goes wrong:** State value goes beyond expected 0-2 range
**Why it happens:** Increment without modulo can overflow with repeated clicks
**How to avoid:**
- Always use `(current + 1) % 3` for three-state cycling
- Validate state values when loading saved grids
- Use constants for state values (UNSET=0, WHITE=1, BLACK=2)
**Warning signs:** Rendering breaks after many clicks on same cell, unexpected colors appear

## Code Examples

### Complete Three-State Click Handler
```javascript
// Source: Combining MDN patterns with three-state logic
class PaintableCanvas {
  constructor(canvasId, gridSize) {
    this.canvas = document.getElementById(canvasId);
    this.ctx = this.canvas.getContext('2d');
    this.grid = new PixelGrid(gridSize);
    this.cellSize = this.canvas.width / gridSize;

    // Disable smoothing for crisp pixels
    this.ctx.imageSmoothingEnabled = false;

    // Pointer event (handles mouse + touch)
    this.canvas.addEventListener('pointerdown', (e) => this.handlePaint(e));

    this.render();
  }

  handlePaint(event) {
    const rect = this.canvas.getBoundingClientRect();
    const canvasX = event.clientX - rect.left;
    const canvasY = event.clientY - rect.top;

    const gridX = Math.floor(canvasX / this.cellSize);
    const gridY = Math.floor(canvasY / this.cellSize);

    // Bounds check
    if (gridX >= 0 && gridX < this.grid.size &&
        gridY >= 0 && gridY < this.grid.size) {
      this.grid.cycle(gridX, gridY);
      this.render();
    }
  }

  render() {
    for (let y = 0; y < this.grid.size; y++) {
      for (let x = 0; x < this.grid.size; x++) {
        const state = this.grid.get(x, y);

        // State-specific colors
        if (state === 0) {
          this.ctx.fillStyle = '#f0f0f0';  // Unset: light gray
        } else if (state === 1) {
          this.ctx.fillStyle = '#ffffff';  // White locked
        } else {
          this.ctx.fillStyle = '#000000';  // Black locked
        }

        this.ctx.fillRect(
          x * this.cellSize,
          y * this.cellSize,
          this.cellSize,
          this.cellSize
        );

        // Grid lines
        this.ctx.strokeStyle = '#cccccc';
        this.ctx.lineWidth = 1;
        this.ctx.strokeRect(
          x * this.cellSize,
          y * this.cellSize,
          this.cellSize,
          this.cellSize
        );
      }
    }
  }

  clear() {
    this.grid.clear();
    this.render();
  }
}
```

### QR Decode with Error Display
```javascript
// Source: jsQR usage + binary success pattern
function decodeAndDisplay(qrCanvas) {
  const ctx = qrCanvas.getContext('2d');
  const imageData = ctx.getImageData(0, 0, qrCanvas.width, qrCanvas.height);
  const code = jsQR(imageData.data, imageData.width, imageData.height);

  const resultDiv = document.getElementById('decode-result');

  if (code) {
    resultDiv.innerHTML = `
      <div class="success">✓ Decode successful</div>
      <div>Data: ${code.data}</div>
      <div>Version: ${code.version}</div>
      <div class="note">Error count: Not available (binary decode only)</div>
    `;
  } else {
    resultDiv.innerHTML = `
      <div class="failure">✗ Decode failed</div>
      <div>QR code could not be decoded</div>
    `;
  }
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Separate mouse/touch handlers | Pointer Events API | ~2015-2017 | Unified event model, simpler code, better stylus support |
| Manual pixel manipulation loops | ImageData + putImageData | Native since Canvas | Direct buffer access, fast bulk updates |
| Table-based layouts for grids | Canvas-based rendering | HTML5 era | Performance with large grids (40×40+), pixel-perfect control |
| External decoder dependencies | Pure JS decoders (jsQR) | ~2016+ | Runs in browser without WASM, easier integration |

**Deprecated/outdated:**
- Flash-based QR readers: Obsolete, browsers no longer support Flash
- jQuery for Canvas manipulation: Unnecessary overhead, use vanilla APIs
- Separate mobile/desktop UIs: Pointer Events + responsive CSS handles all devices

## Open Questions

### 1. Reed-Solomon Error Count Exposure
**What we know:**
- Standard JS decoders (jsQR, zxing-js) return binary success/failure
- QRazyBox demonstrates error counting is technically possible
- Reed-Solomon decoders calculate error count internally but don't expose it

**What's unclear:**
- Can jsQR be modified to expose error count without major refactoring?
- Is there a lightweight RS library that integrates with existing decoders?
- Would forking jsQR and exposing internal RS state be maintainable?

**Recommendation:**
- Phase 2 implements binary decode success/failure as MVP
- Display "Decode: Success/Failed" instead of error count
- Document error counting as Phase 3+ enhancement requiring custom decoder work
- Consider QRazyBox approach: separate RS decoder that accepts QR data blocks

### 2. Function Pattern Mask Accuracy
**What we know:**
- Finder patterns: 7×7 at three corners with 1-module separator
- Timing patterns: Row/column 6
- Alignment patterns: Version-specific coordinates from table
- Format info: Strips around separators
- Version info: Present in versions 7+ (6×3 blocks)

**What's unclear:**
- Do all QR generators (including qrcodejs) follow spec exactly?
- Are there version-specific variations in function pattern placement?
- Should mask include quiet zone (though not strictly function pattern)?

**Recommendation:**
- Implement mask based on official QR spec (thonky.com reference)
- Test with qrcodejs output to verify alignment
- Add visual debug mode showing mask overlay
- Validate with minimal corruption tests (paint 1-2 non-function pixels, verify decode still works)

### 3. Optimal Canvas Size for Grid
**What we know:**
- QR modules range from 25×25 (v2) to 49×49 (v8)
- Canvas should be large enough for visual clarity
- Pixel-perfect requires integer cell sizes

**What's unclear:**
- Best canvas pixel dimensions for each version?
- Should canvas resize dynamically or stay fixed?
- Trade-off between large canvas (clear grid) and performance?

**Recommendation:**
- Use fixed canvas size (e.g., 400×400 or 500×500)
- Calculate cellSize = canvasSize / gridSize at runtime
- Accept fractional cell sizes but round when rendering
- Test on mobile for touch target size (cells should be >= 10-12px for easy tapping)

## Sources

### Primary (HIGH confidence)
- [MDN Canvas API: Pixel Manipulation](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Pixel_manipulation_with_canvas) - Canvas ImageData, getImageData, putImageData patterns
- [Thonky QR Code Tutorial: Module Placement](https://www.thonky.com/qr-code-tutorial/module-placement-matrix) - QR function pattern structure
- [Thonky QR Code Tutorial: Alignment Patterns](https://www.thonky.com/qr-code-tutorial/alignment-pattern-locations) - Version-specific alignment coordinates
- [DENSO WAVE: Error Correction](https://www.qrcode.com/en/about/error_correction.html) - Official QR error correction levels (L/M/Q/H percentages)

### Secondary (MEDIUM confidence)
- [Eloquent JavaScript: Pixel Art Editor](https://eloquentjavascript.net/19_paint.html) - Canvas painting architecture, state management patterns
- [jsQR GitHub](https://github.com/cozmo/jsQR) - Pure JS QR decoder, return data structure
- [html5-qrcode GitHub](https://github.com/mebjas/html5-qrcode) - Alternative decoder using zxing-js
- [QRazyBox Reed-Solomon Decoder](https://merri.cx/qrazybox/help/extension-tools/reed-solomon-decoder.html) - Error counting capabilities demonstration

### Tertiary (LOW confidence - needs validation)
- [Medium: State Management in Vanilla JS 2026](https://medium.com/@chirag.dave/state-management-in-vanilla-js-2026-trends-f9baed7599de) - Modern patterns (Proxy-based reactivity)
- WebSearch results on artistic QR code patterns - Community approaches to QR corruption

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - Canvas API and Pointer Events are well-documented web standards
- Architecture patterns: HIGH - Patterns verified from MDN, Eloquent JavaScript, and QR spec
- QR function patterns: HIGH - Based on official QR spec and authoritative tutorials
- Decoder capabilities: MEDIUM - Verified jsQR exists and works, but error count limitation confirmed through documentation gaps
- Error counting solution: LOW - QRazyBox demonstrates feasibility but implementation path unclear

**Research date:** 2026-02-07
**Valid until:** ~2026-04-07 (60 days - stable web standards, slow-moving QR spec)

**Key gaps requiring validation during implementation:**
1. Verify jsQR return structure matches documentation (or use alternative decoder)
2. Test function pattern mask accuracy against qrcodejs output
3. Determine if error counting is truly required or if binary decode is sufficient for Phase 2 success criteria
4. Validate three-state visual distinction is clear on different displays
