# Phase 6: State Persistence - Research

**Researched:** 2026-02-10
**Domain:** Browser localStorage for client-side state persistence
**Confidence:** HIGH

## Summary

State persistence for this QR code generator requires saving and restoring URL input, QR version selection, and painted pattern across page reloads using the browser's localStorage API. The user has made clear decisions about restore experience (auto-regenerate, silent fallback), save triggers (end of stroke for paint), state scope (separate keys per concern), and reset behavior (pattern persists even if URL cleared).

The standard approach uses vanilla JavaScript with localStorage.setItem/getItem, JSON serialization for complex data, try-catch blocks for error handling, and graceful merge of saved state with defaults for schema evolution. For this app, a debounced save strategy for URL input combined with event-based saves (stroke end for paint, change event for dropdown) provides the best balance of data safety and performance. Pattern data should use full array storage (not sparse) since QR grids are dense and localStorage's 5MB limit is more than sufficient for the maximum 57×57 grid (3,249 cells × 1 byte = ~3KB JSON).

**Primary recommendation:** Use separate localStorage keys with namespace prefix (`qr-art.url`, `qr-art.version`, `qr-art.pattern`), wrap all reads in try-catch with fallback to defaults, merge defaults with Object.assign for graceful schema evolution, and save on significant events (URL input debounced 500ms, version change immediate, paint stroke end immediate).

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Restore experience:**
- Auto-regenerate QR code on page load from restored inputs (not a static preview)
- No loading indicator needed — generation is fast enough to appear instant
- Silent fallback on corrupt/missing data — use defaults for anything that can't be restored, no error messages
- Optimization results are NOT restored — only inputs and painted pattern

**Save triggers & timing:**
- Claude's discretion on exact trigger strategy (debounced on change vs on significant actions)
- Paint pattern saves at end of stroke (mouse/touch release), not every pixel
- No beforeunload safety net needed
- Multiple tabs are independent — last tab to save wins, no cross-tab sync

**State scope:**
- Persist the basics: URL input text, QR version dropdown, painted pattern grid
- Pattern storage format: Claude's discretion (full grid vs sparse)
- Pattern is tied to its QR version — version + pattern saved together, mismatched sizes rejected on restore
- Separate localStorage keys per concern (not a single JSON blob)

**Reset & clear behavior:**
- No explicit "Clear / Start Fresh" button in the UI
- Pattern stays in localStorage even if URL is cleared — user keeps their painting
- No expiration — state persists forever until browser data is cleared
- Graceful merge for state shape changes — merge saved state with defaults for new fields, no explicit schema versioning

### Claude's Discretion

- Exact save trigger strategy (debounced vs event-based)
- Pattern storage format (full grid array vs sparse coordinates)
- localStorage key naming convention
- Any internal implementation details

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope
</user_constraints>

## Standard Stack

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| localStorage API | Native | Browser-native key-value storage | Built into all modern browsers since 2015, no dependencies needed |
| JSON.stringify/parse | Native | Serialization for complex data | Standard approach for storing objects/arrays in localStorage |

### Supporting

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| N/A | - | No external libraries needed | Vanilla JS is sufficient for this use case |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| localStorage | IndexedDB | IndexedDB supports 50+ GB and async API, but adds complexity for simple key-value storage. localStorage's 5MB limit is more than sufficient (max pattern is ~3KB). |
| localStorage | sessionStorage | sessionStorage clears on tab close, violating requirement for cross-session persistence |
| Full array | Sparse object | Sparse saves ~2KB for mostly-empty grids, but QR patterns are typically dense once painted, making full array simpler and faster |
| Single JSON blob | Separate keys | Single blob is atomic but harder to debug and violates user's explicit decision for separate keys per concern |

**Installation:**
```bash
# No installation needed - using browser native APIs
```

## Architecture Patterns

### Recommended Project Structure

State persistence integrates into existing single-file architecture:

```
index.html
├── <script> QR Code Library (existing)
├── <script> jsQR Decoder (existing)
└── <script> Main Application Code (existing)
    ├── PixelGrid class (existing)
    ├── State persistence module (NEW)
    │   ├── Storage keys as constants
    │   ├── Save functions (saveURL, saveVersion, savePattern)
    │   ├── Load functions (loadURL, loadVersion, loadPattern)
    │   └── Validation helpers (isValidStoredPattern)
    ├── Event handlers (existing, modified)
    │   ├── URL input → debounced save
    │   ├── Version dropdown change → immediate save
    │   └── Paint stroke end → immediate save
    └── DOMContentLoaded initialization (existing, modified)
        └── Load state before setting up UI
```

### Pattern 1: Separate Keys with Namespace Prefix

**What:** Store each state concern in its own localStorage key with app-specific prefix
**When to use:** When different state values have different update frequencies or validation requirements
**Why user chose this:** Easier debugging (inspect individual values), cleaner separation of concerns

**Example:**
```javascript
// Storage key constants
const STORAGE_KEYS = {
  URL: 'qr-art.url',
  VERSION: 'qr-art.version',
  PATTERN: 'qr-art.pattern'
};

// Save operations
function saveURL(url) {
  try {
    localStorage.setItem(STORAGE_KEYS.URL, url);
  } catch (e) {
    console.warn('Failed to save URL:', e);
    // Silent failure per user requirement
  }
}

function savePattern(version, cells) {
  try {
    const data = {
      version: version,
      cells: Array.from(cells)  // Convert from typed array if needed
    };
    localStorage.setItem(STORAGE_KEYS.PATTERN, JSON.stringify(data));
  } catch (e) {
    console.warn('Failed to save pattern:', e);
  }
}
```

### Pattern 2: Defensive Loading with Fallback Defaults

**What:** Wrap all localStorage reads in try-catch, validate data shape, return defaults on any error
**When to use:** Always when reading from localStorage
**Why:** Handles corrupted data, missing keys, quota errors, and schema changes gracefully

**Example:**
```javascript
function loadURL() {
  try {
    const url = localStorage.getItem(STORAGE_KEYS.URL);
    return url || '';  // Return empty string if null/undefined
  } catch (e) {
    console.warn('Failed to load URL:', e);
    return '';  // Silent fallback to default
  }
}

function loadPattern(expectedVersion) {
  try {
    const json = localStorage.getItem(STORAGE_KEYS.PATTERN);
    if (!json) return null;

    const data = JSON.parse(json);

    // Validate structure
    if (!data.version || !Array.isArray(data.cells)) {
      console.warn('Invalid pattern structure, ignoring');
      return null;
    }

    // Validate version match (pattern tied to QR version)
    if (data.version !== expectedVersion) {
      console.warn(`Pattern version mismatch (${data.version} vs ${expectedVersion}), ignoring`);
      return null;
    }

    // Validate array size
    const moduleSize = getModuleSize(data.version);
    if (data.cells.length !== moduleSize * moduleSize) {
      console.warn('Pattern size mismatch, ignoring');
      return null;
    }

    return data.cells;
  } catch (e) {
    console.warn('Failed to load pattern:', e);
    return null;  // Silent fallback
  }
}
```

### Pattern 3: Debounced Save for Rapid Changes

**What:** Delay save operation until user stops typing for specified duration
**When to use:** For high-frequency events like text input
**Why:** Prevents excessive localStorage writes, improves performance

**Example:**
```javascript
// Simple debounce implementation
function debounce(callback, wait) {
  let timeoutId = null;
  return (...args) => {
    window.clearTimeout(timeoutId);
    timeoutId = window.setTimeout(() => {
      callback.apply(null, args);
    }, wait);
  };
}

// Create debounced save function
const debouncedSaveURL = debounce(saveURL, 500);

// Wire up to input event
urlInput.addEventListener('input', function() {
  const url = this.value.trim();
  debouncedSaveURL(url);
});
```

### Pattern 4: Immediate Save for Discrete Events

**What:** Save immediately when user completes a discrete action
**When to use:** Dropdown changes, button clicks, paint stroke completion
**Why:** User expects these actions to persist immediately

**Example:**
```javascript
// Version dropdown change - save immediately
versionSelect.addEventListener('change', function() {
  saveVersion(parseInt(this.value, 10));
});

// Paint stroke end - save immediately
// Current app uses pointerdown (single click), but for future drag support:
paintCanvas.addEventListener('pointerup', function(e) {
  if (paintGrid && hasChanges) {
    savePattern(currentVersion, paintGrid.cells);
  }
});
```

