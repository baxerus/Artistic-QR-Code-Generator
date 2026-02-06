# Phase 1: Foundation & QR Generation - Research

**Researched:** 2026-02-06
**Domain:** QR code generation in JavaScript / Single-file HTML applications
**Confidence:** HIGH

## Summary

This phase requires building a single-file HTML tool that generates QR codes from URLs using the HTML5 Canvas API. The research identified node-qrcode (npm: qrcode) as the standard library for browser-based QR generation, with strong support for error correction level H, canvas rendering, and standalone browser bundles. URLs will be encoded in byte mode (since they contain lowercase letters and special characters beyond alphanumeric), requiring careful capacity calculation. The tool must inline all JavaScript dependencies to work via file:// protocol.

Key technical findings: QR codes use four encoding modes (numeric, alphanumeric, byte, kanji), with URLs requiring byte mode. Error correction level H provides 30% error recovery. Version selection must calculate minimum version based on URL byte length and error correction overhead. Modern CSS aspect-ratio property simplifies maintaining square canvas dimensions. URL validation should use native URL constructor API rather than regex to avoid security pitfalls.

**Primary recommendation:** Use node-qrcode with standalone browser bundle, inline via script tag in HTML, render to canvas with error correction level H, validate URLs with URL constructor API.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

#### Page Structure
- Content centered on page
- Minimal title/header at top (tool name only, no description)
- Footer with minimal credit/version info
- Page background: light gray or subtle color (not pure white)

#### Input Layout
- URL input positioned above canvas (traditional form-then-result flow)
- Controls organized in single compact form
- Version dropdown on separate row below URL input
- Generate button positioned below all controls (separate action row)

#### Canvas Presentation
- Responsive to viewport size
- Maintains square aspect ratio always (1:1, no stretching)
- Clear border and shadow around canvas for visual containment
- Canvas prominently displayed in main content area

#### Spacing & Density
- Balanced overall spacing (not too tight, not too loose)
- Medium vertical gaps between major sections (20-30px)
- Sections feel connected but distinct

#### Validation & Feedback
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

### Deferred Ideas (OUT OF SCOPE)
None - discussion stayed within phase scope.

</user_constraints>

## Standard Stack

The established libraries/tools for QR code generation in browser:

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| node-qrcode | Latest (1.5.x+) | QR code generation with canvas support | Most mature JS QR library, supports all error correction levels, has standalone browser build, actively maintained, works in browser without bundler |
| HTML5 Canvas API | Native | Rendering QR code graphics | Built into all modern browsers, no dependencies, works in file:// protocol |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| URL API | Native | URL validation and parsing | Built-in browser API, safer than regex, no XSS vulnerabilities |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| node-qrcode | qrcode.js (davidshimjs) | Simpler API but less flexible, unclear error correction API, less documentation |
| node-qrcode | QRious | Pure canvas library but less features, smaller community |
| node-qrcode | Nayuki's QR-Code-generator | High quality but TypeScript-focused, requires build step |

**Installation:**
For single-file HTML, use CDN:
```html
<script src="https://cdn.jsdelivr.net/npm/qrcode@latest/build/qrcode.min.js"></script>
```

Or download and inline the minified build directly in the HTML file.

## Architecture Patterns

### Single-File HTML Structure
```
index.html
├── <style>             # All CSS inlined
├── <body>              # HTML structure
│   ├── header
│   ├── form            # URL input + version selector + generate button
│   ├── canvas          # QR code display area
│   └── footer
└── <script>            # Inlined QR library + application logic
```

### Pattern 1: Inline Library Dependencies
**What:** Include entire JavaScript library source code directly in HTML file within `<script>` tags
**When to use:** Required for file:// protocol compatibility (no external HTTP requests allowed)
**Example:**
```html
<!DOCTYPE html>
<html>
<head>
    <style>
        /* All CSS here */
    </style>
</head>
<body>
    <!-- HTML structure -->

    <!-- Inline QR code library -->
    <script>
        // Paste entire qrcode.min.js contents here
        // (Download from jsDelivr or npm package build/ folder)
    </script>

    <!-- Application logic -->
    <script>
        // Your QR generation code
    </script>
</body>
</html>
```

