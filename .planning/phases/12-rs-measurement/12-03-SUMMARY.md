---
phase: 12-rs-measurement
plan: 03
subsystem: result-merging
tags: [optimization, results, sorting, reed-solomon]

# Dependency graph
requires:
  - phase: 12-01
    provides: jsQR correctionCount field for RS metrics
  - phase: 12-02
    provides: Worker rsCorrections field in evaluateCandidate results
provides:
  - Main thread mergeWorkerResults sorts by RS corrections
  - Consistent sorting between worker-level and main-thread result ranking
  - Legacy result handling via Infinity fallback
affects: [13-rs-results]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Nullish coalescing (??) for graceful legacy result handling"
    - "Three-tier sort: decodable > rsCorrections > pixelDiff"

key-files:
  created: []
  modified:
    - index.html (mergeWorkerResults function)

key-decisions:
  - "Use nullish coalescing (??) to treat missing rsCorrections as Infinity"
  - "Mirror worker-level sorting logic for consistency"

patterns-established:
  - "RS-aware sorting: decodable first, then RS corrections ascending, then pixelDiff ascending"

# Metrics
duration: 1 min
completed: 2026-02-16
---

# Phase 12 Plan 03: Main Thread RS-Aware Sorting Summary

**Updated mergeWorkerResults to sort by RS corrections as secondary criterion after decodable status, ensuring consistent result ranking across workers and main thread**

## Performance

- **Duration:** 1 min
- **Started:** 2026-02-16T20:23:49Z
- **Completed:** 2026-02-16T20:24:54Z
- **Tasks:** 1
- **Files modified:** 1

## Accomplishments

- mergeWorkerResults now sorts by RS corrections as secondary criterion
- Lower RS corrections (more reliable QR scans) rank higher among decodable results
- Graceful handling of legacy results without rsCorrections field (Infinity fallback)
- Consistent sorting logic between worker updateTopResults() and main thread mergeWorkerResults()

## Task Commits

Each task was committed atomically:

1. **Task 1: Update mergeWorkerResults sorting to include RS corrections** - `7d90335` (feat)

**Plan metadata:** `98d4c09` (docs: complete plan)

## Files Created/Modified

- `index.html` - Modified mergeWorkerResults function (lines 15551-15562):
  - Added RS corrections comparison as secondary sort criterion
  - Uses nullish coalescing for legacy result compatibility
  - PixelDiff now serves as tiebreaker (tertiary criterion)

## Decisions Made

- **Nullish coalescing**: Use `??` operator to handle legacy results without rsCorrections - treats missing as Infinity (sorts last among decodable)
- **Sort order**: Maintain consistency with worker-level sorting (decodable > RS > pixelDiff)

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- Phase 12 (RS Measurement Integration) complete
- All three plans executed: jsQR exposure (01), worker integration (02), main thread sorting (03)
- Ready for Phase 13: Results Ranking & Display
- RS corrections now flow through entire pipeline: jsQR -> worker -> main thread -> results

## Self-Check

Verifying claims:

```
rsCorrections in mergeWorkerResults (line 15559): FOUND
Commit 7d90335: FOUND
12-03-SUMMARY.md: FOUND
```

## Self-Check: PASSED

---
*Phase: 12-rs-measurement*
*Completed: 2026-02-16*
