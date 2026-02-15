# Roadmap: Artistic QR Code Generator

## Milestones

- ✅ **v1.0 MVP** — Phases 1-4 (shipped 2026-02-09) → [archive](milestones/v1-ROADMAP.md)
- ✅ **v1.1 UX Overhaul & Optimization** — Phases 5-9 (shipped 2026-02-14) → [archive](milestones/v1.1-ROADMAP.md)
- 🔄 **v1.2 Visual Consistency & Result Inspection** — Phase 10 (in progress)

## v1.2 Visual Consistency & Result Inspection

**Goal:** Unify protected area visualization (blue everywhere) and make scaled results easier to inspect by showing plain QR codes by default with overlays appearing on hover.

### Phase 10: Visual Polish & Result Inspection

**Goal:** Users can clearly identify protected areas (blue borders) and inspect scaled results without visual clutter.

**Dependencies:** None — builds on existing v1.1 rendering

**Requirements:**
- VIS-01: Protected area borders on paint canvas use blue instead of red
- VIS-02: Scaled results show blue dashed borders around protected areas on hover
- INSP-01: Scaled results display plain QR code by default (no overlay colors)
- INSP-02: Scaled results show error visualization overlay on hover

**Success Criteria:**
1. Paint canvas shows blue dashed borders around finder patterns, timing patterns, and alignment patterns (not red)
2. Scaled results in Top 5 list display clean black/white QR codes without colored overlays
3. Hovering over a scaled result reveals error visualization (red/green pixels showing conflicts)
4. Hovering over a scaled result shows blue dashed borders around protected areas
5. Moving mouse away from scaled result returns it to plain QR view

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

<details open>
<summary>🔄 v1.2 Visual Consistency & Result Inspection (Phase 10) — IN PROGRESS</summary>

- [ ] Phase 10: Visual Polish & Result Inspection (3/3 plans) — in progress
  - [x] 10-01-PLAN.md — Visual consistency and hover-based result inspection
  - [x] 10-02-PLAN.md — UAT gap closure: overlay visibility and hover fixes
  - [ ] 10-03-PLAN.md — UAT revalidation gap closure: overlay anti-aliasing and border visibility

</details>

## Progress

| Phase | Milestone | Plans | Status | Completed |
|-------|-----------|-------|--------|-----------|
| 1. Foundation & QR Generation | v1.0 | 2/2 | Complete | 2026-02-07 |
| 2. Pixel Painting & Corruption | v1.0 | 2/2 | Complete | 2026-02-07 |
| 3. Hash Optimization Loop | v1.0 | 2/2 | Complete | 2026-02-09 |
| 4. Results & Export | v1.0 | 2/2 | Complete | 2026-02-09 |
| 5. Configuration Constants | v1.1 | 1/1 | Complete | 2026-02-10 |
| 6. State Persistence | v1.1 | 1/1 | Complete | 2026-02-10 |
| 7. Painting Overhaul | v1.1 | 2/2 | Complete | 2026-02-13 |
| 8. Generation Safety | v1.1 | 2/2 | Complete | 2026-02-14 |
| 9. Optimization Upgrades | v1.1 | 2/2 | Complete | 2026-02-14 |
| 10. Visual Polish & Result Inspection | v1.2 | 3/3 | In Progress | - |

**Total:** 10 phases across 3 milestones (3 shipped)

---
*Roadmap created: 2026-02-10*
*Last updated: 2026-02-14 (v1.2 completed)*