### Pattern 2: QR Code Generation with Canvas
**What:** Use QRCode.toCanvas() to render directly to canvas element with error correction level H
**When to use:** For all QR generation in this phase
**Example:**
```javascript
// Source: node-qrcode npm documentation
const canvas = document.getElementById('qr-canvas');
const url = document.getElementById('url-input').value;

QRCode.toCanvas(canvas, url, {
  errorCorrectionLevel: 'H',  // 30% error recovery
  version: selectedVersion,    // 1-8, calculated from URL length
  width: canvasSize,           // Responsive sizing
  margin: 4,                   // Required quiet zone
  color: {
    dark: '#000000',
    light: '#FFFFFF'
  }
}, function(error) {
  if (error) {
    console.error(error);
    // Show error feedback to user
  }
});
```

### Pattern 3: URL Validation with Native API
**What:** Use URL constructor for safe validation instead of regex
**When to use:** For all URL input validation
**Example:**
```javascript
// Source: MDN Web Security documentation
function isValidURL(string) {
  try {
    const url = new URL(string);
    // Only allow http/https protocols
    return url.protocol === 'http:' || url.protocol === 'https:';
  } catch (err) {
    return false;
  }
}
```

### Pattern 4: Square Aspect Ratio Canvas
**What:** Use CSS aspect-ratio property to maintain 1:1 ratio
**When to use:** For responsive canvas sizing
**Example:**
```css
/* Source: MDN CSS aspect-ratio documentation */
canvas {
  width: 100%;
  max-width: 512px;
  aspect-ratio: 1 / 1;
  border: 2px solid #ccc;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}
```

### Pattern 5: Version Selection Based on Capacity
**What:** Calculate minimum QR version based on URL byte length and error correction overhead
**When to use:** When user enters/changes URL, to populate version dropdown
**Example:**
```javascript
// QR Code capacity table for error correction level H (byte mode)
const CAPACITIES_H_BYTE = [
  { version: 1, capacity: 7 },
  { version: 2, capacity: 14 },
  { version: 3, capacity: 24 },
  { version: 4, capacity: 34 },
  { version: 5, capacity: 44 },
  { version: 6, capacity: 58 },
  { version: 7, capacity: 64 },
  { version: 8, capacity: 84 }
];

function calculateMinimumVersion(url) {
  const byteLength = new Blob([url]).size;  // Accurate byte count

  for (let entry of CAPACITIES_H_BYTE) {
    if (byteLength <= entry.capacity) {
      return entry.version;
    }
  }

  return 8;  // Max for this phase
}

function updateVersionDropdown(minVersion) {
  const dropdown = document.getElementById('version-select');
  dropdown.innerHTML = '';  // Clear options

  // Only show valid versions (minimum to 8)
  for (let v = minVersion; v <= 8; v++) {
    const option = document.createElement('option');
    option.value = v;
    option.textContent = `Version ${v}`;
    dropdown.appendChild(option);
  }
}
```

### Pattern 6: Inline Validation Feedback
**What:** Visual feedback at input field with icons and color coding
**When to use:** On URL input blur or change events
**Example:**
```html
<div class="input-wrapper">
  <input type="text" id="url-input" placeholder="Enter URL">
  <span class="validation-icon"></span>
</div>
```

```css
.input-wrapper {
  position: relative;
}

.validation-icon {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
}

input.valid {
  border-color: #10b981;
}

input.valid + .validation-icon::after {
  content: '✓';
  color: #10b981;
}

input.invalid {
  border-color: #ef4444;
}

input.invalid + .validation-icon::after {
  content: '✗';
  color: #ef4444;
}
```

