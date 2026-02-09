# Phase 4: Results & Export - Research

**Researched:** 2026-02-09
**Domain:** Canvas export, Clipboard API, UI display patterns, Canvas overlays
**Confidence:** HIGH

## Summary

This phase focuses on presenting the top 5 optimization results to users and enabling export/copy functionality. The research covers four main technical domains: canvas-to-PNG download, clipboard copy operations, results display UI patterns, and error visualization overlays.

The standard approach uses HTML Canvas API's `toBlob()` method (preferred over `toDataURL()` for performance) combined with anchor download attributes for file downloads. The modern Clipboard API's `writeText()` provides secure, promise-based text copying. Error visualization overlays use canvas transparency (RGBA colors or globalAlpha) to distinguish pixel types. Results display can use simple DOM structures with CSS Grid/Flexbox or existing inline patterns.

Since this project uses a single HTML file with no external dependencies, all functionality must be vanilla JavaScript. The existing codebase already tracks top 5 results with moduleData arrays, making export implementation straightforward.

**Primary recommendation:** Use canvas.toBlob() with dynamic anchor downloads for PNG export, navigator.clipboard.writeText() for URL copying, and RGBA semi-transparent overlays on a separate canvas for error visualization. Build results table with vanilla DOM manipulation following existing UI patterns.

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Canvas API | Native | Rendering QR codes and overlays | Browser-native, no dependencies needed |
| Clipboard API | Native | Copying URLs to clipboard | Modern, promise-based replacement for execCommand |
| Blob API | Native | Binary data for downloads | Required for canvas.toBlob() downloads |
| URL.createObjectURL | Native | Temporary URLs for blobs | Memory-efficient download URLs |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| N/A | N/A | Single HTML file constraint | No external libraries allowed |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| toBlob() | toDataURL() | toDataURL() is synchronous and blocks main thread, worse performance for large images, may hit URL length limits |
| Clipboard API | document.execCommand('copy') | execCommand is deprecated, no longer guaranteed to work, not promise-based |
| RGBA colors | globalAlpha property | globalAlpha affects all subsequent draws (more state management), RGBA is more precise per-element |

**Installation:**
```bash
# No installation needed - all browser-native APIs
# Project constraint: single HTML file, no external dependencies
```

## Architecture Patterns

### Recommended Structure
```
Results Display Section:
├── Top 5 slots (already implemented)
│   ├── QR preview canvas (80x80px thumbnails)
│   ├── Metadata (hash, errors, decodability)
│   └── Action buttons (download, copy URL)
├── Expanded result view (new)
│   ├── Full-size QR canvas
│   ├── Error overlay canvas (same size, layered)
│   └── Action buttons (download, copy)
└── URL construction (hash fragment)
```

### Pattern 1: Canvas-to-PNG Download
**What:** Export canvas as downloadable PNG file using Blob API
**When to use:** User clicks download button on a result
**Example:**
```javascript
// Source: MDN - HTMLCanvasElement.toBlob()
// https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toBlob
function downloadCanvasAsPNG(canvas, filename) {
  canvas.toBlob((blob) => {
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    a.click();

    // Clean up object URL to free memory
    URL.revokeObjectURL(url);
  }, 'image/png');
}

// Usage:
const canvas = document.getElementById('qr-canvas');
downloadCanvasAsPNG(canvas, 'qr-code.png');
```

### Pattern 2: Clipboard Copy with Feedback
**What:** Copy text to clipboard with async/await and user feedback
**When to use:** User clicks "Copy URL" button
**Example:**
```javascript
// Source: MDN - Clipboard.writeText()
// https://developer.mozilla.org/en-US/docs/Web/API/Clipboard/writeText
async function copyToClipboard(text, button) {
  try {
    await navigator.clipboard.writeText(text);

    // Visual feedback
    const originalText = button.textContent;
    button.textContent = '✓ Copied!';
    button.disabled = true;

    setTimeout(() => {
      button.textContent = originalText;
      button.disabled = false;
    }, 2000);
  } catch (error) {
    console.error('Failed to copy:', error);
    // Fallback or error message
  }
}

// Security requirement: Must be called from user interaction (button click)
button.addEventListener('click', () => {
  const url = 'https://example.com#abc123';
  copyToClipboard(url, button);
});
```

