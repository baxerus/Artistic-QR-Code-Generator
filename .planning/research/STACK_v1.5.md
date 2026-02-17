# Stack Additions for v1.5 Features

**Project:** QR Art Tool v1.5 Enhancements
**Researched:** Tue Feb 17 2026
**Overall Confidence:** HIGH

## Executive Summary

All three v1.5 features can be implemented with **zero external dependencies** using existing browser APIs and the codebase's already-inlined jsQR VERSIONS table. This research provides concrete implementation patterns for each feature.

**Key Finding:** The existing codebase already contains all required data structures. No new libraries needed.

---

## Feature 1: Reed-Solomon Correction Capacity Display

### Data Source: Existing VERSIONS Table

The jsQR library already inlined in `index.html` (lines 10856-12155) contains the complete VERSIONS table with all RS error correction data for QR versions 1-40, all four EC levels.

**Structure per version (verified from codebase line 10857-10878):**
```javascript
{
  versionNumber: N,
  errorCorrectionLevels: [
    // Index 0: L, Index 1: M, Index 2: Q, Index 3: H
    {
      ecCodewordsPerBlock: N,  // RS codewords per block
      ecBlocks: [
        { numBlocks: N, dataCodewordsPerBlock: N },
        // Optional second block group with different size
      ]
    }
  ]
}
```

### Calculation Formula

Reed-Solomon can correct `t` symbol errors where `2t = n - k` (n = total codewords, k = data codewords). In QR codes:

```
maxCorrectableErrorsPerBlock = ecCodewordsPerBlock / 2
totalMaxCorrections = maxCorrectableErrorsPerBlock × totalBlocks
```

**Implementation:**

```javascript
/**
 * Calculate maximum RS correction capacity for a QR version/EC level
 * @param {number} versionNumber - QR version (1-40)
 * @param {number} ecLevel - 0=L, 1=M, 2=Q, 3=H (app uses 3=H)
 * @returns {Object} Capacity info
 */
function getMaxCorrectionCapacity(versionNumber, ecLevel = 3) {
  // VERSIONS array is 0-indexed, version numbers are 1-indexed
  const version = VERSIONS[versionNumber - 1];
  if (!version) return null;
  
  const ecInfo = version.errorCorrectionLevels[ecLevel];
  
  // Count total blocks across all block groups
  let totalBlocks = 0;
  for (const blockGroup of ecInfo.ecBlocks) {
    totalBlocks += blockGroup.numBlocks;
  }
  
  // RS can correct ecCodewordsPerBlock/2 errors per block
  const maxPerBlock = Math.floor(ecInfo.ecCodewordsPerBlock / 2);
  const maxCorrections = maxPerBlock * totalBlocks;
  
  return {
    maxCorrections,
    totalBlocks,
    ecCodewordsPerBlock: ecInfo.ecCodewordsPerBlock,
    maxPerBlock
  };
}
```

### Accessing the VERSIONS Table

The VERSIONS table is exported at line 10856 in the webpack bundle. Two access approaches:

**Option A: Extract reference after jsQR execution (Recommended)**

jsQR's webpack bundle assigns to `exports.VERSIONS`. After the IIFE executes, extract:

```javascript
// The jsQR IIFE returns the jsQR function but also populates modules
// Look for the versions module (module index 10 in the bundle)
// The VERSIONS array is available via jsQR internals

// Simplest: add extraction during jsQR initialization
// In the existing jsQR inlined code, after line 12155, add:
window.__JSQR_VERSIONS__ = exports.VERSIONS;
```

**Option B: Inline a minimal lookup table**

If modifying jsQR feels risky, copy only the EC data needed (~40 entries):

```javascript
const QR_EC_CAPACITY_H = [
  // [version, ecCodewordsPerBlock, totalBlocks]
  [1, 17, 1],   // Version 1: 17/2 * 1 = 8 max corrections
  [2, 28, 1],   // Version 2: 28/2 * 1 = 14 max corrections
  [3, 22, 2],   // Version 3: 22/2 * 2 = 22 max corrections
  [4, 16, 4],   // Version 4: 16/2 * 4 = 32 max corrections
  // ... etc for versions the app uses
];
```

