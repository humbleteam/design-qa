---
name: design-qa
description: Runs a pre-ship design QA gate on a screenshot, URL, or PR HTML/CSS - checks states, WCAG 2.2 contrast, touch targets, breakpoints, and keyboard paths, then returns a pass/fail table and fix list. Trigger phrases - "QA this screen", "is this ready to ship", "check this PR for accessibility issues", "audit before launch". Skip for style critique - use design-review instead; this checks only verifiable gate criteria.
---

# Design QA

Run every screen through a fixed six-category gate before it ships: component
states, contrast, touch targets, responsive behavior, keyboard access, and
copy conventions. Output is a pass/fail table, a severity-tagged fix list, and
one verdict line. The gate is fail-closed - a category with no evidence in the
input is reported as not verifiable, never marked as a silent pass.

## Step 1 - identify the input and mode

The artifact arrives as one of:

- **Screenshot(s)** pasted into chat. Static states only - see the coverage
  limits in Step 2.
- **A URL.** If a browser or devtools tool is available, drive it: resize the
  viewport, tab through focus order, toggle hover/focus states. If no such
  tool is available, fetch the rendered HTML/CSS and read it statically.
- **HTML/CSS** - a PR diff, a pasted snippet, or local files. Read the source
  directly; this is the richest input and the only one that makes every
  category fully verifiable.
- **Design tokens**, if the user supplies a token file (CSS custom properties
  or JSON) alongside the artifact. Use these as the source of truth for
  spacing, size, and color checks instead of generic heuristics, and flag any
  value in the artifact that doesn't match a token as a deviation.

If nothing artifact-shaped is in the message (no image, no URL, no code), ask
one question: what screen, and how is it best inspected (screenshot, URL, or
code)? Do not guess and do not run the gate against a description alone.

