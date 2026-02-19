# Roadmap: Artistic QR Code Generator

## Milestones

- ✅ **v1.0 MVP** — Phases 1-4 (shipped 2026-02-09) → [archive](milestones/v1-ROADMAP.md)
- ✅ **v1.1 UX Overhaul & Optimization** — Phases 5-9 (shipped 2026-02-14) → [archive](milestones/v1.1-ROADMAP.md)
- ✅ **v1.2 Visual Consistency & Result Inspection** — Phase 10 (shipped 2026-02-15) → [archive](milestones/v1.2-ROADMAP.md)
- ✅ **v1.3 Hash Capacity Optimization** — Phase 11 (shipped 2026-02-15) → [archive](milestones/v1.3-ROADMAP.md)
- ✅ **v1.4 RS Error Measurement** — Phases 12-13 (shipped 2026-02-16)
- ✅ **v1.5 UX & Export Enhancements** — Phases 14-16 (shipped 2026-02-18) → [archive](milestones/v1.5-ROADMAP.md)
- 🚧 **v1.6 QR Code Rotation & Pattern Tools** — Phases 17-19 (in progress)

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

<details>
<summary>✅ v1.5 UX & Export Enhancements (Phases 14-16) — SHIPPED 2026-02-18</summary>

- [x] Phase 14: Shift+Paint Shortcut (1/1 plans) — completed 2026-02-17
- [x] Phase 15: SVG Export (1/1 plans) — completed 2026-02-17
- [x] Phase 16: RS Capacity Display (1/1 plans) — completed 2026-02-18

</details>

### 🚧 v1.6 QR Code Rotation & Pattern Tools (Phases 17-19) — IN PROGRESS

**Milestone Goal:** Improve pattern workflows with rotation-aware QR display and pattern repositioning tools.

- [x] **Phase 17: Decode Metrics Alignment** - Paint Pattern decode test matches result metrics and stable layout (completed 2026-02-18)
- [x] **Phase 18: Move Pattern Tool** - Reposition painted pattern without altering pixel values (completed 2026-02-18)
- [ ] **Phase 19: QR Rotation Controls** - Rotate QR across preview, results, exports, and search

## Phase Details

### Phase 17: Decode Metrics Alignment
**Goal**: Users can validate painted patterns with decode metrics consistent with result cards.
**Depends on**: Phase 16
**Requirements**: DECODE-01, DECODE-02, DECODE-03
**Success Criteria** (what must be TRUE):
  1. User sees Paint Pattern decode corrections displayed as "X of Y" using the same capacity calculation as result cards.
  2. User sees pixel error count alongside corrections in the Paint Pattern decode test.
  3. User can toggle decode status between Success/Fail/Neutral without any layout jump in the status row area.
**Plans**: 3 plans

Plans:
- [ ] 17-01-PLAN.md — Align Paint Pattern decode metrics with result cards
- [ ] 17-02-PLAN.md — Show pixel errors on decode failure
- [ ] 17-03-PLAN.md — Split metrics lines, preserve neutral width, show max corrections on failure

### Phase 18: Move Pattern Tool
**Goal**: Users can reposition the entire painted pattern without changing any painted values.
**Depends on**: Phase 17
**Requirements**: PATTERN-01, PATTERN-02
**Success Criteria** (what must be TRUE):
  1. User can select a "Move Pattern" tool from the paint tool selector alongside Black/White/Unset.
  2. User can drag to shift the full painted pattern, and every pixel retains its original value (black/white/unset) after the move.
**Plans**: 1 plan

Plans:
- [ ] 18-01-PLAN.md — Add Move Pattern tool with drag + arrow-key shifts

### Phase 19: QR Rotation Controls
**Goal**: Users can rotate QR presentation and search without rotating the painted pattern.
**Depends on**: Phase 18
**Requirements**: ROT-01, ROT-02, ROT-03, ROT-04
**Success Criteria** (what must be TRUE):
  1. User can rotate the QR code and protected overlay by 90° in the Paint Pattern section while the painted pattern orientation remains fixed.
  2. User sees result previews and hover overlays reflect the current QR rotation.
  3. User exports PNG/SVG that match the current QR rotation while preserving the painted pattern orientation.
  4. User can enable random rotation for Generate so searches apply 0/90/180/270 rotation to the QR only, reflected in the resulting previews.
**Plans**: 3 plans

Plans:
- [ ] 19-01-PLAN.md — Add rotation control, persistence, and paint-preview rotation
- [ ] 19-02-PLAN.md — Rotate result previews/overlays and align PNG/SVG exports
- [ ] 19-03-PLAN.md — Add random rotation for Generate and worker integration

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
| 16 | v1.5 | Complete    | 2026-02-18 | 2026-02-18 |
| 17 | 3/3 | Complete    | 2026-02-18 | - |
| 18 | v1.6 | Complete    | 2026-02-18 | - |
| 19 | v1.6 | TBD | Not started | - |

**Total:** 19 phases (16 shipped, 3 remaining)

**Coverage:** 9/9 v1.6 requirements mapped ✓

---
*Roadmap created: 2026-02-18*
*Last updated: 2026-02-18 (v1.6 roadmap created)*
