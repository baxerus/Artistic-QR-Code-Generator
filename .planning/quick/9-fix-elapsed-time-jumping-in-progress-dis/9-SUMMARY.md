# Quick Task 9: Fix Elapsed Time Jumping in Progress Display

## Problem
During longer optimization runs, the "X sec of Y sec elapsed" progress display would jump back and forth instead of incrementing smoothly.

## Root Cause
Workers start at staggered intervals (to avoid overwhelming the system), and each worker tracks its own independent `startTime`. When the main thread received PROGRESS messages from different workers, it used the worker's elapsed time directly. Since workers started at different times, their elapsed values differed, causing the display to jump as messages arrived in varying order.

Example with 8 workers (125ms stagger):
- Worker 0 starts at T=0ms, reports 1000ms at T=1000ms
- Worker 1 starts at T=125ms, reports 875ms at T=1000ms
- Worker 2 starts at T=250ms, reports 750ms at T=1000ms

As messages arrived out of order, the display jumped between these values.

## Solution
Changed line 15535 to use the main thread's `searchStartTime` instead of the worker's `e.data.elapsed`:

```javascript
// Before
updateSearchStats(totalAttemptsSoFar, e.data.elapsed);

// After
updateSearchStats(totalAttemptsSoFar, Date.now() - searchStartTime);
```

This ensures consistent elapsed time display regardless of which worker's message arrives.

## Changes
- `index.html`: Line 15535 - use main thread time for elapsed display

## Commit
592bb12
