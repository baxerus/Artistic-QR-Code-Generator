---
phase: quick
plan: 3
type: execute
wave: 1
depends_on: []
files_modified:
  - index.html
autonomous: true

must_haves:
  truths:
    - "Green-highlight animation plays for its full 0.6 seconds when a new result enters the top 10"
    - "Animation completes unless the result is pushed out of top 10 by a better result"
    - "Existing results in the list are not destroyed/recreated when new results arrive"
  artifacts:
    - path: "index.html"
      provides: "Incremental DOM update logic for top-results"
      contains: "Update existing slots in-place"
  key_links:
    - from: "updateTopResults function"
      to: "result-slot DOM elements"
      via: "incremental update instead of full rebuild"
      pattern: "dataset.hash"
---

<objective>
Fix the green-highlight animation to play for its full 0.6 seconds when new results are found.

Purpose: Currently, the animation is interrupted because `updateTopResults` clears and rebuilds all DOM elements every time new results arrive from workers. The animation should complete unless the result is removed from the top 10.

Output: Modified `updateTopResults` function that updates existing slots in-place instead of destroying and recreating them.
</objective>

<execution_context>
@/home/node/.config/Claude/get-shit-done/workflows/execute-plan.md
@/home/node/.config/Claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@.planning/STATE.md
@.planning/PROJECT.md
@index.html (lines 16099-16339 - updateTopResults function)
</context>

<tasks>

<task type="auto">
  <name>Task 1: Implement incremental DOM update for top-results</name>
  <files>index.html</files>
  <action>
Refactor `updateTopResults` function (starting at line 16099) to update the DOM incrementally instead of clearing and rebuilding:

**Current problem (line 16110):**
```javascript
topResultsContainer.innerHTML = "";
```
This destroys all existing slots, including any that have active animations.

**Solution - replace full rebuild with incremental update:**

1. **Map existing slots by hash:** Before any modifications, create a Map of existing slot elements keyed by their `dataset.hash`.

2. **Create/reuse slots:** For each result in `topResults`:
   - If a slot with matching hash already exists in the DOM, reuse it (update rank if position changed, update any dynamic content)
   - If no matching slot exists, create a new slot and add `green-highlight` class

3. **Remove stale slots:** After processing all results, remove any slots whose hash is no longer in the new results.

4. **Reorder slots:** Ensure slots are in correct order by appending them in order (appending existing element moves it).

**Key changes to make:**

Replace lines 16106-16128 with logic like:
```javascript
// Map existing slots by hash
const existingSlots = new Map();
for (const slot of topResultsContainer.children) {
  existingSlots.set(slot.dataset.hash, slot);
}

// Get new hashes for comparison
const newHashes = new Set(topResults.map(r => r.hash));

// Remove slots that are no longer in top results
for (const [hash, slot] of existingSlots) {
  if (!newHashes.has(hash)) {
    slot.remove();
    existingSlots.delete(hash);
  }
}

// Build or update slots in order
topResults.forEach((result, index) => {
  let slot = existingSlots.get(result.hash);
  const isNew = !slot;
  
  if (isNew) {
    // Create new slot (existing slot creation code)
    slot = document.createElement("div");
    slot.className = "result-slot";
    slot.dataset.hash = result.hash;
    slot._resultData = result;
    // Add green-highlight for new results
    slot.classList.add("green-highlight");
    setTimeout(() => slot.classList.remove("green-highlight"), 600);
    // ... rest of slot creation (canvas, labels, buttons)
  } else {
    // Update existing slot
    // Update rank number
    const rankEl = slot.querySelector('.result-rank');
    if (rankEl) rankEl.textContent = `#${index + 1}`;
    // Update _resultData for overlay rendering
    slot._resultData = result;
  }
  
  // Append in order (moves existing elements to correct position)
  topResultsContainer.appendChild(slot);
});
```

**Important:** Keep the existing slot creation code (canvas rendering, labels, buttons) inside the `if (isNew)` block. Only create DOM elements for truly new results.

**Update `previousTopHashes` handling:** The variable `previousTopHashes` is used at line 16123 to detect new results, but with incremental updates, we no longer need it for that purpose - we can detect "new" by checking if the slot doesn't exist. However, keep updating `previousTopHashes` at line 16338 for any other code that might depend on it.
  </action>
  <verify>
1. Open index.html in browser
2. Enter a URL and start optimization
3. Observe the top results section
4. When a new result appears, verify the green glow animation plays for the full 0.6 seconds
5. Verify that existing results don't "flash" or recreate when new results arrive
6. Verify results stay in correct ranked order as better results are found
  </verify>
  <done>
- Green-highlight animation plays for full 0.6 seconds on new results
- Existing result slots remain stable in the DOM during updates
- Results are correctly reordered as rankings change
- No visual flickering or DOM recreation of existing slots
  </done>
</task>

</tasks>

<verification>
1. Start optimization with a painted pattern
2. Watch the top-results section as results stream in
3. Confirm new results animate with green glow for full duration
4. Confirm existing results don't flash/recreate when list updates
5. Confirm final results are in correct order (best first)
</verification>

<success_criteria>
- Animation completes visually (0.6s duration observable)
- No flickering or recreation of stable result slots
- Correct ranking order maintained
- All existing functionality preserved (Download PNG, Copy URL, hover overlays)
</success_criteria>

<output>
After completion, create `.planning/quick/3-fix-green-highlight-animation-to-play-fu/3-SUMMARY.md`
</output>