**Recommendation:** Option A is cleaner since the data already exists in memory.

### Integration Point

The existing UI shows corrections at line 16274-16276:
```javascript
const rsDisplay = result.rsCorrections === Infinity ? "-" : result.rsCorrections;
correctionSpan.innerHTML = `Corrections: <b>${rsDisplay}</b>`;
```

**Enhanced display:**
```javascript
function formatCorrectionStatus(usedCorrections, versionNumber) {
  const capacity = getMaxCorrectionCapacity(versionNumber, 3); // H level
  
  if (usedCorrections === Infinity) {
    return `Corrections: <b>-</b> / ${capacity.maxCorrections}`;
  }
  
  const remaining = capacity.maxCorrections - usedCorrections;
  const percent = Math.round((usedCorrections / capacity.maxCorrections) * 100);
  
  // Color-code based on usage
  let colorClass = 'correction-ok';      // green
  if (percent > 75) colorClass = 'correction-warn';   // yellow
  if (percent > 90) colorClass = 'correction-danger'; // red
  
  return `Corrections: <b class="${colorClass}">${usedCorrections}</b> / ${capacity.maxCorrections} (${remaining} remaining)`;
}
```

### Version-Specific Capacity Table (H Level, Verified from Codebase)

| Version | ecCodewordsPerBlock | Total Blocks | Max Correctable |
|---------|---------------------|--------------|-----------------|
| 1 | 17 | 1 | 8 |
| 2 | 28 | 1 | 14 |
| 3 | 22 | 2 | 22 |
| 4 | 16 | 4 | 32 |
| 5 | 22 | 2+2=4 | 44 |
| 6 | 28 | 4 | 56 |

**Confidence:** HIGH — Direct analysis of inlined jsQR source code (lines 10856-12155).

---

## Feature 2: SVG Export

### No External Libraries Needed

SVG generation is trivial with template strings. The QR module data is a 2D boolean grid — perfect for SVG `<rect>` elements.

### Existing Data Sources

Result objects (line 15362) contain:
```javascript
{
  moduleData,     // Uint8Array - painted state per module
  qrModuleData,   // Original QR modules (for comparison)
  moduleCount     // Grid size (e.g., 33 for Version 4)
}
```

The `renderResultToCanvas` function (referenced at line 16308) already reconstructs the visual QR from this data.

### Implementation Pattern

**Option A: Simple Rect-Based SVG**

```javascript
/**
 * Generate SVG string from QR module data
 * @param {Uint8Array|Array} moduleData - Module states (0=white, 1+=dark)
 * @param {number} moduleCount - Grid dimension
 * @param {Object} options - SVG options
 * @returns {string} Complete SVG document
 */
function generateQRSvg(moduleData, moduleCount, options = {}) {
  const {
    cellSize = 10,
    margin = 4,           // QR spec quiet zone
    foreground = '#000000',
    background = '#ffffff'
  } = options;
  
  const qrSize = moduleCount * cellSize;
  const totalSize = qrSize + (margin * 2 * cellSize);
  const offset = margin * cellSize;
  
  // SVG header with XML declaration for standalone use
  let svg = `<?xml version="1.0" encoding="UTF-8"?>
<svg xmlns="http://www.w3.org/2000/svg" 
     version="1.1" 
     viewBox="0 0 ${totalSize} ${totalSize}" 
     width="${totalSize}" 
     height="${totalSize}">
  <rect width="100%" height="100%" fill="${background}"/>
`;

  // Generate rect for each dark module
  for (let y = 0; y < moduleCount; y++) {
    for (let x = 0; x < moduleCount; x++) {
      const idx = y * moduleCount + x;
      // moduleData values: 0=unpainted, 1=painted white, 2=painted black
      // For export: check if rendered dark (painted black or original dark when unpainted)
      const isDark = moduleData[idx] === 2 || 
                     (moduleData[idx] === 0 && originalIsDark(x, y));
      
      if (isDark) {
        const px = offset + x * cellSize;
        const py = offset + y * cellSize;
        svg += `  <rect x="${px}" y="${py}" width="${cellSize}" height="${cellSize}" fill="${foreground}"/>\n`;
      }
    }
  }
  
  svg += `</svg>`;
  return svg;
}
```