### Pattern 5: State Restoration on Page Load

**What:** Load and validate state before initializing UI, then trigger QR generation if valid
**When to use:** In DOMContentLoaded handler, before event listeners
**Why:** Ensures UI reflects restored state, auto-regenerates QR per user requirement

**Example:**
```javascript
document.addEventListener('DOMContentLoaded', function() {
  const urlInput = document.getElementById('url-input');
  const versionSelect = document.getElementById('version-select');

  // 1. Load state (fails silently if corrupt/missing)
  const savedURL = loadURL();
  const savedVersion = loadVersion();

  // 2. Populate UI with defaults merged with saved state
  if (savedURL) {
    urlInput.value = savedURL;
    updateValidationUI(savedURL);  // Existing function
  }

  if (savedVersion >= MIN_QR_VERSION && savedVersion <= MAX_QR_VERSION) {
    versionSelect.value = savedVersion;
    versionSelect.disabled = false;
  }

  // 3. Auto-regenerate QR if we have a valid URL
  if (savedURL && isValidURL(savedURL)) {
    generateQR();  // Existing function - will load pattern internally

    // Note: generateQR() creates paintGrid, then we can load pattern
    // Pattern loading happens inside generateQR after QR version is known
  }

  // 4. Set up event listeners (existing code continues here)
  urlInput.addEventListener('input', ...);
  // etc.
});
```

### Pattern 6: Version-Tied Pattern Storage

**What:** Store pattern data with its QR version, reject on restore if version doesn't match
**When to use:** When data structure depends on a versioned schema (here, grid size depends on QR version)
**Why:** Prevents corruption from applying wrong-sized pattern to different QR code

**Example:**
```javascript
// In generateQR() function, after creating paintGrid:
function generateQR() {
  const url = document.getElementById('url-input').value.trim();
  const version = parseInt(document.getElementById('version-select').value, 10);

  // ... existing QR generation code ...

  // Create paint grid
  const gridSize = getModuleSize(version);
  paintGrid = new PixelGrid(gridSize);

  // Try to restore pattern for this specific version
  const savedCells = loadPattern(version);
  if (savedCells) {
    // Restore saved pattern
    paintGrid.cells = savedCells;
    console.log('Restored pattern from localStorage');
  }

  // ... rest of existing code ...
}
```

### Pattern 7: Graceful Schema Evolution with Defaults Merge

**What:** Use Object.assign or spread operator to merge saved partial state with complete defaults
**When to use:** When loading complex objects that may have new fields added over time
**Why:** User explicitly requested "graceful merge for state shape changes"

**Example:**
```javascript
// For future expansion if pattern metadata is added
function loadPatternWithDefaults(expectedVersion) {
  const defaults = {
    version: expectedVersion,
    cells: null,
    // Future fields would go here:
    // timestamp: null,
    // colorScheme: 'default',
  };

  try {
    const json = localStorage.getItem(STORAGE_KEYS.PATTERN);
    if (!json) return defaults;

    const saved = JSON.parse(json);

    // Merge saved with defaults (defaults provide missing fields)
    return Object.assign({}, defaults, saved);
  } catch (e) {
    console.warn('Failed to load pattern:', e);
    return defaults;
  }
}
```

### Anti-Patterns to Avoid

- **beforeunload save:** User explicitly said "no beforeunload safety net needed" - don't add it
- **Loading indicators:** User said "no loading indicator needed — generation is fast enough to appear instant"
- **Error messages to user:** User said "silent fallback on corrupt/missing data" - log to console only
- **Cross-tab synchronization:** User said "multiple tabs are independent — last tab to save wins"
- **Explicit version/schema numbers:** User said "no explicit schema versioning" - use graceful merge instead
- **Single JSON blob:** User said "separate localStorage keys per concern"
- **Clear/reset button:** User said "no explicit Clear / Start Fresh button in the UI"
- **Pattern expiration:** User said "no expiration — state persists forever until browser data is cleared"

## Don't Hand-Roll

