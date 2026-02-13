# Phase 8: Generation Safety - Research

**Researched:** 2026-02-13
**Domain:** URL Validation, State Guards, Confirmation Dialogs, Pattern Preservation
**Confidence:** HIGH

## Summary

This phase adds safety guards to protect users from accidental pattern loss and invalid URL inputs. The requirements fall into two categories: (1) input validation — reject URLs containing hash fragments since the optimizer uses hash fragments for variations, and (2) generation guards — prevent automatic QR regeneration when a pattern exists, warn before version changes that would destroy patterns.

The existing codebase has strong foundations to build on: `isValidURL()` for URL validation, `paintGrid.cells` for detecting pattern existence, `savePattern()`/`loadPattern()` for persistence, and the version change event handler in `DOMContentLoaded`. The implementation is straightforward DOM manipulation and state checking — no external libraries needed.

**Primary recommendation:** Add hash fragment detection to `isValidURL()`, guard version change event to check pattern existence before auto-generating, and use `window.confirm()` for version change warnings. Pattern preservation on same-version regeneration requires no new logic since `initPaintCanvas()` already restores saved patterns.

## Standard Stack

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Native URL API | Browser | Parse and validate URLs, detect hash fragments | Built-in, reliable hash property access |
| Native confirm() | Browser | User confirmation dialog for destructive actions | Simple, synchronous, familiar UX |

### Supporting

| Technique | Purpose | When to Use |
|-----------|---------|-------------|
| `URL.hash` property | Detect hash fragment in URLs | SAFE-01: Hash fragment rejection |
| Pattern existence check | Loop over `paintGrid.cells` to detect any non-zero values | SAFE-02/03: Guard generation triggers |
| Early return in event handlers | Skip auto-generation when pattern exists | SAFE-02/03: Block auto-generation |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| `window.confirm()` | Custom modal dialog | More styling control but adds complexity; confirm() is sufficient for MVP |
| URL regex for hash detection | Native URL API | URL API is more reliable, handles edge cases |

## Architecture Patterns

### Pattern 1: Hash Fragment Detection

**What:** Detect and reject URLs that already contain a hash fragment (`#section`)

**When to use:** In URL validation before allowing QR generation

**Example:**
```javascript
// Source: MDN URL.hash documentation
function hasHashFragment(urlString) {
  try {
    const url = new URL(urlString);
    // URL.hash includes the '#' character, so "#" means empty hash, "" means no hash
    // We want to reject any non-empty hash (even "#" alone is problematic)
    return url.hash.length > 0;
  } catch (err) {
    return false; // Invalid URL, let isValidURL() handle it
  }
}

// Integrate into isValidURL or add separate check
function isValidURL(string) {
  try {
    const url = new URL(string);
    if (url.protocol !== "http:" && url.protocol !== "https:") return false;
    
    // SAFE-01: Reject URLs with hash fragments
    if (url.hash.length > 0) return false;
    
    const host = url.hostname;
    const dot = host.indexOf(".");
    return dot > 0 && dot < host.length - 1;
  } catch (err) {
    return false;
  }
}
```

### Pattern 2: Pattern Existence Check

**What:** Helper function to check if any painted pixels exist

**When to use:** Before allowing auto-generation, version changes

**Example:**
```javascript
// Source: Existing pattern in codebase (multiple locations)
function hasPattern() {
  if (!paintGrid) return false;
  
  for (let i = 0; i < paintGrid.cells.length; i++) {
    if (paintGrid.cells[i] !== 0) {
      return true;
    }
  }
  return false;
}
```

### Pattern 3: Guarded Version Change Handler

**What:** Check for pattern existence before auto-generating on version change

**When to use:** SAFE-02/03 — version dropdown change event

**Example:**
```javascript
// Current code (line 15919):
versionSelect.addEventListener("change", function () {
  saveVersion(parseInt(this.value, 10));
  const url = urlInput.value.trim();
  if (isValidURL(url) && !generateBtn.disabled) {
    generateQR(); // Always regenerates - UNSAFE when pattern exists
  }
});

// New guarded version:
versionSelect.addEventListener("change", function () {
  const newVersion = parseInt(this.value, 10);
  saveVersion(newVersion);
  
  const url = urlInput.value.trim();
  if (!isValidURL(url) || generateBtn.disabled) return;
  
  // SAFE-02/03: If pattern exists, don't auto-generate
  if (hasPattern()) {
    // User must click Generate button explicitly
    // Optionally: show visual indicator that version changed but QR not regenerated
    return;
  }
  
  generateQR();
});
```