**Option B: Path-Based SVG (Smaller Files)**

Combines all dark modules into a single `<path>` element:

```javascript
function generateQRSvgOptimized(moduleData, moduleCount, options = {}) {
  const {
    cellSize = 10,
    margin = 4,
    foreground = '#000000',
    background = '#ffffff'
  } = options;
  
  const totalSize = (moduleCount + margin * 2) * cellSize;
  const offset = margin * cellSize;
  
  // Build path data
  let pathData = '';
  for (let y = 0; y < moduleCount; y++) {
    for (let x = 0; x < moduleCount; x++) {
      const idx = y * moduleCount + x;
      const isDark = isDarkModule(moduleData, idx, x, y);
      
      if (isDark) {
        const px = offset + x * cellSize;
        const py = offset + y * cellSize;
        // M=move, h=horizontal, v=vertical, z=close
        pathData += `M${px} ${py}h${cellSize}v${cellSize}h${-cellSize}z`;
      }
    }
  }
  
  return `<?xml version="1.0" encoding="UTF-8"?>
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 ${totalSize} ${totalSize}">
  <rect width="100%" height="100%" fill="${background}"/>
  <path d="${pathData}" fill="${foreground}"/>
</svg>`;
}
```

### Download Implementation

Follows same pattern as existing PNG download (line 16078-16106):

```javascript
/**
 * Download SVG string as file
 * @param {string} svgString - Complete SVG document
 * @param {string} filename - Filename without extension
 */
function downloadSvg(svgString, filename) {
  const blob = new Blob([svgString], { type: 'image/svg+xml;charset=utf-8' });
  const url = URL.createObjectURL(blob);
  
  const a = document.createElement('a');
  a.href = url;
  a.download = `${filename}.svg`;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  
  URL.revokeObjectURL(url);
}
```

### Integration Point

Add SVG button next to existing PNG button (line 16304-16319):

```javascript
// After downloadBtn (PNG), add:
const downloadSvgBtn = document.createElement("button");
downloadSvgBtn.textContent = "Download SVG";
downloadSvgBtn.disabled = isRunning;
downloadSvgBtn.onclick = () => {
  const svg = generateQRSvg(
    result.moduleData,
    result.moduleCount,
    { cellSize: 10, margin: 4 }
  );
  downloadSvg(svg, sanitizedUrl);
};
actions.appendChild(downloadSvgBtn);
```

**Confidence:** HIGH — Standard browser APIs (Blob, URL.createObjectURL), no dependencies.

---

## Feature 3: Shift+Paint for Color Toggle

### Existing Event Structure

The painting system (lines 13555-13604) uses Pointer Events:

```javascript
container.addEventListener("pointerdown", function (e) {
  if (e.button !== 0 && e.button !== 2) return;
  // ...
  currentPaintButton = e.button;
  // ...
});
```

Current behavior:
- Left click (button 0): Paint selected color
- Right click (button 2): Paint opposite color

### Modifier Key Access

All Pointer Events inherit from MouseEvent and include:

```javascript
event.shiftKey   // boolean - Shift key state
event.ctrlKey    // boolean - Ctrl key state  
event.altKey     // boolean - Alt key state
event.metaKey    // boolean - Meta (Cmd/Win) key state
```

**These are read-only properties available on every pointer event.**

### Implementation

**Step 1: Modify `getPaintValue` (line 13469)**

```javascript
/**
 * Get paint value based on mouse button, selected color, and modifiers
 * @param {number} button - Mouse button (0=left, 2=right)
 * @param {boolean} shiftKey - Whether Shift is held
 */
function getPaintValue(button, shiftKey = false) {
  // Shift inverts the effective color
  let effectiveColor = selectedColor;
  if (shiftKey && selectedColor !== "unset") {
    effectiveColor = selectedColor === "black" ? "white" : "black";
  }
  
  if (button === 0) {
    // Left click: paint effective color
    if (effectiveColor === "black") return 2;
    if (effectiveColor === "white") return 1;
    if (effectiveColor === "unset") return 0;
  } else if (button === 2) {
    // Right click: paint opposite of effective color
    if (effectiveColor === "unset") return 0;
    if (effectiveColor === "black") return 1;
    if (effectiveColor === "white") return 2;
  }
  return null;
}
```

