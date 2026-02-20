---
phase: quick/13-fix-eslint-error-identifier-loadselected
plan: 01
type: execute
wave: 1
depends_on: []
files_modified: [index.html]
autonomous: true
requirements: []

must_haves:
  truths:
    - "ESLint error 'Identifier loadSelectedColor has already been declared' is resolved"
  artifacts:
    - path: "index.html"
      provides: "Single loadSelectedColor function declaration"
      contains: "function loadSelectedColor"
      exactly_one: true
---

<objective>
Fix ESLint error: Identifier 'loadSelectedColor' has already been declared in index.html

Purpose: Remove duplicate function declaration that causes ESLint error
Output: Clean index.html with single loadSelectedColor function
</objective>

<context>
@index.html (lines 13855-13868 and 14740-14750)

Two identical function declarations found:

- First (line 13858): Has validation for valid colors ["black", "white", "unset"]
- Second (line 14743): No validation, returns raw localStorage value
  </context>

<tasks>

<task type="auto">
  <name>Remove duplicate loadSelectedColor function</name>
  <files>index.html</files>
  <action>
    Remove the duplicate function declaration at lines 14740-14750 (the comment block and function).
    Keep the first occurrence at lines 13855-13868 as it has better validation (checks color is one of ["black", "white", "unset"]).
    
    Delete these lines in index.html:
    ```
          /**
           * Load selected color from localStorage
           */
          function loadSelectedColor() {
            try {
              return localStorage.getItem(STORAGE_KEYS.COLOR) || "unset";
            } catch (err) {
              console.warn("Failed to load color from localStorage:", err);
              return "unset";
            }
          }
    ```
  </action>
  <verify>grep -c "function loadSelectedColor" index.html returns 1</verify>
  <done>ESLint error "Identifier 'loadSelectedColor' has already been declared" is resolved</done>
</task>

</tasks>

<verification>
- Run ESLint on index.html and confirm no duplicate identifier errors
- grep "loadSelectedColor" shows only one function declaration and two call sites
</verification>

<success_criteria>
ESLint passes with no errors about duplicate 'loadSelectedColor' declaration
</success_criteria>

<output>
After completion, create .planning/quick/13-fix-eslint-error-identifier-loadselected/13-01-SUMMARY.md
</output>
