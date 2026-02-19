---
phase: quick/10-preserve-painted-pattern-when-changing-q
plan: 10
type: execute
wave: 1
depends_on: []
files_modified: [index.html]
autonomous: true
requirements: [QUICK-10]

must_haves:
  truths:
    - "When the QR version changes with a painted pattern, the pattern remains visible after Generate"
    - "The preserved pattern is centered in the new grid and any overflow is clipped"
    - "Version-change warnings describe recentering/clipping instead of pattern loss"
  artifacts:
    - path: "index.html"
      provides: "Pattern remap logic that centers old grid into new grid"
      min_lines: 10
    - path: "index.html"
      provides: "Updated version-change warning/confirm text"
      min_lines: 1
  key_links:
    - from: "generateQR()"
      to: "initPaintCanvas(newVersion, { priorGrid })"
      via: "passing previous paintGrid when version changes"
      pattern: "initPaintCanvas\(version,"
    - from: "initPaintCanvas"
      to: "remapPatternToNewGrid"
      via: "centered copy with clipping"
      pattern: "remapPatternToNewGrid"
---

<objective>
Preserve the painted pattern when changing QR versions by re-centering the old grid into the new grid and clipping overflow.

Purpose: The painted pattern is sacred; version changes should not wipe art. Keeping it centered maintains visual continuity while respecting new grid size.
Output: index.html updated with remap logic and warning copy that matches the new behavior.
</objective>

<execution_context>
@/home/node/.config/opencode/get-shit-done/workflows/execute-plan.md
@/home/node/.config/opencode/get-shit-done/templates/summary.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/STATE.md
@index.html
</context>

<tasks>

<task type="auto">
  <name>Task 1: Remap painted pattern on version change (center + clip)</name>
  <files>index.html</files>
  <action>
    1. Add a helper near the PixelGrid section (before initPaintCanvas) named `remapPatternToNewGrid(oldGrid, newSize)`:
       - Compute `oldSize = oldGrid.size` and center offset: `const delta = (newSize - oldSize) / 2` (both odd sizes so integer).
       - Create `const newGrid = new PixelGrid(newSize)`.
       - For every cell in oldGrid, compute `newX = x + delta`, `newY = y + delta`.
       - If `newX`/`newY` are inside bounds, copy the cell state into `newGrid`.
       - Return `newGrid`.
       - This centers the old pattern in the new grid and clips overflow (cells outside bounds are skipped).

    2. Update `initPaintCanvas(version)` to accept an optional `options` object (e.g., `initPaintCanvas(version, { priorGrid })`).
       - Create the new `paintGrid` as usual.
       - If `priorGrid` is provided and has any painted cells, replace `paintGrid` with `remapPatternToNewGrid(priorGrid, gridSize)`.
       - Otherwise, keep the existing `loadPattern(version)` restore behavior.
       - After applying a remap, call `savePattern(version, paintGrid.cells)` so the preserved pattern is saved under the new version.

    3. In `generateQR()`, capture the existing grid before version change:
       - Store `const previousGrid = paintGrid;` and `const previousVersion = currentVersion;` before `initPaintCanvas` runs.
       - When calling `initPaintCanvas(version, ...)`, pass `priorGrid` only if `previousGrid` exists, `hasPattern()` is true, and `previousVersion !== version`.
       - This ensures we only remap when a real version change occurs with a painted pattern.
  </action>
  <verify>
    1. Open index.html in a browser.
    2. Paint a recognizable pattern near an edge, then switch to a larger version and click Generate.
    3. Confirm the pattern is preserved and centered; extra parts remain clipped if they would exceed bounds.
    4. Switch to a smaller version and repeat; confirm centered clip behavior again.
  </verify>
  <done>Version changes preserve the painted pattern by centering it in the new grid and clipping overflow.</done>
</task>

<task type="auto">
  <name>Task 2: Update version-change warning text to match preservation behavior</name>
  <files>index.html</files>
  <action>
    1. Update the in-form warning text (`#version-warning`) from “will not be copied” to language that explains:
       - The pattern will be preserved
       - It will be re-centered in the new version and clipped if necessary

    2. Update the confirm dialog in `updateVersionDropdown` (URL-forced version bump) to the same message, replacing any wording about clearing or not copying the pattern.
  </action>
  <verify>Search index.html for "will clear your painted pattern" or "will not be copied" and confirm it is replaced with recenter/clip wording.</verify>
  <done>All user-facing warnings describe preservation with centering/clipping instead of pattern loss.</done>
</task>

</tasks>

<verification>
- [ ] Changing QR version with a painted pattern preserves the pattern after Generate
- [ ] Preserved pattern is centered and clipped at the new grid bounds
- [ ] Warning/confirm copy matches the new preservation behavior
</verification>

<success_criteria>
- Pattern survives QR version changes and stays centered in the new grid
- No user-facing text claims the pattern will be cleared or not copied
</success_criteria>

<output>
After completion, create `.planning/quick/10-preserve-painted-pattern-when-changing-q/10-SUMMARY.md`
</output>
