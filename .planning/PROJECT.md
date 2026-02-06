# Artistic QR Code Generator

## What This Is

A single-page HTML tool for creating artistic QR codes by embedding custom pixel patterns. Users paint patterns that are locked in place, then the tool searches for URL hash fragments whose natural QR encoding aligns well with the art, minimizing decoder errors while preserving 100% of the painted pattern.

## Core Value

The painted pattern is sacred and never changes. The tool finds URL variants that naturally align with the art, not the other way around. Even if it means the QR code won't scan, the pattern survives at 100%.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] URL input field with validation
- [ ] Automatic minimum QR version calculation based on URL length + error correction level H
- [ ] QR version selector (Version 2-8) showing valid range for entered URL
- [ ] Painting canvas with grid matching selected QR version dimensions
- [ ] Three-state pixel painting: white (locked), black (locked), unset (QR decides)
- [ ] Click to cycle pixel states: unset → white → black → unset
- [ ] Visual distinction between all three pixel states
- [ ] Clear/reset button to wipe painted pattern
- [ ] Generate button to start optimization
- [ ] Hash fragment optimization algorithm that tries different #fragments
- [ ] Configurable max search time in seconds
- [ ] Manual stop button during search
- [ ] Live progress display: attempts counter, elapsed time
- [ ] Live preview of current best QR code during search
- [ ] Live error count display for current best result
- [ ] QR decoder error counting (Reed-Solomon errors when decoding corrupted QR)
- [ ] Top 5 results display ranked by decoder error count
- [ ] Download QR code as image (PNG) for each result
- [ ] Copy final URL (with hash fragment) for each result
- [ ] Error visualization overlay showing: locked pattern pixels, QR data pixels, error/conflict pixels
- [ ] QR error correction level H hardcoded
- [ ] All JavaScript libraries inlined in HTML (no external files)
- [ ] All CSS inlined in HTML
- [ ] Works when served by web server
- [ ] Works when opened via file:// protocol

### Out of Scope

- QR error correction levels other than H — H provides maximum error tolerance needed for artistic patterns
- Brush tools or drawing modes — simple click-to-cycle is sufficient for pixel art
- Undo/redo — clear/reset is sufficient for initial version
- Saving/loading patterns — focus on generation workflow first
- Mobile touch optimization — desktop-first, modern browsers only
- SVG export — PNG is sufficient for initial version
- Batch processing multiple URLs — single URL workflow only
- Custom QR version ranges beyond 2-8 — sufficient range for most artistic use cases
- Server-side processing — everything client-side in browser

## Context

**Artistic QR Code Use Case:**
The tool is designed for creating artistic/branded QR codes where visual aesthetics matter as much as (or more than) scannability. Users want to embed logos, patterns, or designs into QR codes.

**Algorithm Insight:**
Different URL contents produce different QR code pixel patterns. By varying the hash fragment (#abc123), we search through different natural QR encodings to find ones that happen to align well with the painted pattern. The pattern is fixed; we're finding the lucky URL variant.

**Error Measurement:**
"Error count" means decoder errors when a QR scanner attempts to read the pattern-corrupted code. The algorithm generates a valid QR for URL+hash, overlays the locked pattern (corrupting it), then measures how many errors the decoder reports. Lower errors = better natural alignment.

**Top 5 Results Rationale:**
The mathematically best result (fewest errors) might not be the aesthetically best. Showing top 5 lets users choose based on visual harmony, not just error metrics.

## Constraints

- **Single file**: All code (HTML, CSS, JS, libraries) must be in one .html file
- **No external dependencies**: No CDN links, no separate .js/.css files, no external images
- **Protocol compatibility**: Must work both via http:// (web server) and file:// (local filesystem)
- **Modern browsers**: Target recent Chrome, Firefox, Safari — no IE/legacy support needed
- **Error correction**: Must use QR error correction level H (highest, ~30% error tolerance)
- **QR version range**: Version 2 (25×25) to Version 8 (49×49) — configurable based on URL length

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Hash fragments for URL variants | Clean, doesn't affect destination URL, easy to generate | — Pending |
| Three-state pixels (white/black/unset) | Pattern pixels are locked, others free for QR algorithm | — Pending |
| Top 5 results instead of single best | Aesthetic choice matters beyond pure error metrics | — Pending |
| Error correction level H | Maximum error tolerance for artistic corruption | — Pending |
| Everything in single HTML file | Simplicity, portability, works via file:// | — Pending |
| Modern browsers only | Simplifies development, avoid legacy compatibility | — Pending |

---
*Last updated: 2026-02-06 after initialization*
