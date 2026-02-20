---
phase: quick/11-propose-settings-save-load-reset-menu-wi
plan: 11
type: propose
wave: 1
depends_on: []
files_modified: [".planning/quick/11-propose-settings-save-load-reset-menu-wi/11-PROPOSAL.md"]
autonomous: true
requirements: [QUICK-11]

must_haves:
  truths:
    - "Proposal defines settings save/load/reset UX and scope"
    - "Proposal enumerates JSON schema fields and localStorage keys affected"
    - "Proposal includes confirmation/reset behavior and error handling"
  artifacts:
    - path: ".planning/quick/11-propose-settings-save-load-reset-menu-wi/11-PROPOSAL.md"
      provides: "Settings save/load/reset proposal"
      min_lines: 30
  key_links:
    - from: "Proposal UI spec"
      to: "Header menu with disk icon + dropdown options"
      via: "interaction notes"
      pattern: "Save settings to file|Load settings from file|Reset settings"
---

<objective>
Create a proposal for a lightweight settings menu to export/import JSON state and reset with confirmation.

Purpose: Align on UX, JSON schema, and reset behavior before implementation.
Output: Proposal document with UI spec, schema, behavior, and open questions.
</objective>

<execution_context>
@/home/node/.config/opencode/get-shit-done/workflows/execute-plan.md
@/home/node/.config/opencode/get-shit-done/templates/summary.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/STATE.md
@index.html
</context>

<tasks>

<task type="auto">
  <name>Task 1: Draft proposal for settings save/load/reset</name>
  <files>.planning/quick/11-propose-settings-save-load-reset-menu-wi/11-PROPOSAL.md</files>
  <action>
    1. Summarize the goal and user-facing UX: header button with disk icon, dropdown options for Save/Load/Reset, and reset confirmation dialog.
    2. Define JSON schema fields and localStorage coverage (include URL, version, rotation, random rotation, selected color, painted pattern data) and how to handle missing/extra fields.
    3. Specify import/export flow (file type, filename, error handling) and reset flow (confirmation copy, blocked while search running).
    4. List required implementation touch points in index.html (UI location near headline, handlers, storage keys, preview re-sync).
    5. Add open questions/assumptions for approval.
  </action>
  <verify>
    Proposal document includes UX spec, schema, behavior, and open questions.
  </verify>
  <done>Proposal file is written and ready for discussion and approval.</done>
</task>

</tasks>

<verification>
- [ ] Proposal captures UI, schema, flows, and reset confirmation
</verification>

<success_criteria>
- Proposal is ready for discussion and approval before implementation
</success_criteria>

<output>
After completion, create `.planning/quick/11-propose-settings-save-load-reset-menu-wi/11-SUMMARY.md`
</output>
