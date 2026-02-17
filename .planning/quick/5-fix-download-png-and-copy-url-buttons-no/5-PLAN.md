---
phase: quick
plan: 5
type: execute
wave: 1
depends_on: []
files_modified:
  - index.html
autonomous: true
requirements: []
must_haves:
  truths:
    - "Download PNG and Copy URL buttons are disabled when search starts (Run again/Generate clicked)"
    - "Buttons are re-enabled when search completes"
  artifacts:
    - path: index.html
      contains: "result-actions button"
      action: disable on search start
  key_links:
    - from: startOptimizationUI()
      to: .result-actions button
      via: querySelectorAll + disabled = true
---

<objective>
Fix the Download PNG and Copy URL buttons so they deactivate (disable) when Run again is clicked.

Currently these buttons stay enabled during the search because they're only created with `disabled = isRunning` at creation time, but there's no code to update them when searchState changes to "running".
</objective>

<context>
@index.html

Bug location: `startOptimizationUI()` function (around line 15804)
- Line 15816: Sets `searchState = "running"`
- Line 15821: Adds `results-disabled` class to top results
- Missing: Disables `.result-actions button` elements

Reference for expected behavior: `handleSearchComplete()` function (line 16429-16433) shows how buttons are re-enabled when search completes.
</context>

<tasks>

<task type="auto">
  <name>Task 1: Disable action buttons when search starts</name>
  <files>index.html</files>
  <action>
In the `startOptimizationUI()` function, after adding `results-disabled` class to topResultsEl (around line 15821), add code to disable all result action buttons:

```javascript
// Disable all result card buttons during search
const topResultsContainer = document.getElementById("top-results");
if (topResultsContainer) {
  topResultsContainer
    .querySelectorAll(".result-actions button")
    .forEach((btn) => {
      btn.disabled = true;
    });
}
```

This mirrors the re-enable logic in `handleSearchComplete()` at lines 16429-16433.
  </action>
  <verify>
Grep for "result-actions button" in index.html - should find both the disable code in startOptimizationUI and the enable code in handleSearchComplete.
  </verify>
  <done>
Download PNG and Copy URL buttons are visually disabled (grayed out) immediately after clicking Run again or Generate, and re-enabled when search completes.
  </done>
</task>

</tasks>

<verification>
1. Load the page with a valid URL
2. Click Generate to run search
3. Wait for results to appear
4. Click "Run again" - buttons should be disabled during search
5. Wait for search to complete - buttons should be re-enabled
</verification>

<success_criteria>
Download PNG and Copy URL buttons correctly disable when search starts and enable when it completes.
</success_criteria>

<output>
After completion, create `.planning/quick/5-fix-download-png-and-copy-url-buttons-no/5-SUMMARY.md`
</output>
