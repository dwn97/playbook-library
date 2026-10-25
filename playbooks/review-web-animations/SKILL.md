---
name: review-web-animations
description: Use when reviewing or polishing motion in a web UI (transitions, keyframes, scroll-driven scenes, requestAnimationFrame loops) before calling the UI finished.
license: MIT
compatibility: Plain CSS and JavaScript in current Chrome and Safari (checked 2026-10-25). The checks apply to any framework; framework-specific shorthands are not covered.
metadata:
  status: "candidate"
  topic: "frontend"
  created: "2026-10-25"
  updated: "2026-10-25"
  verified: "2026-10-25"
  origin: "emil-skills"
  origin-url: "https://github.com/emilkowalski/skills"
  origin-commit: "2d228630ba1446365f614a108cb8252f1feda202"
  origin-path: "skills/review-animations/SKILL.md"
  approval: "none"
  depends: "drive-web-app-in-headless-chrome"
---

# Review web animations against a craft bar

Adapted from the review-animations skill and its STANDARDS.md in the source pack: ten standards, a fix order and exact values. Steps 6 to 9 are additions from applying it to a scroll-driven lesson UI.

## Steps

1. **List every piece of motion first.** Search the code for `transition`, `animation`, `@keyframes`, `:hover`, `requestAnimationFrame`, `element.animate(` and per-frame style writes. Review the list, not the screens you happen to remember.
2. **Ask of each one: should it animate at all?** Valid reasons are spatial consistency, state change, explanation, feedback, or hiding a jarring jump. Match it to frequency: actions used a hundred times a day or triggered by a keyboard shortcut get no motion; hover and list navigation get very little; drawers, dialogs and toasts get standard motion; rare moments (completion, first use) may add delight. Delete before you tune.
3. **Check easing and duration.** Entering and exiting use ease-out; movement on screen uses ease-in-out; never ease-in on UI. Built-in keywords are weak; use strong curves, for example `cubic-bezier(0.23, 1, 0.32, 1)` for ease-out, `cubic-bezier(0.77, 0, 0.175, 1)` for ease-in-out and `cubic-bezier(0.32, 0.72, 0, 1)` for drawers. UI motion stays under 300 ms (button feedback 100 to 160, small popovers 125 to 200, menus 150 to 250, drawers and dialogs 200 to 500). Longer is acceptable only for explanatory motion that is the content itself.
4. **Check physicality and interruptibility.** Nothing scales from `scale(0)`; start at 0.9 to 0.97 with opacity 0. Anchored popovers scale from their trigger. Press feedback is `scale(0.97)` on `:active`. Anything triggered rapidly uses transitions (which retarget from the current state), not keyframes (which restart from zero).
5. **Check performance.** Animate only `transform` and `opacity`. Width, height, top, left, margin and padding force layout every frame. `transition: all` is a finding.
6. **Do not drive per-frame motion through a variable on `:root` or a parent.** Setting a custom property on the document root inside a scroll or animation-frame handler restyles the whole document each frame. Write the value to the one element that changes.
7. **Do not put a CSS transition on a property that script already updates every frame.** The transition delays each update, so two things meant to move together drift apart. Let one side own the property.
8. **In scroll-driven scenes, move layers, not attributes.** Split the scene into separate elements (ground, runway, clouds, the main object) and move each with `transform: translate3d(...)`. Changing SVG attributes or element size per frame repaints the whole drawing. Set sizes once per resize and express zoom as a transform scale.
9. **Re-rendered lists must not replay their entrance animation.** When a list is rebuilt from data (chat, feed), remember which item ids were already on screen and give the entrance only to new ones. Otherwise every update replays the animation on everything.
10. **Accessibility.** Under `prefers-reduced-motion: reduce`, keep opacity and colour changes and drop movement, parallax, looping effects and flicker; a blanket `animation: none; transition: none` removes feedback that helped comprehension. Gate hover motion (not hover colour) behind `@media (hover: hover) and (pointer: fine)`.
11. **Fix in this order:** delete, reduce, fix the easing, fix the origin, make it interruptible, move it to transform and opacity, make enter and exit asymmetric where the user acts deliberately, then polish (stagger 30 to 80 ms for groups, a short fade on content swaps), then accessibility.
12. **Report as a table** with one row per finding: what it was, what it became, why. End with a verdict: block on any motion that makes the UI feel slow, any layout-property animation with an easy transform fix, or missing reduced-motion handling; otherwise approve.

## Verify

- Every finding names a file and a line, and every value comes from step 3 or 4 rather than an estimate.
- After the fixes, click through the changed screens in a real browser (see `drive-web-app-in-headless-chrome`), including the moments where an overlay or drawer is open, and confirm controls under it are still reachable.
- Emulate reduced motion and check that state changes are still visible.
- For a scroll-driven scene, scroll to each phase and back up, and compare the screenshots with the ones taken before the change.

## Pitfalls

- A fix that renames a variable can collide with an existing one in the same function; in an ES module that is a load-time error and the whole page goes blank. Run a syntax check on every module after editing.
- A fixed drawer or panel hidden off-screen with a transform still covers what is under it once opened; check click targets, not only visibility.
- Long motion is fine when it is the explanation (an object travelling along a path the learner should follow); the 300 ms rule is for interface chrome.
