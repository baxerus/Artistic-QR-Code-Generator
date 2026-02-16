---
phase: quick-4
plan: 01
type: execute
wave: 1
depends_on: []
files_modified: [index.html]
autonomous: true

must_haves:
  truths:
    - "User can paint black/white pixels over finder patterns, timing patterns, and alignment patterns"
    - "Painted pixels in functional areas are rendered with overlay colors on paint canvas"
    - "Painted functional area pixels are counted in pixel diff calculations"
    - "Painted functional area pixels are applied when evaluating QR decode status"
  artifacts:
    - path: "index.html"
      provides: "Updated painting logic that allows painting into functional patterns"
---

<objective>
Allow painting into functional patterns (finder patterns, timing patterns, alignment patterns)

Purpose: Enable artistic control over ALL QR pixels, not just data areas. Users may want to incorporate functional patterns into their designs.

Output: Modified index.html where:
- Painting is allowed on functional pattern areas (no blocking cursor, no skipping)
- Painted pixels in functional areas show overlay color
- Pixel diff counts include functional area conflicts
- QR decode test includes painted functional area pixels
</objective>

<execution_context>
@/home/node/.config/Claude/get-shit-done/workflows/execute-plan.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/STATE.md
@index.html (lines 13260-13330, 13470-13530, 13590-13680, 13745-13815, 15250-15320)
</context>

<tasks>

<task type="auto">
  <name>Task 1: Remove painting restrictions on functional patterns</name>
  <files>index.html</files>
  <action>
Remove all code that blocks or skips painting in functional/protected areas:

1. **`paintAt()` function (~line 13490-13497)**: Remove the entire block that checks `functionMask[y * paintGrid.size + x]` and returns early. The painting should proceed regardless of whether the cell is in a functional pattern.

2. **`updateCursorForPosition()` function (~line 13315-13321)**: Remove the `if (protectedCell)` block that adds the "blocked" class. This removes the blocked cursor visual feedback. The cursor should always show paintable state.

3. **`runCorruptionPipeline()` decode test (~lines 13265, 13268)**: Change:
   - `if (!isProtected && paintState === 1)` → `if (paintState === 1)`
   - `} else if (!isProtected && paintState === 2)` → `} else if (paintState === 2)`
   This allows painted pixels in functional areas to override QR values for decode testing.

4. **`renderQRWithOverlay()` Layer 1 (~line 13774)**: Remove the line `if (mask && mask[y * gridSize + x]) continue;` so painted overlay is rendered in functional areas too.

5. **`drawPaintedBorders()` (~line 13619)**: Remove the line `if (mask && mask[y * gridSize + x]) continue;` so borders are drawn around painted pixels even in functional areas. Also update the edge checks (lines 13633, 13644, 13655, 13666) to remove the `|| (mask && mask[...])` conditions since we now want to draw borders between painted and non-painted cells regardless of protected status.
  </action>
  <verify>Open index.html in browser, select a color from legend, paint over a finder pattern corner - should accept paint and show colored overlay</verify>
  <done>Painting into functional areas works visually - colored overlay appears, no blocked cursor</done>
</task>

<task type="auto">
  <name>Task 2: Update worker optimization to count functional area conflicts</name>
  <files>index.html</files>
  <action>
Update the worker's `evaluateCandidate()` function to include functional pattern pixels in diff calculations:

1. **Pixel diff calculation (~line 15273)**: Change:
   - `if (paintedState === 0 || isFunctionPattern) continue;`
   → `if (paintedState === 0) continue;`
   
   This includes functional pattern pixels in the pixel diff count when painted.

2. **QR render for decode (~line 15295)**: Change:
   - `if (!isFunctionPattern && paintedState !== 0)`
   → `if (paintedState !== 0)`
   
   This applies painted pixels over functional patterns when rendering for decode test.

Note: The `isFunctionPattern` variable can remain (for reference/future use), just remove it from the conditions.
  </action>
  <verify>Run optimization search with pattern painted over finder pattern - pixel diff should count those conflicts</verify>
  <done>Worker correctly includes functional area pixels in optimization calculations</done>
</task>

</tasks>

<verification>
1. Visual test: Paint over finder pattern (top-left corner square) - should show colored overlay
2. Visual test: Paint over timing pattern (dotted line) - should show colored overlay  
3. Cursor test: Hover over finder pattern - should NOT show blocked cursor
4. Decode test: Paint conflicting pixels over finder pattern - decode status should show "won't scan" or reduced decodability
5. Optimization test: Run search with functional area painting - pixel diff count should include those pixels
</verification>

<success_criteria>
- Painting allowed on ALL grid cells (no functional pattern restrictions)
- Painted overlay visible on functional patterns
- Pixel diff calculations include functional area conflicts
- QR decode testing applies painted pixels over functional patterns
</success_criteria>

<output>
After completion, create `.planning/quick/4-allow-painting-into-functional-patterns/4-SUMMARY.md`
</output>
