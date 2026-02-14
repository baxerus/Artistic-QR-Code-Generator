---
status: diagnosed
phase: 10-visual-polish-result-inspection
source: [10-01-SUMMARY.md]
started: 2026-02-15T00:00:00Z
updated: 2026-02-15T00:10:00Z
---

## Current Test

[testing complete]

## Tests

### 1. Blue Protected Area Borders on Paint Canvas
expected: Open the app and enter a URL to generate a QR code. Look at the paint canvas — the finder patterns (3 large squares in corners), timing patterns (lines connecting finders), and alignment patterns (small squares, if version >= 2) should have BLUE dashed borders around them, not red.
result: issue
reported: "The blue is good. But red was easier to see, than blue is. So the blue is a bit to transparent. Please make it a bit less transparent so it is easier to see."
severity: minor

### 2. Plain QR Code Display by Default
expected: Run the optimizer to generate some results (Top 5 list). The scaled QR codes in the results list should display as clean black/white QR codes without any colored overlays visible. No red/green error pixels, no blue borders — just plain QR.
result: pass

### 3. Error Overlay on Hover
expected: Hover your mouse over one of the scaled results in the Top 5 list. An error visualization overlay should appear showing red pixels (conflicts with painted pattern) and green pixels (matches with painted pattern). The overlay appears immediately without clicking.
result: issue
reported: "The overlay appears. But the transition takes a bit to much time. Also in the NOT SCALED results (so still small) do NOT show the legend ('Match', 'Conflict', 'Function'), because this takes more space, the result gets taller and the page visually jumps on hover. It is enough, if the legend is only shown in the scaled up version"
severity: minor

### 4. Blue Protected Borders on Hover
expected: While hovering over a scaled result, you should also see blue dashed borders around the protected areas (finder patterns, timing patterns, alignment patterns) — same blue color as the paint canvas borders.
result: issue
reported: "The overlay is ALWAYS visible in the scaled up version. Not only when hovering over the scaled up result. Also there is a blue overlay for the functional parts, but it has NO dashed outline."
severity: major

### 5. Overlay Disappears on Mouse Leave
expected: Move your mouse away from the scaled result. The error overlay and blue protected borders should disappear, returning the result to its plain black/white QR code view.
result: issue
reported: "No this is not working for the scaled up result. The overlay is always visible in the scaled up version. In the small results the overlay is going away when the mouse moves away from the result, but not in the scaled up version"
severity: major

### 6. Hover Disabled During Search
expected: Start a new optimization search. While the search is running (spinner visible), hover over existing results in the Top 5 list. The hover effect should NOT show overlays while search is active — overlays only appear when search is stopped/complete.
result: pass

## Summary

total: 6
passed: 2
issues: 4
pending: 0
skipped: 0

## Gaps

- truth: "Blue protected area borders are clearly visible on paint canvas"
  status: failed
  reason: "User reported: The blue is good. But red was easier to see, than blue is. So the blue is a bit to transparent. Please make it a bit less transparent so it is easier to see."
  severity: minor
  test: 1
  root_cause: "Blue (#3b82f6) perceptually dimmer than red (#ff6b6b) - red has max 255 channel, blue has lower values. Same 0.15 overlay opacity insufficient for blue."
  artifacts:
    - path: "index.html"
      issue: "PROTECTED_OVERLAY_COLOR opacity 0.15 too low for blue (lines 13589-13590)"
  missing:
    - "Increase overlay opacity from 0.15 to 0.25-0.30 for blue"
  debug_session: ".planning/debug/blue-border-visibility.md"

- truth: "Error overlay appears immediately on hover without visual disruption"
  status: failed
  reason: "User reported: The overlay appears. But the transition takes a bit to much time. Also in the NOT SCALED results (so still small) do NOT show the legend ('Match', 'Conflict', 'Function'), because this takes more space, the result gets taller and the page visually jumps on hover. It is enough, if the legend is only shown in the scaled up version"
  severity: minor
  test: 3
  root_cause: "Transition has 0.3s duration + 0.3s delay = 0.6s total latency (line 716). Legend shows on all hovers via .result-slot:hover .overlay-legend rule (lines 727-729) instead of only on .expanded."
  artifacts:
    - path: "index.html"
      issue: "Transition too slow (line 716); legend shows on non-expanded hover (lines 727-729)"
  missing:
    - "Reduce transition to 0.1-0.15s with no delay"
    - "Remove .result-slot:hover .overlay-legend rule - keep only .expanded .overlay-legend"
  debug_session: ".planning/debug/overlay-transition-legend.md"

- truth: "Blue dashed borders appear on protected areas only when hovering over scaled result"
  status: failed
  reason: "User reported: The overlay is ALWAYS visible in the scaled up version. Not only when hovering over the scaled up result. Also there is a blue overlay for the functional parts, but it has NO dashed outline."
  severity: major
  test: 4
  root_cause: "CSS rule .result-slot.expanded .result-overlay { opacity: 1 } (lines 719-721) has no :hover condition - unconditionally visible. Dashed lines invisible: lineWidth 0.15 and setLineDash([0.5, 0.25]) too small (lines 15903-15904)."
  artifacts:
    - path: "index.html"
      issue: ".expanded .result-overlay forces opacity:1 unconditionally (lines 719-721)"
    - path: "index.html"
      issue: "lineWidth 0.15 and dash [0.5, 0.25] too small to be visible (lines 15903-15904)"
  missing:
    - "Change to .result-slot.expanded:hover .result-overlay { opacity: 1 } or remove rule"
    - "Increase lineWidth to 0.5-1.0 and setLineDash to [2, 1] or similar"
  debug_session: ".planning/debug/expanded-overlay-always-visible.md"

- truth: "Overlay disappears when mouse leaves scaled result"
  status: failed
  reason: "User reported: No this is not working for the scaled up result. The overlay is always visible in the scaled up version. In the small results the overlay is going away when the mouse moves away from the result, but not in the scaled up version"
  severity: major
  test: 5
  root_cause: "Same as test 4: .result-slot.expanded .result-overlay { opacity: 1 } (lines 719-721) is unconditional - no hover requirement means overlay never hides when expanded."
  artifacts:
    - path: "index.html"
      issue: ".expanded .result-overlay rule has no :hover condition (lines 719-721)"
  missing:
    - "Change to .result-slot.expanded:hover .result-overlay { opacity: 1 } or remove the rule entirely"
  debug_session: ".planning/debug/expanded-overlay-no-hide.md"
