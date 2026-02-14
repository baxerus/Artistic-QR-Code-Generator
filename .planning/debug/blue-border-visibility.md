---
status: diagnosed
trigger: "Blue protected area borders are too transparent and hard to see on paint canvas"
created: 2026-02-15T00:45:00Z
updated: 2026-02-15T00:45:00Z
---

## Current Focus

hypothesis: Blue color (#3b82f6) has inherently lower perceived brightness than red (#ff6b6b), making dashed borders harder to see
test: Compare RGB values and perceived luminance
expecting: Blue has lower luminance/contrast against QR code black/white
next_action: Document root cause for fix by plan-phase --gaps

## Symptoms

expected: Blue dashed borders should be clearly visible around finder, timing, and alignment patterns
actual: User reported blue is harder to see than red was - "a bit too transparent"
errors: None
reproduction: View paint canvas with protected areas visible
started: After commit 6953d04 changed from red to blue

## Eliminated

(none - direct root cause identification)

## Evidence

- timestamp: 2026-02-15T00:45:00Z
  checked: Current color constants at line 13589-13590
  found: PROTECTED_OUTLINE_COLOR = "#3b82f6", PROTECTED_OVERLAY_COLOR = "rgba(59, 130, 246, 0.15)"
  implication: Same 15% opacity as red was, but blue has lower perceived brightness

- timestamp: 2026-02-15T00:45:00Z
  checked: Git commit 6953d04 showing previous values
  found: Red was "#ff6b6b" with "rgba(255, 107, 107, 0.15)" overlay
  implication: Direct 1:1 replacement of color values without adjusting for perceptual differences

- timestamp: 2026-02-15T00:45:00Z
  checked: RGB analysis
  found: Red #ff6b6b = RGB(255, 107, 107) - high red channel. Blue #3b82f6 = RGB(59, 130, 246) - lower overall brightness
  implication: Blue has lower luminance. Red at 255 is brighter than blue at 246, especially with lower R/G channels

- timestamp: 2026-02-15T00:45:00Z
  checked: Human vision science
  found: Human eyes are most sensitive to green, then red, least sensitive to blue
  implication: Blue at same intensity appears dimmer than red to human perception

## Resolution

root_cause: The blue color (#3b82f6) has lower perceived brightness than the previous red (#ff6b6b) due to: (1) lower RGB values overall, and (2) human eyes being less sensitive to blue wavelengths. The overlay opacity was kept at 0.15 without compensation for the perceptual difference.
fix: [to be applied by plan-phase --gaps]
verification: [to be done after fix]
files_changed: []