### Pattern 3: Semi-Transparent Error Overlay
**What:** Layer transparent canvas on top of QR code to highlight pixel types
**When to use:** Displaying error visualization overlay
**Example:**
```javascript
// Source: MDN - Applying styles and colors
// https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Applying_styles_and_colors
function renderErrorOverlay(canvas, moduleCount, paintedPixels, functionMask, qrModules) {
  const scale = Math.floor(canvas.width / moduleCount);
  const ctx = canvas.getContext('2d');
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  for (let row = 0; row < moduleCount; row++) {
    for (let col = 0; col < moduleCount; col++) {
      const idx = row * moduleCount + col;
      const paintedState = paintedPixels[idx];
      const isFunctionPattern = functionMask[idx];
      const qrValue = qrModules[idx];

      let color = null;

      if (isFunctionPattern) {
        // Function pattern pixels (finder patterns, timing, etc) - blue
        color = 'rgb(0 100 255 / 30%)';
      } else if (paintedState !== 0) {
        // Locked pattern pixels - check if matches QR
        const paintedIsDark = (paintedState === 2);
        const qrIsDark = (qrValue === 1);

        if (paintedIsDark === qrIsDark) {
          // Match - green overlay
          color = 'rgb(0 200 0 / 25%)';
        } else {
          // Conflict - red overlay
          color = 'rgb(255 0 0 / 40%)';
        }
      }
      // Unset pixels (paintedState === 0, not function) - no overlay

      if (color) {
        ctx.fillStyle = color;
        ctx.fillRect(col * scale, row * scale, scale, scale);
      }
    }
  }
}
```

