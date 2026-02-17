# Domain Pitfalls: v1.5 Feature Additions

**Domain:** Adding correction capacity display, SVG export, and Shift+paint modifier to existing QR art tool
**Researched:** 2026-02-17
**Confidence:** HIGH (based on existing codebase analysis and official MDN documentation)

---

## Critical Pitfalls

### Pitfall 1: Wrong RS Capacity Formula

**What goes wrong:**
Developer calculates correction capacity using the wrong formula. Common mistakes:
1. Using `ecCodewordsPerBlock` directly instead of `ecCodewordsPerBlock / 2`
2. Confusing "codewords" with "bytes corrected" (they're the same, but the formula is still halved)
3. Not accounting for multiple blocks — using single block capacity instead of summing across blocks

**Why it happens:**
Reed-Solomon can **detect** 2t errors but only **correct** t errors, where t = ecCodewordsPerBlock / 2. The QRCode library's `RS_BLOCK_TABLE` stores the total EC codewords (2t), not the correction capacity (t). Developers often use the raw table value.

**Consequences:**
- "85% capacity used" when it's actually at 170% (undecodable)
- Users think they have room to add more artistic corruption when QR is already at limit
- Search ranking breaks because capacity percentage is wrong

**Prevention:**
```javascript
// WRONG: Using raw EC codewords as capacity
const capacity = ecCodewordsPerBlock; // This is 2t, not t!

// CORRECT: Divide by 2 to get correction capacity
const correctionCapacity = Math.floor(ecCodewordsPerBlock / 2);

// For multi-block QRs, track per-block
blocks.forEach(block => {
  block.capacity = Math.floor(block.ecCodewords / 2);
});
```

**Detection:**
- Capacity shows >100% but QR still decodes
- Test: Manually calculate Version 1-H capacity (should be 17/2 = 8 codewords, not 17)
- Cross-check with QR spec tables

**Phase to address:**
Phase 1 (Capacity Display) — Formula must be correct from the start

---

### Pitfall 2: Version Table Lookup Off-by-One

**What goes wrong:**
RS_BLOCK_TABLE is 0-indexed internally but QR versions are 1-indexed (Version 1 through Version 40). Developer looks up `RS_BLOCK_TABLE[version]` instead of `RS_BLOCK_TABLE[(version - 1) * 4 + ecLevelIndex]`.

**Why it happens:**
The existing codebase uses this formula in `getRsBlockTable()` (line 1745-1757):
```javascript
case d.H:
  return j.RS_BLOCK_TABLE[4 * (a - 1) + 3]; // a is version, 1-indexed
```
Developer copying the pattern might miss the `(a - 1)` part or miscalculate the error correction level offset.

**Consequences:**
- Version 2 shows Version 1 capacity (off by one version)
- Error correction level H shows Q capacity (off by one level)
- Capacity percentages wildly wrong
- Works for Version 1 (by luck), breaks for all others

**Prevention:**
```javascript
// Error correction level indices
const EC_LEVEL_INDEX = { L: 0, M: 1, Q: 2, H: 3 };

// CORRECT formula
const tableIndex = (version - 1) * 4 + EC_LEVEL_INDEX[level];
const rsBlockData = RS_BLOCK_TABLE[tableIndex];

// Extract total/data codewords from table entry
// Format: [numBlocks1, total1, data1, numBlocks2?, total2?, data2?]
```

**Detection:**
- Test with Version 3 or higher (where off-by-one is obvious)
- Compare calculated capacity against QR spec reference table
- Log both version and table index during development

**Phase to address:**
Phase 1 (Capacity Display) — Must be validated against reference table

---

### Pitfall 3: Forgetting App Uses Level H Only

**What goes wrong:**
Developer implements capacity display supporting all error correction levels (L/M/Q/H), but the app only generates Level H QR codes. The extra code:
1. Adds complexity that's never used
2. Creates potential bugs in untested code paths
3. Confuses the implementation

**Why it happens:**
The existing app hardcodes `errorCorrectLevel: d.H` (where d.H = 2 is the H level constant). Developer doesn't notice this constraint and over-engineers.

**Consequences:**
- Wasted development time
- Potential bugs in L/M/Q paths never caught
- Confusion when maintaining code

**Prevention:**
- Check codebase constraints FIRST
- For v1.5: Hardcode Level H capacity lookup, don't implement generic solution
- Add comment: `// App only uses Level H; capacity formula simplified accordingly`

**Detection:**
- Search codebase for `errorCorrectLevel` — confirms H-only usage
- Check if version dropdown shows EC level options (it doesn't)

**Phase to address:**
Phase 1 (Capacity Display) — Scope correctly from start

---

### Pitfall 4: Displaying Codewords When User Expects Modules

**What goes wrong:**
Display shows "RS Corrections: 3/8" meaning 3 codewords corrected out of 8 capacity. User interprets this as "3 pixels wrong" when they've painted 50 pixels differently. The disconnect causes confusion.

**Why it happens:**
RS correction operates on 8-bit codewords, not visual modules. One painted module might affect 0, 1, or multiple codewords depending on:
- Data encoding mode
- Position in data stream
- Interleaving across blocks
- Whether module is in data vs function pattern

**Consequences:**
- User thinks capacity is pixels, not codewords
- "I only changed 3 pixels, why is capacity at 50%?" (3 pixels affected 4 codewords)
- "I changed 50 pixels and it shows 0 corrections?" (all were function pattern)

**Prevention:**
- UI labels must be crystal clear: "RS Corrections: 3 codewords (capacity: 8)"
- Consider adding explanation: "Codewords are internal data units, not pixels"
- Track both metrics: painted pixel count AND RS correction count
- v1.4 already established this distinction — maintain consistency

**Detection:**
- User testing: Ask "what does 3/8 mean?"
- Ensure existing pixel diff metric remains visible alongside RS metrics

**Phase to address:**
Phase 1 (Capacity Display) — Label clarity critical

---

### Pitfall 5: SVG viewBox vs Width/Height Mismatch

**What goes wrong:**
SVG exported with `viewBox="0 0 25 25"` (QR module count) but `width="600" height="600"` (display size). Depending on application:
1. Some programs scale correctly
2. Others show tiny QR in corner
3. Others crop to 25x25 pixels

**Why it happens:**
SVG has two independent size specifications:
- `viewBox`: defines the coordinate system
- `width`/`height`: defines actual size

The existing QRCode library already handles this (line 1841-1846) with:
```javascript
var h = g("svg", {
  viewBox: "0 0 " + String(d) + " " + String(d),
  width: "100%",
  height: "100%",
  // ...
});
```

But if copying/modifying this for export, developer might:
- Use fixed pixel dimensions (breaks in some apps)
- Omit viewBox (breaks scaling)
- Mismatch units (viewBox in modules, width/height in pixels)

**Consequences:**
- SVG appears tiny or huge depending on viewing application
- Professional design tools (Illustrator, Figma) may misinterpret
- Print at wrong size

**Prevention:**
```xml
<!-- RECOMMENDED: viewBox matches modules, dimensions use viewBox for scaling -->
<svg viewBox="0 0 25 25" width="250" height="250" ...>

<!-- OR: Use percentage for responsive sizing -->
<svg viewBox="0 0 25 25" width="100%" height="100%" ...>

<!-- For fixed pixel output at 10px per module: -->
<svg viewBox="0 0 25 25" width="250" height="250" ...>
```

**Detection:**
- Test exported SVG in: browser, Illustrator, Figma, Inkscape
- Check physical size when printing

**Phase to address:**
Phase 2 (SVG Export) — Test in multiple applications

---

### Pitfall 6: SVG Colors Not Matching Canvas

**What goes wrong:**
Canvas uses `#000000` for black modules, SVG uses `rgb(0,0,0)` or `black`. These are equivalent, but other colors diverge:
1. Painted regions use `#404040` and `#d0d0d0` — must match in SVG
2. Protected areas have overlay color — should these export?
3. Green painted borders — should these export?

**Why it happens:**
The canvas rendering code in `renderQRWithOverlay()` uses specific color constants:
```javascript
const PAINTED_WHITE = "#d0d0d0";
const PAINTED_BLACK = "#404040";
const PAINTED_BORDER_COLOR = "#22c55e";
const PROTECTED_OVERLAY_COLOR = "rgba(59, 130, 246, 0.25)";
```

SVG export might not use the same constants, or might try to export UI overlays that shouldn't be in final output.

**Consequences:**
- SVG looks different from canvas preview
- Professional designers can't match brand colors
- UI elements (borders, overlays) pollute export

**Prevention:**
- **Export ONLY the QR data**, not UI overlays
- Use same color constants for SVG as canvas
- Decide: painted regions export as intended final color (black/white) or as preview color (gray)?
- Most likely: Export painted regions as the actual QR color (not gray preview)

**Detection:**
- Visual comparison: screenshot canvas vs exported SVG
- Check exported SVG source for unexpected colors

**Phase to address:**
Phase 2 (SVG Export) — Define export color spec before implementing

---

### Pitfall 7: SVG File Naming Creates Downloads Folder Clutter

**What goes wrong:**
User clicks "Download SVG" multiple times while iterating. Each download is named `qr-code.svg`, overwriting previous or (worse) creating `qr-code (1).svg`, `qr-code (2).svg`, etc.

**Why it happens:**
Simple download implementation uses hardcoded filename:
```javascript
link.download = "qr-code.svg"; // Same name every time
```

**Consequences:**
- User's Downloads folder fills with identically-named files
- Can't tell which version is which
- Accidental overwrites lose work

**Prevention:**
Include distinguishing info in filename:
```javascript
// Option 1: Timestamp
const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
link.download = `qr-${timestamp}.svg`;

// Option 2: URL-based (sanitized)
const urlSlug = new URL(encodedUrl).hostname.replace(/\./g, '-');
link.download = `qr-${urlSlug}.svg`;

// Option 3: Version + hash
link.download = `qr-v${version}-${shortHash}.svg`;
```

**Detection:**
- Test downloading twice in a row
- Check Downloads folder for naming pattern

**Phase to address:**
Phase 2 (SVG Export) — Define naming convention

---

### Pitfall 8: Shift Key State Not Detected in MouseEvent

**What goes wrong:**
Developer adds Shift+click handling but reads `shiftKey` from the wrong event type or timing:
1. Using `keydown` listener instead of reading `event.shiftKey` from click event
2. Not checking `shiftKey` on `pointerdown` (the existing event type)
3. State race: user releases Shift during drag

**Why it happens:**
The existing painting code uses `pointerdown`/`pointermove` events (line 13559-13598). These events have a `shiftKey` property (inherited from MouseEvent) that indicates Shift state at event time.

Developer might:
- Add separate `keydown`/`keyup` listeners creating state management bugs
- Check `shiftKey` on wrong event
- Not handle Shift state changes during drag

**Consequences:**
- Shift+click does nothing (event not checked)
- Modifier state gets "stuck" (separate key listener out of sync)
- Inconsistent behavior during drag

**Prevention:**
```javascript
container.addEventListener("pointerdown", function (e) {
  // CORRECT: Check shiftKey directly on the pointer event
  const isShiftHeld = e.shiftKey;
  
  // Store for use during drag
  currentPaintModifier = isShiftHeld ? 'shift' : 'none';
  
  // Use modifier in paintAt logic
  paintAt(coords.x, coords.y, currentPaintButton, currentPaintModifier);
});

// During drag, check current event's shiftKey
container.addEventListener("pointermove", function (e) {
  // Could update modifier during drag if desired:
  // currentPaintModifier = e.shiftKey ? 'shift' : 'none';
});
```

**Detection:**
- Test: Hold Shift, click, release Shift, click again (behavior should differ)
- Test: Start click, then press/release Shift during drag
- Log `e.shiftKey` to confirm it's being read

**Phase to address:**
Phase 3 (Shift+Paint) — Use event.shiftKey, not separate listener

---

### Pitfall 9: Shift+Click Conflicts with Browser/OS

**What goes wrong:**
Shift+click has existing browser behaviors:
1. **Text selection**: Shift+click extends selection (not relevant for canvas)
2. **Open in new window**: Shift+click on links opens new window
3. **macOS**: Shift+click can trigger accessibility features
4. **Firefox**: Shift+right-click shows context menu even if `contextmenu` is prevented

**Why it happens:**
Shift is a common modifier with pre-existing meanings. The app already prevents `contextmenu` for right-click painting, but Shift+right-click in Firefox bypasses this.

**Consequences:**
- Firefox: Shift+right-click shows context menu instead of painting
- macOS: Potential accessibility interference
- Edge cases where browser captures event before app

**Prevention:**
```javascript
// Already in codebase (line 13601-13603):
container.addEventListener("contextmenu", function (e) {
  e.preventDefault();
});

// This should work, but test Firefox specifically
// For Firefox Shift+right-click: may need additional handling

// Also prevent text selection during painting:
container.style.userSelect = 'none';
```

**Detection:**
- Test Shift+right-click in Firefox
- Test on macOS with accessibility features enabled
- Test with browser extensions that use Shift modifier

**Phase to address:**
Phase 3 (Shift+Paint) — Test cross-browser

---

### Pitfall 10: Shift+Paint Behavior Conflicts with Existing Right-Click

**What goes wrong:**
Existing painting behavior:
- Left-click: Paint selected color
- Right-click: Paint opposite color

Adding Shift modifier:
- Shift+Left-click: Paint opposite (like right-click currently does)

This creates redundant functionality and potential confusion:
- Right-click already does what Shift+left-click would do
- What should Shift+right-click do?

**Why it happens:**
The existing `getPaintValue()` function (line 13469-13482) already maps right-click to opposite color:
```javascript
} else if (button === 2) {
  // Right click: paint opposite color
  if (selectedColor === "black") return 1; // white
  if (selectedColor === "white") return 2; // black
}
```

Adding Shift as a modifier creates overlapping functionality.

**Consequences:**
- Two ways to paint opposite (redundant)
- Shift+right-click behavior undefined
- User confusion about which method to use

**Prevention:**
**Option A: Shift+left replaces right-click entirely**
- Remove right-click opposite behavior
- Shift+left paints opposite
- Right-click reserved for future features

**Option B: Keep both, document as equivalent**
- Right-click and Shift+left-click both paint opposite
- Accessibility benefit: laptop trackpads without right-click
- Shift+right-click = Shift+left-click = right-click (all opposite)

**Option C: Make Shift do something different**
- Instead of "opposite color", Shift could mean "toggle" (if black, make white, if white, make black)
- Or Shift = "force unset" regardless of selection

**Recommendation:** Option B — keep both as equivalent. Some users prefer keyboard modifiers, others prefer mouse buttons. Document that they're equivalent.

**Phase to address:**
Phase 3 (Shift+Paint) — Define behavior before implementing

---

### Pitfall 11: Not Updating `lastPaintedCell` for Shift State Changes

**What goes wrong:**
Existing drag-painting debounces by tracking `lastPaintedCell` to avoid repainting the same cell. If user:
1. Starts painting cell (1,1) with left-click
2. Presses Shift during drag
3. Moves back to (1,1)

The cell (1,1) should be repainted with opposite color, but debounce blocks it.

**Why it happens:**
Current debounce check (line 13518-13522):
```javascript
const cellKey = `${x},${y}`;
// Debounce: don't repaint same cell during drag
if (lastPaintedCell === cellKey) return;
```

This doesn't account for modifier changes during stroke.

**Consequences:**
- Shift press during drag doesn't affect already-painted cells
- User has to release and restart stroke to repaint

**Prevention:**
Include modifier state in debounce key:
```javascript
const modifierState = currentPaintModifier; // or e.shiftKey
const cellKey = `${x},${y}:${modifierState}`;
if (lastPaintedCell === cellKey) return;
```

**Or:** Clear `lastPaintedCell` when modifier changes:
```javascript
if (previousShiftState !== e.shiftKey) {
  lastPaintedCell = null;
  previousShiftState = e.shiftKey;
}
```

**Detection:**
- Test: Start painting, press Shift during drag, move back to earlier cells

**Phase to address:**
Phase 3 (Shift+Paint) — Consider during implementation

---

### Pitfall 12: Touch Devices Can't Use Shift Modifier

**What goes wrong:**
Touch device users have no physical Shift key. If Shift+click is the ONLY way to access a feature, touch users are excluded.

**Why it happens:**
Keyboard modifiers are desktop-centric. The existing app uses `pointerdown` for touch compatibility, but Shift modifier won't work.

**Consequences:**
- Touch users can't access Shift-modified functionality
- Feature parity broken between desktop and mobile

**Prevention:**
Since v1.5 feature is specifically "Shift+paint", it's a desktop enhancement. Ensure:
1. Right-click opposite painting still works (currently exists)
2. Touch users aren't worse off — they just don't get the shortcut
3. Consider future: long-press for modifier on touch?

**For v1.5:** Document that Shift+paint is keyboard shortcut only, touch users use the existing color selector buttons.

**Phase to address:**
Phase 3 (Shift+Paint) — Document as keyboard shortcut

---

## Moderate Pitfalls

### Pitfall 13: Capacity Display Updates Not Synchronized with Decode Result

**What goes wrong:**
RS error count and capacity display are separate UI elements. When a new QR is generated or optimization runs:
- Capacity display updates immediately
- Error count updates after decode
- Brief moment of mismatch

**Prevention:**
- Update both metrics atomically in single render pass
- Or: Hide/gray error count until decode complete
- Ensure same decode result populates both fields

**Phase to address:**
Phase 1 (Capacity Display)

---

### Pitfall 14: SVG Export Button Enabled Before QR Generated

**What goes wrong:**
Export button is enabled when no QR code exists, or when showing stale QR from previous session. Click produces:
- Empty SVG
- SVG of wrong QR
- Error

**Prevention:**
- Disable export button when `qrModel` is null
- Tie button state to same conditions as paint section visibility
- Show tooltip: "Generate a QR code first"

**Phase to address:**
Phase 2 (SVG Export)

---

### Pitfall 15: Large QR Versions Create Huge SVG Files

**What goes wrong:**
Version 40 QR has 177x177 = 31,329 modules. If each module is an SVG `<rect>` element:
- 31,329 elements × ~50 bytes each = ~1.5MB SVG
- Slow to open in browsers
- Design tools may choke

**Why it happens:**
Naive SVG generation creates one element per module:
```xml
<rect x="0" y="0" width="1" height="1"/>
<rect x="1" y="0" width="1" height="1"/>
<!-- ... 31,327 more -->
```

**Prevention:**
Option A: Use `<use>` with template (existing code does this):
```xml
<rect id="template" width="1" height="1"/>
<use href="#template" x="0" y="0"/>
<use href="#template" x="1" y="0"/>
```
Still many elements, but smaller file size.

Option B: Consolidate runs of dark modules:
```xml
<rect x="0" y="0" width="3" height="1"/> <!-- 3 consecutive dark modules -->
```
Complex to implement, significant size reduction.

Option C: Use a path with all modules as one element:
```xml
<path d="M0,0h1v1h-1z M1,0h1v1h-1z ..."/>
```
Single element, reasonable size.

For v1.5: Option A (existing pattern) is acceptable. Add optimization if file size becomes issue.

**Phase to address:**
Phase 2 (SVG Export) — Monitor file sizes

---

### Pitfall 16: Modifier Key State Persists After Window Blur

**What goes wrong:**
User holds Shift, clicks outside app window (blur), releases Shift. Returns to app — the `shiftKey` state from last event is stale.

**Why it happens:**
If storing modifier state in variable rather than reading from each event:
```javascript
let isShiftHeld = false;
document.addEventListener('keydown', e => { if (e.key === 'Shift') isShiftHeld = true; });
document.addEventListener('keyup', e => { if (e.key === 'Shift') isShiftHeld = false; });
// Misses keyup when window unfocused
```

**Prevention:**
Read `event.shiftKey` from each mouse/pointer event, don't maintain separate state:
```javascript
container.addEventListener("pointerdown", function (e) {
  // Fresh read every event
  const isShift = e.shiftKey;
  paintAt(coords.x, coords.y, e.button, isShift);
});
```

**Phase to address:**
Phase 3 (Shift+Paint)

---

## Minor Pitfalls

### Pitfall 17: SVG Download Opens in New Tab (Safari)

**What goes wrong:**
Safari sometimes opens data URIs in new tab instead of downloading. User expects file download, gets new tab with raw SVG.

**Prevention:**
Use Blob URL instead of data URI:
```javascript
const blob = new Blob([svgContent], { type: 'image/svg+xml' });
const url = URL.createObjectURL(blob);
link.href = url;
link.click();
URL.revokeObjectURL(url); // Clean up
```

**Phase to address:**
Phase 2 (SVG Export)

---

### Pitfall 18: Capacity Percentage Shows "Infinity%" for Zero Capacity

**What goes wrong:**
If capacity calculation returns 0 (shouldn't happen with valid QR), percentage calculation divides by zero:
```javascript
const percentage = (errors / capacity) * 100; // NaN or Infinity if capacity=0
```

**Prevention:**
```javascript
const percentage = capacity > 0 ? (errors / capacity) * 100 : 0;
```

**Phase to address:**
Phase 1 (Capacity Display)

---

### Pitfall 19: Painted Regions Don't Export to SVG

**What goes wrong:**
User paints a pattern, exports SVG, painted regions are missing because export only includes base QR structure.

**Why it matters:**
If user intends to share their artistic QR, the painted pattern IS the art.

**Prevention:**
When exporting, render the QR as it appears on canvas:
- Modules with paintState=1 (white paint) → white fill
- Modules with paintState=2 (black paint) → black fill
- Modules with paintState=0 → QR's natural state

**Phase to address:**
Phase 2 (SVG Export) — Must include painted regions

---

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| Capacity from RS_BLOCK_TABLE | Using raw value as capacity | Divide by 2: `capacity = ecCodewords / 2` |
| Version lookup | Using version directly as index | Use `(version - 1) * 4 + ecLevelIndex` |
| SVG export colors | Using different constants than canvas | Share color constants: `PAINTED_WHITE`, `PAINTED_BLACK` |
| Shift detection | Adding keydown/keyup listeners | Use `event.shiftKey` from pointer events |
| Painted region export | Exporting base QR only | Include paint state in export rendering |

## Phase-Specific Warnings

| Phase | Topic | Likely Pitfall | Mitigation |
|-------|-------|----------------|------------|
| 1 | Capacity Display | Wrong formula (not dividing by 2) | Test against spec table |
| 1 | Capacity Display | Off-by-one version lookup | Test version 2+ explicitly |
| 2 | SVG Export | Colors don't match canvas | Use same color constants |
| 2 | SVG Export | Painted regions missing | Include paint state in render |
| 2 | SVG Export | viewBox/size mismatch | Test in multiple apps |
| 3 | Shift+Paint | Separate key listener stale state | Use event.shiftKey |
| 3 | Shift+Paint | Conflicts with right-click | Define relationship clearly |
| 3 | Shift+Paint | Debounce blocks modifier change | Include modifier in debounce key |

## "Looks Done But Isn't" Checklist

- [ ] **Capacity formula:** Divides EC codewords by 2, not using raw value
- [ ] **Version lookup:** Tested with Version 2+ (catches off-by-one)
- [ ] **Level H hardcoded:** Not over-engineering for L/M/Q
- [ ] **SVG colors:** Match canvas exactly
- [ ] **SVG painted regions:** Included in export
- [ ] **SVG file naming:** Includes timestamp or identifier
- [ ] **Shift detection:** Using event.shiftKey, not separate listener
- [ ] **Shift + right-click:** Behavior defined
- [ ] **Touch fallback:** Documented that Shift is desktop-only shortcut

## Sources

**MDN Documentation (authoritative):**
- [KeyboardEvent.shiftKey](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/shiftKey) — "Widely available since July 2015"
- [MouseEvent.shiftKey](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/shiftKey) — Inherited by PointerEvent
- [SVG Tutorial: Getting Started](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorials/SVG_from_scratch/Getting_started)
- [Element: contextmenu event](https://developer.mozilla.org/en-US/docs/Web/API/Element/contextmenu_event) — Firefox Shift+right-click note

**QR Code RS Specification:**
- [Thonky QR Tutorial: Error Correction](https://www.thonky.com/qr-code-tutorial/error-correction-coding)
- RS_BLOCK_TABLE structure: `/workspace/index.html` lines 1568-1729
- Correction capacity formula: `t = ecCodewordsPerBlock / 2`

**Existing Codebase Analysis:**
- Paint event handling: `/workspace/index.html` lines 13553-13604
- `getPaintValue()` function: lines 13469-13482
- Color constants: lines 13606-13611
- QRCode library SVG generation: lines 1820-1878
- Version table lookup: lines 1745-1758

---
*Pitfalls research for: v1.5 Feature Additions (Capacity Display, SVG Export, Shift+Paint)*
*Researched: 2026-02-17*
