---
phase: quick
plan: 6
type: execute
wave: 1
depends_on: []
files_modified: [index.html]
autonomous: true
requirements: []
must_haves:
  truths:
    - "User can click Run again while a result is expanded"
    - "Expanded result collapses when Run again is clicked"
    - "New search runs with collapsed results view"
  artifacts:
    - path: "index.html"
      provides: "startOptimizationUI function updated to collapse expanded results"
      contains: "result-slot.expanded"
  key_links:
    - from: "startOptimizationUI()"
      to: ".result-slot.expanded"
      via: "querySelectorAll and classList.remove"
      pattern: "querySelectorAll.*expanded"

---

<objective>
Collapse any expanded result when "Run again" is clicked so users don't see stale expanded states from previous results.
</objective>

<context>
@index.html (lines 15804-15842: startOptimizationUI function)
</context>

<tasks>

<task type="auto">
  <name>Collapse expanded results on Run again</name>
  <files>index.html</files>
  <action>
In the `startOptimizationUI()` function (around line 15813, after `togglePaintSection(true)`), add code to collapse any expanded result slots:

1. Find all elements matching `.result-slot.expanded` in the top-results container
2. For each expanded slot:
   - Remove the "expanded" class
   - Re-render the overlay at module resolution (null size, no borders) using the same pattern as lines 16287-16308

The code should look something like:

```javascript
// Collapse any expanded result slots before starting new search
const topResultsContainer = document.getElementById("top-results");
const expandedSlots = topResultsContainer.querySelectorAll(".result-slot.expanded");
expandedSlots.forEach((slot) => {
  slot.classList.remove("expanded");
  // Re-render overlay at module resolution
  const overlay = slot.querySelector(".result-overlay");
  if (overlay) {
    const mc = parseInt(overlay.dataset.moduleCount);
    const qrData = JSON.parse(overlay.dataset.qrModuleData);
    const painted = JSON.parse(overlay.dataset.paintedPixels);
    const ver = parseInt(overlay.dataset.version);
    const fm = createFunctionPatternMask(ver);
    renderErrorOverlay(overlay, qrData, mc, painted, fm, null, false);
  }
});
```
  </action>
  <verify>
Manual test:
1. Click on a result to expand it
2. Click "Run again" button
3. Verify the expanded result collapses back to normal view
4. Run the search and verify it completes with results displayed normally
  </verify>
  <done>Expanded results are collapsed when "Run again" is clicked</done>
</task>

</tasks>

<verification>
- [ ] Click a result to expand it (click on the QR preview)
- [ ] Click "Run again" 
- [ ] The previously expanded result should be collapsed
- [ ] Search runs and completes normally
</verification>

<success_criteria>
Expanded result collapses when "Run again" is clicked
</success_criteria>

<output>
After completion, create `.planning/quick/6-an-expanded-result-should-get-collapsed-/6-SUMMARY.md`
</output>