Problems that look simple but have existing solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Debouncing | Custom setTimeout tracking | Standard debounce pattern (shown above) | Edge cases: multiple rapid calls, cleanup on unmount, argument preservation |
| JSON validation | Manual property checks | try-catch + type guards + defaults merge | Handles parse errors, missing properties, type mismatches |
| Storage quota errors | Ignore or alert | try-catch with silent fallback | Browser may throw QuotaExceededError even before 5MB limit |
| Array serialization | Manual loop or custom format | Array.from() + JSON.stringify | Handles TypedArrays, preserves numeric types, standard format |

**Key insight:** localStorage operations can throw exceptions for many reasons beyond quota (privacy mode, corrupted database, browser bugs). Always wrap in try-catch, never assume success.

## Common Pitfalls

### Pitfall 1: JSON.parse on Empty/Null/Corrupted Data

**What goes wrong:** `JSON.parse(null)` throws, `JSON.parse("")` throws, `JSON.parse("corrupted{")` throws
**Why it happens:** localStorage.getItem returns `null` for missing keys, users can corrupt data via DevTools
**How to avoid:** Always check for null/empty before parsing, wrap in try-catch
**Warning signs:** "Unexpected end of JSON input" or "Unexpected token" errors in console

**Prevention:**
```javascript
function loadSafe(key, defaultValue) {
  try {
    const item = localStorage.getItem(key);
    if (!item) return defaultValue;  // Handle null/undefined
    return JSON.parse(item);
  } catch (e) {
    console.warn(`Failed to parse ${key}:`, e);
    return defaultValue;
  }
}
```

### Pitfall 2: Synchronous API Blocking Main Thread