If the request names a specific breakpoint or theme ("check the dark mode
version", "just check mobile"), scope the run to what was asked but still
report every category - the unscoped categories become not verifiable rather
than skipped.

## Step 2 - run each category and collect evidence

Work through all six categories every time, in this order. For each, the
result is one of **PASS**, **FAIL**, or **N/V** (not verifiable from this
input). Full detail and copy-paste checklist form: [references/qa-checklist.md](references/qa-checklist.md).

### 1. Component states
Check for: default, hover, focus, active, disabled, loading, empty, error,
and long-content/overflow. A static screenshot shows exactly one state per
element - mark every other state N/V and name what additional screenshots or
code would make it verifiable (e.g. "hover state: N/V - provide a screenshot
with the cursor over the primary button, or the CSS `:hover` rule"). For
HTML/CSS, grep the stylesheet for `:hover`, `:focus`, `:disabled`,
`:empty`/skeleton classes, and `.error`/`.is-error` style classes; a state
with zero matching rules is a FAIL if the component is interactive, N/V if
you cannot tell whether the component is interactive at all. Every one of
these nine states has a severity in Step 3 - none of them is reported and
then left out of the fix list.

### 2. Contrast (WCAG 2.2 SC 1.4.3, SC 1.4.11)
Text against its background needs a 4.5:1 ratio (3:1 for text 24px+ or 19px+
bold - "large text" under SC 1.4.3). Non-text UI elements (icons that convey
meaning, input borders, focus indicators) need 3:1 against adjacent colors
under SC 1.4.11. Get exact hex values from CSS if available; from a
screenshot, only use a hex value you can state with confidence (sampled from
a solid-fill area, not a gradient or photo) - otherwise mark the pair N/V and
ask for the CSS or the exact hex. Compute the ratio with the relative
luminance formula in the reference file; do not eyeball it. If dark mode is
also provided, run contrast on both themes separately and report both rows.

### 3. Touch / click targets (WCAG 2.2 SC 2.5.8)
Minimum 24x24 CSS px for any target that isn't inline text, with narrow
exceptions (inline links in a text block, targets with an equivalent nearby
control). Note where a target meets the 24px legal floor but falls short of
the platform comfort size - 44x44pt per iOS Human Interface Guidelines,
48x48dp per Android Material Design - as a P1, not a P0 - see severity
rules in Step 3. For CSS, read `width`/`height`/`padding` on the
interactive element and its hit area.
For a screenshot, only measure in CSS px if the viewport width is stated or
inferable (a known device size, or visible browser chrome); otherwise mark
N/V rather than guessing a scale factor.

### 4. Responsive (WCAG 2.2 SC 1.4.10)
Content must reflow at a 320 CSS px viewport width with no horizontal
scrolling and no loss of content or function. For a URL with a resizable
viewport tool, actually resize to 320px and look. For CSS, check for fixed
`width`/`min-width` values on containers wider than 320px, and for
`overflow-x` handling. For screenshots, this is verifiable only if the user
supplied a screenshot at or near 320px width; otherwise N/V.

### 5. Keyboard (WCAG 2.2 SC 2.4.7, SC 2.1.2)
Every interactive element needs a visible focus indicator (SC 2.4.7) that
survives your own review, not a suppressed one - `outline: none` or
`outline: 0` with no replacement focus style is a FAIL. Tab order should
follow visual/reading order (compare DOM order to layout for CSS input; a
`tabindex` greater than 0 is a code smell worth flagging). No keyboard traps
(SC 2.1.2) - a modal or widget that can be tabbed into but not out of. A
static screenshot cannot show any of this; mark the whole category N/V unless
CSS/JS or a live, drivable page is available.

### 6. Copy
Icon-label pairs mean what they say (a trash icon next to "Archive" is a
mismatch). Headlines, buttons, and labels use sentence case unless the
artifact's own system consistently uses another convention - if so, check
against that system instead. Error messages state what happened and what to
do about it ("Card declined - check the number and try again" passes;
"Something went wrong" fails). This category is verifiable from a screenshot
whenever text is legible in it.

### Rolling item results into one category status

Every category above checks more than one thing: nine states in category 1, one
row per text and non-text pair in category 2, every interactive target in
category 3. The table in Step 4 has one status cell per category, and the items
inside a category routinely disagree, so the roll-up needs a rule instead of a
judgment call each run. Resolve the cell by precedence, highest first:

1. **FAIL** if any item in the category failed. A found failure stands whatever
   else could not be checked.
2. **N/V** if nothing failed and at least one item could not be checked. A
   category where two items passed and seven were invisible has not been
   checked, and PASS would claim it was.
3. **PASS** only when every item resolved and none failed.

The evidence cell then names the split rather than one example, so N/V does not
erase the items that did pass: `2 of 9 states verifiable (default, focus); no
hover, active, disabled, loading, empty, error or overflow rule in the input`.

Coverage counts the same way: a category counts toward `<n>/6` only when no item
in it is left N/V, whether it ended PASS or FAIL. The status cell answers whether
anything failed, the coverage number answers whether everything was looked at,
and a failure found in a category nobody could finish inspecting is not full
coverage of that category.

## Step 3 - assign severity and write fixes

Every FAIL gets one severity tag:

- **P0** - a binary WCAG 2.2 success-criterion breach: contrast below the SC
  1.4.3/1.4.11 threshold, a target under 24x24 CSS px (SC 2.5.8), no focus
  indicator at all (SC 2.4.7), a keyboard trap (SC 2.1.2), or horizontal
  scroll at 320px (SC 1.4.10).
- **P1** - a real gap that isn't a binary accessibility breach: a missing
  component state (loading/empty/error) on a flow that needs it, a target
  between 24px and 44px, illogical tab order, an error message that doesn't
  say what to do next.
- **P2** - polish: icon/label mismatch, inconsistent casing, a state that
  exists but looks unfinished.

**Every FAIL gets a bucket.** The three definitions above name the failures
they were written for, and category 1 checks nine states while only two of
them appear: a missing focus indicator is P0 under SC 2.4.7, and a missing
loading, empty, or error state is P1. Hover, active, disabled, and long
content are checked in Step 2 and mandated as a FAIL when an interactive
component has no rule for them, so leaving them unbucketed forces the fix
list either to drop a failure the table already reported or to cite a
success criterion that does not cover it. Map them like this:

| Missing or broken state | Severity |
|---|---|
| Focus indicator | P0 - SC 2.4.7, a keyboard user cannot see where they are |
| Long content that clips or overlaps at 320px | P0 - SC 1.4.10, content is lost at the reflow width |
| Loading, empty, or error on a flow that reaches it | P1 - the flow has no defined behavior at a point normal use reaches |
| Disabled on a control that can become unavailable | P1 - the control still looks operable while it is not |
| Hover on a pointer-operated control | P1 - nothing marks the element as interactive before it is clicked |
| Long content that clips or overlaps above 320px | P1 - content is lost, outside the width the criterion names |
| Active or pressed | P2 - the action still completes and reports its result |
| Default inconsistent with the rest of the interface | P2 - unless the mismatch itself breaches another category, which files there instead |
| A state that exists but looks unfinished | P2 |

A FAIL matching no line here is P1 with the reason stated. It is never
promoted to P0: P0 means a named success criterion was breached, and citing
one that does not cover the finding is the invented violation the closing
section rules out. Dropping it instead is the other illegal move - the
category already reported FAIL, so a fix list without it contradicts the
table directly above it.

Write each fix as:

```
N. <short title> - P<0/1/2>
   Before: <specific, observable fact - what's actually there>
   After: <concrete fix - implementable in under an hour, or the real fix at
   its real size when it is larger>
```

Every FAIL in the table has a line here, and the list is ordered rather than
trimmed: P0 first, then P1, then P2, and inside a tag the category order of
the table above. Ordering is what puts the most-impactful items first. A cap
would instead drop failures the table already reported, which is the illegal
move the severity rules above name and which
[references/qa-checklist.md](references/qa-checklist.md) states as nothing
that failed is left out of the fix list.

Grouping is the one thing that shortens the list. The same failure on several
elements is one line carrying the count ("applies to all 4 nav icons"),
because one fix closes all of them. Failures needing separate fixes stay
separate lines however many there are - a screen with twenty distinct
failures gets twenty lines, and the length is itself the finding.

The hour bound on After is there to keep a fix concrete: `improve
keyboard support` is not a fix. It bounds how the fix is written, and it is
not a test a finding has to pass to be reported. P0 is the tag most likely
to exceed it, because a binary criterion breach is often structural - a
keyboard trap inside a third-party date picker is closed by patching or
replacing the widget, and a contrast breach carried by a design-system token
is closed in the token set and at every call site. Where the real fix is
larger than the bound, After states it at its real size and names what it
touches instead of shrinking to fit: `replace or patch the vendor date picker
(vendor-side fix, not an afternoon)`. Shrinking it is the failure the bound
invites - an After small enough to pass the bound describes a workaround, the
breach stays live, and the verdict line directly below still blocks the merge
on it. An interim mitigation can ride along on the same line where one
exists, labeled as holding the line rather than closing it, under the real
fix and never in place of it. A P0 written at its real size is also the one
the verdict line names: see what FAIL asks of the merge, in Step 4.

## Step 4 - render the output

Exact shape:

```
# Design QA - <artifact name or URL>

| Category | Status | Evidence |
|---|---|---|
| Component states | PASS / FAIL / N/V | <one line> |
| Contrast | PASS / FAIL / N/V | <one line> |
| Touch targets | PASS / FAIL / N/V | <one line> |
| Responsive | PASS / FAIL / N/V | <one line> |
| Keyboard | PASS / FAIL / N/V | <one line> |
| Copy | PASS / FAIL / N/V | <one line> |

## Fixes

1. <title> - P0
   Before: ...
   After: ...

2. <title> - P1
   Before: ...
   After: ...

**Verdict: FAIL** - <N> P0 issue(s) in <category list>. The merge is blocked
until each is closed or recorded as an accepted breach. <For each P0 whose
After is stated at its real size: name it and say where the fix lands.>
Coverage: <n>/6 categories verifiable. <When n < 6, add whichever of these
two groups has members:> Not verifiable: <category list> - <what input would
resolve it>. <category> ended FAIL with <k> of <m> items unresolved - <what
input would finish it>.
```

Verdict logic - the six rungs:

- **FAIL** - one or more P0 issues anywhere.
- **NOT VERIFIABLE** - no P0, and not one item in any category resolved. The
  gate did not run; say which input would let it run, and never render this as
  a pass.
- **PASS WITH FOLLOW-UPS** - no P0, at least one P1 or P2, and `Coverage: 6/6`.
- **PASS WITH FOLLOW-UPS AND GAPS** - no P0, at least one P1 or P2, and
  coverage below 6.
- **PASS** - zero issues at any severity and `Coverage: 6/6` - every category
  resolved every item it opened.
- **PASS WITH GAPS** - zero issues found, coverage below 6, and at least one
  item resolved somewhere.

Every rung states its own coverage condition, so all six are exclusive and the
order of the lines cannot decide a run. Coverage splits the two rungs that
report findings for the same reason it splits the two that report none: a
category that resolved two items and left seven N/V has produced evidence while
counting nothing toward coverage, so a condition reading "all six produced
evidence" sends the commonest partial run - the one the roll-up rule at the end
of Step 2 exists for - to the unqualified rung under a table showing an N/V
cell. The split used to run on the clean half of the ladder only, and one P2
found in the one category a screenshot can finish took the same four words as a
screen that cleared all six. The count was on the line in both runs; the word a
reader stops at was the same, and the word is what a merge gate acts on. The
ladder answers two questions - did anything fail, and was everything looked at -
so it closes at these six rungs and no further one is reachable.

Coverage never upgrades a verdict, and it qualifies every rung whose first word
is PASS: a run that found nothing wrong in two categories and could not see the
other four is not the same result as a screen that cleared all six, and a run
that found one P2 in those two categories is not the same result either. Count
the
categories where every item resolved to PASS or FAIL with none left N/V, put
`Coverage: <n>/6 categories verifiable` on the verdict line whatever the verdict
is, and when `n` is below 6 name the N/V categories and what would unlock each.

Name the categories that ended FAIL with items still unchecked on that line too,
with what is missing, kept separate from the N/V list. Such a category is in
neither group the line currently reports: it is not N/V, because it found a
failure, and it does not count toward `<n>`, because it was never finished. Leave
it out and the line stops adding up - `Coverage: 2/6` above three named N/V
categories sends the reader hunting for a fourth. The three numbers are meant to
reconcile: categories counted, plus categories not verifiable, plus categories
that failed before they finished, equals six.

`Coverage: 0/6` and NOT VERIFIABLE are different tests and must not be collapsed.
Coverage counts categories finished to the last item, so a run that read three
states and a contrast pair and could see nothing else reports 0/6 while having
produced real evidence - and a failure among those items is a finding that keeps
its severity and its fix line. NOT VERIFIABLE is for the run where no item
anywhere resolved, which is the only case where an empty fix list means nothing
was checked.

### What FAIL asks of the merge

FAIL blocks the merge, at any coverage - a P0 is a breach of a named criterion
and the count beside it changes nothing about that. Everything below is about
what the line says next, not about whether the gate blocks.

The line said `Fix P0s before merge` until it met the branch Step 3 already
carries: a P0 whose real fix is larger than the merge. A keyboard trap inside a
third-party date picker is closed by patching or replacing the widget, and a
contrast breach carried by a design-system token is closed in the token set and
at every call site. Neither lands before a PR merges, so the instruction asks
for something the run has already shown cannot happen, and both ways of making
it read true are failures this skill names elsewhere: shrink the After until it
fits the hour bound, which Step 3 rules out because the breach stays live behind
a workaround, or re-tag the P0 as P1 so the verdict leaves FAIL, which the
severity rules rule out because a tag tracks the criterion that was breached and
not the calendar the fix lands on.

So the line states the condition and names the only exit. The merge is blocked
until each P0 is closed, or until it is recorded as an accepted breach: the
criterion, the real fix at its real size, who owns it, and the interim
mitigation where Step 3 produced one, labeled as holding the line rather than
closing it. Accepting a breach is a call the people shipping make with what is
in front of them; the gate's job is that the record exists and says what is
live. Where a P0's After is already written at its real size, name that P0 on
the verdict line and say where its fix lands, so the two are read together
rather than the reader meeting a blocked merge and an afternoon-sized fix list
that does not contain the blocker.

An accepted breach moves nothing else. The verdict stays FAIL, the tag stays P0,
the fix line stays in the list, and coverage is unchanged - the screen was
inspected exactly as well either way. A record that reads as a pass is how a
breach stops being tracked, which is the same failure as a workaround standing
in for a fix, arriving one line further down.

## Edge cases

- **Single static screenshot.** States, keyboard, and often responsive land
  as N/V - say exactly what additional input (hover screenshot, CSS, a
  drivable URL) would unlock each one. Contrast, touch targets (if scale is
  known), and copy usually stay verifiable.
- **Nothing came back verifiable.** A screenshot at an unstated scale, with
  no legible text and no solid-fill color to sample, supports none of the six
  categories. The verdict is NOT VERIFIABLE, never PASS: an empty fix list
  here means nothing was checked, not that nothing was wrong.
- **A category where some items pass and others cannot be checked.** The cell
  is N/V, not PASS, and the evidence names both halves - see the roll-up rule
  at the end of Step 2. A screenshot showing a clean default state has verified
  one of nine states, not the category. If the run found nothing wrong anywhere,
  the verdict is PASS WITH GAPS: the category produced evidence but did not
  finish, so it does not count toward coverage, and `Coverage: 6/6` is what PASS
  asks for. If it found a P1 or P2 anywhere, the verdict is PASS WITH FOLLOW-UPS
  AND GAPS, on the same reading of the same count - the findings decide which
  pair of rungs the run is on, coverage decides which of the two.
- **A category that failed before it finished.** Contrast samples one pair
  below threshold and cannot reach two others: the cell is FAIL, the failure
  keeps its severity and its fix line, and the category counts toward neither
  `<n>/6` nor the not-verifiable list. Name it on the verdict line as its own
  group, with what would finish it. Counted, plus not verifiable, plus
  failed-but-unfinished comes to six, and a line carrying only the first two
  leaves the reader hunting for the categories missing from both.
- **A P0 whose fix cannot land before the merge.** A keyboard trap in a vendor
  widget, or a contrast breach carried by a token used in forty places. After
  states the real fix at its real size (Step 3), the verdict stays FAIL, and
  the verdict line names that P0 and where its fix lands. The merge proceeds
  only as a recorded accepted breach, with an owner and the real fix against
  it; it is never unblocked by shrinking the After to a workaround or by
  moving the finding to P1.
- **Dark mode supplied.** Run contrast as two separate rows, one per theme,
  in the same table.
- **Design tokens supplied.** Check artifact values against the token set
  first; a value that doesn't match any token is a deviation worth its own
  fix line even if it happens to pass WCAG.
- **URL unreachable or behind auth.** Say so, ask for a screenshot or an
  HTML export instead of guessing at the page.
- **Everything passes.** Still show the full table and write "no P0/P1/P2
  issues found" - do not skip the table because there's nothing to fix.
- **User asks to skip a category.** Run it anyway and mark it "skipped per
  request" in the evidence column rather than omitting the row - the table
  shape stays fixed so nothing silently drops out of the gate.

## What this skill does not do

It does not judge visual style, brand fit, or information hierarchy - that's
a design-review call, not a QA gate. It does not invent WCAG violations from
a component it cannot see (see N/V above). It does not run in a real browser
or screen reader by itself - when the environment has no such tool, it says
so instead of pretending to have tested behavior it only read as code.
