# Design QA checklist

Read this when running the design-qa gate, and reuse it as a standalone
manual checklist - copy the six tables below into a PR description or a
ticket and check items off by hand.

## 1. Component states

- [ ] Default state matches the rest of the interface (spacing, type, color)
- [ ] Hover state exists and is visually distinct from default
- [ ] Focus state exists and is visually distinct from hover (see keyboard
      section - focus needs its own visible indicator, not just a hover echo)
- [ ] Active/pressed state exists for buttons and controls
- [ ] Disabled state is visually distinct and not just lower opacity on an
      otherwise identical control (contrast on disabled text still matters
      if it's not purely decorative)
- [ ] Loading state exists for anything that fetches or submits
- [ ] Empty state exists for any list, table, or search result area
- [ ] Error state exists for any input or async action that can fail
- [ ] Long content (a name, a label, a number) doesn't break the layout -
      check truncation, wrapping, or scroll behavior

## 2. Contrast - WCAG 2.2 SC 1.4.3 (text), SC 1.4.11 (non-text)

- [ ] Body text: 4.5:1 against its background
- [ ] Large text (24px+, or 19px+ and bold): 3:1 against its background
- [ ] Icons that convey meaning (not purely decorative): 3:1 against
      adjacent colors
- [ ] Input borders and other non-text UI boundaries: 3:1 against adjacent
      colors
- [ ] Focus indicators: 3:1 against the adjacent background

### Computing a contrast ratio by hand

Both colors as sRGB hex. For each channel `c` in 0-255:

```
c_s = c / 255
c_lin = c_s / 12.92                          if c_s <= 0.03928
c_lin = ((c_s + 0.055) / 1.055) ^ 2.4         otherwise
```

Relative luminance: `L = 0.2126*R_lin + 0.7152*G_lin + 0.0722*B_lin`

Contrast ratio: `(L1 + 0.05) / (L2 + 0.05)`, where `L1` is the lighter color's
luminance and `L2` the darker one's, so the ratio is always >= 1.

## 3. Touch / click targets - WCAG 2.2 SC 2.5.8

- [ ] Every interactive target is at least 24x24 CSS px, including its
      padding/hit area, not just its visible glyph
- [ ] Exceptions applied correctly: inline text links, and targets that have
      an equivalent-sized alternative nearby, are exempt from the 24px floor
- [ ] Primary actions (buttons a user taps often, or under time pressure)
      meet the platform comfort size - 44x44pt per iOS Human Interface
      Guidelines, 48x48dp per Android Material Design - not a legal minimum
      but a real usability bar
- [ ] Adjacent targets have enough space between them that a touch on one
      doesn't reliably clip its neighbor

## 4. Responsive - WCAG 2.2 SC 1.4.10

- [ ] Content reflows at 320 CSS px width with no horizontal scrolling
- [ ] No content or functionality is lost at 320px - collapsed nav, hidden
      columns, and truncated text still need an equivalent way to reach the
      same information or action
- [ ] Text isn't clipped or overlapping at any breakpoint between 320px and
      the widest tested width
- [ ] Images and media scale down instead of forcing overflow

## 5. Keyboard - WCAG 2.2 SC 2.4.7, SC 2.1.2

- [ ] Every interactive element has a visible focus indicator when tabbed to
- [ ] `outline: none` or `outline: 0` never appears without a replacement
      focus style on the same selector
- [ ] Tab order follows reading/visual order - no unexplained jumps
- [ ] No positive `tabindex` values (they override natural order and are
      almost always a bug)
- [ ] Modals, dropdowns, and custom widgets can be exited with the keyboard
      alone - no trap that only a mouse click escapes
- [ ] All actions available by mouse are also available by keyboard

## 6. Copy

- [ ] Icon and label agree on meaning (a save icon next to "Delete" fails)
- [ ] Headlines, buttons, and field labels use sentence case, unless the
      artifact has its own consistent system - check against that system
      instead of defaulting to sentence case
- [ ] Error messages state what happened, in plain language
- [ ] Error messages state what to do next
- [ ] Button labels are verbs that describe the action ("Save changes", not
      "OK") wherever the action isn't a universally understood convention

## Severity reference

| Tag | Meaning | Forces FAIL verdict |
|---|---|---|
| P0 | Binary WCAG 2.2 success-criterion breach | Yes |
| P1 | Real gap, not a binary accessibility breach | No |
| P2 | Polish | No |

A category with no evidence in the input is **not verifiable** - it is never
recorded as a pass.

Section 1 checks nine states and the three definitions above name only two of
them, so a missing hover or disabled rule can be reported as a failure and then
have nowhere to go in the fix list. Each state maps to a tag:

| Missing or broken state | Tag |
|---|---|
| Focus indicator | P0 (SC 2.4.7) |
| Long content clipping or overlapping at 320px | P0 (SC 1.4.10) |
| Loading, empty, or error state a flow reaches in normal use | P1 |
| Disabled state on a control that can become unavailable | P1 |
| Hover state on a pointer-operated control | P1 |
| Long content clipping or overlapping above 320px | P1 |
| Active/pressed state | P2 |
| Default state inconsistent with the rest of the interface | P2 |
| A state that exists but looks unfinished | P2 |

Anything else that fails is P1 with the reason stated. Nothing is promoted to
P0 without a success criterion that actually covers it, and nothing that failed
is left out of the fix list.

## Coverage reference

Coverage decides which verdict is available. Count the categories that
produced evidence and put `Coverage: <n>/6` on the verdict line.

| Coverage | No issues found | Issues found |
|---|---|---|
| 6 of 6 | PASS | FAIL if any P0, else PASS WITH FOLLOW-UPS |
| 1-5 of 6 | PASS WITH GAPS | FAIL if any P0, else PASS WITH FOLLOW-UPS |
| 0 of 6 | NOT VERIFIABLE | - |

PASS is the one verdict coverage can veto: it needs all six categories to have
been checked, not merely to have turned up nothing.
