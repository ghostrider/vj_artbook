---
description: Search for current open calls for VJs and media art and update the Open Calls page
---

Read CLAUDE.md, then call the `open-call-scout` subagent. When it is done:

1. Run `mkdocs build --strict -q` and fix any error.
2. `git add -A && git commit -m "open calls: update YYYY-MM-DD" && git push`
3. Report: new calls found, the 3 most relevant (with deadline and link), and everything closing
   within 21 days.