### Pattern 4: Results Table Display
**What:** Display top 5 results in ranked order with actions
**When to use:** After optimization completes
**Example:**
```javascript
// Source: Existing project pattern (index.html lines 14517-14583)
function displayResultsTable(topResults, baseURL, version) {
  const container = document.getElementById('results-container');
  container.innerHTML = '';

  topResults.forEach((result, index) => {
    const resultCard = document.createElement('div');
    resultCard.className = 'result-card';

    // Rank and metadata
    const rank = document.createElement('div');
    rank.className = 'result-rank';
    rank.textContent = `#${index + 1}`;

    // QR preview canvas
    const canvas = document.createElement('canvas');
    canvas.width = result.moduleCount;
    canvas.height = result.moduleCount;
    renderModuleData(canvas, result.moduleData, result.moduleCount);

    // Metadata
    const meta = document.createElement('div');
    meta.innerHTML = `
      <div>Hash: ${result.hash}</div>
      <div>Errors: ${result.decodable ? 0 : result.pixelDiff}</div>
      <div>Status: ${result.decodable ? '✓ Decodable' : '✗ Not decodable'}</div>
    `;

    // Action buttons
    const actions = document.createElement('div');

    const downloadBtn = document.createElement('button');
    downloadBtn.textContent = 'Download PNG';
    downloadBtn.onclick = () => downloadResult(result);

    const copyBtn = document.createElement('button');
    copyBtn.textContent = 'Copy URL';
    copyBtn.onclick = () => copyToClipboard(`${baseURL}#${result.hash}`, copyBtn);

    actions.appendChild(downloadBtn);
    actions.appendChild(copyBtn);

    resultCard.append(rank, canvas, meta, actions);
    container.appendChild(resultCard);
  });
}
```

### Anti-Patterns to Avoid
- **Using toDataURL() for downloads:** Blocks main thread, worse performance, may exceed URL length limits. Use toBlob() instead.
- **execCommand('copy') for clipboard:** Deprecated API, no longer guaranteed to work. Use Clipboard API.
- **Multiple canvas layers for overlays:** Unnecessary complexity for this use case. Single overlay canvas is sufficient.
- **Forgetting URL.revokeObjectURL():** Memory leak when creating object URLs. Always revoke after download completes.
- **Not handling Clipboard API errors:** Promise may reject due to permissions or browser support. Always use try/catch.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Canvas export | Custom base64 encoding | canvas.toBlob() | Browser-optimized, async, handles encoding |
| File download | Manual blob construction | toBlob() + anchor download | Standard pattern, works across browsers |
| Clipboard copy | Custom selection + execCommand | navigator.clipboard.writeText() | Modern API, promise-based, better error handling |
| Semi-transparent overlays | Manual alpha blending | RGBA colors or globalAlpha | Canvas API handles compositing efficiently |
| URL object cleanup | Manual memory tracking | URL.revokeObjectURL() | Browser manages memory correctly |

**Key insight:** Browser APIs for canvas export and clipboard are mature, well-tested, and handle edge cases (CORS, permissions, memory management) that custom implementations would miss.

## Common Pitfalls

### Pitfall 1: Tainted Canvas SecurityError
**What goes wrong:** Attempting to export canvas throws SecurityError: "canvas has been tainted by cross-origin data"
**Why it happens:** Canvas becomes "tainted" when drawing images from different origins without CORS headers. Once tainted, toBlob(), toDataURL(), and getImageData() throw SecurityError.
**How to avoid:**
- For this project: QR codes are generated in-canvas (not loaded from external sources), so no CORS issues
- If loading external images: Use img.crossOrigin = 'anonymous' and server must send proper CORS headers
**Warning signs:** SecurityError exceptions when calling toBlob() or toDataURL()

### Pitfall 2: Clipboard API Requires User Activation
**What goes wrong:** navigator.clipboard.writeText() fails or throws NotAllowedError
**Why it happens:** Clipboard API requires "transient user activation" (user recently interacted with page) for security. Automated/background clipboard access is blocked.
**How to avoid:**
- Always call clipboard operations from event handlers (click, keydown, etc)
- Never call from setTimeout, Promise.then, or other async contexts not triggered by user
- Check for HTTPS context (API doesn't work on HTTP)
**Warning signs:** NotAllowedError DOMException, clipboard operations silently fail

### Pitfall 3: toBlob() Callback with null
**What goes wrong:** toBlob() callback receives null instead of Blob object
**Why it happens:** Canvas failed to create image (e.g., canvas height/width is 0, canvas exceeds maximum size)
**How to avoid:**
- Always check if blob is null before using it
- Validate canvas dimensions before export
- Handle error case with user-friendly message
**Warning signs:** TypeError when trying to use blob result, "Cannot read property of null"

### Pitfall 4: Forgetting to Revoke Object URLs
**What goes wrong:** Memory leaks accumulate as user downloads multiple files
**Why it happens:** URL.createObjectURL() creates a URL that references a Blob in memory. Browser keeps Blob in memory until URL is revoked or page unloads.
**How to avoid:**
- Call URL.revokeObjectURL(url) after download triggered
- Can revoke immediately after click() since download is already initiated
- Not critical for single-page short sessions, but important for long-running apps
**Warning signs:** Memory usage grows with repeated downloads, browser becomes sluggish

### Pitfall 5: RGBA vs globalAlpha Confusion
**What goes wrong:** Overlay transparency affects all subsequent drawing operations
**Why it happens:** Using globalAlpha sets transparency for ALL future draws until changed. Forgetting to reset it causes unintended transparency.
**How to avoid:**
- Prefer RGBA colors for individual elements: `ctx.fillStyle = 'rgb(255 0 0 / 50%)'`
- If using globalAlpha, save/restore context state or explicitly reset after
- RGBA is more explicit and easier to debug
**Warning signs:** Unexpected transparency in subsequent canvas draws, hard-to-debug rendering issues

### Pitfall 6: Download Attribute Same-Origin Limitation
**What goes wrong:** Download attribute ignored, browser navigates to URL instead
**Why it happens:** download attribute only works for same-origin URLs, blob: URLs, and data: URLs. Cross-origin URLs ignore the attribute.
**How to avoid:**
- For this project: Using canvas.toBlob() creates blob: URL, which works fine
- Never try to use download attribute with cross-origin HTTP URLs
- Use blob: or data: URLs for client-generated content
**Warning signs:** Browser navigates away instead of downloading, download attribute ignored

## Code Examples

### Complete Canvas Download Implementation
```javascript
// Source: MDN toBlob() + project requirements
// https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toBlob

