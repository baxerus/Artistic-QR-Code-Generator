# Artistic QR Code Generator

## What This Is

A single-file HTML tool (16,353 lines) for creating artistic QR codes by embedding custom pixel patterns. Users paint three-state patterns onto a QR grid using an interactive legend with drag painting, then the tool runs a multi-worker optimization search through URL hash fragments to find encodings that naturally align with the art, minimizing pixel conflicts while preserving 100% of the painted pattern.

## Core Value

The painted pattern is sacred and never changes. The tool finds URL variants that naturally align with the art, not the other way around. Even if it means the QR code won't scan, the pattern survives at 100%.

## Current State

**Shipped:** v1.3 Hash Capacity Optimization (2026-02-15)

**Codebase:** ~16,500 lines of HTML/CSS/JS in a single file. Tech stack: vanilla HTML5/CSS3/ES6, qrcodejs (inlined), jsQR (inlined), multi-worker optimization via Blob URLs.

**Milestones Complete:**

- v1.0 MVP — 4 phases, 8 plans, 59 commits (2026-02-09)
- v1.1 UX Overhaul — 5 phases, 8 plans, 16 commits (2026-02-14)
- v1.2 Visual Consistency — 1 phase, 3 plans (2026-02-15)
- v1.3 Hash Capacity — 1 phase, 1 plan, 4 commits (2026-02-15)

**Total:** 11 phases, 20 plans across 4 milestones

## Requirements

### Validated

**v1.0 MVP:**

- ✓ URL input field with validation — v1.0
- ✓ Automatic minimum QR version calculation — v1.0
- ✓ QR version selector (Version 2-10) — v1.0 (extended v1.1)
- ✓ Three-state pixel painting (white/black/unset) — v1.0
- ✓ Hash fragment optimization algorithm — v1.0
- ✓ Configurable max search time — v1.0
- ✓ Live preview of top 5 results during search — v1.0
- ✓ Pixel diff error measurement — v1.0
- ✓ Download QR as PNG / Copy URL — v1.0
- ✓ Error visualization overlay — v1.0
- ✓ Single-file architecture (works via file://) — v1.0

**v1.1 UX Overhaul & Optimization:**

- ✓ State persistence via localStorage — v1.1
- ✓ Paint pattern overlayed on QR preview — v1.1
- ✓ Protected QR areas visually marked — v1.1
- ✓ Legend-based color selection with drag painting — v1.1
- ✓ Right-click paints opposite color — v1.1
- ✓ Hash fragment URL rejection — v1.1
- ✓ Pattern-aware version change with confirmation — v1.1
- ✓ Auto-stop when 5 perfect results found — v1.1
- ✓ Multi-worker parallel optimization (2-8 workers) — v1.1
- ✓ QR version range expanded to Version 10 — v1.1
- ✓ MAX_QR_VERSION as named constant — v1.1

**v1.2 Visual Consistency & Result Inspection:**

- ✓ Protected area borders use blue (not red) on paint canvas — v1.2
- ✓ Scaled results show plain QR by default (no overlay) — v1.2
- ✓ Scaled results show error visualization + blue dashed borders on hover — v1.2

**v1.3 Hash Capacity Optimization:**

- ✓ Dynamic hash length fills available QR capacity — v1.3
- ✓ MIN_HASH_LENGTH constant (renamed from HASH_LENGTH) — v1.3
- ✓ Hash expands to use remaining bytes after URL — v1.3

### Active

(None — all milestones shipped. Run `/gsd-new-milestone` to plan next features.)

### Out of Scope

- QR error correction levels other than H — H provides maximum error tolerance
- Undo/redo — clear/reset sufficient; complexity not warranted
- Saving/loading patterns to file — localStorage persistence sufficient
- Mobile touch optimization — desktop-first
- SVG export — PNG sufficient
- Batch processing — single URL workflow only
- Server-side processing — everything client-side

## Constraints

- **Single file**: All code in one .html file
- **No external dependencies**: No CDN links, no separate files
- **Protocol compatibility**: Works both via http:// and file://
- **Modern browsers**: Chrome, Firefox, Safari — no IE/legacy
- **Error correction**: QR level H (30% error tolerance)
- **QR version range**: 2-10 (controlled by MAX_QR_VERSION constant)

## Key Decisions

| Decision                               | Rationale                               | Outcome       |
| -------------------------------------- | --------------------------------------- | ------------- |
| Hash fragments for URL variants        | Clean, doesn't affect destination URL   | ✓ Good        |
| Three-state pixels (white/black/unset) | Pattern locked, others free for QR      | ✓ Good        |
| Top 5 results instead of single best   | Aesthetic choice matters                | ✓ Good        |
| Error correction level H               | Maximum error tolerance for art         | ✓ Good        |
| Everything in single HTML file         | Simplicity, portability                 | ✓ Good        |
| Pixel diff instead of RS errors        | Practical proxy, jsQR doesn't expose RS | ✓ Good        |
| Web Workers via Blob URL               | Non-blocking search in single-file      | ✓ Good        |
| MAX_QR_VERSION as constant             | Easy to expand version range            | ✓ Good — v1.1 |
| Legend-based painting                  | Natural UI for drag painting            | ✓ Good — v1.1 |
| Right-click opposite color             | Fast workflow without legend switching  | ✓ Good — v1.1 |
| Multi-worker optimization              | 2-8x throughput improvement             | ✓ Good — v1.1 |
| Auto-stop on 5 perfect results         | Saves time when ideal found             | ✓ Good — v1.1 |
| Pattern-aware version changes          | Prevents accidental pattern loss        | ✓ Good — v1.1 |
| Hash fragment URL rejection            | Prevents silent failures                | ✓ Good — v1.1 |

## Future Considerations

Based on v1.1 development, potential future work:

**UX Enhancements:**

- Save/load patterns to/from file (JSON export)
- Keyboard shortcuts for painting tools
- Mobile/touch support

**Output Enhancements:**

- SVG export format

---

_Last updated: 2026-02-15 after v1.3 shipped_
