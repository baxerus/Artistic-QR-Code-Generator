# Project Milestones: Artistic QR Code Generator

## v1 MVP (Shipped: 2026-02-09)

**Delivered:** Single-file HTML tool for creating artistic QR codes by painting pixel patterns and searching for URL hash fragments that naturally align with the art.

**Phases completed:** 1-4 (8 plans total)

**Key accomplishments:**
- Single-file HTML architecture (508KB, 15,161 lines) with qrcodejs and jsQR libraries inlined -- works via both http:// and file:// protocol
- Three-state pixel painting canvas with click-to-cycle interaction and real-time QR corruption feedback
- Function pattern masking system protecting QR structural elements (finders, timing, alignment) from artistic corruption
- Web Worker hash optimization engine running non-blocking search with live progress and top 5 result tracking across runs
- Complete export suite with PNG download at 1:1 module resolution, clipboard URL copy with file:// fallback, and color-coded error visualization overlay

**Stats:**
- 1 file (index.html), 15,161 lines of HTML/CSS/JS
- 4 phases, 8 plans, 59 commits
- 4 days from start to ship (2026-02-06 -> 2026-02-09)

**Git range:** `3ddf08b` -> `97d10be`

---

## v1.1 UX Overhaul & Optimization (In Progress: 2026-02-10)

**Goal:** Improve the painting workflow (overlay on QR, drag painting, state persistence), add safety guards (hash fragment rejection, version change warnings), and speed up optimization (multi-worker, auto-stop).

**Phases:** 5-9 (8 plans estimated)

**Scope:**
- Configuration constants: max version as named constant, expanded range to Version 10
- State persistence: localStorage save/restore for full app state
- Painting overhaul: overlay on QR preview, protected area visualization, legend-based drag painting
- Generation safety: hash fragment rejection, version change guards, pattern preservation
- Optimization upgrades: auto-stop on perfect results, multi-worker parallelization

**Requirements:** 16 (CONF-01/02, PERS-01/02, PAINT-01-05, SAFE-01-05, OPT-01/02)

---
