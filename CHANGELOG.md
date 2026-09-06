# Changelog

## [1.4.0] - 2026-09-06

- A category whose items disagree now resolves by a stated rule. Every category
  checks several things - nine states in category 1, one row per pair in
  category 2, every target in category 3 - and the output table has one status
  cell per category, with no rule for the mixed case. The two available moves
  were both wrong: PASS on a category where two states were read and seven were
  invisible claims a check that never ran, and N/V erases the items that did
  resolve. Status now rolls up by precedence (FAIL if anything failed, else N/V
  if anything is unchecked, else PASS), and the evidence cell names the split.
- Coverage counts the same way. A category counts toward `<n>/6` only when no
  item in it is left unverifiable, whatever its status cell says, so a screenshot
  run can no longer report full coverage on a category it saw one frame of. This
  is the coverage veto the 1.2.0 verdict rules applied between categories, now
  applied inside one.
- `references/qa-checklist.md` carries the roll-up rule under Coverage reference,
  where the nine boxes of section 1 make the mixed case concrete for a manual run.
- `Coverage: 0/6` and the NOT VERIFIABLE verdict are now stated as different
  tests. Under the stricter count a run can read a few items across several
  categories, finish none of them, and report 0 while having found something
  real, so NOT VERIFIABLE is now reserved for the run where no item anywhere
  resolved. Without that split, a P1 found in a half-read category would have
  been reported as a gate that never ran.
- The README's screenshot usage line said states and keyboard come back "mostly"
  not verifiable, which was the ambiguity in one word. Under the roll-up rule
  they come back not verifiable, and the How-it-works list says why.

## [1.3.0] - 2026-08-18

- Every component-state failure now has a severity bucket. Step 2 checks nine
  states and mandates a FAIL when an interactive component has no rule for one
  of them, while Step 3 named only two: focus (P0) and loading/empty/error
  (P1). Hover, active, disabled and long content matched nothing, so a stated
  FAIL had to be dropped from the fix list or promoted to P0 by citing a
  success criterion that does not cover it. Added a state-to-severity map and a
  P1 default for anything it does not name, with promotion to P0 ruled out.
- `references/qa-checklist.md` carries the same map under Severity reference.
- Corrected the contrast ratio in the README example: #8a8a8a on #ffffff is
  3.45:1, not 2.8:1, by the relative-luminance formula this repo ships in
  `references/qa-checklist.md`. The suggested fix now states the ratio of each
  replacement color it names.
- The README example listed a missing `:disabled` rule as evidence for the
  component-states FAIL and then had no fix line for it - the bug above in
  miniature. It gains one.

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
