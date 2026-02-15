---
type: quick
task: 1
slug: fix-url-too-long-alert-cancel-should-rev
date: 2026-02-15
commits: [59c41c5, 652b352]
---

# Quick Task 1: Fix URL Too Long Alert Cancel Behavior

**Fixed confirm dialog cancel not reverting URL change when version bump rejected**

## What Changed

1. Reordered `updateVersionDropdown()` to check for version bump confirmation BEFORE modifying dropdown:
   - **Before:** `dropdown.innerHTML = ""` cleared options, then showed confirm dialog. Cancel couldn't properly restore state.
   - **After:** Check if version bump needed + pattern exists FIRST. Show dialog. Only modify dropdown if confirmed or no pattern.

2. Added `saveURL(previousValidURL)` call when cancel is clicked to restore localStorage immediately (overriding the debounced save of the rejected URL).

## Verification

1. Paint a pattern on a QR code
2. Edit URL to make it longer (requiring higher version)
3. Confirm dialog appears
4. Click "Cancel"
5. URL reverts to previous value, version dropdown unchanged
6. Refresh page - previous URL is restored (localStorage correct)

## Commits

- `59c41c5` - fix: cancel URL change when version bump is rejected
- `652b352` - fix: also restore localStorage URL when version bump cancelled
