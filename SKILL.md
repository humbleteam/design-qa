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
you cannot tell whether the component is interactive at all.

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

Write each fix as:

```
N. <short title> - P<0/1/2>
   Before: <specific, observable fact - what's actually there>
   After: <concrete fix, implementable in under an hour>
```

Cap the list at the most-impactful items if there are many; do not pad with
trivial duplicates of the same issue on different elements - group repeats
into one fix line and note the count ("applies to all 4 nav icons").

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

**Verdict: FAIL** - <N> P0 issue(s) in <category list>. Fix P0s before
merge.
Coverage: <n>/6 categories verifiable. <If n < 6, add:> Not verifiable:
<category list> - <what input would resolve it>.
```

Verdict logic - first match wins:

- **FAIL** - one or more P0 issues anywhere.
- **NOT VERIFIABLE** - no P0, and no category produced evidence. The gate did
  not run; say which input would let it run, and never render this as a pass.
- **PASS WITH FOLLOW-UPS** - no P0, at least one P1 or P2, at least one
  category with evidence.
- **PASS** - zero issues at any severity, and all six categories produced
  evidence.
- **PASS WITH GAPS** - zero issues found, but one or more categories are N/V.

Coverage never upgrades a verdict, and PASS is the one outcome it can veto: a
run that found nothing wrong in two categories and could not see the other
four is not the same result as a screen that cleared all six. Count the
categories that produced evidence, put `Coverage: <n>/6 categories verifiable`
on the verdict line whatever the verdict is, and when `n` is below 6 name the
N/V categories and what would unlock each.

## Edge cases

- **Single static screenshot.** States, keyboard, and often responsive land
  as N/V - say exactly what additional input (hover screenshot, CSS, a
  drivable URL) would unlock each one. Contrast, touch targets (if scale is
  known), and copy usually stay verifiable.
- **Nothing came back verifiable.** A screenshot at an unstated scale, with
  no legible text and no solid-fill color to sample, supports none of the six
  categories. The verdict is NOT VERIFIABLE, never PASS: an empty fix list
  here means nothing was checked, not that nothing was wrong.
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
