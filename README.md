<div align="center">

<h1>Design QA</h1>

**A pre-ship design QA gate for Claude Code: check any screen's states, WCAG contrast, touch targets, breakpoints, and keyboard paths, and get a pass/fail table with a fix list before it merges.**

[![CI](https://img.shields.io/github/actions/workflow/status/humbleteam/design-qa/validate.yml?branch=main&style=for-the-badge&logo=github&label=CI)](https://github.com/humbleteam/design-qa/actions/workflows/validate.yml)
[![GitHub stars](https://img.shields.io/github/stars/humbleteam/design-qa?style=for-the-badge&logo=github&color=181717)](https://github.com/humbleteam/design-qa/stargazers)
[![Last commit](https://img.shields.io/github/last-commit/humbleteam/design-qa?style=for-the-badge&color=339933)](https://github.com/humbleteam/design-qa/commits/main)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-skill-D97757?style=for-the-badge&logo=anthropic&logoColor=white)](https://github.com/humbleteam/design-qa/blob/main/SKILL.md)

</div>

Design QA is a Claude Code skill that runs a fixed six-category gate -
states, contrast, touch targets, responsive behavior, keyboard access, and
copy - against a screenshot, a live URL, or a PR's HTML/CSS. It returns a
pass/fail table, a severity-tagged fix list, and one verdict line, so a
screen either clears the gate or gets a specific, checkable reason it didn't.

## Table of contents

- [What it does](#what-it-does)
- [Quick start](#quick-start)
- [Usage](#usage)
- [Example output](#example-output)
- [How it works](#how-it-works)
- [How is this different from just asking the model?](#how-is-this-different-from-just-asking-the-model)
- [FAQ](#faq)
- [Related skills](#related-skills)
- [Who maintains this](#who-maintains-this)

## What it does

- Runs six fixed categories every time - states, contrast, touch targets,
  responsive behavior, keyboard access, copy - so a pass never skips one.
- Computes contrast against WCAG 2.2 SC 1.4.3 (text, 4.5:1) and SC 1.4.11
  (non-text, 3:1) with the relative-luminance formula, not by eye.
- Checks touch targets against SC 2.5.8's 24x24 CSS px floor, and flags
  anything under the 44px platform comfort size separately.
- Checks reflow at 320px width (SC 1.4.10), focus visibility (SC 2.4.7),
  and keyboard traps (SC 2.1.2).
- Marks any category the input can't support as "not verifiable" - a
  screenshot can't prove a hover exists.
- Tags every failure P0/P1/P2 and gives a fail-closed verdict: any P0
  forces a FAIL.

## Quick start

**Personal install** (available in every project):

```bash
git clone https://github.com/humbleteam/design-qa ~/.claude/skills/design-qa
```

**Project install** (checked into one repo, shared with your team):

```bash
git clone https://github.com/humbleteam/design-qa .claude/skills/design-qa
```

**Other agents** (Cursor, Codex, or any LLM agent): the skill is plain
[Agent Skills](https://github.com/humbleteam/design-qa/blob/main/SKILL.md) markdown.
Paste `SKILL.md` into the system prompt or a project rules file.

**Verify:** restart Claude Code, then ask it to list its available skills -
`design-qa` should appear, loaded from `~/.claude/skills/` or
`.claude/skills/`.

## Usage

- "QA this screenshot before we ship it" - runs the full gate against a
  pasted image; states and keyboard checks come back mostly "not
  verifiable" with a note on what would unlock them.
- "Run a design QA pass on this PR" - point it at the changed HTML/CSS
  files; every category is fully checkable from source.
- "Is humbleteam.com/pricing ready to ship at mobile width?" - point it at
  a live URL; if a browser tool is available it resizes the viewport and
  drives focus order directly.

## Example output

The following is an illustrative example, not a real audit of any product.

```
# Design QA - checkout-summary.html

| Category | Status | Evidence |
|---|---|---|
| Component states | FAIL | No `:disabled` or `.error` rule for the promo input |
| Contrast | FAIL | Body text #8a8a8a on #ffffff = 2.8:1 (needs 4.5:1) |
| Touch targets | PASS | All buttons >= 44x44 CSS px |
| Responsive | FAIL | Fixed `min-width: 400px` overflows at 320px |
| Keyboard | FAIL | `.promo-input:focus { outline: none; }`, no replacement |
| Copy | PASS | Error copy states cause and next step |

## Fixes

1. Body text fails contrast - P0
   Before: #8a8a8a on #ffffff = 2.8:1
   After: darken to #6b6b6b or darker (4.5:1 minimum)

2. Summary card overflows at 320px - P0
   Before: `.summary-card { min-width: 400px; }`
   After: replace with `min-width: 0; width: 100%;` and let the grid handle sizing

3. Promo input has no visible focus state - P0
   Before: `outline: none` with no replacement
   After: add `:focus-visible { outline: 2px solid #2563eb; outline-offset: 2px; }`

4. Promo input has no error state - P1
   Before: invalid codes submit silently
   After: add an `.error` class with a red border and inline message

**Verdict: FAIL** - 3 P0 issues (contrast, responsive, keyboard). Fix P0s
before merge.
```

## How it works

- Six categories run every time so coverage doesn't shrink to whatever the
  input happens to make easy.
- Evidence beats inference. Contrast is computed with the WCAG
  relative-luminance formula from real hex values, not eyeballed. Target
  sizes are read from CSS or measured against a stated viewport, never
  assumed.
- A category with nothing to check against is reported as not verifiable,
  with a note on what would unlock it - a gate that rounds uncertainty up
  to "pass" isn't a gate.
- Every failure gets a severity tag: P0 for a binary WCAG 2.2
  success-criterion breach, P1 for a real gap that isn't a legal-grade
  breach, P2 for polish. Only P0 forces the verdict to FAIL.
- Fixes follow a Before/After shape: Before is the specific, observable
  problem; After is a change implementable in under an hour.
- A supplied design-token file is checked against first, ahead of generic
  heuristics.
- The full checklist stands alone as a manual reference:
  [references/qa-checklist.md](references/qa-checklist.md), including the
  contrast formula for hand computation.

## How is this different from just asking the model?

A bare "does this look okay" prompt returns a vibe, not a verdict - it
comments on whatever catches its attention and skips the rest, with no
fixed category list to keep it honest. It also tends to round an unclear
screenshot up to a pass instead of flagging what it can't check. This
skill pins six categories every run, forces a real contrast computation
instead of a guess, tags severity so a P2 note can't outweigh a P0
failure, and ends with one verdict line, not a paragraph to interpret.

## FAQ

**What should a design QA checklist include?**
At minimum: component states (hover, focus, active, disabled, loading,
empty, error), contrast against WCAG 2.2 SC 1.4.3 and SC 1.4.11, touch
target size against SC 2.5.8, responsive behavior at 320px width (SC
1.4.10), keyboard access and focus visibility (SC 2.4.7, SC 2.1.2), and
copy conventions. Those are the six categories this skill runs.

**How do I automate design QA?**
Point this skill at a PR's changed HTML/CSS files, or run it against a
staging URL before a release. It runs as a Claude Code skill;
`references/qa-checklist.md` also works as a manual gate with no agent
involved.

**What is the minimum touch target size?**
WCAG 2.2 SC 2.5.8 sets 24x24 CSS px as the minimum, with exceptions for
inline links and targets with an equivalent-sized alternative nearby. iOS
Human Interface Guidelines recommend 44x44pt for primary actions; Android
Material Design recommends 48x48dp - a legal floor and a usability bar,
not the same number.

**Can Claude check color contrast automatically?**
Yes, given two hex values - it applies the WCAG relative-luminance formula
and reports the exact ratio against the 4.5:1 (text) or 3:1 (large
text/non-text) thresholds. From a screenshot alone it only reports a ratio
it can state with confidence; ambiguous colors are marked not verifiable.

**Do I still need axe-core or Lighthouse?**
Yes, and they run in different places. Automated scanners are rule engines
against a rendered DOM: they catch missing alt text, unlabeled inputs, ARIA
misuse, and contrast on elements they can compute, and they run on every
commit without anyone deciding to look. What they cannot report is a state
that was never built, a tab order that follows the DOM but not the reading
order a person sees, or an error message that names the problem without
saying what to do next. Those are three of the six categories in this gate.
Keep a scanner in CI as the floor, and run this gate against the screens a
PR actually changes.

**What's the difference between this and a design review?**
This skill checks pass/fail criteria against fixed standards. A design
review judges subjective quality - hierarchy, polish, layout intent. Use
[design-review](https://github.com/humbleteam/design-review) for that second
kind of question.

## Related skills

Part of a 10-skill open-source kit for design teams by Humbleteam.

- [design-review](https://github.com/humbleteam/design-review) - structured UX critique with a 0-4 score, Before/After/Why fixes, and a citation for every claim.
- [ascii-wireframes](https://github.com/humbleteam/ascii-wireframes) - three distinct layout hypotheses as ASCII wireframes before any hi-fi work.
- [html-mockup](https://github.com/humbleteam/html-mockup) - census-first HTML mockups that match a reference screenshot: exact palette, item counts, component states.
- [extract-design-tokens](https://github.com/humbleteam/extract-design-tokens) - pull palette, type, spacing, radii, and shadows from a URL or screenshot into CSS variables and JSON.
- [audit-design-tokens](https://github.com/humbleteam/audit-design-tokens) - find token drift in a codebase: raw hex values, off-scale spacing, near-duplicate colors.
- [design-handoff](https://github.com/humbleteam/design-handoff) - turn a finished mockup into a dev-ready spec: tokens, states, accessibility annotations, open questions.
- [accessibility-audit](https://github.com/humbleteam/accessibility-audit) - WCAG 2.2-grounded accessibility review with success-criterion citations and severity levels.
- [ux-writing](https://github.com/humbleteam/ux-writing) - interface copy that reads human: plain-verb microcopy rules and an AI-tell strip pass.
- [design-brief](https://github.com/humbleteam/design-brief) - extract a 5-bullet design brief from messy project inputs, with a gap report for what is missing.

## Who maintains this

[Humbleteam](https://humbleteam.com/) is a digital product design and AI-engineering studio: founded in 2017, working from Prague and Dubai, with 80+ digital awards to the name, including 14 Awwwards wins, a Webby, and a Red Dot. We design digital products for startups and enterprises in fintech, healthtech, sports, and AI, and we build AI infrastructure for design teams - agents, workflows, and skills like this one.

This skill is distilled from the internal playbooks we run on client work: the same checklists behind the case studies at [humbleteam.com/work](https://humbleteam.com/work), for clients like Tinder and Acronis.

- The full 10-skill kit: [Related skills](#related-skills) above, or all repos at [github.com/humbleteam](https://github.com/humbleteam)
- What we do with AI for design teams: [humbleteam.com/ai](https://humbleteam.com/ai)
- Design and AI writing: [humbleteam.com/blog](https://humbleteam.com/blog)
- LinkedIn: [linkedin.com/company/humbleteam](https://www.linkedin.com/company/humbleteam/)
- Talk to us: [hi@humbleteam.com](mailto:hi@humbleteam.com)

Issues and PRs welcome.

MIT - see [LICENSE](LICENSE).