### Pattern 4: Version Change Confirmation Dialog

**What:** Warn user when changing versions would destroy their pattern

**When to use:** SAFE-05 — version change (dropdown or URL-forced) when pattern exists

**Example:**
```javascript
// In version change handler:
versionSelect.addEventListener("change", function () {
  const newVersion = parseInt(this.value, 10);
  const previousVersion = currentVersion; // Track current version globally
  
  if (hasPattern() && newVersion !== previousVersion) {
    const confirmed = confirm(
      "Changing QR version will clear your painted pattern. Continue?"
    );
    
    if (!confirmed) {
      // Revert dropdown to previous version
      this.value = previousVersion;
      return;
    }
    
    // User confirmed - clear pattern and proceed
    paintGrid.clear();
    savePattern(newVersion, paintGrid.cells);
  }
  
  saveVersion(newVersion);
  
  // ... rest of handler
});
```

### Pattern 5: Same-Version Pattern Preservation

**What:** When regenerating QR at same version, preserve painted pixels

**When to use:** SAFE-04 — URL change or explicit Generate button click at same version

**Analysis:** The current `initPaintCanvas()` function already does this:
```javascript
function initPaintCanvas(version) {
  // ...
  paintGrid = new PixelGrid(gridSize);
  
  // Try to restore saved pattern for this version
  const savedCells = loadPattern(version);
  if (savedCells) {
    paintGrid.cells = savedCells;  // <-- Pattern preserved!
  }
  // ...
}
```

The pattern is saved to localStorage per-version by `savePattern(currentVersion, paintGrid.cells)`. When regenerating at the same version, `loadPattern()` returns the existing pattern. **No new code needed for SAFE-04** — it already works.

### Pattern 6: Error Message for Hash Fragment

**What:** Show clear error message when URL contains hash fragment

**When to use:** SAFE-01 — user feedback

**Example:**
```javascript
// Option A: Extend validation UI to show specific error messages
function updateValidationUI(url) {
  const input = document.getElementById("url-input");
  const icon = document.getElementById("validation-icon");
  const button = document.getElementById("generate-btn");
  
  if (!url) {
    // ... empty state
    return;
  }
  
  // Check for hash fragment first (before other validation)
  if (hasHashFragment(url)) {
    input.classList.remove("valid");
    input.classList.add("invalid");
    icon.classList.remove("valid");
    icon.classList.add("invalid");
    button.disabled = true;
    
    // Show error message (could add error message div to UI)
    showError("URLs with hash fragments (#section) are not supported");
    return;
  }
  
  // ... rest of validation
}

// Option B: Simple alert/tooltip on input
function showError(message) {
  // Could be: tooltip, inline error div, or alert
  const errorDiv = document.getElementById("url-error");
  if (errorDiv) {
    errorDiv.textContent = message;
    errorDiv.style.display = "block";
  }
}
```

### Anti-Patterns to Avoid

- **Silently clearing pattern on version change:** User loses work without warning — always confirm first
- **Checking pattern existence in multiple places inconsistently:** Extract to `hasPattern()` helper
- **Using URL string parsing instead of URL API:** The `URL.hash` property handles edge cases correctly
- **Auto-generating when pattern exists:** Defeats the purpose of persistence — require explicit action

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| URL hash detection | Regex for `#` character | `new URL(str).hash` | Handles URL encoding, edge cases |
| Confirmation dialogs | Custom modal | `window.confirm()` | Simpler, synchronous, familiar UX |
| Pattern dirty state | Custom change tracking | Check `paintGrid.cells` directly | Already have the data, no need to duplicate |

**Key insight:** This phase is mostly about connecting existing pieces with guard logic. The codebase already has pattern storage, version tracking, and validation UI. New code is minimal — mostly conditionals and one confirm dialog.

## Common Pitfalls

