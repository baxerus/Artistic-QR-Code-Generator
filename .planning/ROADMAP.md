# Roadmap: Artistic QR Code Generator

## Milestones

- ✅ **v1.0 MVP** — Phases 1-4 (shipped 2026-02-09) → [archive](milestones/v1-ROADMAP.md)
- ✅ **v1.1 UX Overhaul & Optimization** — Phases 5-9 (shipped 2026-02-14) → [archive](milestones/v1.1-ROADMAP.md)
- ✅ **v1.2 Visual Consistency & Result Inspection** — Phase 10 (shipped 2026-02-15) → [archive](milestones/v1.2-ROADMAP.md)
- 🔄 **v1.3 Hash Capacity Optimization** — Phase 11 (active)

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

### v1.3 Hash Capacity Optimization (Phase 11) — ACTIVE

#### Phase 11: Dynamic Hash Capacity

**Goal:** Hash fragment length fills available QR capacity, maximizing data area variation for better pattern alignment.

**Dependencies:** None (builds on existing hash optimization from v1.0/v1.1)

**Requirements:** HASH-01, HASH-02, HASH-03

**Success Criteria:**
1. Hash length is calculated dynamically based on QR version capacity minus URL byte length
2. MIN_HASH_LENGTH constant exists (renamed from HASH_LENGTH) clarifying minimum, not fixed
3. Generated hashes expand to fill remaining byte capacity of selected QR version
4. Longer hashes produce more data area variation during optimization search

## Progress

| Phase | Milestone | Plans | Status | Completed |
|-------|-----------|-------|--------|-----------|
| 1-4 | v1.0 | 8/8 | Complete | 2026-02-09 |
| 5-9 | v1.1 | 8/8 | Complete | 2026-02-14 |
| 10 | v1.2 | 3/3 | Complete | 2026-02-15 |
| 11 | v1.3 | 0/? | Active | — |

**Total:** 11 phases, 19 plans across 4 milestones (3 shipped, 1 active)

---
*Roadmap created: 2026-02-10*
*Last updated: 2026-02-15 (v1.3 Phase 11 added)*
