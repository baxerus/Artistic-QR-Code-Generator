# Phase 5: Configuration Constants - Research

**Researched:** 2026-02-10
**Domain:** JavaScript configuration constants, QR code specifications
**Confidence:** HIGH

## Summary

This phase extracts hardcoded QR version limits into a single governing constant and expands the supported range from versions 2-8 to versions 2-10. The current implementation has the magic number `8` scattered across multiple locations (capacity table, dropdown loop, validation messages). The standard approach is to centralize this as a named constant using JavaScript's `const` keyword with UPPER_SNAKE_CASE naming.

QR code versions 9 and 10 have well-defined specifications: Version 9 is 53×53 modules with 85-98 bytes capacity at error correction level H, and Version 10 is 57×57 modules with 99-119 bytes capacity at level H. The grid size formula `modules = 17 + 4 × version` is already correctly implemented via `getModuleSize()`.

Visual feedback for version auto-adjustment should use the existing `result-highlight` animation (green box-shadow glow, 0.6s duration) which matches modern micro-interaction patterns (100-300ms for feedback, up to 600ms for attention-drawing effects).

**Primary recommendation:** Define `const MAX_QR_VERSION = 10` and `const MIN_QR_VERSION = 2` at module level (near existing CAPACITIES_H_BYTE), extend capacity table to version 10, replace all hardcoded `8` and `2` references with constants, apply `result-highlight` class to dropdown on auto-bump.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Default version selection:**
- Default version on fresh load is always Version 2 (smallest)
- No session persistence for this phase (Phase 6 handles that)
- Version dropdown is locked/disabled until a URL is entered
- Once a URL is entered, only show versions that can encode that URL (hide versions that are too small)

**Version auto-adjustment UX:**
- When a URL requires a higher version than currently selected, auto-bump to the exact minimum required version
- Provide visual feedback on auto-bump: a short green glow animation on the dropdown (same style as the updated-results animation)
- When user manually selects a version higher than needed, allow it silently (they may want more painting room)
- Version only auto-increases, never auto-decreases — if a URL gets shorter, version stays where it is; user manually lowers if desired

### Claude's Discretion

- Version dropdown display format (labels, whether to show grid dimensions)
- Exact constant naming and file location
- How the glow animation is implemented (CSS transition details)

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope

</user_constraints>

## Standard Stack

This is a single-file vanilla JavaScript application using the qrcodejs library for QR generation. No build tooling or module system is used.

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| qrcodejs | embedded | QR code generation | Lightweight, client-side QR encoding with error correction support |
| Native JavaScript | ES6+ | Application logic | No framework needed for this scope |
| Native CSS | CSS3 | Styling & animations | Sufficient for keyframe animations and transitions |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| jsQR | embedded | QR decoding (Web Worker) | Validation during hash optimization |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Inline constants | External config file | Adds complexity for single-file app |
| Magic numbers | Named constants | Constants improve maintainability (RECOMMENDED) |

**Installation:**
No installation needed - libraries are embedded in index.html.

## Architecture Patterns

### Recommended Constant Location

Place constants at module level (script scope) where related data structures already exist:

```javascript
// Current location (around line 13144):
// QR capacity table for error correction level H, byte mode (versions 1-8)
const CAPACITIES_H_BYTE = [ ... ];

// ADD HERE:
const MIN_QR_VERSION = 2;
const MAX_QR_VERSION = 10;
```

**Why this location:**
- Co-located with capacity data that depends on these values
- Module-level scope accessible to all functions
- Follows existing pattern (CAPACITIES_H_BYTE already uses UPPER_SNAKE_CASE)
- Clear visibility for future modifications

### Pattern 1: Single Source of Truth for Limits

**What:** Define version range once, derive all behavior from it

**When to use:** Any configurable range or limit that affects multiple parts of the codebase

