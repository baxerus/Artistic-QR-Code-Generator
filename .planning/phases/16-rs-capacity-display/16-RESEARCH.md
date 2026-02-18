# Phase 16: RS Capacity Display - Research

**Researched:** 2026-02-18
**Domain:** QR Reed-Solomon correction capacity display (jsQR VERSIONS table)
**Confidence:** HIGH

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
## Implementation Decisions

### Placement and label
- Replace existing "Corrections: X" text with "Corrections: X of Y" wherever that text appears (result card and any other views that show it).
- Keep the label as "Corrections".

### Text format and ordering
- Exact inline format: "Corrections: X of Y".
- Use integer values only (no decimals).
- Corrections appears before Pixels in the metrics order.

### Hover hint behavior
- Keep the existing hover hint and extend it to explain what the max value (Y) represents.
- Show the hover hint even when placeholders are shown.

### Edge/unknown states
- If X is missing: show placeholder (e.g., "Corrections: — of Y").
- If Y is unavailable or computes to 0/negative: treat as unavailable and show placeholder ("Corrections: X of —").

### Visual emphasis
- No warning cues or icons; keep the existing styling with no visual emphasis changes.

### Claude's Discretion
- Placeholder glyph choice and exact hover hint copy, as long as it explains the max value.

### Deferred Ideas (OUT OF SCOPE)
## Deferred Ideas

None — discussion stayed within phase scope.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| RSCAP-01 | Result card shows correction count as "X of Y" where Y is max capacity for QR version at level H | UI integration point in `updateTopResults()` plus placeholder/tooltip rules in this research |
| RSCAP-02 | Max capacity calculated from jsQR VERSIONS table using formula: `sum(ecBlocks × ecCodewordsPerBlock) / 2` | jsQR VERSIONS structure and capacity formula documented here |
</phase_requirements>

## Summary

The codebase already exposes the two inputs needed for the display: current correction count (`result.rsCorrections`) and the QR version (`currentVersion`). The max capacity (Y) is derived from the inlined jsQR `VERSIONS` table (module 10 in the webpack bundle) using the required formula `sum(ecBlocks × ecCodewordsPerBlock) / 2` at error correction level H. This is a pure UI change centered in the result card rendering (`updateTopResults`), with a small helper to compute capacity.

The critical risk is a wrong capacity calculation or lookup (off-by-one version index, wrong EC level index). The H-level index in the jsQR `errorCorrectionLevels` array is 3. The `ecBlocks` array can include multiple block groups and must be summed before halving the EC codewords per block. Placeholders need to follow the locked decisions for missing X/Y while preserving the hover hint and the existing styling.

**Primary recommendation:** Implement a `getMaxCorrectionsCapacity(version)` helper using `jsQR`’s `VERSIONS` data and update the result card’s correction label to `Corrections: X of Y` with placeholder rules and extended tooltip copy.

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Inlined jsQR bundle | embedded in `index.html` | VERSIONS table + RS decode count | Already shipped in page, authoritative QR metadata |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| None | N/A | N/A | No new dependencies required |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| jsQR VERSIONS table | Hard-coded capacity map | Increases duplication, risk of drift |

**Installation:**
```bash
# No new packages required
```

## Architecture Patterns

### Recommended Project Structure
```
index.html               # Single-file app, inlined libraries and UI logic
```

### Pattern 1: UI-derived metrics in result cards
**What:** Compute derived metrics in `updateTopResults()` and render them into the result card DOM nodes.
**When to use:** Display-only metrics derived from existing result objects and current app state (version).
**Example:**
```javascript
// Source: /workspace/index.html (updateTopResults)
const rsDisplay = result.rsCorrections === Infinity ? "-" : result.rsCorrections;
correctionSpan.innerHTML = `Corrections: <b>${rsDisplay}</b>`;
```

### Pattern 2: QR metadata from jsQR VERSIONS
**What:** Read `errorCorrectionLevels` metadata from jsQR’s `VERSIONS` table.
**When to use:** Need QR structural data (blocks, EC codewords) for a version.
**Example:**
```javascript
// Source: /workspace/index.html (jsQR VERSIONS table)
exports.VERSIONS = [
  {
    versionNumber: 1,
    errorCorrectionLevels: [
      { ecCodewordsPerBlock: 7, ecBlocks: [{ numBlocks: 1, dataCodewordsPerBlock: 19 }] },
      { ecCodewordsPerBlock: 10, ecBlocks: [{ numBlocks: 1, dataCodewordsPerBlock: 16 }] },
      { ecCodewordsPerBlock: 13, ecBlocks: [{ numBlocks: 1, dataCodewordsPerBlock: 13 }] },
      { ecCodewordsPerBlock: 17, ecBlocks: [{ numBlocks: 1, dataCodewordsPerBlock: 9 }] }
    ]
  }
];
```

