# Check

    ./pb check

Exit code 0 means the library obeys `CONTRACT.md`. Exit code 1 means at least one violation, each printed on a line starting `FAIL`. A hook or a scheduled run can depend on the exit code. `./pb check --stale` also lists entries not verified for 180 days; those lines start `STALE` and do not change the exit code.

Run it before and after any change to the library.

## Fixing what it reports

Fix the cause. Never make a violation go away by weakening the contract, by editing a line of the log, or by declaring an approval class the steps do not need.

| the check says | what to do |
|---|---|
| a field is missing or invalid | Correct the entry's frontmatter against section 3 of the contract. |
| the content changed without a log event | Record the change with `./pb log edit <id> --why "<reason>"`. |
| a trusted entry changed since its last passing review | The owner or an independent reviewer reads the new text; record it with `./pb log review <id> --reviewer owner --verdict pass`. Until then the check stays red, which is the point. |
| the steps contain a command of an approval class that is not declared | Add the class to `approval`, or remove the command if the entry does not need it. |
| invisible characters, an HTML comment, a base64 block, an email address, a home-directory path, or text that claims to grant permission | Remove it. If it arrived with adapted source text, tell the owner: that is a finding about the source. |
| an installed entry's body does not end with the `## Log the outcome` section | Put the section back as the last thing in the body, word for word from section 3 of the contract with the entry's id filled in. Changing an installed entry needs the owner's yes. |
| the `skill` mark is on a retired entry | Remove the mark, then run `./pb install` on the owner's yes so that the link goes too. |
| a `confirm` names something that is not a `use`, or a use has more than one | `./pb log` refuses to write either, so the log was edited by hand. Restore `log/events.jsonl` from git. |
| a generated file differs from a fresh build | `./pb build`. Never edit a generated file. |
| committed log lines changed or were removed | Restore `log/events.jsonl` from git. Corrections are new events. |
| a document names a command or option that does not exist | Correct the document, or tell the owner the command is missing. |
| the contract lists a field the checker has no validator for | The contract and the command have drifted apart. Tell the owner; do not delete the row. |

## What a clean check does not mean

It means the library is well formed and every change is recorded. It does not mean an entry's advice is right, or that an entry is safe. Sections 4 and 6 of the contract say what the evidence and the trust rules do and do not establish.
