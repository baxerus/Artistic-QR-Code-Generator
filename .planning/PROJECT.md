# Artistic QR Code Generator

## What This Is

A single-file HTML tool (508KB, 15,161 lines) for creating artistic QR codes by embedding custom pixel patterns. Users paint three-state patterns onto a QR grid, then the tool runs a Web Worker-based search through URL hash fragments to find encodings that naturally align with the art, minimizing pixel conflicts while preserving 100% of the painted pattern.

## Core Value

The painted pattern is sacred and never changes. The tool finds URL variants that naturally align with the art, not the other way around. Even if it means the QR code won't scan, the pattern survives at 100%.

## Requirements

### Validated

- URL input field with validation — v1
- Automatic minimum QR version calculation based on URL length + error correction level H — v1
- QR version selector (Version 2-8) showing valid range for entered URL — v1
- Painting canvas with grid matching selected QR version dimensions — v1
- Three-state pixel painting: white (locked), black (locked), unset (QR decides) — v1
- Click to cycle pixel states: unset → white → black → unset — v1
- Visual distinction between all three pixel states — v1
- Clear/reset button to wipe painted pattern — v1
- Generate button to start optimization — v1
- Hash fragment optimization algorithm that tries different #fragments — v1
- Configurable max search time in seconds — v1
- Manual stop button during search — v1
- Live progress display: attempts counter, elapsed time — v1
- Live preview of top 5 current best QR codes during search — v1
- Live error count display for top 5 results — v1
- Pixel diff error measurement (painted pixel vs QR module alignment) — v1
- Top 5 results display ranked by decoder error count — v1
- Download QR code as image (PNG) for each result — v1
- Copy final URL (with hash fragment) for each result — v1
- Error visualization overlay showing: locked pattern pixels, QR data pixels, error/conflict pixels — v1
- QR error correction level H hardcoded — v1
- All JavaScript libraries inlined in HTML (no external files) — v1
- All CSS inlined in HTML — v1
- Works when served by web server — v1
- Works when opened via file:// protocol — v1

### Active

**v1.1 — UX Overhaul & Optimization**

- [ ] State persistence via localStorage (URL, painted pattern, settings survive reload)
- [ ] Paint pattern overlayed directly on QR code preview
- [ ] Protected QR areas (finders, timing, alignment) visually marked as non-editable
- [ ] Legend-based color selection replacing click-to-cycle (black/white/unset)
- [ ] Click and drag painting with selected color; right-click paints opposite (white↔black)
- [ ] Reject URLs ending in hash fragments
- [ ] Smart generation: pattern exists → only button triggers generation, not dropdown
- [ ] Version change with pattern: same size preserves pattern, size change warns user
- [ ] Auto-stop optimization when 5 results with 0 errors found
- [ ] Multiple Web Workers for parallel optimization search
- [ ] QR version range expanded to Version 10 (57×57)
- [ ] Max QR version extracted as a constant for easy future adjustment

### Out of Scope

- QR error correction levels other than H — H provides maximum error tolerance needed for artistic patterns
- Brush tools or advanced drawing modes (fill, shapes) — legend selection + drag painting is sufficient
- Undo/redo — clear/reset is sufficient for initial version
- Saving/loading patterns to file — localStorage persistence is sufficient for now
- Mobile touch optimization — desktop-first, modern browsers only
- SVG export — PNG is sufficient for initial version
- Batch processing multiple URLs — single URL workflow only
- Custom QR version ranges beyond 2-10 — sufficient range for most artistic use cases
- Server-side processing — everything client-side in browser

## Context

**Current Milestone: v1.1 UX Overhaul & Optimization**
Goal: Improve the painting workflow (overlay on QR, drag painting, state persistence), add safety guards (hash fragment rejection, version change warnings), and speed up optimization (multi-worker, auto-stop).

**Current State:**
Shipped v1 MVP with 15,161 lines of HTML/CSS/JS in a single file. Tech stack: vanilla HTML5/CSS3/ES6, qrcodejs (inlined), jsQR (inlined), Web Workers via Blob URL. All 31 v1 requirements satisfied across 4 phases, 8 plans, 59 commits over 4 days.

