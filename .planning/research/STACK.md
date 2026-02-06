# Stack Research: Artistic QR Code Generator

**Domain:** Browser-based QR code generation with artistic pixel manipulation
**Researched:** 2026-02-06
**Confidence:** MEDIUM-HIGH

## Executive Summary

For a single-file HTML artistic QR code generator, the 2026 stack requires:
1. **QR Generator** with error correction level H and module access: Nayuki's QR Code generator (TypeScript/JS)
2. **QR Decoder** for error measurement: @paulmillr/qr or @nuintun/qrcode
3. **Canvas API** for pixel manipulation (native browser)
4. **No build step**: All libraries must be inlineable as single-file JavaScript

The critical constraint is finding libraries that expose QR code internals (module matrix, error correction) for artistic manipulation, not just black-box encoding/decoding.

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| **Nayuki QR Code Generator** | Latest (TypeScript/JS port) | QR code generation with full control | Only generator with clean access to module matrix, ~970 lines of pure TypeScript, zero dependencies, MIT licensed, actively maintained. Provides `getModule(x, y)` for pixel-level access. |
| **@paulmillr/qr** (or npm `qr`) | 0.5.3+ | QR decoding and error measurement | Zero dependencies, minimal decoder (~162 ops/sec), exposes error correction via Reed-Solomon, works standalone. Renamed to `qr` package on npm (latest 0.5.3, Jan 2026). |
| **Canvas API** (native) | HTML5 standard | Pixel manipulation and rendering | Native browser API, no dependencies, `ImageData` API provides direct pixel access via `Uint8ClampedArray` (RGBA format). |
| **ES6 Modules** (native) | ES2015+ | Code organization | Inline `<script type="module">` allows clean code structure in single HTML file without build tools. |

**Confidence: HIGH** - All technologies verified via official documentation and current in 2026.

### Supporting Libraries (Optional)

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| **@nuintun/qrcode** | 5.0.2 | Alternative encoder/decoder | If you need both encode and decode in one library; pure JavaScript, TypeScript-built, exposes internal structures. Use instead of separate encoder/decoder. |
| **jsQR (cozmo)** | 1.4.0 | Fallback decoder | Pure JavaScript decoder, but UNMAINTAINED (last update 2021). Only use if @paulmillr/qr fails your use case. |
| **QRCode.js (davidshimjs)** | Latest | Simple generation fallback | Zero dependencies, canvas support, but NO module-level access. Only for basic QR generation without artistic manipulation. |

**Confidence: MEDIUM** - Supporting libraries verified but may have maintenance concerns (jsQR) or limited feature sets.

## Installation (for reference - will be inlined)

```bash
# If developing with npm first (optional)
npm install qr  # @paulmillr/qr renamed to qr
# OR
npm install @nuintun/qrcode

# For Nayuki: Download from GitHub releases
# https://github.com/nayuki/QR-Code-generator/releases
# Use: qrcodegen-javascript.js or qrcodegen-typescript.ts (compile first)
```

**For single-file HTML:** Copy library source code directly into `<script>` tags or inline as ES6 module.

## Architecture for Single-File HTML

```html
<!DOCTYPE html>
<html>
<head>
  <title>Artistic QR Code Generator</title>
</head>
<body>
  <canvas id="qrCanvas" width="800" height="800"></canvas>

  <script>
    // Inline Nayuki QR Code Generator (~970 lines)
    // Copy from: https://github.com/nayuki/QR-Code-generator/blob/master/typescript-javascript/qrcodegen-v1-js.js

    // OR inline @paulmillr/qr decoder/encoder
    // Available as single file from npm package

    // Your application code
    const canvas = document.getElementById('qrCanvas');
    const ctx = canvas.getContext('2d');

    // Generate QR code
    const qr = qrcodegen.QrCode.encodeText("https://example.com#hash",
                                            qrcodegen.QrCode.Ecc.HIGH);

    // Access module matrix for artistic manipulation
    for (let y = 0; y < qr.size; y++) {
      for (let x = 0; x < qr.size; x++) {
        const isBlack = qr.getModule(x, y);
        // Your 3-state logic: white/black/unset
        // Paint locked pixels, search for hash values that fit art
      }
    }

    // Use decoder to measure errors
    // @paulmillr/qr or @nuintun/qrcode
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const decoded = decode(imageData); // Returns error count or null
  </script>
</body>
</html>
```

