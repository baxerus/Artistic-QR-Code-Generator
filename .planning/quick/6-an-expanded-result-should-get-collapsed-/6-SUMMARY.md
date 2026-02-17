---
phase: quick
plan: 6
subsystem: UI/Results Display
tags: [bug-fix, ux, results-expansion]
dependency_graph:
  requires: []
  provides:
    - startOptimizationUI collapses expanded results
  affects:
    - index.html (startOptimizationUI function)
tech_stack:
  - JavaScript (inline)
  - DOM manipulation
key_files:
  created: []
  modified:
    - index.html
decisions: []
metrics:
  duration: "~1 minute"
  completed: "2026-02-17"
---

# Quick Task 6: Expand Result Collapse Summary

**One-liner:** Collapse any expanded result when "Run again" Overview is clicked

##

Added functionality to collapse any expanded result when the user clicks "Run again" to start a new search. This prevents users from seeing stale expanded states from previous search results.

## Changes Made

### Modified: `index.html`

Added code to `startOptimizationUI()` function (line ~15815-15838) that:
1. Finds all `.result-slot.expanded` elements in the top-results container
2. Removes the "expanded" class from each
3. Re-renders the overlay at module resolution (null size, no borders) using the same pattern as the existing result-click handling code

### Code Pattern

```javascript
const resultsContainer = document.getElementById("top-results");
const expandedSlots = resultsContainer?.querySelectorAll(".result-slot.expanded");
expandedSlots?.forEach((slot) => {
  slot.classList.remove("expanded");
  const collapsedOverlay = slot.querySelector(".result-overlay");
  if (collapsedOverlay) {
    // Re-render at module resolution
    renderErrorOverlay(collapsedOverlay, qrData, mc, painted, fm, null, false);
  }
});
```

## Verification

- [x] Click a result to expand it (click on the QR preview)
- [x] Click "Run again" 
- [x] The previously expanded result should be collapsed
- [x] Search runs and completes normally

## Deviations from Plan

None - plan executed exactly as written.

## Commit

- `bf5ed9d`: fix(quick-6): collapse expanded results when Run again is clicked

---

## Self-Check: PASSED

- [x] index.html modified with new code
- [x] Commit bf5ed9d exists in git log
- [x] Code contains result-slot.expanded selector
- [x] startOptimizationUI function updated
