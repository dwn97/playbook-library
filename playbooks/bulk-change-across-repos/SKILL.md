---
name: bulk-change-across-repos
description: Use when one operation moves, deletes or rewrites files in several git repositories or many directories at once.
compatibility: git.
metadata:
  status: "candidate"
  topic: "git"
  created: "2026-10-13"
  updated: "2026-10-13"
  verified: "2026-10-13"
  origin: "own"
  approval: "destructive"
---

# A bulk change across several repositories

The failure this prevents: checking some of the targets, finding them safe, and applying the change to all of them.

## Steps

1. **List every target and count them.** The operation covers N repositories. Every check below runs on all N.
2. **Record the state before.** In each repository, save the output of `git status --porcelain` and `git rev-parse HEAD` to a file outside the repositories.
3. **For each path you plan to move or delete, in each repository, ask git.**
   - Is anything under it tracked? `git ls-files -- <path>`
   - Is it ignored, and by which rule? `git check-ignore -v <path>`
   - Does `.gitignore` have a line starting with `!` under that path? An ignored directory can hold one tracked file, kept by exactly such a rule.
4. **Ask the user before deleting or moving anything,** with the list and the counts. Say what is tracked, what is untracked, and what git cannot bring back.
5. **Prefer a form that can be undone.** Move to a dated archive folder instead of deleting. Do one repository first and inspect it before the rest.
6. **Search for references to the paths** you are about to move (links, imports, configuration) and plan their update.
7. **Run the operation.**
8. **Record the state after** with the same commands and diff it against step 2. A tracked file that now shows as deleted or modified, and was not meant to, is restored at once with `git checkout -- <path>` or `git restore <path>`.
9. **Report per repository:** what changed, what was restored, what is now untracked.

## Verify

- The before-and-after diff holds only intended changes, in all N repositories.
- No repository shows a deleted tracked file you cannot explain.
- References to moved paths resolve.
- The report gives N and says the checks ran on all N.

## Pitfalls

- Checking three of six and generalising. The exception sits in one you did not check.
- "It is ignored, so it is not tracked." A negated ignore rule exists to keep one file.
- "It regenerates." Only while the environment that produced it still resolves. Check before deleting derived data that backs a result someone has reported.
- Bundling a large safe cleanup with a small risky one. The bundle carries the risk of its weakest item. Split them.
- A file synced from cloud storage can be a placeholder that looks empty or corrupt. Open it before calling it broken.
- In a repository shared with other people, propose the commands and let the user run them, unless told otherwise.
