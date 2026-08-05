# Changelog

## [1.1.0] - 2026-08-05

- FAQ: answer where this gate sits next to axe-core and Lighthouse - scanners
  stay in CI as the floor, this gate covers the three categories a rule engine
  cannot report (states never built, tab order that reads wrong, error copy
  that omits the next step).

## [1.0.0] - 2026-07-12

- Initial release: six-category QA gate (component states, contrast, touch
  targets, responsive, keyboard, copy) with a pass/fail table and a
  severity-tagged Before/After fix list.
- Fail-closed verdict logic - any P0 issue forces a FAIL; categories with no
  supporting evidence are reported as not verifiable, never rounded to pass.
- `references/qa-checklist.md` - the full gate as a standalone copy-paste
  markdown checklist, including the WCAG relative-luminance contrast formula.
