---
type: quick
task: 1
slug: fix-url-too-long-alert-cancel-should-rev
date: 2026-02-15
commit: 59c41c5
---

# Quick Task 1: Fix URL Too Long Alert Cancel Behavior

**Fixed confirm dialog cancel not reverting URL change when version bump rejected**

## What Changed

Reordered `updateVersionDropdown()` to check for version bump confirmation BEFORE modifying dropdown:

- **Before:** `dropdown.innerHTML = ""` cleared options, then showed confirm dialog. Cancel couldn't properly restore state.
- **After:** Check if version bump needed + pattern exists FIRST. Show dialog. Only modify dropdown if confirmed or no pattern.

## Verification

1. Paint a pattern on a QR code
2. Edit URL to make it longer (requiring higher version)
3. Confirm dialog appears
4. Click "Cancel"
5. URL reverts to previous value, version dropdown unchanged

## Commit

`59c41c5` - fix: cancel URL change when version bump is rejected
