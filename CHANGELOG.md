# Changelog

## [1.7.0] - 2026-09-22

- The verdict-line template in Step 4 carries the third group. It offered two slots -
  `Coverage: <n>/6` and a not-verifiable list - while the prose two paragraphs below it, the
  coverage reference in `references/qa-checklist.md`, and the partial-coverage example in the
  README all require a third: a category that ended FAIL with items still unchecked counts
  toward neither, and the three groups are meant to add up to six. The template was the one
  place a reader copies, and its single `<If n < 6, add:>` condition introduced only the
  not-verifiable list, so a run whose coverage fell short purely because a category failed
  before it finished had to write an empty list or write nothing and leave the numbers short.
- That case is now an edge case, where a reader meets it. The list already carried its
  sibling - a category where some items pass and others cannot be checked - and not the one
  where a failure is found before the category is finished, which is the case the verdict
  line has to report separately and the one the README example spends its closing paragraph
  on.

## [1.6.0] - 2026-09-16

- PASS now asks for `Coverage: 6/6` instead of "all six categories produced
  evidence". The two are different tests, and the difference is the whole point of
  the roll-up rule added in 1.4.0: a category that resolved two of nine states
  produced evidence while counting nothing toward coverage. The verdict ladder is
  first-match-wins with PASS above PASS WITH GAPS, so on the commonest partial run -
  a screenshot, nothing wrong in what could be seen - the loose condition matched
  first and the run reported PASS under a table showing an N/V cell, with PASS WITH
  GAPS reachable only when a category resolved nothing at all. `references/qa-checklist.md`
  has keyed both verdicts to the coverage count since 1.4.0 and the README says PASS
  needs all six categories actually checked; SKILL.md was the file out of step.
- PASS and PASS WITH GAPS are stated as exclusive conditions, so the order of the two
  lines can no longer decide a run, and the edge case for a half-checked category now
  names the verdict it produces.

## [1.5.0] - 2026-09-11

- The README gains a second worked example: one screenshot, partial coverage. Every
  version since 1.2.0 has written rules for the run that cannot finish - the coverage
  veto, the roll-up precedence, the split evidence cell, `Coverage: 0/6` against NOT
  VERIFIABLE - and the only rendered output in the repo was an HTML/CSS run where all
  six categories resolve. The input the skill documents as its most common one had no
  example of its output anywhere. The new block shows a category reading N/V with two of
  nine states verified, a category reading FAIL on a pair it sampled while two others
  were never sampleable, `Coverage: 2/6`, and a verdict that is still actionable.
- Writing that example truthfully turned up a hole in the verdict line. Coverage counts
  only categories finished to the last item, and the not-verifiable list names only
  categories that are N/V, so a category that ended FAIL with items still unchecked
  appears in neither: `Coverage: 2/6` printed above three named N/V categories leaves a
  fourth unaccounted for, and the reader cannot tell whether it was missed or never
  existed. Step 4 now names those categories on the same line, kept separate from the
  N/V list, so counted plus not verifiable plus failed-before-finishing comes to six.
- `references/qa-checklist.md` carries the same reconciliation under Coverage reference.

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