### Anti-Patterns to Avoid
- **External dependencies via CDN in production:** Breaks file:// protocol requirement. Always inline libraries.
- **Regex-based URL validation:** Complex, error-prone, vulnerable to ReDoS attacks. Use URL constructor instead.
- **Alphanumeric mode encoding for URLs:** URLs contain lowercase letters and special chars, must use byte mode.
- **Canvas size mismatch:** Setting canvas.width/height attributes different from CSS dimensions causes blurry QR codes. Keep them synchronized or use responsive CSS.
- **Missing quiet zone:** QR codes need 4-module margin around edges for reliable scanning. Don't set margin: 0.
- **Assuming string.length = byte length:** Unicode characters can be multiple bytes. Use `new Blob([string]).size` for accurate byte count.

## Don't Hand-Roll

Problems that look simple but have existing solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| QR code encoding algorithm | Custom QR generator from scratch | node-qrcode library | QR spec is complex: error correction polynomials, Reed-Solomon encoding, mask pattern selection, format info bits. Library handles all edge cases. |
| URL validation regex | Custom regex patterns | Native URL constructor API | URLs have many edge cases (IDN, IPv6, ports, fragments). Regex is vulnerable to ReDoS. Browser API is safer and more complete. |
| Byte length calculation | string.length property | new Blob([string]).size | JavaScript strings are UTF-16 internally but need byte count for QR capacity. Multibyte chars break string.length accuracy. |
| Canvas aspect ratio enforcement | Manual resize calculations | CSS aspect-ratio property | Modern CSS handles responsive 1:1 ratio automatically. Old padding-based tricks are obsolete. |
| QR version capacity table | Manual lookups | Pre-computed lookup array | QR spec defines exact capacities. Calculate once, use lookup table to avoid errors. |

**Key insight:** QR code generation involves complex mathematical operations (Galois fields, polynomial division, optimal encoding mode selection). The library has handled thousands of edge cases over years of development. Focus on UI/UX, not reimplementing QR spec.

## Common Pitfalls

### Pitfall 1: File Protocol CORS Restrictions
**What goes wrong:** Attempting to load external resources (scripts, stylesheets) via `<script src="https://...">` fails when HTML file is opened via file:// protocol
**Why it happens:** Browsers treat file:// URLs as opaque origins for security. CORS only works with http:// or https://. Modern browsers block cross-origin requests from local files.
**How to avoid:** Inline ALL dependencies directly in HTML. Download qrcode.min.js contents and paste into `<script>` tag. No external resource loads.
**Warning signs:** Console errors like "Cross-Origin Request Blocked" or "Not allowed to load local resource" when opening file locally.

### Pitfall 2: Canvas Size vs CSS Size Mismatch
**What goes wrong:** QR code appears blurry or pixelated despite being generated correctly
**Why it happens:** Canvas has two dimensions: internal drawing buffer (canvas.width/height attributes) and display size (CSS width/height). Mismatch causes browser scaling and blur.
**How to avoid:** Either (a) let library handle sizing with width option, or (b) keep canvas attributes and CSS dimensions identical. For responsive design, use CSS aspect-ratio and let library calculate pixel dimensions.
**Warning signs:** QR code scans correctly but looks fuzzy visually. High DPI displays show more blur.

### Pitfall 3: Encoding Mode Assumptions
**What goes wrong:** QR code generation fails or produces unnecessarily large versions when URL length seems within capacity
**Why it happens:** Alphanumeric mode capacity tables don't apply to URLs. URLs contain lowercase letters (a-z) and many special characters (://?=&), forcing byte mode encoding. Byte mode has ~40% less capacity than alphanumeric.
**How to avoid:** Always use byte mode capacity table for URL encoding. A Version 3 holds 35 alphanumeric chars but only 24 bytes. Calculate based on byte length via `new Blob([url]).size`.
**Warning signs:** Library auto-selects higher version than expected. Error: "Data too long for version X".

