# Roadmap: Artistic QR Code Generator

## Milestones

- ✅ **v1.0 MVP** — Phases 1-4 (shipped 2026-02-09) → [archive](milestones/v1-ROADMAP.md)
- ✅ **v1.1 UX Overhaul & Optimization** — Phases 5-9 (shipped 2026-02-14) → [archive](milestones/v1.1-ROADMAP.md)
- ✅ **v1.2 Visual Consistency & Result Inspection** — Phase 10 (shipped 2026-02-15) → [archive](milestones/v1.2-ROADMAP.md)
- ✅ **v1.3 Hash Capacity Optimization** — Phase 11 (shipped 2026-02-15) → [archive](milestones/v1.3-ROADMAP.md)
- ✅ **v1.4 RS Error Measurement** — Phases 12-13 (shipped 2026-02-16)
- 🔄 **v1.5 UX & Export Enhancements** — Phases 14-16 (in progress)

## Phases

<details>
<summary>✅ v1.0 MVP (Phases 1-4) — SHIPPED 2026-02-09</summary>

- [x] Phase 1: Foundation & QR Generation (2/2 plans) — completed 2026-02-07
- [x] Phase 2: Pixel Painting & Corruption (2/2 plans) — completed 2026-02-07
- [x] Phase 3: Hash Optimization Loop (2/2 plans) — completed 2026-02-09
- [x] Phase 4: Results & Export (2/2 plans) — completed 2026-02-09

</details>

<details>
<summary>✅ v1.1 UX Overhaul & Optimization (Phases 5-9) — SHIPPED 2026-02-14</summary>

- [x] Phase 5: Configuration Constants (1/1 plan) — completed 2026-02-10
- [x] Phase 6: State Persistence (1/1 plan) — completed 2026-02-10
- [x] Phase 7: Painting Overhaul (2/2 plans) — completed 2026-02-13
- [x] Phase 8: Generation Safety (2/2 plans) — completed 2026-02-14
- [x] Phase 9: Optimization Upgrades (2/2 plans) — completed 2026-02-14

</details>

<details>
<summary>✅ v1.2 Visual Consistency & Result Inspection (Phase 10) — SHIPPED 2026-02-15</summary>

- [x] Phase 10: Visual Polish & Result Inspection (3/3 plans) — completed 2026-02-15

</details>

<details>
<summary>✅ v1.3 Hash Capacity Optimization (Phase 11) — SHIPPED 2026-02-15</summary>

- [x] Phase 11: Dynamic Hash Capacity (1/1 plan) — completed 2026-02-15

</details>

<details>
<summary>✅ v1.4 RS Error Measurement (Phases 12-13) — SHIPPED 2026-02-16</summary>

- [x] Phase 12: RS Measurement Integration (3/3 plans) — completed 2026-02-16
  - [x] 12-01-PLAN.md — Modify jsQR to expose RS correction count
  - [x] 12-02-PLAN.md — Worker captures and returns RS data
  - [x] 12-03-PLAN.md — Main thread RS-aware result merging
- [x] Phase 13: Results Ranking & Display (2/2 plans) — completed 2026-02-16
  - [x] 13-01-PLAN.md — Update result card UI to display RS + pixel metrics
  - [x] 13-02-PLAN.md — Redefine perfect result as RS=0 for auto-stop

</details>

### v1.5 UX & Export Enhancements (Phases 14-16)

- [x] **Phase 14: Shift+Paint Shortcut** — Shift+click/drag paints "unset" for quick erasing (completed 2026-02-17)
- [x] **Phase 15: SVG Export** — Download result cards as vector SVG files (completed 2026-02-17)
- [ ] **Phase 16: RS Capacity Display** — Show corrections as "X of Y" with max capacity

## Phase Details

### Phase 14: Shift+Paint Shortcut

**Goal:** Users can quickly paint "unset" to erase painted pixels without changing legend selection.

**Dependencies:** Phase 13 (v1.4 complete)

**Requirements:**
- PAINT-01: Shift+click/drag (left or right button) paints "unset" regardless of selected color or button
- PAINT-02: Shift state captured at stroke start and held for entire drag

**Success Criteria:**
1. User can hold Shift and left-click to paint "unset" even when white or black is selected in legend
2. User can hold Shift and right-click to paint "unset" (same behavior as Shift+left-click)
3. Shift state at stroke start determines behavior for entire drag operation (releasing Shift mid-drag doesn't change behavior)
4. Normal painting behavior unchanged when Shift is not held

**Plans:** 1/1 plans complete

Plans:
- [x] 14-01-PLAN.md — Shift+paint override and legend visual feedback

---

### Phase 15: SVG Export

**Goal:** Users can download result QR codes as scalable vector graphics for high-quality printing.

**Dependencies:** Phase 14 (isolated, but sequenced for risk ordering)

**Requirements:**
- SVG-01: Result card has "Download SVG" button alongside existing PNG button
- SVG-02: SVG contains black/white modules only (final QR, not preview colors)
- SVG-03: Downloaded SVG filename follows same pattern as PNG export (different extension only)

**Success Criteria:**
1. User can click "Download SVG" button on any result card
2. Downloaded SVG opens in vector editors (Illustrator, Inkscape) at any size without pixelation
3. SVG modules are pure black (#000) on white (#FFF) background, matching final QR output
4. Filename matches PNG pattern with .svg extension (e.g., "qr-abc123.svg")

**Plans:** 1/1 plans complete

Plans:
- [x] 15-01-PLAN.md — SVG generation function and Download SVG button on result cards

---

### Phase 16: RS Capacity Display

**Goal:** Users see how close they are to QR scan failure with "X of Y" correction capacity display.

**Dependencies:** Phase 15 (isolated, but sequenced for risk ordering)

**Requirements:**
- RSCAP-01: Result card shows correction count as "X of Y" where Y is max capacity for QR version at level H
- RSCAP-02: Max capacity calculated from jsQR VERSIONS table using formula: `sum(ecBlocks × ecCodewordsPerBlock) / 2`

**Success Criteria:**
1. Result card displays "RS: X of Y" where X is current corrections and Y is maximum capacity
2. Y value is accurate for the QR version being displayed (verified against QR spec)
3. Display updates correctly when results change (search produces new candidates)

**Plans:** 1 plan

Plans:
- [ ] 16-01-PLAN.md — Add RS capacity helper and update correction display

---

## Progress

| Phase | Milestone | Plans | Status | Completed |
|-------|-----------|-------|--------|-----------|
| 1-4 | v1.0 | 8/8 | Complete | 2026-02-09 |
| 5-9 | v1.1 | 8/8 | Complete | 2026-02-14 |
| 10 | v1.2 | 3/3 | Complete | 2026-02-15 |
| 11 | v1.3 | 1/1 | Complete | 2026-02-15 |
| 12 | v1.4 | 3/3 | Complete | 2026-02-16 |
| 13 | v1.4 | 2/2 | Complete | 2026-02-16 |
| 14 | v1.5 | 1/1 | Complete | 2026-02-17 |
| 15 | v1.5 | 1/1 | Complete | 2026-02-17 |
| 16 | v1.5 | 0/? | Not started | - |

**Total:** 16 phases (14 shipped, 2 remaining)

**Coverage:** 7/7 v1.5 requirements mapped ✓

---
*Roadmap created: 2026-02-10*
*Last updated: 2026-02-17 (v1.5 roadmap created)*
