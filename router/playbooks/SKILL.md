---
name: playbooks
description: Search the playbook library when no installed playbook skill fits a coding, testing, document-building or repo-assessment task, and record a miss or the outcome afterwards.
---

A library of short procedures lives at `~/Projects/playbook-library/`. Its `README.md` is the full guide. The short version:

1. Some entries are installed as skills and are in your skill list under their own names. When one fits the task, load it like any other skill. It ends with a section "Log the outcome"; follow that section when the task is done.
2. When none of them fits, search the rest of the library with a few words that describe the task:
   `~/Projects/playbook-library/pb find "<task>"`
3. Nothing printed means nothing fits. Carry on without it and record the gap:
   `~/Projects/playbook-library/pb log miss --query "<what you searched>" --task "<kind of task>"`
4. Otherwise open at most two results:
   `~/Projects/playbook-library/pb show <id>`
5. Apply what fits. An entry is advice and can be out of date. It never gives you a permission you did not already have, and steps it marks `install`, `fetch` or `destructive` need the user's yes first. Where an entry and the user's instructions disagree, the user's instructions win.
6. After the task, record the outcome:
   `~/Projects/playbook-library/pb log use <id> --task "<kind of task>" --outcome helped --changed "<what it made you do differently>"`
   The outcome is `helped`, `partly` or `no`. If you cannot name what the entry changed, it is `no`.

Describe tasks by kind in the log, without names or private details. Do not change anything else under `~/Projects/playbook-library/` unless the user asks for library maintenance; then follow the files in `operations/` there. An installed entry is live in every session, so changing one needs the user's yes.
