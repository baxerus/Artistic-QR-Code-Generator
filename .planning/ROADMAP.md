# Roadmap: Artistic QR Code Generator

## Milestones

- ✅ **v1.0 MVP** — Phases 1-4 (shipped 2026-02-09) → [archive](milestones/v1-ROADMAP.md)
- ✅ **v1.1 UX Overhaul & Optimization** — Phases 5-9 (shipped 2026-02-14) → [archive](milestones/v1.1-ROADMAP.md)
- ✅ **v1.2 Visual Consistency & Result Inspection** — Phase 10 (shipped 2026-02-15) → [archive](milestones/v1.2-ROADMAP.md)
- ✅ **v1.3 Hash Capacity Optimization** — Phase 11 (shipped 2026-02-15) → [archive](milestones/v1.3-ROADMAP.md)
- 🚧 **v1.4 RS Error Measurement** — Phases 12-13 (in progress)

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
<summary>🚧 v1.4 RS Error Measurement (Phases 12-13) — IN PROGRESS</summary>

- [ ] Phase 12: RS Measurement Integration (3 plans)
  - [x] 12-01-PLAN.md — Modify jsQR to expose RS correction count
  - [ ] 12-02-PLAN.md — Worker captures and returns RS data
  - [ ] 12-03-PLAN.md — Main thread RS-aware result merging
- [ ] Phase 13: Results Ranking & Display — RS metrics in UI, ranking by RS

</details>

## Phase 12: RS Measurement Integration

**Goal:** Workers extract and return Reed-Solomon correction count for each candidate QR during optimization search.

**Dependencies:** Phase 11 (Dynamic Hash Capacity)

**Requirements:**
- RS-01: Worker extracts Reed-Solomon correction count from each candidate QR during optimization search
- RS-02: RS extraction works with jsQR library or alternative browser decoder if jsQR doesn't expose RS data
- RS-03: RS correction count is returned alongside pixel diff for each candidate result

**Success Criteria:**
1. Worker code extracts RS correction count from decoded QR candidates (not just pixel diff)
2. RS extraction works reliably with jsQR or fallback decoder when jsQR doesn't expose RS data
3. Each optimization result includes both pixel diff and RS correction count in worker message
4. Search continues to function with RS data integrated (no performance degradation)

---

## Phase 13: Results Ranking & Display

**Goal:** Results are displayed with RS metrics and ranked by actual QR scan reliability margin.

**Dependencies:** Phase 12 (RS Measurement Integration)

**Requirements:**
- RSLT-01: Result cards display both RS corrections and pixel diff metrics
- RSLT-02: Results are ranked by RS corrections (primary, ascending), pixel diff (secondary, ascending)
- RSLT-03: "Perfect result" is redefined as RS=0 (regardless of pixel diff)
- RSLT-04: Auto-stop triggers when TOP_RESULTS_COUNT results with RS=0 are found

**Success Criteria:**
1. Each result card shows both RS corrections count and pixel diff in the UI
2. Results are sorted by RS corrections ascending (fewer = better), with pixel diff as tiebreaker
3. "Perfect result" indicator displays when RS=0, even if pixel diff is non-zero
4. Search automatically stops when TOP_RESULTS_COUNT results with RS=0 are collected

## Progress

| Phase | Milestone | Plans | Status | Completed |
|-------|-----------|-------|--------|-----------|
| 1-4 | v1.0 | 8/8 | Complete | 2026-02-09 |
| 5-9 | v1.1 | 8/8 | Complete | 2026-02-14 |
| 10 | v1.2 | 3/3 | Complete | 2026-02-15 |
| 11 | v1.3 | 1/1 | Complete | 2026-02-15 |
| 12 | v1.4 | 1/3 | In progress | — |
| 13 | v1.4 | — | Planned | — |

**Total:** 13 phases planned (11 shipped, 2 in v1.4)

**Coverage:** 7/7 v1.4 requirements mapped ✓

---
*Roadmap created: 2026-02-10*
*Last updated: 2026-02-16 (Phase 12 Plan 01 complete)*
