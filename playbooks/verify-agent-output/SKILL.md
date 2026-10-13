---
name: verify-agent-output
description: Use when relying on work returned by another agent or subagent, such as research findings, review findings, extracted data or a claimed verification.
metadata:
  status: "candidate"
  topic: "agents"
  created: "2026-10-13"
  updated: "2026-10-13"
  verified: "2026-10-13"
  origin: "own"
  approval: "none"
---

# Checking what another agent hands back

An agent's report arrives in the same confident voice whether the work was done or not.

## Steps

1. **Read the report as a set of claims.** A well-formed answer does not show the work was done. Before the conclusions, look for placeholder content (a source on an example domain, a claim that reads "test"), empty sections and reported failures.
2. **Sort the claims.** Which will something be built on, sent to someone, or acted on in a way that cannot be undone? Check those yourself, first-hand: open the file, run the command, fetch the page.
3. **The user outranks an agent on facts about the user.** Where an agent's finding contradicts what the user told you directly, ask the user. Do not build on the agent's version.
4. **For a batch of findings, run a separate verification pass.** Use a fresh agent per finding or per group. Tell it to try to refute the finding, give it the paths to the evidence, and tell it that "not a defect" is an acceptable answer. Expect a large share of findings to fall.
5. **Do not let an agent grade its own work.** The author's "all checks pass" is not evidence. Verification comes from a context that has not seen the author's reasoning.
6. **Brief a reviewer with** the user's request in the user's words, the files, the sources and any safety limits. Leave out your conclusions and your guess at what is weak.
7. **Check the method as well as the totals.** How was it counted? Was the input deduplicated? Was the right time window selected?
8. **Before an irreversible action that rests on an agent's claim** ("these files are gone", "this file is corrupt", "nothing uses this"), confirm the claim yourself.
9. **Report the status of each important claim:** checked first-hand, verified by a second agent, or unverified.

## Verify

- Every claim you pass on carries one of the three labels from step 9.
- At least one finding was rejected or corrected, or you can say why none needed to be. A pass that confirms everything has usually tested nothing.
- Facts newer than the model's training, and facts that depend on one page, were fetched again.

## Pitfalls

- Two agents that read the same summary and agree are one source.
- A verifier that is handed the author's conclusion tends to return it confirmed.
- A finding can be an artifact of how the material was fed in, such as a flattened text dump that scrambles which caption belongs to which figure.
- "Not found" from an agent describes one search.
- Fixing something because a review flagged it, when the flag was wrong, makes good work worse.
