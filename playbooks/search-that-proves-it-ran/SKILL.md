---
name: search-that-proves-it-ran
description: Use when a search, grep or count over files or logs returns nothing or a surprising number and a conclusion will rest on it.
metadata:
  status: "candidate"
  topic: "verification"
  created: "2026-10-13"
  updated: "2026-10-13"
  verified: "2026-10-13"
  origin: "own"
  approval: "none"
---

# A search result you can rest a conclusion on

An empty result has two explanations: nothing is there, or the search did not run as intended. They look the same.

## Steps

1. **Prove the search ran before trusting an empty result.** Print how many files it covered. Then run the same command for a string you know is present. No hit on the known string means the command is broken.
2. **Check the usual breakages.**
   - An unquoted variable holding several flags or patterns. zsh does not split it into words, so it arrives as one bad argument. Use an array.
   - A pattern that matched no files, or that the shell expanded before the tool saw it.
   - Ignore files and hidden directories that the tool skips by default.
   - Binary or compressed files that are skipped, and files that exist only as cloud placeholders.
   - Case, encoding, line endings, and markup inside the text. Search the text with markup stripped as well.
   - A match that spans lines, searched with a tool that works line by line.
3. **Before trusting a count, check the unit.** Lines in logs and transcripts repeat. Deduplicate by the record's own id before counting. Select by the timestamp inside the record, not by the file's modification time.
4. **Confirm a lookup returned what you asked for.** A lookup that takes several identifying parameters can ignore one and return a neighbour. Check that the response names your item (id, name, date) before reading anything else from it.
5. **Try a second route before concluding absence.** "Nothing found" describes one query against one source at one moment. Try another spelling or language, another endpoint, the subject's own site. For a live page, say "empty when checked at" and the time.
6. **If you checked a sample, say so:** "k of N checked". Do not extend the result to N.
7. **Report the result with its denominator:** what was searched, how many files or records, which query, and when.

## Verify

- The control search for a known string returned a hit with the exact command used.
- The count of files searched is above zero and plausible.
- A surprising number was computed a second way, or again after deduplication.
- The conclusion is no stronger than the search: "not found in X", where X is what was searched.

## Pitfalls

- Three empty results in a row, one of them for a string that must be there, mean the search is broken.
- A detail in the result that contradicts what you asked for, such as the wrong weekday or the wrong name, invalidates the result. Do not pass it on with a caveat.
- A text replacement keyed on a short pattern also hits a second, unrelated place. Anchor it on surrounding words and compare before and after.
- A headline number that comes from duplicated lines.