## Detailed Technology Analysis

### 1. Nayuki QR Code Generator (PRIMARY ENCODER)

**Why this is the standard choice:**
- **Module-level access**: `qr.getModule(x, y)` returns boolean for each pixel
- **Zero dependencies**: Pure TypeScript/JavaScript, only needs language standard library
- **Error correction control**: Full support for L, M, Q, H levels with explicit control
- **Version support**: All 40 QR versions (1-40), easily constrain to versions 2-8 for your use case
- **Clean API**: `QrCode.encodeText(text, errorCorrectionLevel)` or manual segment creation
- **Well-documented**: Extensive documentation at https://www.nayuki.io/page/qr-code-generator-library
- **File protocol compatible**: No external dependencies, pure computation
- **Inlineable**: ~970 lines of TypeScript compiles to standalone JavaScript
- **License**: MIT (permissive)

**How to access QR internals:**
```javascript
const qr = qrcodegen.QrCode.encodeText("text", qrcodegen.QrCode.Ecc.HIGH);
console.log(qr.size); // Matrix size (21 for version 1, etc.)
for (let y = 0; y < qr.size; y++) {
  for (let x = 0; x < qr.size; x++) {
    const module = qr.getModule(x, y); // true = black, false = white
    // Manipulate or read individual modules
  }
}
```

**What you can access:**
- Module matrix (pixel grid)
- QR version
- Error correction level
- Size

**What you CANNOT access directly:**
- Reed-Solomon error correction codewords (handled internally)
- Masking patterns (automatically selected)
- Format/version information bits (encoded automatically)