### Anti-Patterns to Avoid
- **Hard-coded RS capacity table:** duplicates source-of-truth in jsQR and risks drift.
- **Using byte capacity (CAPACITIES_H_BYTE) for RS capacity:** that table is for payload size, not RS correction capacity.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| RS capacity data | New static tables | jsQR `VERSIONS` table | Already embedded, authoritative, complete |

**Key insight:** The RS capacity formula requires only jsQR’s EC metadata, so a custom dataset is unnecessary and riskier.

## Common Pitfalls

### Pitfall 1: Wrong EC level index
**What goes wrong:** Using the wrong `errorCorrectionLevels` index (L/M/Q instead of H) yields incorrect capacity.
**Why it happens:** The array is positional with no labels in the bundled data.
**How to avoid:** Use index 3 for H (L=0, M=1, Q=2, H=3).
**Warning signs:** Y values that mismatch QR spec tables or decrease unexpectedly across versions.

### Pitfall 2: Off-by-one version lookup
**What goes wrong:** Using `VERSIONS[version]` instead of `VERSIONS[version - 1]`.
**Why it happens:** Version numbers are 1-based while arrays are 0-based.
**How to avoid:** Always subtract 1 before indexing.
**Warning signs:** Y values from the wrong version; mismatch with module count.

### Pitfall 3: Not summing multiple block groups
**What goes wrong:** Only the first `ecBlocks` group is used, undercounting capacity.
**Why it happens:** Some versions have two block groups.
**How to avoid:** Sum `numBlocks` across all groups before multiplying by `ecCodewordsPerBlock`.
**Warning signs:** Capacity for higher versions looks too low compared to spec.

### Pitfall 4: Skipping the divide-by-2 rule
**What goes wrong:** Using total EC codewords as errors instead of half.
**Why it happens:** RS correction rule (2t parity) is easy to miss.
**How to avoid:** Apply `totalEcCodewords / 2` explicitly.
**Warning signs:** Y values double expected; “0 of Y” looks too large.

## Code Examples

Verified patterns from codebase:

### Extract correction count for display
```javascript
// Source: /workspace/index.html
const rsDisplay = result.rsCorrections === Infinity ? "-" : result.rsCorrections;
correctionSpan.innerHTML = `Corrections: <b>${rsDisplay}</b>`;
```

### Read EC metadata from jsQR VERSIONS
```javascript
// Source: /workspace/index.html (jsQR VERSIONS table)
const version = VERSIONS[versionNumber - 1];
const ecInfo = version.errorCorrectionLevels[3]; // H level
// ecInfo.ecCodewordsPerBlock, ecInfo.ecBlocks
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Show only current corrections | Show current corrections plus capacity | Phase 16 | Users see headroom before scan failure |

**Deprecated/outdated:**
- Using byte payload capacity as a proxy for RS correction capacity.

## Open Questions

1. **Where to expose VERSIONS for UI access?**
   - What we know: jsQR bundles `exports.VERSIONS` internally in the inlined script.
   - What’s unclear: Best low-risk hook to access it from the app scope.
   - Recommendation: Add a minimal export after jsQR IIFE (e.g., `window.__JSQR_VERSIONS__ = jsQR.__VERSIONS__` if available) or add a small helper that reads a shared variable created during bundling. Validate in code review.

## Sources

### Primary (HIGH confidence)
- `/workspace/index.html` - jsQR `VERSIONS` table and result card rendering

### Secondary (MEDIUM confidence)
- `/workspace/.planning/research/SUMMARY.md` - prior v1.5 research synthesis

### Tertiary (LOW confidence)
- None

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - no external dependencies, embedded jsQR data
- Architecture: HIGH - single-file UI, clear integration point
- Pitfalls: HIGH - direct inspection of EC data structure and rendering code

**Research date:** 2026-02-18
**Valid until:** 2026-03-20