**Artistic QR Code Use Case:**
The tool is designed for creating artistic/branded QR codes where visual aesthetics matter as much as (or more than) scannability. Users want to embed logos, patterns, or designs into QR codes.

**Algorithm Insight:**
Different URL contents produce different QR code pixel patterns. By varying the hash fragment (#abc123), we search through different natural QR encodings to find ones that happen to align well with the painted pattern. The pattern is fixed; we're finding the lucky URL variant. Hash charset is a-z0-9, length 6 (~2.2B combinations).

**Error Measurement:**
"Error count" means pixel diff — the number of painted pixels that conflict with the QR module states. The algorithm generates a valid QR for URL+hash, overlays the locked pattern, then counts conflicts. Lower conflicts = better natural alignment. Results are sorted by decodability first, then by pixel diff ascending.

**Top 5 Results Rationale:**
The mathematically best result (fewest errors) might not be the aesthetically best. Showing top 5 lets users choose based on visual harmony, not just error metrics. Results accumulate across multiple optimization runs.

**v1.1 User Feedback (from testing v1):**
- Reloading page loses all state (URL, pattern) — frustrating during iterative testing
- Painting on a separate canvas from the QR preview makes alignment hard to judge
- Click-to-cycle is slow for painting larger patterns — drag painting needed
- Protected QR areas (finders, timing) not visually obvious — users accidentally paint over them
- Entering a URL with a hash fragment causes silent issues
- Changing version dropdown while pattern exists silently destroys the pattern
- Optimization runs to timeout even when perfect results (0 errors) already found
- ~75 attempts/second feels slow — room for parallelization

## Constraints

- **Single file**: All code (HTML, CSS, JS, libraries) must be in one .html file
- **No external dependencies**: No CDN links, no separate .js/.css files, no external images
- **Protocol compatibility**: Must work both via http:// (web server) and file:// (local filesystem)
- **Modern browsers**: Target recent Chrome, Firefox, Safari — no IE/legacy support needed
- **Error correction**: Must use QR error correction level H (highest, ~30% error tolerance)
- **QR version range**: Version 2 (25x25) to Version 10 (57x57) — max version is a constant for easy adjustment

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Hash fragments for URL variants | Clean, doesn't affect destination URL, easy to generate | ✓ Good — core algorithm works well |
| Three-state pixels (white/black/unset) | Pattern pixels are locked, others free for QR algorithm | ✓ Good — intuitive UX |
| Top 5 results instead of single best | Aesthetic choice matters beyond pure error metrics | ✓ Good — users can pick visually best |
| Error correction level H | Maximum error tolerance for artistic corruption | ✓ Good — essential for pattern survival |
| Everything in single HTML file | Simplicity, portability, works via file:// | ✓ Good — zero setup, easy sharing |
| Modern browsers only | Simplifies development, avoid legacy compatibility | ✓ Good — no compatibility issues |
| Pixel diff instead of Reed-Solomon errors | jsQR doesn't expose RS error counts; pixel diff is practical proxy | ✓ Good — effective alignment metric |
| Web Worker via Blob URL | Non-blocking search in single-file architecture | ✓ Good — UI stays responsive |
| Batched worker loop (100 + setTimeout) | Allows stop messages to interrupt search | ✓ Good — clean stop behavior |
| 1:1 module resolution PNG export | No scaling artifacts, users resize as needed | ✓ Good — cleanest output |
| Click-to-expand error overlay | Full-width preview better than toggle button | ✓ Good — clear visualization |
| Paint locking during optimization | Prevents stale results from pattern changes | ✓ Good — data integrity |

| Max QR version as constant | Frequently adjusted during testing, needs to be easy to change | — Pending |
| Legend-based painting replacing click-to-cycle | Drag painting requires pre-selected color, legend is natural UI | — Pending |
| Right-click paints opposite color | Fast workflow: left=selected, right=opposite, no legend switching | — Pending |
| Multi-worker optimization | Single worker bottleneck at ~75 attempts/sec | — Pending |

---
*Last updated: 2026-02-10 after v1.1 milestone start*
