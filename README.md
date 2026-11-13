# 1000a — VJ Art Book

A living book of visual ideas for live VJ performance, grown weekly by a team of Claude agents
and published with MkDocs Material on GitHub Pages.

## Agents (`.claude/agents/`)
| Agent | Job |
|---|---|
| idea-scout | finds new ideas on the web → `pipeline/backlog.yml` |
| idea-researcher | explores one idea → `pipeline/work/<slug>/research.md` |
| code-sketcher | ISF shader + Wire/Unreal notes → `sketches/<slug>/` |
| book-editor | compiles the chapter → `docs/ideas/<slug>.md` |
| open-call-scout | open calls for VJs and media art → `docs/open-calls.md` |

## Run manually (Claude Code, in this repo)
- `/grow-book 3`: add 3 new chapters
- `/find-open-calls`: refresh the open-call page

## Preview locally
```
pip install -r requirements.txt
mkdocs serve
```

## Steer the book
- Add or remove inspiration areas in `pipeline/sources.md`
- Add your own idea to `pipeline/backlog.yml` with `status: new`: the next run picks it up
- Set an idea to `status: rejected` to drop it
