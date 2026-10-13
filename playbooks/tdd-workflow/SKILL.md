---
name: tdd-workflow
description: Use when adding a feature or fixing a bug test-first, so that a test is seen failing for the intended reason before any production code changes.
license: MIT
metadata:
  status: "candidate"
  topic: "testing"
  created: "2026-10-13"
  updated: "2026-10-13"
  verified: "2026-10-13"
  origin: "ecc"
  origin-url: "https://github.com/affaan-m/ECC"
  origin-commit: "b92f181b3b748d1abbc2afd94af6aa7d8c8fca29"
  origin-path: "skills/tdd-workflow/SKILL.md"
  approval: "none"
---

# Test-first workflow

Adapted from the upstream skill. What is kept is the part that is easy to skip: seeing the test fail for the right reason, and reporting the evidence.

## Steps

1. **Find the project's real test command** before anything else: the test script in the manifest, the test configuration, the test files next to the code. The package manager and the test runner are two different things; check both. Run the existing suite and record the baseline.
2. **State the behaviour,** one sentence per case: the normal case, the edge cases, the error case. For a bug, the exact input that fails today.
3. **Write the test first.** For a bug, write a reproducer that fails because of the bug.
4. **Red gate: run it and read the failure.** It counts only if the test was collected and executed, and it failed for the intended reason, which is the missing behaviour or the bug. A failure from a syntax error, a wrong import, a missing dependency or broken setup is not red. Fix the test until the failure is the right one. Do not touch production code before this.
5. **Green.** Write the smallest change that makes the test pass. Rerun the same test, then the whole suite.
6. **Refactor** with the tests green: names, duplication, structure. Rerun after each change.
7. **Report the evidence:** the command, the failing output from step 4, the passing output from step 5, and one line per new test saying what it guarantees. Quote real output. Never write that something passes when it was not run.

## Verify

- Red was observed. The failure message from step 4 is in the report.
- Undo the production change for a moment and the new test fails again. Put it back and the test passes.
- The whole suite passes, not only the new test.
- No existing test was weakened, skipped or deleted to reach green. If one had to change, say which and why.

## Pitfalls

- Writing test and implementation together and running once. The test was never seen failing, so it may assert nothing.
- A test that passes on the first run, before any production change, is testing something that already works or nothing at all. Find out which.
- Upstream requires a checkpoint commit after each stage. Commit only if the user asked for commits. Otherwise the stages live in the report.
- Upstream sets a coverage floor of 80% and prescribes end-to-end tests. Use the project's own standard.
- Upstream starts by running a detector script that ships with that pack. It is not used here; step 1 replaces it.
- When no test can be written first, as in an exploratory spike or pure layout work, say so. Write the test once the behaviour is known.
