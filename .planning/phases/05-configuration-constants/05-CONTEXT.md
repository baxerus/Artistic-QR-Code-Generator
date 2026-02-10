# Phase 5: Configuration Constants - Context

**Gathered:** 2026-02-10
**Status:** Ready for planning

<domain>
## Phase Boundary

Extract QR version limits into a single governing constant and expand the supported range from versions 2-8 to versions 2-10. All version-dependent logic (dropdown range, validation, min-version calculation) derives from this constant. No new features — pure infrastructure refactor with a range bump.

</domain>

<decisions>
## Implementation Decisions

### Default version selection
- Default version on fresh load is always Version 2 (smallest)
- No session persistence for this phase (Phase 6 handles that)
- Version dropdown is locked/disabled until a URL is entered
- Once a URL is entered, only show versions that can encode that URL (hide versions that are too small)

### Version auto-adjustment UX
- When a URL requires a higher version than currently selected, auto-bump to the exact minimum required version
- Provide visual feedback on auto-bump: a short green glow animation on the dropdown (same style as the updated-results animation)
- When user manually selects a version higher than needed, allow it silently (they may want more painting room)
- Version only auto-increases, never auto-decreases — if a URL gets shorter, version stays where it is; user manually lowers if desired

### Claude's Discretion
- Version dropdown display format (labels, whether to show grid dimensions)
- Exact constant naming and file location
- How the glow animation is implemented (CSS transition details)

</decisions>

<specifics>
## Specific Ideas

- Green glow animation for version auto-bump should match the existing animation used for updated results in the app
- Hiding invalid versions rather than graying them out keeps the dropdown clean

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 05-configuration-constants*
*Context gathered: 2026-02-10*