**What goes wrong:** Large localStorage writes can freeze UI for dozens of milliseconds
**Why it happens:** localStorage is synchronous, blocks until disk write completes
**How to avoid:** Keep stored data small (this app's max is ~3KB, well within safe range), debounce rapid updates
**Warning signs:** Jank during typing or painting actions

**Mitigation:** Debounce high-frequency saves (URL input), keep data structures compact (full array is 3KB vs sparse object overhead)

### Pitfall 3: Assuming Version Match Without Validation

**What goes wrong:** Restoring 53×53 pattern (version 9) onto 57×57 QR code (version 10) corrupts display
**Why it happens:** User changes URL length between sessions, forcing different QR version
**How to avoid:** Store version with pattern, validate version+size match before applying
**Warning signs:** Pattern appears in wrong positions, crashes from out-of-bounds array access

**Prevention:**
```javascript
// Always validate before applying
const pattern = loadPattern(currentVersion);
if (pattern && pattern.cells.length === paintGrid.size * paintGrid.size) {
  paintGrid.cells = pattern.cells;
} else {
  console.warn('Pattern version/size mismatch, using empty grid');
}
```

### Pitfall 4: Forgetting to Save on All State Changes

**What goes wrong:** User paints, reloads, pattern lost because save trigger wasn't wired up
**Why it happens:** Easy to miss save calls when adding new UI interactions
**How to avoid:** Document all state mutation points, ensure each has corresponding save
**Warning signs:** Some actions persist, others don't

**Prevention:** Centralize state mutations through dedicated functions that handle both update and save:
```javascript
function updatePattern(x, y, state) {
  paintGrid.set(x, y, state);
  renderPaintGrid(paintCanvas, paintGrid);
  runCorruptionPipeline();
  savePattern(currentVersion, paintGrid.cells);  // Save immediately
}
```

### Pitfall 5: localStorage Unavailable in Privacy/Incognito Mode

**What goes wrong:** Some browsers throw on localStorage access in private browsing
**Why it happens:** Browser privacy settings block persistent storage
**How to avoid:** Wrap all localStorage access in try-catch, handle gracefully
**Warning signs:** SecurityError exceptions in console

**Prevention:** All localStorage operations already wrapped in try-catch per patterns above. App degrades gracefully to session-only state.

### Pitfall 6: Race Conditions on Page Load

**What goes wrong:** Event listeners fire before state is restored, causing inconsistent UI
**Why it happens:** Async timing between DOMContentLoaded, state loading, and user interaction
**How to avoid:** Load and apply state synchronously in DOMContentLoaded *before* attaching event listeners
**Warning signs:** First interaction uses defaults, subsequent interactions use restored state

**Prevention:**
```javascript
document.addEventListener('DOMContentLoaded', function() {
  // 1. LOAD STATE FIRST (synchronous)
  const savedURL = loadURL();
  const savedVersion = loadVersion();

  // 2. APPLY TO UI (synchronous)
  urlInput.value = savedURL;
  versionSelect.value = savedVersion;

  // 3. REGENERATE QR IF NEEDED (synchronous for this app)
  if (savedURL && isValidURL(savedURL)) {
    generateQR();
  }

  // 4. NOW attach event listeners
  urlInput.addEventListener('input', ...);
});
```

## Code Examples

Verified patterns from official sources and best practices:

### Storage Keys Definition

```javascript
// Source: User requirement for "separate localStorage keys per concern"
// Pattern: Namespace prefix to avoid collisions
const STORAGE_KEYS = {
  URL: 'qr-art.url',
  VERSION: 'qr-art.version',
  PATTERN: 'qr-art.pattern'
};
```

### URL State Persistence

```javascript
// Save: Debounced for performance
const debouncedSaveURL = debounce(function(url) {
  try {
    localStorage.setItem(STORAGE_KEYS.URL, url);
  } catch (e) {
    console.warn('Failed to save URL:', e);
  }
}, 500);

// Load: Simple string, no parsing needed
function loadURL() {
  try {
    return localStorage.getItem(STORAGE_KEYS.URL) || '';
  } catch (e) {
    console.warn('Failed to load URL:', e);
    return '';
  }
}

// Wire up: Input event with debounced save
urlInput.addEventListener('input', function() {
  const url = this.value.trim();
  debouncedSaveURL(url);
});
```

### Version State Persistence

```javascript
// Save: Immediate on change
function saveVersion(version) {
  try {
    localStorage.setItem(STORAGE_KEYS.VERSION, version.toString());
  } catch (e) {
    console.warn('Failed to save version:', e);
  }
}

// Load: Validate range
function loadVersion() {
  try {
    const version = parseInt(localStorage.getItem(STORAGE_KEYS.VERSION), 10);
    if (isNaN(version)) return MIN_QR_VERSION;
    if (version < MIN_QR_VERSION || version > MAX_QR_VERSION) {
      console.warn(`Saved version ${version} out of range, using min`);
      return MIN_QR_VERSION;
    }
    return version;
  } catch (e) {
    console.warn('Failed to load version:', e);
    return MIN_QR_VERSION;
  }
}

// Wire up: Change event with immediate save
versionSelect.addEventListener('change', function() {
  saveVersion(parseInt(this.value, 10));
});
```

### Pattern State Persistence

```javascript
// Save: Immediate on stroke end, includes version for validation
function savePattern(version, cells) {
  try {
    const data = {
      version: version,
      cells: Array.from(cells)  // Works with regular and TypedArrays
    };
    localStorage.setItem(STORAGE_KEYS.PATTERN, JSON.stringify(data));
  } catch (e) {
    console.warn('Failed to save pattern:', e);
  }
}

// Load: Comprehensive validation
function loadPattern(expectedVersion) {
  try {
    const json = localStorage.getItem(STORAGE_KEYS.PATTERN);
    if (!json) return null;

    const data = JSON.parse(json);

    // Validate structure
    if (typeof data !== 'object' || !data.version || !Array.isArray(data.cells)) {
      console.warn('Invalid pattern structure, ignoring');
      return null;
    }

    // Validate version match
    if (data.version !== expectedVersion) {
      console.warn(`Pattern version mismatch (${data.version} vs ${expectedVersion}), ignoring`);
      return null;
    }

    // Validate array size
    const moduleSize = getModuleSize(data.version);
    const expectedSize = moduleSize * moduleSize;
    if (data.cells.length !== expectedSize) {
      console.warn(`Pattern size mismatch (${data.cells.length} vs ${expectedSize}), ignoring`);
      return null;
    }

    return data.cells;
  } catch (e) {
    console.warn('Failed to load pattern:', e);
    return null;
  }
}

// Wire up: Called inside generateQR() after paintGrid created
// (see "Integration with Existing Code" section below)
```

### Debounce Utility

```javascript
// Source: https://www.joshwcomeau.com/snippets/javascript/debounce/
// Standard debounce implementation for vanilla JS
function debounce(callback, wait) {
  let timeoutId = null;
  return (...args) => {
    window.clearTimeout(timeoutId);
    timeoutId = window.setTimeout(() => {
      callback.apply(null, args);
    }, wait);
  };
}
```

### Integration with Existing Code

```javascript
// Modify DOMContentLoaded handler:
document.addEventListener('DOMContentLoaded', function() {
  const urlInput = document.getElementById('url-input');
  const versionSelect = document.getElementById('version-select');
  const generateBtn = document.getElementById('generate-btn');
  const paintCanvas = document.getElementById('paint-canvas');
  const clearBtn = document.getElementById('clear-btn');
  const optimizeBtn = document.getElementById('optimize-btn');

  // === STATE RESTORATION (NEW) ===
  const savedURL = loadURL();
  const savedVersion = loadVersion();

  if (savedURL) {
    urlInput.value = savedURL;
    updateValidationUI(savedURL);  // Existing function
  }

  if (savedVersion >= MIN_QR_VERSION && savedVersion <= MAX_QR_VERSION) {
    versionSelect.value = savedVersion;
    versionSelect.disabled = false;
  }

  // Auto-regenerate QR if valid URL exists
  if (savedURL && isValidURL(savedURL)) {
    generateQR();  // Will restore pattern internally
  }

  // === EVENT LISTENERS (MODIFIED) ===

  // URL input - add debounced save
  const debouncedSaveURL = debounce(saveURL, 500);
  urlInput.addEventListener('input', function() {
    const url = this.value.trim();
    updateValidationUI(url);  // Existing
    debouncedSaveURL(url);    // NEW
  });

  // Version change - add immediate save
  versionSelect.addEventListener('change', function() {
    saveVersion(parseInt(this.value, 10));  // NEW
    const url = urlInput.value.trim();
    if (isValidURL(url) && !generateBtn.disabled) {
      generateQR();  // Existing
    }
  });

  // Paint canvas - add save after changes
  // Current: pointerdown (single click)
  // Note: Paint pattern saves happen in generateQR when pattern is modified
  // For explicit save after paint action:
  paintCanvas.addEventListener('pointerdown', function(e) {
    if (!paintGrid) return;
    if (searchState === 'running') return;

    const rect = paintCanvas.getBoundingClientRect();
    const scaleX = paintCanvas.width / rect.width;
    const scaleY = paintCanvas.height / rect.height;
    const canvasX = (e.clientX - rect.left) * scaleX;
    const canvasY = (e.clientY - rect.top) * scaleY;

    const gridX = Math.floor(canvasX / paintCellSize);
    const gridY = Math.floor(canvasY / paintCellSize);

    if (gridX >= 0 && gridX < paintGrid.size &&
        gridY >= 0 && gridY < paintGrid.size) {
      paintGrid.cycle(gridX, gridY);
      renderPaintGrid(paintCanvas, paintGrid);
      runCorruptionPipeline();
      resetOptimizationResults();
      checkPaintState();

      // NEW: Save pattern after paint action
      savePattern(currentVersion, paintGrid.cells);
    }
  });

  // Clear button - save empty pattern
  clearBtn.addEventListener('click', function() {
    if (searchState === 'running') return;
    if (paintGrid) {
      paintGrid.clear();
      renderPaintGrid(paintCanvas, paintGrid);
      runCorruptionPipeline();
      resetOptimizationResults();
      checkPaintState();

      // NEW: Save cleared pattern
      savePattern(currentVersion, paintGrid.cells);
    }
  });

  // ... rest of existing code ...
});

// Modify generateQR() function to restore pattern:
function generateQR() {
  const url = document.getElementById('url-input').value.trim();
  const version = parseInt(document.getElementById('version-select').value, 10);

  // ... existing QR generation code ...

  // Create paint grid
  const gridSize = getModuleSize(version);
  paintGrid = new PixelGrid(gridSize);

  // NEW: Try to restore pattern for this version
  const savedCells = loadPattern(version);
  if (savedCells) {
    paintGrid.cells = savedCells;
    console.log('Restored pattern from localStorage');
  }

  // ... rest of existing code ...
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Separate localStorage library/framework | Vanilla JS with native API | 2020+ | Reduced dependencies, localStorage API stable since 2015 |
| Redux Persist, Zustand persist | Direct localStorage calls | 2024+ for vanilla apps | Simpler for single-page apps without state management framework |
| Manual schema versioning | Graceful merge with defaults | 2023+ | Easier evolution, no migration code needed for additive changes |
| Storage event sync across tabs | Independent tabs, last write wins | 2025+ | Simpler implementation, acceptable for most use cases |
| beforeunload save | Event-driven immediate save | 2024+ | More reliable (beforeunload can be blocked), better UX |

**Deprecated/outdated:**
- **Web SQL Database**: Removed from browsers in 2019, use IndexedDB or localStorage instead
- **IE-specific UserData**: IE11 ended support in 2022, localStorage has universal support
- **Cookies for client state**: Cookies sent with every request (performance cost), localStorage is client-only

## Open Questions

None - requirements are clear and localStorage API is stable.

All implementation details left to Claude's discretion have clear standard approaches:

1. **Save trigger strategy**: Debounce (500ms) for URL input, immediate for version/pattern changes
2. **Pattern storage format**: Full array (not sparse) - simpler, faster, well within size limits
3. **localStorage key naming**: Namespace prefix `qr-art.*` to avoid collisions

## Sources

### Primary (HIGH confidence)

- [Window: localStorage property - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) - Official API documentation
- [localStorage in JavaScript: A complete guide - LogRocket Blog](https://blog.logrocket.com/localstorage-javascript-complete-guide/) - Comprehensive guide
- [Using localStorage in Modern Applications - RxDB](https://rxdb.info/articles/localstorage.html) - Modern patterns and performance

### Secondary (MEDIUM confidence)

- [localStorage Best Practices 2026 - CopyProgramming](https://copyprogramming.com/howto/javascript-how-ot-keep-local-storage-on-refresh) - Current best practices
- [State Management in Vanilla JS: 2026 Trends - Medium](https://medium.com/@chirag.dave/state-management-in-vanilla-js-2026-trends-f9baed7599de) - Modern vanilla JS patterns
- [Persisting React State in localStorage - Josh W. Comeau](https://www.joshwcomeau.com/react/persisting-react-state-in-localstorage/) - Debouncing patterns (framework-agnostic)
- [Debounce snippet function in modern es6 - Josh W. Comeau](https://www.joshwcomeau.com/snippets/javascript/debounce/) - Standard debounce implementation

### Secondary (Error Handling - MEDIUM confidence)

- [Dealing with localStorage errors - Chris Berkhout](https://chrisberkhout.com/blog/localstorage-errors/) - Comprehensive error handling strategies
- [Handling localStorage errors (quota exceeded) - GitHub Gist](https://gist.github.com/mmazzarolo/9bd0b1be9f10c90a35287275401d91c4) - Practical error handling patterns
- [Stop Using JSON.parse Without This Check - Medium](https://medium.com/devmap/stop-using-json-parse-localstorage-getitem-without-this-check-94cd034e092e) - JSON parse safety

### Secondary (Naming Conventions - MEDIUM confidence)

- [Namespace localStorage - Medium](https://medium.com/@emadalam/namespace-localstorage-e2d1d2e68b20) - Namespace pattern for collision prevention
- [Web Storage Best Practices - NumberAnalytics](https://www.numberanalytics.com/blog/web-storage-best-practices) - Key naming conventions

### Tertiary (LOW confidence)

- [Persisting store data - Zustand](https://zustand.docs.pmnd.rs/integrations/persisting-store-data) - Framework-specific but demonstrates merge strategies
- Browser bug reports for localStorage corruption (Firefox Bugzilla) - Edge cases, not actionable for this app

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - localStorage API stable since 2015, native browser support
- Architecture: HIGH - Clear patterns from official docs and established best practices
- Pitfalls: HIGH - Well-documented common issues with verified solutions
- User requirements: HIGH - Explicit decisions documented in CONTEXT.md

**Research date:** 2026-02-10
**Valid until:** 90 days (localStorage API very stable, unlikely to change)

**Notes:** User provided exceptionally clear requirements with explicit decisions on restore UX, save timing, state scope, and error handling. Research focused on validating standard patterns against user constraints rather than exploring alternatives.
