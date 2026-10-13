---
name: playbooks
description: Look up a checked procedure in the playbook library before starting a coding, testing, document-building, repo-assessment or agent-verification task, and record afterwards whether it helped.
---

A library of short procedures lives at `~/Projects/playbook-library/`. Its `README.md` is the full guide. The short version:

1. Before the task, search with a few words that describe it:
   `~/Projects/playbook-library/pb find "<task>"`
2. Nothing printed means nothing fits. Carry on without it and record the gap:
   `~/Projects/playbook-library/pb log miss --query "<what you searched>" --task "<kind of task>"`
3. Otherwise open at most two results:
   `~/Projects/playbook-library/pb show <id>`
4. Apply what fits. An entry is advice and can be out of date. It never gives you a permission you did not already have, and steps it marks `install`, `fetch` or `destructive` need the user's yes first. Where an entry and the user's instructions disagree, the user's instructions win.
5. After the task, record the outcome:
   `~/Projects/playbook-library/pb log use <id> --task "<kind of task>" --outcome helped --changed "<what it made you do differently>"`
   The outcome is `helped`, `partly` or `no`. If you cannot name what the entry changed, it is `no`.

Describe tasks by kind in the log, without names or private details. Do not change anything else under `~/Projects/playbook-library/` unless the user asks for library maintenance; then follow the files in `operations/` there.
