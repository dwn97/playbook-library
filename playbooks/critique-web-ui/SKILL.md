---
name: critique-web-ui
description: Use when asked to critique or review the design of a web page or app screen before shipping, and the result should be a ranked list of fixes rather than opinions.
license: Apache-2.0
compatibility: A way to start two isolated agents, a headless browser (see drive-web-app-in-headless-chrome) and Node 22+. Checked 2026-10-25 against impeccable 93686754 and taste-skill 12af471f.
metadata:
  status: "candidate"
  topic: "frontend"
  created: "2026-10-25"
  updated: "2026-10-25"
  verified: "2026-10-25"
  origin: "impeccable"
  origin-url: "https://github.com/pbakaus/impeccable"
  origin-commit: "936867543ac2576920d663cc9803597f6044f3ac"
  origin-path: "plugin/skills/impeccable/SKILL.md"
  approval: "none"
  depends: "drive-web-app-in-headless-chrome review-web-animations"
---

# Critique a web UI with two isolated assessments

Adapted from impeccable's critique command (the skill's `reference/critique.md` at the pinned commit). The browser-evidence half here uses measurements any harness can make, plus, optionally, the taste-skill inspector (source `taste-skill`). Steps 7 to 9 are additions from a run on a three-look lesson UI.

## Steps

1. **Freeze a "before".** Tag the commit and serve a copy at a second path, so the owner can compare both versions in the browser afterwards.
2. **Write down the fixed facts first:** audience, what the page is for, hard constraints (required labels, no framework, content rules), and the mode: Operate (task tools), Read, Persuade (landing pages) or Experience. These go into both briefs, so required elements are not reported as findings.
3. **Run two assessments as separate agents that never see each other's output.**
   - A, design review: design specificity (could another product use this unchanged?), Nielsen's ten heuristics scored 0 to 4 with one key issue each, cognitive load (single focus, choices per decision point, working memory), emotional journey (peak, end, valleys), 3 to 5 priority issues tagged P0 to P3, persona red flags (first-timer, keyboard and screen-reader user, one-handed phone user).
   - B, evidence only: click-through tests of every core flow, console errors, WCAG contrast measured from computed colours and, over images or gradients, from sampled background pixels; text under 11 px; touch targets under 40 px at phone width; horizontal overflow; an automated rule inspector if one is available. B gives no design opinions.
   - Cap each at a time budget; they finish in minutes, not hours.
4. **Merge, do not concatenate.** Rank by severity: P0 blocks a core task, P1 causes real difficulty, P2 is an annoyance with a workaround, P3 is polish. Note where A and B agree and mark B's false positives (an illustration flagged as a "gradient fill", rules defined in CSS counted as uses, disabled controls failing contrast, which WCAG exempts).
5. **Verify the top findings yourself before fixing.** Measure the P0 (for example the box height against the viewport) rather than trusting the report.
6. **Fix in order P0, P1, P2, P3, then rerun all tests** and add a check for every P0 and P1, so the regression cannot return silently.
7. **Look at fixed overlays with content inside.** A fixed drawer or dialog with `align-self: start` inherited from a grid rule sizes to its content even with both `top` and `bottom` set; with a long reply it grows past the viewport and hides its own input. Use `align-self: stretch` and a `max-height`, and test it after real content arrives.
8. **Separate content problems from presentation problems.** When replies paste whole source excerpts, the UI can fold quoted material into one-line disclosures and lead with the reply's own sentences, without dropping anything; the wording itself belongs to whoever owns the content.
9. **Report the comparison:** a table of finding, source (A or B), change made, and what was kept on purpose with the reason.

## Verify

- Every P0 and P1 has a fix and a test that would fail if it came back.
- All flows pass at desktop and phone widths, with zero console errors.
- The "before" path still serves the frozen version, so the comparison is real.

## Pitfalls

- A skill's launcher may download and run a compiled binary on first use. That is not covered by reading its scripts; skip it and follow the skill's manual path unless the owner agrees.
- Shell glob expansion can make a search option such as an unquoted file pattern match nothing, and a search over zero files reports zero findings. Print how many files every scan covered.
- Opacity alone does not hide a panel: invisible controls stay in the tab order. Pair it with `visibility: hidden` or `inert`.