### Pitfall 1: URL Encoding Edge Cases

**What goes wrong:** URLs like `example.com/page%23section` (encoded hash) slip through simple string checks

**Why it happens:** The `#` character can be URL-encoded as `%23`

**How to avoid:** Use `new URL(str).hash` which normalizes and decodes the URL first

**Warning signs:** URLs with `%23` in the path being incorrectly rejected or accepted

### Pitfall 2: Confirm Dialog Blocking

**What goes wrong:** `confirm()` blocks JavaScript execution, which could cause issues with async operations

**Why it happens:** `confirm()` is synchronous

**How to avoid:** Call confirm() before starting any async operations, not during them. In this case, version change handler is synchronous anyway.

**Warning signs:** UI freezes or async state corruption

### Pitfall 3: Version vs Pattern Grid Size Mismatch

**What goes wrong:** Saved pattern for version N is loaded into version M grid (different sizes)

**Why it happens:** Pattern is stored per-version in localStorage, but size validation could fail

**How to avoid:** `loadPattern()` already validates grid size matches:
```javascript
// Existing code (line 13875):
if (stored.cells.length !== getModuleSize(version) ** 2) {
  return null; // Size mismatch - return null (will create fresh grid)
}
```
This is already handled correctly.

### Pitfall 4: Race Condition on Rapid Version Changes

**What goes wrong:** User rapidly changes version, confirm dialogs stack or state gets inconsistent

**Why it happens:** Multiple change events fire in quick succession

**How to avoid:** Track pending confirmation state, ignore new changes while dialog is open. Or: confirm() is synchronous and blocks, so this may not be an issue in practice.

**Warning signs:** Multiple dialogs appearing, incorrect version selected after confirmation

### Pitfall 5: URL-Forced Version Change Not Triggering Warning

**What goes wrong:** User changes URL to longer text, requiring higher version — pattern is lost without warning

**Why it happens:** `updateVersionDropdown()` auto-bumps version, but this happens in `updateValidationUI()`, not version change event

**How to avoid:** Add pattern check in URL input handler when version would change:
```javascript
// In URL input handler or updateVersionDropdown:
if (hasPattern() && newMinVersion > currentVersion) {
  // This would force a version change - warn user
  const confirmed = confirm(
    "This URL requires a larger QR version and will clear your pattern. Continue?"
  );
  if (!confirmed) {
    // Revert URL to previous value
    urlInput.value = previousURL;
    return;
  }
}
```

**Warning signs:** Pattern disappears when typing longer URLs

## Code Examples

### Complete Hash Fragment Rejection

```javascript
// Source: MDN URL API + existing isValidURL pattern
/**
 * Check if URL contains a hash fragment
 * @param {string} urlString - URL to check
 * @returns {boolean} True if URL has hash fragment
 */
function hasHashFragment(urlString) {
  try {
    const url = new URL(urlString);
    return url.hash.length > 0;
  } catch (err) {
    return false;
  }
}

/**
 * Validate URL - extended with hash fragment rejection (SAFE-01)
 */
function isValidURL(string) {
  try {
    const url = new URL(string);
    
    // Only allow http/https
    if (url.protocol !== "http:" && url.protocol !== "https:") return false;
    
    // SAFE-01: Reject URLs with hash fragments
    if (url.hash.length > 0) return false;
    
    // Require valid hostname with at least one dot
    const host = url.hostname;
    const dot = host.indexOf(".");
    return dot > 0 && dot < host.length - 1;
  } catch (err) {
    return false;
  }
}
```

### Complete Generation Guards

