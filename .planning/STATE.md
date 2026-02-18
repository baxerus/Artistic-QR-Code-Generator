# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-02-18)

**Core value:** The painted pattern is sacred and never changes. The tool finds URL variants that naturally align with the art.
**Current focus:** Planning next milestone

## Current Position

Phase: Not started (defining requirements)
Plan: —
Status: Defining requirements
Last activity: 2026-02-18 — Milestone v1.6 started

Progress: ████████████████████████ 6 milestones shipped (v1.0-v1.5), 16/16 phases complete

## Milestones

**v1.0 MVP (complete):**
- Phases: 1-4
- Plans: 8
- Timeline: 4 days (2026-02-06 → 2026-02-09)
- Archive: `.planning/milestones/v1-ROADMAP.md`

**v1.1 UX Overhaul & Optimization (complete):**
- Phases: 5-9
- Plans: 8
- Timeline: 5 days (2026-02-10 → 2026-02-14)
- Archive: `.planning/milestones/v1.1-ROADMAP.md`

**v1.2 Visual Consistency & Result Inspection (complete):**
- Phases: 10
- Plans: 3
- Timeline: 2 days (2026-02-14 → 2026-02-15)
- Archive: `.planning/milestones/v1.2-ROADMAP.md`

**v1.3 Hash Capacity Optimization (complete):**
- Phases: 11
- Plans: 1
- Timeline: Same day (2026-02-15)
- Archive: `.planning/milestones/v1.3-ROADMAP.md`

**v1.4 RS Error Measurement (complete):**
- Phases: 12-13
- Plans: 5
- Timeline: Same day (2026-02-16)
- Archive: `.planning/milestones/v1.4-ROADMAP.md`

**Total:** 16 phases planned, 16 shipped

## Accumulated Context

Key decisions archived in milestone summaries. See `.planning/MILESTONES.md`.

### Phase 12: RS Measurement Integration ✓
Goal: Workers extract and return Reed-Solomon correction count for each candidate QR
Requirements: RS-01, RS-02, RS-03
Verification: 7/7 must-haves passed (2026-02-16)

### Phase 14: Shift+Paint Shortcut ✓
Goal: Shift+paint shortcut for quick unset painting with legend visual feedback
Requirements: PAINT-01, PAINT-02
Verification: Complete (2026-02-17)

### Phase 14 Decisions

- **14-01**: isShiftStroke captured at pointerdown only — mid-stroke Shift press/release ignored
- **14-02**: Legend feedback uses same .active CSS class as normal selection, no distinct temporary style

### Phase 13: Results Ranking & Display ✓
Goal: Results displayed with RS metrics and ranked by QR reliability
Requirements: RSLT-01, RSLT-02, RSLT-03, RSLT-04
Verification: Complete (2026-02-16)

### Phase 13 Decisions

- **13-01**: Display format "RS: N | Pixels: M" for dual metrics; perfect-rs class for RS=0 results
- **13-02**: RS=0 defines perfect result for auto-stop, not pixelDiff=0

### Phase 12 Decisions

- **12-01**: jsQR correctionCount exposed via object return pattern { bytes, correctionCount }
- **12-02**: Worker rsCorrections field tracks correction count per candidate
- **12-03**: Main thread mergeWorkerResults sorts by RS corrections with nullish coalescing fallback

### Blockers/Concerns

None.

### Quick Tasks Completed

| # | Description | Date | Commit | Directory |
|---|---|---|---|---|
| 1 | Fix URL too long alert - cancel should revert URL change | 2026-02-15 | 9ae4bac | [1-fix-url-too-long-alert-cancel-should-rev](./quick/1-fix-url-too-long-alert-cancel-should-rev/) |
| 2 | Show top 10 results instead of top 5 | 2026-02-15 | 3e22ec6 | [2-instead-of-top-5-results-show-the-top-10](./quick/2-instead-of-top-5-results-show-the-top-10/) |
| 3 | Fix green-highlight animation to play full duration | 2026-02-16 | 54fcae9 | [3-fix-green-highlight-animation-to-play-fu](./quick/3-fix-green-highlight-animation-to-play-fu/) |
| 4 | Allow painting into functional patterns | 2026-02-16 | ccc913c | [4-allow-painting-into-functional-patterns](./quick/4-allow-painting-into-functional-patterns/) |
| 5 | Fix Download PNG and Copy URL buttons to disable during search | 2026-02-17 | 72e4425 | [5-fix-download-png-and-copy-url-buttons-no](./quick/5-fix-download-png-and-copy-url-buttons-no/) |
| 6 | Collapse expanded result when Run again is clicked | 2026-02-17 | bf5ed9d | [6-an-expanded-result-should-get-collapsed-](./quick/6-an-expanded-result-should-get-collapsed-/) |
| 7 | Add X button to collapse expanded result | 2026-02-17 | 0d7935b | [7-add-x-button-to-collapse-expanded-result](./quick/7-add-x-button-to-collapse-expanded-result/) |
| 8 | Prevent page jump when results cleared during painting | 2026-02-17 | cfdb4df | [8-prevent-page-jump-when-results-cleared-d](./quick/8-prevent-page-jump-when-results-cleared-d/) |
| 9 | Fix elapsed time jumping in progress display | 2026-02-17 | 592bb12 | [9-fix-elapsed-time-jumping-in-progress-dis](./quick/9-fix-elapsed-time-jumping-in-progress-dis/) |

## v1.5 Phases

### Phase 14: Shift+Paint Shortcut ✓
- **Goal:** Users can quickly paint "unset" to erase painted pixels without changing legend selection
- **Requirements:** PAINT-01, PAINT-02
- **Risk:** LOW (standard pointer event pattern)
- **Status:** v1.5 milestone complete

### Phase 15: SVG Export ✓
- **Goal:** Users can download result QR codes as scalable vector graphics
- **Requirements:** SVG-01, SVG-02, SVG-03
- **Risk:** LOW (well-documented SVG generation)
- **Status:** Complete (2026-02-17)

### Phase 16: RS Capacity Display ✓
- **Goal:** Users see how close they are to QR scan failure with "X of Y" capacity display
- **Requirements:** RSCAP-01, RSCAP-02
- **Risk:** MEDIUM (formula complexity - divide by 2 critical)
- **Status:** Complete (2026-02-18)

### Phase 16 Decisions

- **16-01**: Use jsQR VERSIONS metadata to compute level H correction capacity and display "Corrections: X of Y" with placeholders

## Session Continuity

Last session: 2026-02-18
Stopped at: Completed 16-01-PLAN.md
Resume: None

---
*v1.5 roadmap created: 2026-02-17*
