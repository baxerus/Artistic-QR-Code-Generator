---
status: diagnosed
trigger: "Overlay doesn't disappear when mouse leaves scaled-up (expanded) result"
created: 2026-02-15T00:00:00Z
updated: 2026-02-15T00:00:00Z
---

## Current Focus

hypothesis: Expanded state has unconditional `opacity: 1` rule that overrides hover behavior
test: Compare CSS specificity between `.result-slot.expanded .result-overlay` and `.result-slot:hover .result-overlay`
expecting: Expanded state rule takes precedence regardless of hover state
next_action: Document root cause

## Symptoms

expected: Overlay disappears when mouse leaves scaled (expanded) result, returning to plain QR view
actual: Overlay always visible in expanded/scaled-up version, works correctly in small (non-expanded) results
errors: None
reproduction: Test 5 in UAT - expand a result and move mouse away
started: Discovered during UAT

## Eliminated

(none)

## Evidence

- timestamp: 2026-02-15T00:01:00Z
  checked: CSS rules for .result-overlay in index.html
  found: |
    Line 714-717: `.result-overlay` has default `opacity: 0`
    Line 719-721: `.result-slot.expanded .result-overlay` sets `opacity: 1` unconditionally
    Line 723-725: `.result-slot:hover .result-overlay` sets `opacity: 1` on hover
  implication: The expanded rule (line 719-721) ALWAYS forces opacity:1 when expanded class is present, with no consideration for hover state

- timestamp: 2026-02-15T00:01:30Z
  checked: CSS specificity comparison
  found: |
    Both selectors have same specificity (2 classes):
    - `.result-slot.expanded .result-overlay` (lines 719-721)
    - `.result-slot:hover .result-overlay` (lines 723-725)
    
    However, .expanded rule comes FIRST and is unconditional - it doesn't depend on :hover.
    The hover rule only ADDS opacity:1 on hover, but can't REMOVE it from expanded state.
  implication: There is no CSS rule to hide overlay when mouse leaves expanded result - the expanded rule forces it always visible

## Resolution

root_cause: |
  CSS rule at lines 719-721 forces overlay visible unconditionally when expanded:
  
  ```css
  .result-slot.expanded .result-overlay {
    opacity: 1;
  }
  ```
  
  This rule has no :hover condition, so the overlay is ALWAYS visible when expanded.
  
  For non-expanded results, the base rule (opacity: 0) applies, and only :hover (line 723-725) shows the overlay.
  
  But for expanded results, the .expanded rule immediately sets opacity: 1, ignoring hover state entirely.
  
  The design intent seems to be: "show overlay on hover OR when expanded" but the actual behavior is 
  "show overlay on hover, AND always show when expanded (regardless of hover)".
  
  To fix: The expanded state should follow the same hover pattern as non-expanded - only show overlay
  when hovering, not unconditionally. Remove or modify the rule at lines 719-721.

fix: (not applied - find_root_cause_only mode)
verification: (not applied)
files_changed: []
