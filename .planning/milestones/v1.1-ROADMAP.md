# Roadmap: Artistic QR Code Generator

## Milestones

- v1.0 MVP - Phases 1-4 (shipped 2026-02-09)
- v1.1 UX Overhaul & Optimization - Phases 5-9 (shipped 2026-02-14)

## Phases

<details>
<summary>v1.0 MVP (Phases 1-4) - SHIPPED 2026-02-09</summary>

Archived to `.planning/milestones/v1-ROADMAP.md`

- Phase 1: Foundation & QR Generation (2 plans, complete)
- Phase 2: Pixel Painting & Corruption (2 plans, complete)
- Phase 3: Hash Optimization Loop (2 plans, complete)
- Phase 4: Results & Export (2 plans, complete)

</details>

### v1.1 UX Overhaul & Optimization (Complete)

**Milestone Goal:** Improve the painting workflow (overlay on QR, drag painting, state persistence), add safety guards (hash fragment rejection, version change warnings), and speed up optimization (multi-worker, auto-stop).

#### Phase 5: Configuration Constants
**Goal**: QR version range and limits are governed by a single constant, expanded to Version 10
**Depends on**: Phase 4 (v1 codebase)
**Requirements**: CONF-01, CONF-02
**Success Criteria** (what must be TRUE):
  1. QR version dropdown offers versions 2 through 10 (previously capped at 8)
  2. Changing the max version constant in one place updates all version-related logic (dropdown range, validation, min-version calculation)
  3. Version 10 QR codes (57x57) generate, display, and optimize correctly
**Plans**: 1 plan

Plans:
- [x] 05-01-PLAN.md -- Extract version constants, extend capacity table to v10, implement dropdown UX (disable, auto-bump, glow)

#### Phase 6: State Persistence
**Goal**: Users never lose their work when reloading the page
**Depends on**: Phase 5
**Requirements**: PERS-01, PERS-02
**Success Criteria** (what must be TRUE):
  1. User reloads page and sees the same URL, QR version, painted pattern, and settings they had before
  2. Restored state re-renders the QR code with pattern overlay (not just raw data in localStorage)
  3. First visit (no saved state) works normally with default/empty state
**Plans**: 1 plan

Plans:
- [x] 06-01-PLAN.md -- Add localStorage persistence module and wire save/restore for URL, version, and pattern with auto-QR regeneration on load

#### Phase 7: Painting Overhaul
**Goal**: Users paint directly on the QR code preview with drag painting and clear visual feedback about protected areas
**Depends on**: Phase 6
**Requirements**: PAINT-01, PAINT-02, PAINT-03, PAINT-04, PAINT-05
**Success Criteria** (what must be TRUE):
  1. Paint pattern renders as a translucent overlay on top of the QR code preview (single combined view, not separate canvases)
  2. QR structural areas (finder patterns, timing patterns, alignment patterns) are visually distinct and cannot be painted
  3. User selects a color (black/white/unset) from a legend, then clicks or drags to paint multiple pixels in one stroke
  4. Right-click and drag paints the opposite color (black becomes white, white becomes black); right-click does nothing when unset is selected
  5. Pattern state persists correctly after painting overhaul (localStorage updated with new overlay data)
**Plans**: 2 plans

Plans:
- [x] 07-01-PLAN.md — Refactor to unified QR canvas with overlay rendering, protected area border outlines
- [x] 07-02-PLAN.md — Interactive color legend with drag painting (left-click) and opposite color (right-click)

#### Phase 8: Generation Safety
**Goal**: Users are protected from accidental pattern loss and invalid URL inputs
**Depends on**: Phase 7
**Requirements**: SAFE-01, SAFE-02, SAFE-03, SAFE-04, SAFE-05
**Success Criteria** (what must be TRUE):
  1. Entering a URL that ends with a hash fragment (e.g., "example.com/page#section") shows a clear rejection error and prevents generation
  2. When a painted pattern exists, changing the version dropdown does NOT auto-generate a new QR code
  3. When a painted pattern exists, only the "Generate QR Code" button triggers QR generation
  4. Generating a new QR code at the same version preserves all painted pixels on the new code
  5. Changing to a different QR version (via dropdown or URL-forced version change) shows a confirmation dialog warning about pattern loss before proceeding
**Plans**: 2 plans

Plans:
- [x] 08-01-PLAN.md — Hash fragment URL rejection (SAFE-01) and generation guards (SAFE-02, SAFE-03)
- [x] 08-02-PLAN.md — Version change confirmation dialog (SAFE-05) and pattern preservation verification (SAFE-04)

#### Phase 9: Optimization Upgrades
**Goal**: Optimization searches faster with multiple workers and stops automatically when ideal results are found
**Depends on**: Phase 8
**Requirements**: OPT-01, OPT-02
**Success Criteria** (what must be TRUE):
  1. Optimization automatically stops when 5 results with 0 pixel errors have been found
  2. Multiple Web Workers run in parallel during optimization, with combined throughput visibly higher than single-worker baseline
  3. Auto-stop and multi-worker features work together (all workers stop when 5 perfect results accumulated)
  4. Results from all workers merge correctly into the top-5 ranking (no duplicates, proper sorting)
**Plans**: 2 plans

Plans:
- [x] 09-01-PLAN.md — Auto-stop when 5 results with 0 pixel errors found (OPT-01)
- [x] 09-02-PLAN.md — Multi-worker parallelization with result merging (OPT-02)

## Progress

**Execution Order:** Phases 5 -> 6 -> 7 -> 8 -> 9

| Phase | Milestone | Plans Complete | Status | Completed |
|-------|-----------|----------------|--------|-----------|
| 1. Foundation & QR Generation | v1.0 | 2/2 | Complete | 2026-02-07 |
| 2. Pixel Painting & Corruption | v1.0 | 2/2 | Complete | 2026-02-07 |
| 3. Hash Optimization Loop | v1.0 | 2/2 | Complete | 2026-02-09 |
| 4. Results & Export | v1.0 | 2/2 | Complete | 2026-02-09 |
| 5. Configuration Constants | v1.1 | 1/1 | Complete | 2026-02-10 |
| 6. State Persistence | v1.1 | 1/1 | Complete | 2026-02-10 |
| 7. Painting Overhaul | v1.1 | 2/2 | Complete | 2026-02-13 |
| 8. Generation Safety | v1.1 | 2/2 | Complete | 2026-02-14 |
| 9. Optimization Upgrades | v1.1 | 2/2 | Complete | 2026-02-14 |

---
*Roadmap created: 2026-02-10*
*Last updated: 2026-02-14 (Phase 9 complete - v1.1 Milestone Complete)*
