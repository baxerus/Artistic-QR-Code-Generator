---
phase: quick
plan: 7
type: execute
wave: 1
depends_on: []
files_modified:
  - index.html
autonomous: true
requirements: []
---

<objective>
Add an X button to collapse expanded result results
</objective>

<context>
This is a QR code painting tool. Results are displayed in a grid. When clicked, a result expands to show more detail (larger QR preview with overlay legend). Currently, clicking on the expanded result again collapses it. The user wants an explicit X (close) button that also collapses the expanded result.
</context>

<tasks>

<task type="auto">
  <name>Add X button for collapsing expanded results</name>
  <files>index.html</files>
  <action>
1. Add CSS for the close button:
   - Add `.result-close-btn` styles (position absolute top-right of slot, visible only when `.result-slot.expanded`)
   - Style: dark circle with white X, cursor pointer, hover effect

2. Add the X button element in the slot creation code (after legend creation ~line 16391):
   - Create button element with class `result-close-btn`
   - Inner HTML: "&times;" for the X symbol
   - Click handler: same collapse logic as canvas click (lines 16304-16372)
   - Only show when expanded (CSS handles this)

3. Add the close button to the slot (line ~16398):
   - `slot.appendChild(closeBtn);`
  </action>
  <verify>After implementation:
1. Click a result to expand it
2. X button should appear in top-right corner of the result slot
3. Click the X button - result should collapse
4. X button should not be visible when result is collapsed
  </verify>
  <done>
- X button visible only when result is expanded
- Clicking X collapses the result (same as clicking canvas)
- X button styled consistently with the UI
  </done>
</task>

</tasks>

<success_criteria>
- [ ] X button appears when result is expanded
- [ ] X button collapses the result when clicked
- [ ] X button hidden when result is collapsed
- [ ] Styling is consistent with the application
</success_criteria>

<output>
After completion, create `.planning/quick/7-add-x-button-to-collapse-expanded-result/7-SUMMARY.md`
</output>
