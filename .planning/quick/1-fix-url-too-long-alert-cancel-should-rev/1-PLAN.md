---
type: quick
task: 1
slug: fix-url-too-long-alert-cancel-should-rev
date: 2026-02-15
---

# Quick Task 1: Fix URL Too Long Alert Cancel Behavior

## Problem

When editing a URL that requires a larger QR version and the user has a painted pattern:
1. Confirm dialog asks "This URL requires a larger QR version (X) and will clear your painted pattern. Continue?"
2. User clicks "Cancel"
3. Bug: The version dropdown was ALREADY modified before the dialog appeared
4. The URL was supposed to revert but dropdown state was corrupted

## Root Cause

In `updateVersionDropdown()`, the code cleared `dropdown.innerHTML = ""` BEFORE checking if a confirm dialog was needed. When the user cancelled, the function tried to restore the previous URL, but the dropdown was already corrupted.

## Solution

Move the confirm dialog check BEFORE any dropdown modifications:
1. Check if version bump is needed AND pattern exists
2. Show confirm dialog
3. If cancelled: restore URL and return immediately (dropdown untouched)
4. Only AFTER confirmation (or no pattern): modify dropdown

## Files Modified

- index.html: `updateVersionDropdown()` function reordered
