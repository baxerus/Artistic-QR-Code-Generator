---
phase: quick/2-instead-of-top-5-results-show-the-top-10
plan: 2
type: execute
wave: 1
depends_on: []
files_modified: [index.html]
autonomous: true

must_haves:
  truths:
    - "Number of top results is configurable via a constant"
    - "Constant is used in both main thread and worker code"
    - "Comments no longer contain hardcoded numbers (e.g., 'Top results' not 'Top 5 results')"
    - "The constant is set to 10 instead of 5"
  artifacts:
    - path: "index.html"
      provides: "TOP_RESULTS_COUNT constant defined and used everywhere"
      min_lines: 1
  key_links:
    - from: "Main thread (searchConfig)"
      to: "Worker code (searchConfig.topResultsCount)"
      via: "postMessage when starting optimization"
---

<objective>
Change the number of top results displayed from 5 to 10, making it configurable via a constant that is passed to the worker code. Remove hardcoded numbers from comments.

Purpose: Make the number of top results easily changeable in the future and ensure it's not hardcoded in multiple places.
Output: Updated index.html with TOP_RESULTS_COUNT constant.
</objective>

<execution_context>
@/home/node/.config/opencode/get-shit-done/workflows/execute-plan.md
@/home/node/.config/opencode/get-shit-done/templates/summary.md
</execution_context>

<context>
@.planning/STATE.md
@index.html
</context>

<tasks>

<task type="auto">
  <name>Task 1: Add TOP_RESULTS_COUNT constant and pass to worker</name>
  <files>index.html</files>
  <action>
    1. Near the top of the main script section (around line 13818 where MAX_QR_VERSION is defined), add a constant: `const TOP_RESULTS_COUNT = 10;`
    
    2. In the `startOptimization()` function (around line 15661), when creating the message sent to workers, add: `topResultsCount: TOP_RESULTS_COUNT` to the message object
    
    3. In the worker code (inside the Blob), where `searchConfig` is received (around line 15469), ensure the worker reads `searchConfig.topResultsCount`
    
    4. In the worker's `updateTopResults()` function (around line 15352), change the hardcoded `5` to use `searchConfig.topResultsCount`:
       - Change: `if (topResults.length > 5)` 
       - To: `if (topResults.length > searchConfig.topResultsCount)`
  </action>
  <verify>grep -n "TOP_RESULTS_COUNT" index.html returns the constant definition and usage</verify>
  <done>Constant is defined at module level, passed to worker in message, and used in worker's slice() call</done>
</task>

<task type="auto">
  <name>Task 2: Update HTML comment and JSDoc comments</name>
  <files>index.html</files>
  <action>
    1. Change HTML comment on line 916 from:
       `<!-- Top 5 results row -->`
       To:
       `<!-- Top results row -->`
    
    2. Change JSDoc comment on line 16093-16095 from:
       `/**
        * Update top 5 results display
        */`
       To:
       `/**
        * Update top results display
        */`
  </action>
  <verify>grep -n "Top 5" index.html returns no matches</verify>
  <done>No hardcoded "Top 5" references remain in comments; only "Top results" used</done>
</task>

<task type="auto">
  <name>Task 3: Update CSS class name (if exists) and verify layout</name>
  <files>index.html</files>
  <action>
    1. Search for any CSS class names or IDs that reference "top-5" or "top5" and update to "top-results" if appropriate (the HTML element already uses id="top-results" so this may not be necessary)
    
    2. Check if any CSS rules reference the "top-results" class and verify the layout can accommodate 10 items instead of 5 (may need to adjust grid/flex properties)
    
    3. If the current layout uses a horizontal row that might not fit 10 items, add CSS to allow wrapping:
       - Find `.top-results` CSS rules
       - Add `flex-wrap: wrap` if needed to accommodate 10 items
  </action>
  <verify>Layout displays 10 results without breaking (test with browser or visual inspection)</verify>
  <done>CSS accommodates 10 items in the top results display, no layout breakage</done>
</task>

</tasks>

<verification>
- [ ] TOP_RESULTS_COUNT = 10 is defined as a constant
- [ ] Constant is passed to worker via searchConfig
- [ ] Worker uses the constant instead of hardcoded 5
- [ ] All "Top 5" references in comments changed to "Top results"
- [ ] Layout can display 10 results
</verification>

<success_criteria>
- Constant TOP_RESULTS_COUNT exists and equals 10
- Worker receives and uses the constant
- No hardcoded "Top 5" in any comments
- Results display shows up to 10 items
</success_criteria>

<output>
After completion, create `.planning/quick/2-instead-of-top-5-results-show-the-top-10/2-SUMMARY.md`
</output>
