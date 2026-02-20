---
phase: quick/12-implement-settings-save-load-reset-menu-
plan: 12
type: execute
wave: 1
depends_on: []
files_modified: ["index.html"]
autonomous: true
requirements: [QUICK-12]

must_haves:
  truths:
    - "User can export current settings and painted pattern to a JSON file"
    - "User can import a JSON file to restore settings and pattern (with safe validation)"
    - "User can reset settings with confirmation, and reset is blocked while search is running"
  artifacts:
    - path: "index.html"
      provides: "Settings menu UI plus save/load/reset handlers"
      contains: "Save settings to file"
  key_links:
    - from: "Settings menu button"
      to: "export/import/reset handlers"
      via: "click handlers on dropdown items"
      pattern: "Save settings to file|Load settings from file|Reset settings"
    - from: "Import handler"
      to: "runtime state + localStorage persistence"
      via: "apply imported fields then re-sync preview/results"
      pattern: "runCorruptionPipeline|resetOptimizationResults"
---

<objective>
Implement a header settings menu that saves, loads, and resets configuration via JSON files.

Purpose: Enable explicit settings portability and quick reset beyond localStorage persistence.
Output: UI menu + handlers for export/import/reset with validation and confirmation.
</objective>

<execution_context>
@/home/node/.config/opencode/get-shit-done/workflows/execute-plan.md
@/home/node/.config/opencode/get-shit-done/templates/summary.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/STATE.md
@.planning/quick/11-propose-settings-save-load-reset-menu-wi/11-PROPOSAL.md
@index.html
</context>

<tasks>

<task type="auto">
  <name>Task 1: Add settings menu UI and dropdown behavior</name>
  <files>index.html</files>
  <action>
    1. Add a compact disk-icon button in the header (aligned to the right of the headline) that toggles a dropdown menu.
    2. Implement dropdown options with exact labels from the proposal: “Save settings to file”, “Load settings from file”, “Reset settings”.
    3. Add menu open/close behavior: clicking button toggles; clicking outside closes; selecting an option closes the menu.
    4. Style the button and dropdown (icon button, panel, hover/focus states) to match existing header UI patterns.
  </action>
  <verify>
    Menu button appears in header and dropdown opens/closes with the three options.
  </verify>
  <done>Settings menu UI exists with correct labels and expected toggle behavior.</done>
</task>

<task type="auto">
  <name>Task 2: Implement settings export/import JSON handlers</name>
  <files>index.html</files>
  <action>
    1. Implement export handler that builds a JSON snapshot matching the proposal schema (url, version, rotation, randomRotation, selectedColor, pattern with version/size/cells) and downloads as `qr-art-settings.json` with application/json.
    2. Implement import handler that opens a JSON file picker, parses JSON safely, and applies fields in order: url, version, rotation/randomRotation, selectedColor, pattern (only if grid size matches current version).
    3. Use existing setters/helpers for URL, version, rotation, color, and pattern persistence; keep best-effort import (missing fields leave current values unchanged; extra fields ignored).
    4. After applying import, re-sync preview/results (corruption pipeline + optimization results reset) and handle invalid JSON with an alert while leaving state unchanged.
  </action>
  <verify>
    Export creates a JSON file and import restores settings without console errors on valid file.
  </verify>
  <done>Save/load flows produce and consume the expected JSON schema with safe validation.</done>
</task>

<task type="auto">
  <name>Task 3: Implement reset with confirmation and search-state guard</name>
  <files>index.html</files>
  <action>
    1. Add reset handler that shows a confirm dialog with copy: “Reset settings and clear stored pattern?”; default cancel.
    2. Block reset when search is running (mirror existing guard used for clear/optimize).
    3. On confirm, clear the defined localStorage keys (url, version, rotation, random rotation, selected color, pattern), reset runtime state to defaults, and persist an empty pattern for the default version.
    4. Re-sync preview/results after reset and ensure UI inputs reflect defaults.
  </action>
  <verify>
    Reset requires confirmation and is blocked during active search; defaults restore afterward.
  </verify>
  <done>Reset flow clears settings safely and returns the app to default state.</done>
</task>

</tasks>

<verification>
- [ ] Settings menu exports a JSON file, imports it back, and reset prompts with guard during search
</verification>

<success_criteria>
- Settings save/load/reset menu works end-to-end with validation and state re-sync
</success_criteria>

<output>
After completion, create `.planning/quick/12-implement-settings-save-load-reset-menu-/12-SUMMARY.md`
</output>
