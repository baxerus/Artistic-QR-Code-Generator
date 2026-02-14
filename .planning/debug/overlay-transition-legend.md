---
status: diagnosed
trigger: "Error overlay transition too slow and legend causes visual jump in small results"
created: 2026-02-15T00:00:00Z
updated: 2026-02-15T00:00:00Z
symptoms_prefilled: true
goal: find_root_cause_only
---

## Current Focus

hypothesis: CONFIRMED - Two CSS issues found
test: Examined CSS transition and legend display rules
expecting: Found transition value > 150ms and legend always visible on hover
next_action: Return diagnosis

## Symptoms

expected: Overlay appears immediately on hover without visual disruption
actual: Overlay transition takes too much time. Legend shows in non-scaled results causing visual jump on hover.
errors: None reported
reproduction: Test 3 in UAT
started: Discovered during UAT

## Eliminated

## Evidence

- timestamp: 2026-02-15T00:00:00Z
  checked: CSS transition property on .result-overlay
  found: Line 716 - `transition: opacity 0.3s ease 0.3s` - This has TWO delays: 0.3s duration AND 0.3s delay before starting
  implication: Total time before overlay fully appears is 0.6s (600ms) which is very slow

- timestamp: 2026-02-15T00:00:00Z
  checked: CSS rules for overlay-legend visibility on hover
  found: Line 727-729 - `.result-slot:hover .overlay-legend { display: flex; }` - This shows legend on ALL hover states, not just expanded
  implication: Legend appears on non-expanded (small) results causing visual jump due to added height

- timestamp: 2026-02-15T00:00:00Z
  checked: Expanded state legend rule
  found: Lines 771-773 - `.result-slot.expanded .overlay-legend { display: flex; }` correctly shows legend for expanded
  implication: The expanded rule is correct, but the hover rule (line 727-729) overrides for non-expanded results

## Resolution

root_cause: |
  TWO CSS ISSUES FOUND:
  
  1. SLOW OVERLAY TRANSITION (Line 716):
     `transition: opacity 0.3s ease 0.3s`
     - 0.3s duration + 0.3s delay = 0.6s total before overlay fully visible
     - User perceives this as "too slow"
  
  2. LEGEND VISIBLE ON ALL HOVERS (Lines 727-729):
     `.result-slot:hover .overlay-legend { display: flex; }`
     - Shows legend on ALL result-slot hovers, including non-expanded (small) results
     - Legend adds height to result slot, causing visual jump
     - Should ONLY show legend on .expanded state, not on simple hover

fix: 
verification: 
files_changed: []