### Pitfall 4: Missing Quiet Zone
**What goes wrong:** Generated QR codes fail to scan with some readers, especially mobile apps
**Why it happens:** QR spec requires 4-module white border (quiet zone) around code. Scanners use this to detect code boundaries. Setting margin: 0 or cropping too tightly breaks scanning.
**How to avoid:** Always use margin: 4 (default) in QRCode.toCanvas options. Don't crop canvas output. Design page layout to accommodate margin naturally.
**Warning signs:** QR code works on some scanners but not others. Scanning requires precise alignment or multiple attempts.

### Pitfall 5: URL Validation with Regex
**What goes wrong:** Valid URLs rejected or invalid URLs accepted. Potential ReDoS security vulnerability with complex regex patterns.
**Why it happens:** URLs are complex (protocols, IDN, IPv6, query params, fragments). Comprehensive regex is hundreds of characters and still misses edge cases. Catastrophic backtracking in regex can freeze browser.
**How to avoid:** Use native URL constructor: `new URL(string)` throws TypeError for invalid URLs. Check protocol separately for http/https only.
**Warning signs:** User reports valid URL rejected. Browser freezes on specific malformed input. Security audit flags regex complexity.

### Pitfall 6: Unicode/Multibyte Characters in URLs
**What goes wrong:** QR version calculation underestimates required capacity, generation fails
**Why it happens:** string.length counts UTF-16 code units, not bytes. Non-ASCII characters (emojis, international domains, special chars) are multiple bytes. QR capacity is measured in bytes.
**How to avoid:** Calculate byte length correctly: `new Blob([url]).size` or `new TextEncoder().encode(url).length`. Use this for version calculation.
**Warning signs:** Short URLs with emojis/international chars fail. Error: "Data too long" on unexpectedly short input.

## Code Examples

Verified patterns from research sources:

### Complete QR Generation Flow
```javascript
// Validate URL
function validateAndGenerate() {
  const input = document.getElementById('url-input');
  const url = input.value.trim();

  // Validate with URL constructor
  if (!isValidURL(url)) {
    input.classList.add('invalid');
    input.classList.remove('valid');
    return;
  }

  input.classList.add('valid');
  input.classList.remove('invalid');

  // Calculate minimum version
  const minVersion = calculateMinimumVersion(url);
  updateVersionDropdown(minVersion);

  // Get selected version
  const version = parseInt(document.getElementById('version-select').value);

  // Generate QR code
  generateQR(url, version);
}

function isValidURL(string) {
  try {
    const url = new URL(string);
    return url.protocol === 'http:' || url.protocol === 'https:';
  } catch (err) {
    return false;
  }
}

function generateQR(url, version) {
  const canvas = document.getElementById('qr-canvas');

  QRCode.toCanvas(canvas, url, {
    errorCorrectionLevel: 'H',
    version: version,
    margin: 4,
    width: 400,  // Will scale responsively via CSS
    color: {
      dark: '#000000',
      light: '#FFFFFF'
    }
  }, function(error) {
    if (error) {
      console.error('QR generation failed:', error);
      alert('Failed to generate QR code. URL may be too long.');
    }
  });
}
```