/**
 * Download canvas as PNG file
 * @param {HTMLCanvasElement} canvas - Canvas to export
 * @param {string} filename - Filename without extension (e.g., 'qr-code')
 */
function downloadCanvasAsPNG(canvas, filename) {
  // Validate canvas
  if (!canvas || canvas.width === 0 || canvas.height === 0) {
    console.error('Invalid canvas for download');
    return;
  }

  canvas.toBlob((blob) => {
    // Handle null blob (canvas failed to create image)
    if (!blob) {
      console.error('Failed to create image blob');
      return;
    }

    // Create temporary object URL
    const url = URL.createObjectURL(blob);

    // Create and trigger download
    const a = document.createElement('a');
    a.href = url;
    a.download = `${filename}.png`;
    a.style.display = 'none';
    document.body.appendChild(a);
    a.click();

    // Cleanup
    document.body.removeChild(a);
    URL.revokeObjectURL(url); // Free memory
  }, 'image/png'); // PNG format (default, best for QR codes)
}

// Usage for result export:
const canvas = renderResultToCanvas(result);
const filename = `qr-${result.hash}`;
downloadCanvasAsPNG(canvas, filename);
```

### Complete Clipboard Copy with Error Handling
```javascript
// Source: MDN Clipboard API + project requirements
// https://developer.mozilla.org/en-US/docs/Web/API/Clipboard/writeText

/**
 * Copy text to clipboard with visual feedback
 * @param {string} text - Text to copy
 * @param {HTMLButtonElement} button - Button that triggered copy (for feedback)
 */
async function copyToClipboard(text, button) {
  // Check if Clipboard API is available
  if (!navigator.clipboard) {
    console.error('Clipboard API not available (requires HTTPS)');
    showTemporaryMessage(button, '✗ Copy failed');
    return;
  }

  const originalText = button.textContent;
  const originalDisabled = button.disabled;

  try {
    // Write to clipboard (requires user activation)
    await navigator.clipboard.writeText(text);

    // Success feedback
    button.textContent = '✓ Copied!';
    button.disabled = true;

    setTimeout(() => {
      button.textContent = originalText;
      button.disabled = originalDisabled;
    }, 2000);

  } catch (error) {
    // Handle errors (permissions, user activation, etc)
    console.error('Copy failed:', error);

    // Show error feedback
    button.textContent = '✗ Copy failed';
    setTimeout(() => {
      button.textContent = originalText;
    }, 2000);
  }
}

/**
 * Construct final URL with hash fragment
 */
function constructResultURL(baseURL, hashFragment) {
  // Hash fragment format: baseURL + '#' + hash
  return `${baseURL}#${hashFragment}`;
}

// Usage from button click:
copyBtn.addEventListener('click', () => {
  const url = constructResultURL(currentURL, result.hash);
  copyToClipboard(url, copyBtn);
});
```

### Error Visualization Overlay
```javascript
// Source: MDN Canvas colors + project requirements
// https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Applying_styles_and_colors

/**
 * Render error visualization overlay on top of QR code
 * Shows: locked pattern pixels, function patterns, conflict pixels
 * @param {HTMLCanvasElement} overlayCanvas - Canvas for overlay (same size as QR)
 * @param {number} moduleCount - QR module dimensions
 * @param {Uint8Array} paintedPixels - Painted pattern (0=unset, 1=white, 2=black)
 * @param {Uint8Array} functionMask - Function pattern mask (1=function pattern)
 * @param {Uint8Array} qrModules - Final QR module data (0=white, 1=black)
 */