```javascript
// Source: Adapted from existing codebase patterns

/**
 * Check if any pixels are painted
 * @returns {boolean} True if pattern exists
 */
function hasPattern() {
  if (!paintGrid) return false;
  
  for (let i = 0; i < paintGrid.cells.length; i++) {
    if (paintGrid.cells[i] !== 0) {
      return true;
    }
  }
  return false;
}

// Updated version change handler (SAFE-02, SAFE-03, SAFE-05)
versionSelect.addEventListener("change", function () {
  const newVersion = parseInt(this.value, 10);
  const url = urlInput.value.trim();
  
  // SAFE-05: Warn before version change when pattern exists
  if (hasPattern() && newVersion !== currentVersion) {
    const confirmed = confirm(
      "Changing QR version will clear your painted pattern. Continue?"
    );
    
    if (!confirmed) {
      // Revert dropdown to current version
      this.value = currentVersion;
      return;
    }
    
    // User confirmed - pattern will be cleared by initPaintCanvas
  }
  
  saveVersion(newVersion);
  
  // SAFE-02/03: If pattern exists and version unchanged, don't auto-generate
  // User must click Generate button explicitly
  if (hasPattern() && newVersion === currentVersion) {
    // Show visual indicator that manual generation is needed?
    return;
  }
  
  if (isValidURL(url) && !generateBtn.disabled) {
    generateQR();
  }
});
```

### Error Message UI (Optional Enhancement)

```html
<!-- Add after url-input wrapper -->
<div id="url-error" class="error-message" style="display: none;"></div>
```

```css
.error-message {
  color: #ef4444;
  font-size: 0.875rem;
  margin-top: 4px;
  padding: 4px 8px;
}
```

```javascript
function showURLError(message) {
  const errorDiv = document.getElementById("url-error");
  if (errorDiv) {
    errorDiv.textContent = message;
    errorDiv.style.display = message ? "block" : "none";
  }
}

// In updateValidationUI:
if (hasHashFragment(url)) {
  // ... mark as invalid ...
  showURLError("URLs with # fragments are not supported. The optimizer adds its own hash suffixes.");
  return;
} else {
  showURLError(null); // Clear error
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Auto-regenerate on any change | Guard with pattern checks | Phase 8 | Prevents accidental pattern loss |
| Accept any valid URL | Reject hash fragments | Phase 8 | Prevents optimizer conflicts |
| Silent version changes | Confirmation dialog | Phase 8 | User awareness of destructive actions |

**Deprecated/outdated:**
- Nothing deprecated in this phase — these are additive safety features

## Open Questions

1. **Should URL-forced version changes also trigger confirmation?**
   - What we know: Version dropdown changes trigger confirmation. URL changes that require a version bump currently don't.
   - What's unclear: Should typing a longer URL also warn about pattern loss?
   - Recommendation: Yes, implement warning in `updateVersionDropdown()` when minVersion > currentVersion and pattern exists. This is SAFE-05's "URL-forced version change" case.

2. **Visual indicator when generation is blocked?**
   - What we know: When pattern exists and version changes, we skip auto-generation
   - What's unclear: Should we show something to indicate the QR preview is stale?
   - Recommendation: Optional enhancement — could add "Regenerate needed" badge or glow on Generate button. Not strictly required by success criteria.

3. **What about hash fragments in result URLs from optimizer?**
   - What we know: Optimizer adds `#xxxx` hash suffixes to find scannable variants
   - What's unclear: User might copy a result URL and re-paste it — would be rejected
   - Recommendation: This is an edge case. Could strip hash before validation, or document this behavior. Not blocking for Phase 8.

## Sources

### Primary (HIGH confidence)

- MDN URL API documentation - `URL.hash` property behavior
- MDN Window.confirm() - synchronous confirmation dialog
- Existing codebase:
  - `isValidURL()` (line 13902) - current URL validation
  - `initPaintCanvas()` (line 13996) - pattern restoration logic
  - `loadPattern()` (line 13858) - localStorage pattern loading with size validation
  - `savePattern()` (line 13807) - localStorage pattern saving
  - Version change handler (line 15919) - current auto-generate behavior
  - `checkPaintState()` (line 15337) - pattern existence check pattern

### Secondary (MEDIUM confidence)

- Phase 7 RESEARCH.md - pattern storage architecture decisions
- Phase 6 implementation - localStorage persistence patterns

### Tertiary (LOW confidence)

- None — all patterns are well-established browser APIs and existing codebase patterns

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - Native browser APIs only, no external dependencies
- Architecture: HIGH - Straightforward guard logic, clear patterns from existing code
- Pitfalls: HIGH - Common browser behaviors, well-documented edge cases

**Research date:** 2026-02-13
**Valid until:** 2026-03-13 (30 days - stable browser APIs)