### Event Listeners Setup
```javascript
document.addEventListener('DOMContentLoaded', function() {
  const input = document.getElementById('url-input');
  const generateBtn = document.getElementById('generate-btn');

  // Validate on input change
  input.addEventListener('input', function() {
    const url = input.value.trim();
    if (url.length === 0) {
      input.classList.remove('valid', 'invalid');
      return;
    }

    if (isValidURL(url)) {
      const minVersion = calculateMinimumVersion(url);
      updateVersionDropdown(minVersion);
    }
  });

  // Validate on blur
  input.addEventListener('blur', function() {
    const url = input.value.trim();
    if (url.length > 0) {
      if (isValidURL(url)) {
        input.classList.add('valid');
        input.classList.remove('invalid');
      } else {
        input.classList.add('invalid');
        input.classList.remove('valid');
      }
    }
  });

  // Generate on button click
  generateBtn.addEventListener('click', validateAndGenerate);

  // Generate on Enter key
  input.addEventListener('keypress', function(e) {
    if (e.key === 'Enter') {
      validateAndGenerate();
    }
  });
});
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Padding-based aspect ratio | CSS aspect-ratio property | 2021 (wide support by 2023) | Simpler, more maintainable responsive squares |
| Complex URL regex validation | Native URL constructor API | Always available but best practice emerged ~2020 | Safer, more accurate, prevents ReDoS |
| External CDN for libraries | Inline dependencies for file:// support | Requirement-driven | Enables offline/local usage |
| QRCode.js (davidshimjs) | node-qrcode | node-qrcode gained popularity 2018+ | Better docs, more features, active maintenance |

**Deprecated/outdated:**
- `<table>` rendering for QR codes: Some old libraries (qrcode.js) support this. Canvas is now standard for better performance and modern browser support.
- jQuery dependencies: Early QR libraries required jQuery. Modern libraries are vanilla JS.
- Browserify for bundling: While still works, modern tools (Rollup, esbuild) are faster. For single-file HTML, direct CDN download of pre-built bundles is simplest.

## Open Questions

Things that couldn't be fully resolved:

1. **Exact byte capacity formula for QR versions**
   - What we know: Official capacity tables exist for each version + error correction level + encoding mode. Values confirmed for V1-V8 with error correction H in byte mode.
   - What's unclear: The mathematical formula to calculate capacity without lookup table (involves error correction codewords, data codewords, block structure).
   - Recommendation: Use hardcoded lookup table for V1-V8 byte mode with error correction H. Library handles calculation internally, we just need to predict minimum version for UI.

2. **Optimal canvas pixel dimensions for high-DPI displays**
   - What we know: node-qrcode's width option sets canvas dimensions. Larger = sharper on high-DPI but slower to generate.
   - What's unclear: Performance impact of very large canvas (e.g., 2048px) vs visual quality gain.
   - Recommendation: Start with 400-512px width, test on high-DPI screens. Library scales well. Can increase if blurriness reported.

3. **Loading state duration**
   - What we know: QR generation is synchronous and fast (< 100ms typically).
   - What's unclear: Whether loading spinner is needed or if instant feedback is better UX.
   - Recommendation: Test with real URLs. If generation is imperceptible (< 100ms), skip loading state. If user notices delay, add spinner.

## Sources

### Primary (HIGH confidence)
- node-qrcode npm package documentation - API, error correction levels, canvas rendering
- MDN Web Docs - URL API, Canvas API, aspect-ratio CSS property
- QR Code Official Spec (via qrcode.com) - Error correction levels, version structure
- Ryan Gibson's QR Character Limits Table - Capacity data for error correction level H

### Secondary (MEDIUM confidence)
- Multiple blog posts and tutorials on QR generation (cross-verified common patterns)
- Browser security documentation on file:// protocol CORS restrictions
- CSS-Tricks and W3Schools for aspect-ratio property examples

### Tertiary (LOW confidence - marked for validation)
- Exact performance characteristics of different canvas sizes (requires real-world testing)
- Optimal loading state patterns (UX preference, should validate with users)

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - node-qrcode is clearly established leader with active maintenance and comprehensive docs
- Architecture patterns: HIGH - Single-file HTML with inlined dependencies is well-documented approach for file:// protocol
- QR encoding details: HIGH - Verified with official QR spec and multiple authoritative sources
- Pitfalls: MEDIUM-HIGH - Based on common issues documented in library repos, stack overflow, and security advisories
- UI patterns: MEDIUM - CSS aspect-ratio is modern standard, validation patterns follow current best practices

**Research date:** 2026-02-06
**Valid until:** 2026-03-06 (30 days - QR generation is stable domain, libraries mature)