**Example:**
```javascript
// Source: JavaScript configuration best practices
// https://medium.com/@z_callan/javascript-project-architecture-constants-deddfde3c8a8

// Define once
const MIN_QR_VERSION = 2;
const MAX_QR_VERSION = 10;

// Use everywhere
const CAPACITIES_H_BYTE = [
  { version: MIN_QR_VERSION, capacity: 14 },
  // ... up to ...
  { version: MAX_QR_VERSION, capacity: 119 },
];

function updateVersionDropdown(minVersion) {
  for (let v = minVersion; v <= MAX_QR_VERSION; v++) {
    // ...
  }
}

function calculateMinimumVersion(url) {
  // ...
  return -1; // URL too long for MAX_QR_VERSION
}
```

### Pattern 2: Visual Feedback via CSS Class Toggle

**What:** Apply/remove animation class to trigger keyframe animation

**When to use:** Temporary visual feedback that should play once and automatically end

**Example:**
```javascript
// Source: Existing codebase pattern (result-highlight animation)
// Applied to search results when updated

function updateVersionDropdown(minVersion) {
  const dropdown = document.getElementById("version-select");
  const previousValue = dropdown.value;

  // ... populate dropdown ...

  dropdown.value = minVersion;

  // Trigger animation if value changed (auto-bumped)
  if (previousValue && parseInt(previousValue) !== minVersion) {
    dropdown.classList.add('result-highlight');
    // Animation completes automatically after 0.6s
    // Optional: remove class after animation
    setTimeout(() => dropdown.classList.remove('result-highlight'), 600);
  }
}
```

**Note:** The `result-highlight` class is already defined in CSS and applies a green box-shadow pulse animation for 0.6 seconds.

### Pattern 3: Capacity Table Extension

**What:** Extend array with additional entries while maintaining structure

**When to use:** Adding new versions to existing capacity lookup table

**Example:**
```javascript
// Source: Official QR specifications
// https://ryanagibson.com/extra/qr-character-limits/

const CAPACITIES_H_BYTE = [
  { version: 1, capacity: 7 },
  { version: 2, capacity: 14 },
  { version: 3, capacity: 24 },
  { version: 4, capacity: 34 },
  { version: 5, capacity: 44 },
  { version: 6, capacity: 58 },
  { version: 7, capacity: 64 },
  { version: 8, capacity: 84 },
  { version: 9, capacity: 98 },   // NEW: 53×53 grid
  { version: 10, capacity: 119 }, // NEW: 57×57 grid
];
```

**Capacity values verified:** Version 9-H supports 85-98 bytes (use upper bound 98), Version 10-H supports 99-119 bytes (use upper bound 119).

### Anti-Patterns to Avoid

- **Hardcoding limits in multiple places:** Creates maintenance burden and inconsistency risk. ALWAYS use named constants.
- **Silent auto-adjustment:** Users need feedback when system changes their selection. Per requirements, use green glow animation.
- **Auto-decreasing version:** Violates user expectation of stability. Only auto-increase, never decrease (per locked decisions).
- **Removing class before animation completes:** Let CSS animation finish naturally, or use proper timing (600ms minimum for 0.6s animation).

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| QR capacity calculation | Custom byte-counting logic | Official capacity table lookup | QR spec includes overhead (mode bits, length bits, error correction) that's complex to calculate correctly |
| Animation timing | JavaScript-based transitions | CSS `@keyframes` with class toggle | CSS animations are hardware-accelerated and automatically clean up; JS-based animation requires manual cleanup |
| Version range validation | Multiple boundary checks | Single constant reference | Eliminates desync risk between validation, UI, and error messages |

**Key insight:** QR code capacity isn't just URL byte length - the spec includes mode indicators (4 bits), character count indicators (variable bits), and error correction overhead. Always use pre-calculated capacity tables from official sources rather than attempting to derive capacity from scratch.

## Common Pitfalls

### Pitfall 1: Off-by-One in Capacity Boundaries

