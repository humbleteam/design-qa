# Changelog

## [1.2.0] - 2026-08-11

- Verdict logic now depends on coverage, closing a hole that let a fail-closed
  gate return PASS without checking anything: an artifact supporting none of
  the six categories produced no issues, and "zero issues across every
  category that had evidence" was vacuously true. Two verdicts added -
  NOT VERIFIABLE (no category produced evidence) and PASS WITH GAPS (clean run,
  some categories not verifiable) - and PASS now requires all six categories to
  have been checked.
- Every verdict line carries `Coverage: <n>/6 categories verifiable`, whatever
  the verdict.
- Edge case: a screenshot at an unstated scale with no legible text and no
  sampleable fill supports no category - the empty fix list means nothing was
  checked, not that nothing was wrong.
- `references/qa-checklist.md` gains a coverage reference table mapping
  coverage against findings to the five verdicts.

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
