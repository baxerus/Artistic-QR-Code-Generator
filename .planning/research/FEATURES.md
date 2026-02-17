# Feature Research: v1.5 UX & Export Enhancements

**Domain:** QR Art Tool UX improvements
**Researched:** 2026-02-17
**Confidence:** HIGH (based on existing codebase analysis, MDN documentation, standard UX patterns)

## Context

This is milestone-specific research for v1.5 features being added to an existing Artistic QR Code Generator. The app already has:
- RS correction count display ("Corrections: N")
- PNG download button per result card
- Left-click/right-click painting with legend-based color selection
- Drag painting support with pointer capture

**Goal:** Add three UX enhancements:
1. "X of Y" capacity indicator for RS corrections
2. SVG export button alongside existing PNG export
3. Shift+paint keyboard shortcut for opposite color painting

## Feature Landscape

### Feature 1: RS Corrections Capacity Indicator

#### Table Stakes (Users Expect These)

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Show current RS corrections | Already have ("Corrections: N") | DONE | Existing implementation |
| Show maximum capacity | Users want to know "how much headroom remains" | LOW | Requires lookup table by version+EC level |
| Clear "X of Y" or "X/Y" format | Standard UX pattern for capacity | LOW | Match existing label style |

#### Differentiators

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Visual progress bar | Quick visual assessment of headroom | MEDIUM | Optional; text is sufficient for v1.5 |
| Color coding (green/yellow/red) | Instant "is this safe?" feedback | LOW | Nice but not essential |
| Per-block breakdown | Technical users may want detail | HIGH | Defer - total count is sufficient |

#### Anti-Features

| Feature | Why Avoid | Alternative |
|---------|-----------|-------------|
| Percentage display only | "75% capacity" loses precision; X/Y is clearer | Show both count and total |
| Predicted success probability | Too many scanner-dependent variables | Show raw RS usage instead |

#### Display Format Options

| Format | Example | Pros | Cons |
|--------|---------|------|------|
| "RS: X of Y" | "RS: 3 of 30" | Clear, matches existing | Slightly longer |
| "RS: X/Y" | "RS: 3/30" | Compact | May look like fraction |
| "X corrections (Y max)" | "3 corrections (30 max)" | Very explicit | Verbose |

**Recommendation:** `RS: X of Y` format - clear, consistent with existing "Corrections: N" label style, easily understood.

#### RS Capacity Lookup

QR error correction capacity depends on version and EC level. App uses EC level H exclusively.

| Version | Total Codewords | EC Codewords (Level H) | Max Correctable* |
|---------|-----------------|------------------------|------------------|
| 1 | 26 | 17 | 8 |
| 2 | 44 | 28 | 14 |
| 3 | 70 | 44 | 22 |
| 4 | 100 | 64 | 32 |
| 5 | 134 | 88 | 44 |
| 6 | 172 | 112 | 56 |
| 7 | 196 | 130 | 65 |
| 8 | 242 | 156 | 78 |
| 9 | 292 | 192 | 96 |
| 10 | 346 | 224 | 112 |

*Max correctable = EC codewords / 2 (RS can correct t errors using 2t parity bytes)

**Implementation:** Lookup table keyed by version, returns max correctable for level H.

```javascript
// RS capacity lookup for EC level H
const RS_CAPACITY_H = {
  1: 8, 2: 14, 3: 22, 4: 32, 5: 44,
  6: 56, 7: 65, 8: 78, 9: 96, 10: 112,
  // Continue for versions 11-40 if needed
};

function getRSCapacity(version) {
  return RS_CAPACITY_H[version] || null;
}
```

#### Dependencies on Existing Features

- **Requires:** `result.rsCorrections` field (already available from Phase 12)
- **Requires:** `currentVersion` (already tracked globally)
- **Integrates with:** Result card display (lines 16259-16289)

---

### Feature 2: SVG Export Button

#### Table Stakes (Users Expect These)

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Download SVG button | Users expect format choice for vector vs raster | LOW | New button alongside PNG |
| Vector-crisp output | SVG should scale infinitely | LOW | QR is grid-based, maps to rects |
| Same filename pattern | Consistency with PNG export | LOW | Just change extension |
| Works on all result cards | Consistency with existing PNG button | LOW | Same integration pattern |

#### Differentiators

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Embedded metadata | URL in SVG comments | LOW | Nice for traceability |
| Custom dimensions dialog | Let user specify output size | MEDIUM | Defer - SVG scales anyway |
| Styled modules (rounded corners) | Artistic flair | HIGH | Out of scope for v1.5 |