function renderErrorOverlay(overlayCanvas, moduleCount, paintedPixels, functionMask, qrModules) {
  const scale = Math.floor(overlayCanvas.width / moduleCount);
  const ctx = overlayCanvas.getContext('2d');

  // Clear previous overlay
  ctx.clearRect(0, 0, overlayCanvas.width, overlayCanvas.height);

  for (let row = 0; row < moduleCount; row++) {
    for (let col = 0; col < moduleCount; col++) {
      const idx = row * moduleCount + col;
      const paintedState = paintedPixels[idx];
      const isFunctionPattern = functionMask[idx];
      const qrValue = qrModules[idx];

      let color = null;

      // Determine pixel type and color
      if (isFunctionPattern) {
        // Function pattern pixels (finder, timing, alignment, format info)
        // Blue overlay - not modifiable
        color = 'rgb(0 100 255 / 30%)';

      } else if (paintedState !== 0) {
        // Locked pattern pixels - user painted these
        const paintedIsDark = (paintedState === 2);
        const qrIsDark = (qrValue === 1);

        if (paintedIsDark === qrIsDark) {
          // Match - locked pixel matches QR data requirement
          // Green overlay - successful lock
          color = 'rgb(0 200 0 / 25%)';
        } else {
          // Conflict - locked pixel conflicts with QR data
          // Red overlay - error/conflict
          color = 'rgb(255 0 0 / 40%)';
        }
      }
      // else: Unset pixels (QR data pixels) - no overlay

      if (color) {
        ctx.fillStyle = color;
        ctx.fillRect(col * scale, row * scale, scale, scale);
      }
    }
  }
}

// Usage:
const overlayCanvas = document.getElementById('error-overlay');
overlayCanvas.width = 400;
overlayCanvas.height = 400;
renderErrorOverlay(
  overlayCanvas,
  result.moduleCount,
  paintedPixelsArray,
  functionMaskArray,
  result.moduleData
);
```

### Render Result to Full-Size Canvas
```javascript
/**
 * Render result's moduleData to full-size canvas for display/export
 * @param {Uint8Array|Array} moduleData - Module data (0=white, 1=black)
 * @param {number} moduleCount - Dimensions of QR code
 * @param {number} canvasSize - Target canvas size in pixels
 * @returns {HTMLCanvasElement} Canvas with rendered QR code
 */
function renderResultToCanvas(moduleData, moduleCount, canvasSize) {
  const canvas = document.createElement('canvas');
  canvas.width = canvasSize;
  canvas.height = canvasSize;

  const scale = Math.floor(canvasSize / moduleCount);
  const actualSize = moduleCount * scale;

  const ctx = canvas.getContext('2d');

  // Render modules
  for (let row = 0; row < moduleCount; row++) {
    for (let col = 0; col < moduleCount; col++) {
      const idx = row * moduleCount + col;
      const isDark = (moduleData[idx] === 1);

      ctx.fillStyle = isDark ? '#000000' : '#FFFFFF';
      ctx.fillRect(col * scale, row * scale, scale, scale);
    }
  }

  return canvas;
}

// Usage for export:
const canvas = renderResultToCanvas(result.moduleData, result.moduleCount, 400);
downloadCanvasAsPNG(canvas, `qr-${result.hash}`);
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| document.execCommand('copy') | navigator.clipboard.writeText() | Deprecated ~2020 | Clipboard API is promise-based, more secure, better error handling |
| toDataURL() for downloads | toBlob() + URL.createObjectURL() | Best practice since ~2018 | Async processing, better performance, no URL length limits |
| Multiple stacked canvas elements | Single overlay canvas | Design pattern | Simpler architecture, easier state management for overlays |
| Base opacity with globalAlpha | RGBA colors with alpha channel | Modern CSS3 | More explicit per-element control, easier debugging |

