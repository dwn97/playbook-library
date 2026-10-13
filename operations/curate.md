# Curate

The maintenance pass. Run it on request, or on a schedule once the owner sets one. Mechanical checks come first and judgment second. Every promotion, retirement and dropped source cites log events by `eid`.

## 1. Mechanical

1. `./pb check --stale`. Fix every violation before anything else, following `operations/check.md`.
2. `./pb stats`. Read the table, the list of entries never used, and the searches where nothing fit.

## 2. Entries

3. **Promote.** An entry with at least three uses, two thirds of them `helped` and one confirmed by the owner is ready for review. Ask the owner or an independent reviewer with fresh context to read it. Record the result, then, on a pass, set `status` to `trusted` in the file and record the promotion.

       ./pb log review <id> --reviewer owner --verdict pass
       ./pb log promote <id> --uses "<eid> <eid> <eid>" --review <eid>

   If the uses are there and none is confirmed by the owner, ask the owner about one of them. Do not mark a use confirmed yourself.
4. **Retire.** Propose retirement for an entry that was used at least three times and helped in fewer than one third, that another entry replaces, or that is wrong. Set `status` to `retired`, add `superseded-by` when there is a replacement, and record it.

       ./pb log retire <id> --why "<reason, citing the use events>"

   Lack of use alone retires nothing. An entry for a kind of task that has not come up yet stays.
5. **Merge.** Two entries that answer the same task become one: keep or write the better one, retire the other with `superseded-by`.
6. **Re-verify.** For each stale entry, run its steps or compare them with current tool versions and current upstream. Update `verified`, and `updated` if the text changed.

       ./pb log verify <id> --how "<what was checked>"
       ./pb log edit <id> --why "<what changed>"

   An edit to a `trusted` entry needs a new passing review before the check is clean again.

## 3. Gaps

7. **Misses.** Group the searches where nothing fit. Two or more misses for the same kind of task are a reason to propose an entry, following `operations/ingest.md`.
8. **Unused entries.** If the harness keeps transcripts and the owner allows reading them, sample recent sessions for tasks where a fitting entry existed and was not consulted. Record each.

       ./pb log unused <id> --task "<kind of task>"

   Many of these mean the router or the descriptions need work, not the entries.

## 4. Sources and documentation

9. **Sources.** With the owner's yes, compare each pin with its upstream and report what changed. Moving a pin follows `operations/ingest.md`. Dropping a source is the owner's decision: set its status to `dropped` in the manifest and record it.

       ./pb log drop --source <id> --why "<reason, citing the log>"

10. **Documentation.** Re-read `README.md`, `CONTRACT.md` and the files in `operations/` against what the library now does. Propose each edit to the owner. A change to the contract also needs a new version number and a record.

        ./pb log decide --what "<the decision>" --why "<the reason>"

## 5. Report

11. `./pb build`, `./pb check`, then tell the owner: what was promoted, retired, merged, added and re-verified, each with its `eid`; what needs the owner's decision; and the numbers from `./pb stats`.