#### Anti-Features

| Feature | Why Avoid | Alternative |
|---------|-----------|-------------|
| Canvas-to-SVG conversion | Rasterizes, defeats purpose | Generate SVG from moduleData directly |
| Complex path optimization | Over-engineering for grid | Simple rect elements |
| Inline CSS in SVG | Browser compatibility issues | Use attributes |

#### SVG Generation Pattern

QR codes map perfectly to SVG: each module becomes a `<rect>` element.

```javascript
function generateQRSvg(moduleData, moduleCount, pixelSize = 10) {
  const size = moduleCount * pixelSize;
  let rects = '';
  
  for (let y = 0; y < moduleCount; y++) {
    for (let x = 0; x < moduleCount; x++) {
      if (moduleData[y * moduleCount + x] === 1) { // Black module
        rects += `<rect x="${x * pixelSize}" y="${y * pixelSize}" width="${pixelSize}" height="${pixelSize}" fill="black"/>`;
      }
    }
  }
  
  return `<?xml version="1.0" encoding="UTF-8"?>
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 ${size} ${size}" width="${size}" height="${size}">
  <rect width="100%" height="100%" fill="white"/>
  ${rects}
</svg>`;
}
```

#### Download Pattern

Use same Blob+download pattern as PNG export:

```javascript
function downloadSvg(svgString, filename) {
  const blob = new Blob([svgString], { type: 'image/svg+xml' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `${filename}.svg`;
  a.click();
  URL.revokeObjectURL(url);
}
```

#### Button Placement

Current result card actions (line 16298-16328):
```
[Download PNG] [Copy URL]
```

After v1.5:
```
[Download PNG] [Download SVG] [Copy URL]
```

#### Dependencies on Existing Features

- **Requires:** `result.moduleData` (already available)
- **Requires:** `result.moduleCount` (already available)
- **Integrates with:** Result actions container (line 16298)
- **Pattern follows:** Existing `downloadCanvasAsPNG()` pattern

---

### Feature 3: Shift+Paint Keyboard Shortcut

#### Table Stakes (Users Expect These)

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Shift+click paints opposite color | Standard drawing app convention | LOW | Check `event.shiftKey` |
| Works with drag painting | Consistent with existing drag | LOW | Check shiftKey on each event |
| Visual feedback that Shift is held | Users know mode is active | LOW-MEDIUM | Optional cursor change or legend highlight |

#### Differentiators

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Cursor change when Shift held | Visual confirmation | MEDIUM | CSS cursor + keydown/keyup listeners |
| Legend indicator | Show which color will paint | LOW | Highlight opposite legend button |
| Keyboard shortcut for colors (B/W/U) | Power user efficiency | MEDIUM | Defer to v1.6 |

#### Anti-Features

| Feature | Why Avoid | Alternative |
|---------|-----------|-------------|
| Hold-to-toggle (releases on mouseup) | Unpredictable behavior | Shift+click = discrete opposite |
| Ctrl modifier | Reserved for zoom/pan in browsers | Use Shift |
| Alt modifier | Reserved for browser menus | Use Shift |

#### Behavior Matrix

Current behavior (from `getPaintValue()` at lines 13464-13482):

| Action | Selected Color | Result |
|--------|----------------|--------|
| Left-click | Black | Paint black |
| Left-click | White | Paint white |
| Left-click | Unset | Paint unset |
| Right-click | Black | Paint white (opposite) |
| Right-click | White | Paint black (opposite) |
| Right-click | Unset | Paint unset |

**With Shift modifier:**

| Action | Selected Color | Result |
|--------|----------------|--------|
| Shift+Left-click | Black | Paint white (opposite) |
| Shift+Left-click | White | Paint black (opposite) |
| Shift+Left-click | Unset | Paint unset (no opposite) |
| Shift+Right-click | Black | Paint black (reverts to normal) |
| Shift+Right-click | White | Paint white (reverts to normal) |

**Simplest implementation:** Treat Shift+Left-click same as Right-click.

```javascript
function getPaintValue(button, shiftKey = false) {
  // Shift+left-click behaves like right-click (paints opposite)
  const effectiveButton = (button === 0 && shiftKey) ? 2 : button;
  
  if (effectiveButton === 0) {
    // Left click: paint selected color
    if (selectedColor === "black") return 2;
    if (selectedColor === "white") return 1;
    if (selectedColor === "unset") return 0;
  } else if (effectiveButton === 2) {
    // Right click (or Shift+left): paint opposite
    if (selectedColor === "unset") return 0;
    if (selectedColor === "black") return 1;
    if (selectedColor === "white") return 2;
  }
  return null;
}
```