**What goes wrong:** Using `<` instead of `<=` in capacity lookup causes URLs at exact capacity limit to be rejected

**Why it happens:** Confusion between "less than capacity" vs "fits within capacity"

**How to avoid:** Capacity check should be `byteLength <= entry.capacity` (not `<`)

**Warning signs:** URLs that should fit in a version are marked as "too long"

### Pitfall 2: Inconsistent Version References

**What goes wrong:** Updating MAX_VERSION constant but missing a hardcoded reference somewhere, causing partial upgrade

**Why it happens:** Search-and-replace misses edge cases (comments, strings, console messages)

**How to avoid:**
1. Define constants first
2. Search for ALL instances of literal `8` in JavaScript section
3. Replace each with constant reference
4. Search for "version 8" strings in error messages

**Warning signs:** Error message says "too long for version 8" when MAX_VERSION = 10

### Pitfall 3: Animation Class Accumulation

**What goes wrong:** Adding animation class multiple times without removal causes CSS to ignore subsequent triggers (animation won't replay)

**Why it happens:** Browser won't restart animation if class is already present

**How to avoid:** Two strategies:
1. Remove class before re-adding: `element.classList.remove('class'); setTimeout(() => element.classList.add('class'), 10);`
2. Remove class after animation completes: `setTimeout(() => element.classList.remove('class'), 600);`

**Warning signs:** Animation plays on first auto-bump but not on subsequent changes

### Pitfall 4: Wrong Grid Dimensions for New Versions

**What goes wrong:** Calculating grid size incorrectly for versions 9-10

**Why it happens:** Misunderstanding the formula or off-by-one error

**How to avoid:** Formula is `modules = 17 + (version × 4)`, already correctly implemented in `getModuleSize(version)`. No changes needed to this function.

**Warning signs:** QR display canvas is wrong size, painting doesn't align with QR modules

**Verification:**
- Version 9: `17 + (9 × 4) = 53` ✓
- Version 10: `17 + (10 × 4) = 57` ✓

### Pitfall 5: Auto-Bump Triggering on Initial Load

**What goes wrong:** Green glow plays when user first enters URL, even though no "change" occurred

**Why it happens:** Not tracking previous value before setting new value

**How to avoid:** Only trigger animation if dropdown had a previous value AND it's different from new value:

```javascript
const previousValue = dropdown.value;
dropdown.value = minVersion;
if (previousValue && parseInt(previousValue) !== minVersion) {
  // Trigger animation
}
```

**Warning signs:** Dropdown glows green on first URL entry (should only glow on auto-increase)

## Code Examples

Verified patterns from the existing codebase and official sources:

### Extending Capacity Table

```javascript
// Source: Existing pattern (line 13145) + official QR specs
// https://ryanagibson.com/extra/qr-character-limits/

// BEFORE (versions 1-8):
const CAPACITIES_H_BYTE = [
  { version: 1, capacity: 7 },
  { version: 2, capacity: 14 },
  { version: 3, capacity: 24 },
  { version: 4, capacity: 34 },
  { version: 5, capacity: 44 },
  { version: 6, capacity: 58 },
  { version: 7, capacity: 64 },
  { version: 8, capacity: 84 },
];

// AFTER (versions 1-10):
const MIN_QR_VERSION = 2;
const MAX_QR_VERSION = 10;

const CAPACITIES_H_BYTE = [
  { version: 1, capacity: 7 },
  { version: 2, capacity: 14 },
  { version: 3, capacity: 24 },
  { version: 4, capacity: 34 },
  { version: 5, capacity: 44 },
  { version: 6, capacity: 58 },
  { version: 7, capacity: 64 },
  { version: 8, capacity: 84 },
  { version: 9, capacity: 98 },   // NEW: 53×53 modules
  { version: 10, capacity: 119 }, // NEW: 57×57 modules
];
```

### Updating Version Dropdown Loop

```javascript
// Source: Existing function (line 13285)

// BEFORE:
function updateVersionDropdown(minVersion) {
  const dropdown = document.getElementById("version-select");
  dropdown.innerHTML = "";

  for (let v = minVersion; v <= 8; v++) {  // ← HARDCODED
    const option = document.createElement("option");
    option.value = v;
    option.textContent = "Version " + v + " (" + getModuleSize(v) + "\u00D7" + getModuleSize(v) + ")";
    dropdown.appendChild(option);
  }

  dropdown.value = minVersion;
}

// AFTER:
function updateVersionDropdown(minVersion) {
  const dropdown = document.getElementById("version-select");
  const previousValue = dropdown.value;
  dropdown.innerHTML = "";

  for (let v = minVersion; v <= MAX_QR_VERSION; v++) {  // ← USE CONSTANT
    const option = document.createElement("option");
    option.value = v;
    option.textContent = "Version " + v + " (" + getModuleSize(v) + "\u00D7" + getModuleSize(v) + ")";
    dropdown.appendChild(option);
  }

  dropdown.value = minVersion;

  // Visual feedback on auto-bump (per locked decisions)
  if (previousValue && parseInt(previousValue) !== minVersion) {
    dropdown.classList.add('result-highlight');
    setTimeout(() => dropdown.classList.remove('result-highlight'), 600);
  }
}
```

### Updating Error Messages

```javascript
// Source: Existing validation function (line 13365)

// BEFORE:
function updateValidationUI(url) {
  // ...
  if (isValidURL(url)) {
    const minVersion = calculateMinimumVersion(url);

    if (minVersion === -1) {
      // ...
      console.error("URL too long for QR version 8");  // ← HARDCODED
      return;
    }
    // ...
  }
}

// AFTER:
function updateValidationUI(url) {
  // ...
  if (isValidURL(url)) {
    const minVersion = calculateMinimumVersion(url);

    if (minVersion === -1) {
      // ...
      console.error(`URL too long for QR version ${MAX_QR_VERSION}`);  // ← USE CONSTANT
      return;
    }
    // ...
  }
}
```

### Updating calculateMinimumVersion Comment

```javascript
// Source: Existing function (line 13177)

// BEFORE:
/**
 * Calculate minimum QR version needed for URL
 * Returns version number (1-8) or -1 if too long
 */

// AFTER:
/**
 * Calculate minimum QR version needed for URL
 * Returns version number (1-10) or -1 if too long
 */
```

### CSS Animation (Already Exists)

```css
/* Source: Existing codebase (line 540)
 * No changes needed - reuse existing animation
 */

@keyframes result-highlight {
  0% {
    box-shadow: 0 0 0 0 rgba(76, 175, 80, 0.7);
  }
  50% {
    box-shadow: 0 0 10px 5px rgba(76, 175, 80, 0.5);
  }
  100% {
    box-shadow: 0 0 0 0 rgba(76, 175, 80, 0);
  }
}

.result-highlight {
  animation: result-highlight 0.6s ease-out;
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Magic numbers scattered in code | Named constants with UPPER_SNAKE_CASE | Modern JS era (ES6+) | Easier maintenance, single source of truth |
| JavaScript-based animation loops | CSS `@keyframes` with class toggle | CSS3 adoption (~2012) | Better performance (GPU acceleration), cleaner code |
| Silent auto-adjustments | Visual micro-interactions for feedback | UX best practices 2020+ | Users understand system behavior, feel in control |

**Deprecated/outdated:**
- Using `var` for constants (ES5): Use `const` keyword for immutable values
- Inline animation via `setInterval`: Use CSS animations with class toggle for better performance

## Open Questions

1. **Should MIN_QR_VERSION constant start at 1 or 2?**
   - What we know: CONTEXT.md specifies "Default version on fresh load is always Version 2 (smallest)"
   - What's unclear: Whether version 1 should be in capacity table but filtered from dropdown, or excluded entirely
   - Recommendation: Include version 1 in CAPACITIES_H_BYTE for completeness (follows existing pattern), but MIN_QR_VERSION = 2 controls what's actually offered in dropdown. Dropdown loop starts at `Math.max(minVersion, MIN_QR_VERSION)`.

2. **Should version dropdown show grid dimensions?**
   - What we know: Current implementation shows "Version 5 (37×37)", CONTEXT.md marks format as Claude's discretion
   - What's unclear: Whether to keep this format or simplify to "Version 5"
   - Recommendation: KEEP grid dimensions - users benefit from understanding QR size, especially when painting. Format is informative without being cluttered.

## Sources

### Primary (HIGH confidence)
- Existing codebase analysis: `/workspace/index.html` - Current implementation patterns, animation definitions, function signatures
- QR Code capacity reference: [Complete Tables of QR Code Character Limits](https://ryanagibson.com/extra/qr-character-limits/) - Official byte mode capacities for versions 9-10 at level H
- QR Code specifications: [QRcode.com Information capacity and versions](https://www.qrcode.com/en/about/version.html) - Grid size formula verification

### Secondary (MEDIUM confidence)
- JavaScript constants best practices: [JavaScript Project Architecture: Constants](https://medium.com/@z_callan/javascript-project-architecture-constants-deddfde3c8a8) - Naming conventions, organization patterns
- JavaScript constants organization: [10 JavaScript Constants File Best Practices](https://climbtheladder.com/10-javascript-constants-file-best-practices/) - UPPER_SNAKE_CASE, descriptive naming
- CSS animation timing: [Enhancing User Experience With CSS Animations](https://stephaniewalter.design/blog/enhancing-user-experience-with-css-animations/) - Micro-interaction timing (100-300ms for feedback)
- Dropdown UX patterns: [Dropdown UI Design: Anatomy, UX, and Use Cases](https://www.setproduct.com/blog/dropdown-ui-design) - Visual feedback for auto-select
- CSS glow effects: [47 Best Glowing Effects in CSS [2026]](https://www.testmuai.com/blog/glowing-effects-in-css/) - Box-shadow animation techniques

### Tertiary (LOW confidence)
- General QR capacity info: [QR Code Storage Capacity by Mode, Version & Error Correction](https://www.qrcodechimp.com/qr-code-storage-capacity-guide/) - Confirmed grid formula but lacked exact byte capacities

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - Direct codebase analysis, no external dependencies needed
- Architecture: HIGH - Patterns verified in existing code, official sources confirm QR specs
- Pitfalls: HIGH - Based on common JavaScript/CSS patterns and existing codebase structure

**Research date:** 2026-02-10
**Valid until:** 2026-03-10 (30 days - stable domain, vanilla JS patterns unlikely to change)

---

## Research Notes

**Key implementation points:**
1. Two constants needed: `MIN_QR_VERSION = 2` and `MAX_QR_VERSION = 10`
2. Capacity table extends with 2 new entries (versions 9, 10)
3. Four code locations need updates: capacity table, dropdown loop, validation message, function comment
4. Animation uses existing `result-highlight` class (0.6s green glow)
5. Auto-bump detection: compare `dropdown.value` before/after, only trigger if changed AND had previous value

**Verified specs:**
- Version 9: 53×53 modules, 98 bytes capacity (level H)
- Version 10: 57×57 modules, 119 bytes capacity (level H)
- Grid formula: `modules = 17 + (version × 4)` ✓ already implemented
- Animation timing: 600ms (0.6s) matches modern micro-interaction patterns (100-600ms range)

**Locked by user:**
- Default to version 2 (not 1)
- Dropdown disabled until URL entered
- Show only versions >= minimum required (hide too-small versions)
- Auto-bump to exact minimum (not higher)
- Green glow on auto-bump (matches result-highlight style)
- Manual selection of higher version is allowed
- Never auto-decrease, only auto-increase
