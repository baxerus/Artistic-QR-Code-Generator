---
status: diagnosed
trigger: "Overlay is ALWAYS visible in scaled-up (expanded) version, not just on hover; no dashed outline on blue protected areas"
created: 2026-02-15T00:00:00Z
updated: 2026-02-15T00:00:00Z
---

## Current Focus

hypothesis: CSS rule forces overlay visible in expanded state unconditionally
test: Reviewed CSS at lines 719-721
expecting: Should only show on hover
next_action: Return diagnosis

## Symptoms

expected: Blue dashed borders appear on protected areas only when hovering over scaled result
actual: Overlay is ALWAYS visible in expanded version; blue overlay has NO dashed outline
errors: None
reproduction: Test 4 in UAT
started: Discovered during UAT

## Eliminated

(none)

## Evidence

- timestamp: 2026-02-15T00:00:00Z
  checked: CSS rules for .result-overlay (lines 713-725)
  found: Line 719-721 has `.result-slot.expanded .result-overlay { opacity: 1; }` - unconditionally sets opacity to 1 when expanded
  implication: Overlay is ALWAYS visible when a result slot is expanded, regardless of hover state

- timestamp: 2026-02-15T00:00:00Z
  checked: renderProtectedBordersOverlay function (lines 15900-15943)
  found: Function DOES call `ctx.setLineDash([0.5, 0.25])` at line 15904 - dashed lines ARE being set correctly
  implication: The dashes are being rendered, but may be too fine/small to see at the module scale (0.5 and 0.25 are very small values)

- timestamp: 2026-02-15T00:00:00Z
  checked: Line width in renderProtectedBordersOverlay
  found: Line 15903 sets `ctx.lineWidth = 0.15` - extremely thin line at module scale
  implication: Combined with tiny dash values (0.5, 0.25), the dashed effect may not be visible - appears as solid or invisible line

## Resolution

root_cause: |
  TWO ISSUES:
  
  1. **Overlay always visible in expanded state (CSS issue):**
     Line 719-721 in index.html: `.result-slot.expanded .result-overlay { opacity: 1; }`
     This CSS rule unconditionally makes the overlay visible when expanded, without requiring hover.
     The hover rule at lines 723-725 (`.result-slot:hover .result-overlay { opacity: 1 }`) is intended 
     to show on hover, but the expanded rule overrides this by always showing it.
  
  2. **Dashed outline not visible (rendering issue):**
     Lines 15903-15904: `ctx.lineWidth = 0.15` and `ctx.setLineDash([0.5, 0.25])`
     The line width (0.15 modules) and dash segments (0.5 and 0.25 modules) are extremely small.
     At typical QR display sizes, these render as nearly invisible or appear as a solid thin line
     rather than a visible dashed pattern.

fix: (not applied - goal is find_root_cause_only)
verification: (pending)
files_changed: []