**Deprecated/outdated:**
- **document.execCommand('copy')**: Deprecated, no longer guaranteed to work across browsers. Use Clipboard API instead.
- **Canvas toDataURL() for large images**: Not deprecated but discouraged for performance. Use toBlob() for downloads.
- **Non-secure contexts (HTTP)**: Clipboard API requires HTTPS. Modern web features increasingly require secure contexts.

## Open Questions

1. **Should error overlay be always visible or toggled?**
   - What we know: Requirements specify error visualization overlay should be provided (OUTPUT-03)
   - What's unclear: Whether overlay should be always-on, toggled by button, or shown only on hover/click
   - Recommendation: Implement as toggle button ("Show Errors") per result, default OFF to keep main view clean

2. **Export canvas size for downloaded PNG?**
   - What we know: Current preview canvases are 80x80px thumbnails, moduleCount varies by QR version
   - What's unclear: Should exported PNG be native module size, scaled to standard size (e.g., 512px), or user-configurable?
   - Recommendation: Export at 512x512px (512/moduleCount = ~16-20px per module for versions 5-8), good balance of quality and file size

3. **Should overlay be included in downloaded PNG?**
   - What we know: Requirements separate "download QR code" (OUTPUT-01) from "error visualization overlay" (OUTPUT-03)
   - What's unclear: Does download include overlay, or only clean QR code?
   - Recommendation: Download clean QR code only. Error overlay is debugging/understanding tool, not part of final output

4. **Hash fragment format and validation?**
   - What we know: Project uses hash fragments for URL variants, optimization generates random hashes
   - What's unclear: Hash length, character set, collision handling
   - Recommendation: Use existing hash generation from worker (appears to be 6-8 character alphanumeric), validate against base URL structure

## Sources

### Primary (HIGH confidence)
- [MDN: HTMLCanvasElement.toDataURL()](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toDataURL) - Canvas PNG export API
- [MDN: HTMLCanvasElement.toBlob()](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toBlob) - Preferred async canvas export
- [MDN: Clipboard.writeText()](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard/writeText) - Modern clipboard copy API
- [MDN: Clipboard API](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard_API) - Security requirements and usage
- [MDN: HTMLAnchorElement.download](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAnchorElement/download) - Download attribute specification
- [MDN: CanvasRenderingContext2D.globalAlpha](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/globalAlpha) - Canvas transparency
- [MDN: Canvas API - Applying styles and colors](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Applying_styles_and_colors) - RGBA colors and transparency

### Secondary (MEDIUM confidence)
- [WebSearch: toBlob vs toDataURL performance](https://github.com/Infocatcher/Right_Links/issues/25) - Performance comparison, community consensus
- [WebSearch: Canvas CORS and SecurityError](https://developer.mozilla.org/en-US/docs/Web/HTML/How_to/CORS_enabled_image) - Tainted canvas issues
- Web.dev patterns for clipboard operations - Modern best practices

### Tertiary (LOW confidence)
- WebSearch results for QR code error visualization patterns - Limited specific guidance, general visualization principles apply

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - All browser-native APIs with official MDN documentation
- Architecture: HIGH - Well-established patterns for canvas export and clipboard operations, existing project provides results structure
- Pitfalls: HIGH - Official MDN documentation covers security errors, permissions, and common mistakes

**Research date:** 2026-02-09
**Valid until:** 2026-03-09 (30 days - stable browser APIs, unlikely to change)

**Key project constraints applied:**
- Single HTML file, no external dependencies
- All JavaScript inline
- Modern browsers only (Chrome, Firefox, Safari)
- Works via file:// and http:// protocols (affects Clipboard API availability)
- Existing codebase already tracks top 5 results with moduleData

**Implementation notes:**
- Top 5 results already tracked in Phase 3 worker
- Result structure includes: hash, pixelDiff, decodable, moduleData (Uint8Array), moduleCount
- No additional libraries needed - all native browser APIs
- Clipboard API requires HTTPS for production, may not work with file:// protocol (fallback handling needed)
