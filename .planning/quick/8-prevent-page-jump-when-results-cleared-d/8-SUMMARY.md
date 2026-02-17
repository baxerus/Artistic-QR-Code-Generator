# Quick Task 8: Prevent Page Jump When Results Cleared During Painting

## Problem
When painting a pattern with existing results displayed, clearing the results caused the page to jump vertically. This was disruptive to users actively painting.

## Root Cause
- Body used `display: flex; align-items: center; justify-content: center` for vertical centering
- When results were cleared, content height decreased
- Flexbox recalculated centered position, causing visible jump
- Additionally, scrollbar appearing/disappearing caused horizontal shift

## Solution
1. Removed flexbox centering from body
2. Added 40px top margin to `.container` for visual spacing
3. Added `html { overflow-y: scroll; }` to always show scrollbar, preventing horizontal shift

## Changes
- `index.html`: CSS changes to body and container styles

## Commit
cfdb4df