**Confidence: HIGH** - Verified from official documentation (https://www.nayuki.io/page/qr-code-generator-library) and GitHub repository.

**Sources:**
- [QR Code generator library (Nayuki)](https://www.nayuki.io/page/qr-code-generator-library)
- [GitHub: nayuki/QR-Code-generator](https://github.com/nayuki/QR-Code-generator)

### 2. @paulmillr/qr (now `qr`) - PRIMARY DECODER

**Why recommended for decoding:**
- **Zero dependencies**: Minimal, auditable codebase
- **Encoder AND decoder**: Rare to find decoder with encoder in same package
- **Error correction support**: Custom levels (7% low, 15% medium default, 25% quartile, 30% high)
- **Decoder internals**: "Reads information via zig-zag pattern, de-interleaves bytes, corrects errors, converts to bits"
- **Performance**: ~162 ops/sec (sufficient for iterative hash search)
- **Active maintenance**: Renamed from @paulmillr/qr to `qr` in early 2026, latest version 0.5.3 (Jan 2026)
- **File protocol compatible**: No external dependencies
- **License**: Likely MIT/permissive (verify in package)

**How to measure decoder errors:**
The library's decoder performs Reed-Solomon error correction. To measure errors:
1. Generate corrupted QR code (with artistic pixels)
2. Attempt to decode
3. Check if decoding succeeds (returns data) or fails (returns null/error)
4. For more precise error measurement, you may need to access intermediate decoder state (check source code)

**Limitations:**
- ECI (Extended Channel Interpretation) not yet supported
- Decoder at "near parity with ZXing" but not 100%
- Documentation is minimal; may need to read source code for error measurement internals

**Confidence: MEDIUM-HIGH** - Package actively maintained (Jan 2026 release), but limited documentation on error measurement internals.

**Sources:**
- [npm: @paulmillr/qr](https://www.npmjs.com/package/@paulmillr/qr)
- [npm: qr (renamed package)](https://www.npmjs.com/package/qr)
- [GitHub: paulmillr/qr](https://github.com/paulmillr/qr)
- [QR code generator and scanner (Paul Millr)](https://paulmillr.com/apps/qr/)

### 3. @nuintun/qrcode (ALTERNATIVE ENCODER/DECODER)

**Why consider this:**
- **Both encoder and decoder** in one package
- **Pure JavaScript** (TypeScript-built)
- **Module access**: Exposes finder patterns, timing patterns, corners
- **Error correction**: Supports L, M, Q, H levels
- **Image-based decoding**: Can decode from image source
- **Latest version**: 5.0.2 (published May 2025, stable)

**API structure:**
```javascript
// Encoder
import { Encoder } from '@nuintun/qrcode';
const qrcode = new Encoder();
qrcode.write('text');
const result = qrcode.toDataURL();

// Decoder
import { Decoder } from '@nuintun/qrcode';
const decoder = new Decoder();
const result = decoder.decode(imageSrc);
```

**Why not primary recommendation:**
- Less documentation than Nayuki
- Heavier weight than separate Nayuki + @paulmillr/qr
- Unclear if module-level access is as clean as Nayuki's `getModule(x, y)`

**When to use instead:**
- If you want single library for both encode/decode
- If @paulmillr/qr decoder doesn't meet your needs
- If you need advanced features like Hanzi/Kanji encoding modes

**Confidence: MEDIUM** - Verified via npm (v5.0.2) but limited documentation access.

**Sources:**
- [npm: @nuintun/qrcode](https://www.npmjs.com/package/@nuintun/qrcode)
- [GitHub: nuintun/qrcode](https://github.com/nuintun/qrcode)

### 4. Canvas API (Native Browser)

**Why this is the standard:**
- **Native to all modern browsers** (no library needed)
- **ImageData API**: Direct pixel manipulation via `Uint8ClampedArray`
- **RGBA format**: 4 bytes per pixel (red, green, blue, alpha)
- **Methods available**:
  - `ctx.getImageData(x, y, width, height)` - Read pixels
  - `ctx.putImageData(imageData, x, y)` - Write pixels
  - `ctx.createImageData(width, height)` - Create blank ImageData
- **File protocol compatible**: Native API, no external resources

**Pixel manipulation pattern:**
```javascript
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

// Read pixel data
const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
const data = imageData.data; // Uint8ClampedArray

// Manipulate pixels (RGBA format)
for (let i = 0; i < data.length; i += 4) {
  data[i]     = 255; // Red
  data[i + 1] = 0;   // Green
  data[i + 2] = 0;   // Blue
  data[i + 3] = 255; // Alpha
}

// Write pixel data back
ctx.putImageData(imageData, 0, 0);
```

**3-state pixel logic for artistic QR:**
```javascript
// State: 0=white, 1=black, 2=unset (to be searched)
const pixelState = new Uint8Array(qr.size * qr.size);

// User paints art pattern -> lock those pixels
function lockPixel(x, y, isBlack) {
  pixelState[y * qr.size + x] = isBlack ? 1 : 0;
}

// Search for URL hash that aligns with locked pixels
function searchHash() {
  for (let hash of generateHashes()) {
    const qr = qrcodegen.QrCode.encodeText(`https://example.com#${hash}`,
                                            qrcodegen.QrCode.Ecc.HIGH);
    let errors = 0;
    for (let y = 0; y < qr.size; y++) {
      for (let x = 0; x < qr.size; x++) {
        const state = pixelState[y * qr.size + x];
        if (state !== 2) { // Locked pixel
          const qrModule = qr.getModule(x, y);
          if ((state === 1) !== qrModule) errors++;
        }
      }
    }
    if (errors === 0) return hash; // Perfect match
  }
}
```

**Confidence: HIGH** - Native browser API, verified from MDN documentation (2026).

**Sources:**
- [MDN: Pixel manipulation with canvas](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Pixel_manipulation_with_canvas)
- [MDN: getImageData()](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/getImageData)
- [MDN: putImageData()](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/putImageData)

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Nayuki QR Generator | **qrcode** (npm) | If you need PNG/JPEG output or Node.js integration. But it lacks module-level access for artistic manipulation. |
| Nayuki QR Generator | **QRCode.js** (davidshimjs) | For simple QR generation without artistic features. Zero dependencies, canvas support, but NO `getModule()` access. |
| @paulmillr/qr | **jsQR** (cozmo) | Pure JavaScript decoder, but UNMAINTAINED since 2021. Avoid unless you have specific needs. |
| @paulmillr/qr | **zxing-js** | Multi-format barcode library, but unmaintained ("developers do not have time to actively maintain"). Overkill for QR-only project. |
| @nuintun/qrcode | **Nayuki + @paulmillr/qr** | If single library is preferred over two separate libraries. Trade-off: heavier weight, less documentation. |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| **node-qrcode** (`qrcode` on npm) | Designed for Node.js; browser bundle requires build step; no module-level access for artistic manipulation. | Nayuki QR Code Generator |
| **jsQR** (cozmo) | UNMAINTAINED since 2021 (5 years old as of 2026); no updates, potential security/compatibility issues. | @paulmillr/qr or @nuintun/qrcode |
| **zxing-js** | UNMAINTAINED ("developers do not have time to actively maintain"); overkill for QR-only; harder to inline. | @paulmillr/qr for decoding |
| **QRCode.js** (davidshimjs) | No module-level access; cannot read individual QR pixels for artistic manipulation. Good for simple generation only. | Nayuki QR Code Generator |
| **Any library requiring build tools** (Webpack, Rollup, etc.) | Your constraint: single HTML file, no build step, file:// protocol. | Only use libraries with standalone JS files |
| **Any library with external dependencies** | Cannot inline dependencies; breaks single-file requirement. | Zero-dependency libraries only |

## Error Correction Internals: How to Access

### QR Code Error Correction Levels

All libraries support 4 standard levels:
- **L (Low)**: 7% of codewords can be restored
- **M (Medium)**: 15% of codewords can be restored (default for many libraries)
- **Q (Quartile)**: 25% of codewords can be restored
- **H (High)**: 30% of codewords can be restored

**For artistic QR codes, always use H (High)** to maximize tolerance for pixel manipulation.

### Reed-Solomon Error Correction

QR codes use Reed-Solomon error correction (works in Galois Field GF(256)). The algorithm:
1. Encodes data into polynomials
2. Adds redundant error correction codewords
3. Decoder can correct up to N/2 byte errors, where N = number of error correction bytes

**Example**: Version 2 QR with H correction has 34 bytes data + 28 bytes error correction = can correct up to 14 byte errors.

### How to Measure Decoder Errors

**Approach 1: Binary success/failure**
```javascript
// Generate artistic QR code
const qr = generateArtisticQR(url, artPattern);

// Try to decode
const result = decode(qr);
if (result) {
  console.log("Decode succeeded:", result);
} else {
  console.log("Decode failed - too many errors");
}
```

**Approach 2: Count pixel differences (proxy for errors)**
```javascript
// Generate two QR codes: artistic vs. perfect
const artisticQR = generateArtisticQR(url, artPattern);
const perfectQR = qrcodegen.QrCode.encodeText(url, qrcodegen.QrCode.Ecc.HIGH);

let differences = 0;
for (let y = 0; y < perfectQR.size; y++) {
  for (let x = 0; x < perfectQR.size; x++) {
    if (artisticQR.getModule(x, y) !== perfectQR.getModule(x, y)) {
      differences++;
    }
  }
}
console.log(`Pixel differences: ${differences}`);
```

**Approach 3: Access Reed-Solomon internals (advanced)**
- Requires reading decoder source code
- Look for `ReedSolomon`, `rsDecoder`, or similar classes
- May expose error count or correction status
- Not guaranteed in all libraries

**Confidence: MEDIUM** - Error correction theory is well-established, but accessing internals varies by library.

**Sources:**
- [Error correction feature (DENSO WAVE)](https://www.qrcode.com/en/about/error_correction.html)
- [QR Code Error Correction Explained (Scanova)](https://scanova.io/blog/qr-code-error-correction/)
- [Reed-Solomon error correction (Wikipedia)](https://en.wikipedia.org/wiki/Reed%E2%80%93Solomon_error_correction)
- [Let's develop a QR Code Generator, part III: error correction (DEV Community)](https://dev.to/maxart2501/let-s-develop-a-qr-code-generator-part-iii-error-correction-1kbm)

## File:// Protocol Compatibility

**All recommended libraries work with file:// protocol** because:
1. **Pure JavaScript computation** - no server requests
2. **No external dependencies** - no CDN or network calls
3. **Canvas API is native** - no external resources

**Potential pitfalls:**
- **DO NOT use**: CDN links (e.g., `<script src="https://cdn.example.com/qr.js">`) - breaks offline use
- **DO**: Inline all JavaScript in `<script>` tags or use data URLs
- **Camera access**: May require HTTPS for getUserMedia() API; if scanning via camera, file:// won't work

**Testing on file:// protocol:**
```bash
# Create single HTML file
cat > qr-generator.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>QR Generator</title></head>
<body>
  <canvas id="qr"></canvas>
  <script>
    // Inline Nayuki library here (~970 lines)
    // Your code here
  </script>
</body>
</html>
EOF

# Open in browser
open qr-generator.html  # macOS
xdg-open qr-generator.html  # Linux
start qr-generator.html  # Windows
```

**Confidence: HIGH** - Verified that pure JavaScript libraries work with file:// protocol.

**Sources:**
- [GitHub: six-two/qr.html](https://github.com/six-two/qr.html) - Example of standalone offline QR generator

## Stack Patterns by Use Case

**If building from scratch (greenfield):**
- Use **Nayuki** for encoding (best module access)
- Use **@paulmillr/qr** (`qr` on npm) for decoding (minimal, maintained)
- Use **Canvas API** for rendering
- **Total size**: ~1500-2000 lines of inlined JavaScript

**If you need single library:**
- Use **@nuintun/qrcode** for both encode/decode
- Use **Canvas API** for rendering
- **Trade-off**: Heavier weight, less documentation than Nayuki

**If maximum simplicity (no artistic features):**
- Use **QRCode.js** (davidshimjs) for basic generation
- **Limitation**: Cannot implement artistic QR features (no module access)

**If decoding is not critical:**
- Use **Nayuki** only for encoding
- Skip decoder (save ~500 lines)
- **Limitation**: Cannot measure errors, must rely on scanning QR codes externally

## Version Compatibility

| Package | Compatible With | Notes |
|---------|-----------------|-------|
| Nayuki QR Generator | ES6+ browsers (2015+) | Uses classes, let/const, arrow functions, for-of loops. Can compile to ES5 for older browsers. |
| @paulmillr/qr (`qr`) | ES6+ browsers | Zero dependencies, modern JavaScript. |
| @nuintun/qrcode | ES6+ browsers | TypeScript-built, compiles to modern JavaScript. |
| Canvas API | All modern browsers | IE10+, Safari 5.1+, all evergreen browsers. ImageData with Float16Array requires very recent browsers (2024+). |

**Target: Modern browsers only (2020+)** - Chrome 90+, Firefox 88+, Safari 14+, Edge 90+

**Confidence: HIGH** - ES6 support is universal in 2026 browsers.

## Sources Summary

**HIGH confidence sources (verified via official documentation):**
- [QR Code generator library (Nayuki)](https://www.nayuki.io/page/qr-code-generator-library)
- [GitHub: nayuki/QR-Code-generator](https://github.com/nayuki/QR-Code-generator)
- [MDN: Canvas API Pixel Manipulation](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Pixel_manipulation_with_canvas)
- [Error correction feature (DENSO WAVE)](https://www.qrcode.com/en/about/error_correction.html)

**MEDIUM confidence sources (npm packages, active maintenance):**
- [npm: @paulmillr/qr](https://www.npmjs.com/package/@paulmillr/qr)
- [npm: qr (renamed package)](https://www.npmjs.com/package/qr)
- [GitHub: paulmillr/qr](https://github.com/paulmillr/qr)
- [npm: @nuintun/qrcode](https://www.npmjs.com/package/@nuintun/qrcode)
- [GitHub: nuintun/qrcode](https://github.com/nuintun/qrcode)

**LOW confidence sources (unmaintained or unverified):**
- [GitHub: cozmo/jsQR](https://github.com/cozmo/jsQR) - Last update 2021, unmaintained
- [GitHub: zxing-js/library](https://github.com/zxing-js/library) - Developers state no active maintenance

**WebSearch-verified sources:**
- [QR Code Error Correction Explained (Scanova)](https://scanova.io/blog/qr-code-error-correction/)
- [Reed-Solomon error correction (Wikipedia)](https://en.wikipedia.org/wiki/Reed%E2%80%93Solomon_error_correction)
- [Let's develop a QR Code Generator, part III (DEV Community)](https://dev.to/maxart2501/let-s-develop-a-qr-code-generator-part-iii-error-correction-1kbm)

---

## Confidence Assessment by Category

| Area | Confidence | Rationale |
|------|------------|-----------|
| **QR Generation (Nayuki)** | HIGH | Official documentation verified, actively maintained, well-established library with 6+ language ports. |
| **QR Decoding (@paulmillr/qr)** | MEDIUM-HIGH | Package recently renamed and updated (Jan 2026), zero dependencies, but limited documentation on error measurement internals. |
| **Canvas API** | HIGH | Native browser API, MDN documentation current, standard across all modern browsers. |
| **Error Correction Theory** | HIGH | Reed-Solomon algorithm is well-documented standard, QR spec is public (ISO/IEC 18004). |
| **Error Measurement Practice** | MEDIUM | Theory is solid, but accessing decoder internals for precise error counts requires source code inspection. Binary success/failure is straightforward. |
| **File:// Protocol** | HIGH | Pure JavaScript libraries with no external dependencies work on file:// protocol. Verified with standalone examples. |
| **Single-file Inlining** | HIGH | All recommended libraries are single-file or can be concatenated into single file. No build tools required. |

## Open Questions & Validation Needed

1. **Error measurement precision**: Can @paulmillr/qr or @nuintun/qrcode expose exact error counts from Reed-Solomon decoder? Need to inspect source code.

2. **Performance of hash search**: At ~162 ops/sec for decoding, searching thousands of hash values may be slow. May need optimization or WebWorkers.

3. **QR masking patterns**: Does locking pixels break mask pattern selection? Nayuki auto-selects masks; need to test if artistic pixels cause readability issues.

4. **Version constraints**: You specified versions 2-8. Need to verify these provide enough capacity for URLs with hash fragments at error level H.

5. **Browser compatibility edge cases**: Float16Array ImageData is cutting-edge (2024-2025). Stick to Uint8ClampedArray for broader compatibility.

---

## Final Recommendation

**For artistic QR code generation in single HTML file (2026):**

```javascript
// 1. Inline Nayuki QR Code Generator (~970 lines)
//    Download: https://github.com/nayuki/QR-Code-generator/releases
//    File: qrcodegen-v1-js.js (ES6) or qrcodegen-v1-es5.js (ES5)

// 2. Inline @paulmillr/qr (or npm `qr`) for decoding
//    Download from npm: https://www.npmjs.com/package/qr
//    Or build from source: https://github.com/paulmillr/qr

// 3. Use native Canvas API for rendering

// Total: ~1500-2000 lines of JavaScript, zero dependencies, MIT licensed
```

**This stack is:**
- ✅ **Zero dependencies** (pure JavaScript)
- ✅ **Single-file compatible** (inline all code)
- ✅ **File:// protocol compatible** (no external resources)
- ✅ **Module-level access** (Nayuki's `getModule(x, y)`)
- ✅ **Error correction level H** (30% tolerance)
- ✅ **Decoder for error measurement** (@paulmillr/qr)
- ✅ **Modern browsers** (ES6+, Canvas API)
- ✅ **Well-documented** (Nayuki has extensive docs)
- ✅ **Actively maintained** (Nayuki: ongoing, @paulmillr/qr: Jan 2026 update)

**Overall Confidence: MEDIUM-HIGH** - Core libraries are solid and verified, but error measurement internals need source code inspection for precise implementation.

---

*Stack research for: Artistic QR Code Generator*
*Researched: 2026-02-06*
*Researcher: GSD Project Researcher*
