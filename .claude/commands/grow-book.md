---
description: Run the VJ art book pipeline — scout new ideas, research, design and publish them as chapters
argument-hint: "[number of new ideas, default 3]"
---

Run the 1000a VJ Art Book pipeline. Read CLAUDE.md first.

N = $ARGUMENTS (if empty: 3). Today's date: use `date +%F`.

1. **Catch up on unfinished work first.** Read `pipeline/backlog.yml`. Any idea with status
   `new`, `researched` or `designed` from an earlier run is finished first, before new ideas
   (it counts towards N).
2. **Scout.** If fewer than N ideas are waiting, call the `idea-scout` subagent to add the missing number.
3. **For each waiting idea**, run the chain one stage at a time:
   - status `new` → `idea-researcher` subagent with the slug
   - status `researched` → `design-sketcher` subagent with the slug
   - status `designed` → `book-editor` subagent with the slug
   Different ideas may run in parallel; the stages of one idea must run in order.
   After each step, check backlog.yml to confirm the status moved on. If an agent fails,
   retry once; if it fails again, leave the status as is and note it in the report.
4. **Verify.** Run `mkdocs build --strict -q`. Fix problems or revert the broken chapter.
5. **Commit and push.** `git add -A && git commit -m "book: add <slugs>" && git push`.
6. **Report** in a few lines: new chapters (title + one-line hook), rejected ideas with reason,
   anything that failed.