**Step 2: Store shift state on paint start (line 13559)**

```javascript
let currentShiftState = false; // Add to module-level state

container.addEventListener("pointerdown", function (e) {
  if (e.button !== 0 && e.button !== 2) return;
  if (searchState === "running") return;

  e.preventDefault();
  
  isPainting = true;
  currentPaintButton = e.button;
  currentShiftState = e.shiftKey; // Store shift state at paint start
  lastPaintedCell = null;
  container.setPointerCapture(e.pointerId);

  const canvas = container.querySelector("canvas");
  if (canvas) {
    const coords = getGridCoordinates(canvas, e);
    paintAt(coords.x, coords.y, currentPaintButton, currentShiftState);
  }
});
```

**Step 3: Update `paintAt` to accept shift state (line 13509)**

```javascript
function paintAt(x, y, button, shiftKey = false) {
  // ... bounds check ...
  
  const paintValue = getPaintValue(button, shiftKey);
  if (paintValue === null) return;
  
  // ... rest of function unchanged ...
}
```

**Step 4: Pass shift state in pointermove (line 13577)**

```javascript
container.addEventListener("pointermove", function (e) {
  if (!isPainting) return;

  const canvas = container.querySelector("canvas");
  if (canvas) {
    const coords = getGridCoordinates(canvas, e);
    paintAt(coords.x, coords.y, currentPaintButton, currentShiftState);
  }
});
```

### UX Decision: Locked vs Real-time Shift

**Locked (Recommended):** Store `shiftKey` on `pointerdown`, use that value for entire stroke.
- Consistent with existing `currentPaintButton` behavior
- Prevents jarring color changes mid-stroke
- Simpler mental model

**Real-time:** Check `e.shiftKey` on each `pointermove`.
- More flexible
- May be confusing if color changes unexpectedly

**Recommendation:** Locked at paint-start.

### Optional: Cursor Feedback

Update cursor to indicate shift state (visual feedback):

```javascript
// In updateCursorForPosition or mousemove handler
function updateCursorHint(e, wrapper) {
  if (e.shiftKey && selectedColor !== "unset") {
    wrapper.style.cursor = 'crosshair'; // Indicate inversion mode
  } else {
    wrapper.style.cursor = 'pointer';
  }
}
```

Or add a CSS class:

```css
#qr-canvas-container.shift-mode {
  cursor: url('data:image/svg+xml,...'), crosshair;
}
```

**Confidence:** HIGH — Standard DOM event properties, tested across all browsers.

---

## Summary: No New Dependencies

| Feature | Implementation | New Dependencies |
|---------|---------------|------------------|
| Correction Capacity | Extract from existing VERSIONS table | None |
| SVG Export | Template strings + Blob API | None |
| Shift+Paint | `event.shiftKey` property | None |

### Changes Required

| Feature | Files/Functions to Modify |
|---------|--------------------------|
| Correction Capacity | Add utility function, modify UI near line 16275 |
| SVG Export | Add `generateQRSvg` + `downloadSvg` functions, add button near line 16304 |
| Shift+Paint | Modify `getPaintValue` (line 13469), event handlers (lines 13559, 13577) |

### Estimated Code Additions

| Feature | Lines of Code | Complexity |
|---------|---------------|------------|
| Correction Capacity | ~30 lines | Low |
| SVG Export | ~50 lines | Low |
| Shift+Paint | ~20 lines (modifications) | Low |

**Total:** ~100 lines of new/modified code, zero dependencies.

---

## Sources

- **jsQR VERSIONS table:** Lines 10856-12155 of `/workspace/index.html`
- **Existing PNG export:** Lines 16078-16106 of `/workspace/index.html`
- **Existing paint system:** Lines 13469-13604 of `/workspace/index.html`
- **MDN PointerEvent.shiftKey:** Inherited from MouseEvent, standard property
- **SVG 1.1 Specification:** Inline SVG uses standard elements (rect, path)
- **Reed-Solomon theory:** RS can correct t errors where 2t = n-k codewords
