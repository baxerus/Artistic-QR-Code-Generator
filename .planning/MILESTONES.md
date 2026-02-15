# Project Milestones: Artistic QR Code Generator

## v1.1 UX Overhaul & Optimization (Shipped: 2026-02-14)

**Delivered:** Major UX improvements with interactive painting overlay, state persistence, generation safety guards, and multi-worker optimization with auto-stop.

**Phases completed:** 5-9 (8 plans total)

**Key accomplishments:**
- Extended QR version range to Version 10 (57×57) with configurable MAX_QR_VERSION constant for easy adjustment
- Full state persistence via localStorage — URL, version, and painted pattern survive page reloads with automatic QR regeneration
- Unified QR canvas with painted overlay rendering, protected area visualization (red dashed borders), and hover grid lines
- Interactive legend-based color selection with drag painting (left-click) and opposite color (right-click)
- Generation safety guards: hash fragment URL rejection, pattern-aware version changes with confirmation dialogs
- Multi-worker parallel optimization (2-8 workers based on CPU) with auto-stop when 5 perfect results found

**Stats:**
- 1 file (index.html), 16,353 lines of HTML/CSS/JS (+1,192 lines from v1.0)
- 5 phases, 8 plans, 16 commits
- 5 days from start to ship (2026-02-10 → 2026-02-14)

**Git range:** `c104790` → `7cb2d22`

---

## v1.0 MVP (Shipped: 2026-02-09)

**Delivered:** Single-file HTML tool for creating artistic QR codes by painting pixel patterns and searching for URL hash fragments that naturally align with the art.

**Phases completed:** 1-4 (8 plans total)

**Key accomplishments:**
- Single-file HTML architecture (508KB, 15,161 lines) with qrcodejs and jsQR libraries inlined — works via both http:// and file:// protocol
- Three-state pixel painting canvas with click-to-cycle interaction and real-time QR corruption feedback
- Function pattern masking system protecting QR structural elements (finders, timing, alignment) from artistic corruption
- Web Worker hash optimization engine running non-blocking search with live progress and top 5 result tracking across runs
- Complete export suite with PNG download at 1:1 module resolution, clipboard URL copy with file:// fallback, and color-coded error visualization overlay

**Stats:**
- 1 file (index.html), 15,161 lines of HTML/CSS/JS
- 4 phases, 8 plans, 59 commits
- 4 days from start to ship (2026-02-06 → 2026-02-09)

**Git range:** `3ddf08b` → `97d10be`

---

**Total Project Stats:**
- 4 milestones shipped
- 11 phases, 20 plans
- 10 days total development (2026-02-06 → 2026-02-15)

## v1.2 Visual Consistency & Result Inspection (Shipped: 2026-02-15)

**Delivered:** Visual polish for paint canvas protected areas and result card hover interactions for error visualization inspection.

**Phases completed:** 10 (3 plans)

**Key accomplishments:**
- Protected area borders on paint canvas use blue instead of red (avoiding confusion with error colors)
- Scaled result cards show plain QR by default (cleaner default view)
- Hover over result cards reveals error visualization with blue dashed borders

**Stats:**
- 1 file (index.html) modified
- 1 phase, 3 plans
- 2 days (2026-02-14 → 2026-02-15)

---


## v1.3 Hash Capacity Optimization (Shipped: 2026-02-15)

**Delivered:** Dynamic hash length calculation that fills available QR capacity, giving the optimization search more data area variation for better painted pattern alignment.

**Phases completed:** 11 (1 plan, 3 tasks)

**Key accomplishments:**
- Dynamic hash length calculation based on QR version capacity minus URL byte length
- Renamed HASH_LENGTH to MIN_HASH_LENGTH to clarify it's a floor, not fixed value
- Worker integration receives hashLength dynamically via message protocol
- Longer hashes create more variation in QR data regions for better optimization candidates

**Stats:**
- 1 file (index.html) modified
- 1 phase, 1 plan, 4 commits
- Same day (2026-02-15)

**Git range:** `5d1d771` → `8c0e210`

---