#### Event Handler Changes

In `pointerdown` handler (line 13559):
```javascript
container.addEventListener("pointerdown", function (e) {
  // ...
  paintAt(coords.x, coords.y, currentPaintButton, e.shiftKey);
});
```

In `pointermove` handler (line 13577):
```javascript
container.addEventListener("pointermove", function (e) {
  // ...
  paintAt(coords.x, coords.y, currentPaintButton, e.shiftKey);
});
```

#### Visual Feedback (Optional Enhancement)

Simplest: Change cursor to "crosshair" when Shift is held.

```javascript
document.addEventListener('keydown', (e) => {
  if (e.key === 'Shift') {
    container.style.cursor = 'crosshair';
  }
});
document.addEventListener('keyup', (e) => {
  if (e.key === 'Shift') {
    container.style.cursor = 'pointer';
  }
});
```

#### Dependencies on Existing Features

- **Modifies:** `getPaintValue()` function (lines 13464-13482)
- **Modifies:** `pointerdown`/`pointermove` handlers (lines 13559-13584)
- **Integrates with:** Existing painting system
- **Non-breaking:** Existing behavior preserved when Shift not held

---

## Feature Dependencies

```
[Existing RS Display]
    └── extends-to ──> [RS Capacity Indicator]
                           └── requires ──> [Version lookup table]
                           └── requires ──> [currentVersion state]

[Existing PNG Export]
    └── parallel-to ──> [SVG Export]
                           └── requires ──> [moduleData from result]
                           └── uses ──> [same download pattern]

[Existing Paint System]
    └── extends-to ──> [Shift+Paint Shortcut]
                           └── modifies ──> [getPaintValue() function]
                           └── modifies ──> [pointer event handlers]
```

---

## MVP Definition (v1.5)

### Launch With

- [x] RS capacity indicator showing "RS: X of Y" format
- [x] SVG export button on result cards
- [x] Shift+left-click paints opposite color

### Add After Validation (v1.5.x)

- [ ] Visual health bar for RS capacity (Trigger: users want quick visual)
- [ ] Color-coded RS capacity (Trigger: users want warning when near limit)
- [ ] Cursor change when Shift held (Trigger: users confused about mode)
- [ ] Keyboard shortcuts for color selection (Trigger: power user requests)

### Future Consideration (v2+)

- [ ] Per-block RS breakdown (Why defer: adds complexity, total count is sufficient)
- [ ] Custom SVG styling options (Why defer: out of scope for utility tool)
- [ ] Multiple modifier keys for paint modes (Why defer: complexity)

---

## Complexity Assessment

| Feature | Complexity | Estimated Changes | Risk |
|---------|------------|-------------------|------|
| RS Capacity Indicator | LOW | ~30 lines (lookup table + display update) | None |
| SVG Export Button | LOW | ~50 lines (SVG generator + button handler) | None |
| Shift+Paint Shortcut | LOW | ~15 lines (event handler updates) | None |

**Total estimated changes:** ~95 lines across 3 features.

---

## Sources

### Primary (HIGH confidence)

- **MDN Blob API**: https://developer.mozilla.org/en-US/docs/Web/API/Blob - SVG download pattern
- **MDN SVG Tutorial**: https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Getting_Started - SVG structure
- **Existing codebase analysis**: index.html lines 13464-13604 (paint system), 16298-16328 (export buttons)
- **QR Code spec (ISO/IEC 18004)**: EC capacity tables for RS correction limits

### Secondary (MEDIUM confidence)

- **UX patterns observation**: Standard "X of Y" format used in storage indicators, progress bars, pagination
- **Drawing tool conventions**: Shift modifier for opposite/constrain is standard (Photoshop, Figma, etc.)

---

## Confidence Assessment

| Area | Confidence | Reason |
|------|------------|--------|
| RS capacity lookup | HIGH | QR spec defines EC codeword counts per version |
| SVG generation | HIGH | QR is grid-based, maps directly to rect elements |
| Shift modifier pattern | HIGH | Standard UX convention, native browser support |
| Integration points | HIGH | Direct codebase analysis, clear modification points |

---
*Feature research for: v1.5 UX & Export Enhancements*
*Milestone: v1.5*
*Researched: 2026-02-17*
