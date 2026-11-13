# VJ Art Book — Sun92

A living book of visual ideas for VJing and live audiovisual performance, grown
by a team of Claude agents. It is published as a MkDocs Material site.

Owner: Uli (Sun92), VJ in Schaffhausen. Main tools: Resolume Arena + Wire,
Unreal Engine 5.7 (Niagara, Remote Control via OSC/MIDI), Spout, NDI,
Arturia KeyStep 37 / BeatStep Pro, Rust, TouchDesigner.

## Repository layout

```
docs/                     the published book (MkDocs source)
  index.md                landing page
  ideas/<slug>.md         one chapter per idea (written by book-editor)
  ideas/index.md          table of contents for all ideas (auto-maintained)
  open-calls.md           current open calls (written by open-call-scout)
sketches/<slug>/          runnable code per idea (written by code-sketcher)
pipeline/
  backlog.yml             idea queue and status, the single source of truth
  sources.md              where the scout looks for inspiration
  open-calls.yml          raw open-call records (incl. expired ones)
  work/<slug>/            intermediate notes: research.md, code.md
.claude/agents/           the agent team
.claude/commands/         /grow-book and /find-open-calls
```

## Pipeline (see .claude/commands/grow-book.md)

1. **idea-scout**: finds new ideas on the web and appends them to `pipeline/backlog.yml` with status `new`
2. **idea-researcher**: explores one idea, writes `pipeline/work/<slug>/research.md`, sets status `researched`
3. **code-sketcher**: designs the implementation, writes `sketches/<slug>/` + `pipeline/work/<slug>/code.md`, sets status `coded`
4. **book-editor**: compiles the chapter `docs/ideas/<slug>.md`, updates the index, sets status `published`
5. **open-call-scout** (separate run): maintains `pipeline/open-calls.yml` and `docs/open-calls.md`

## Rules for all agents

- English for all book content. Short, concrete, practical. No filler.
- Always cite sources as Markdown links. Never invent artists, works, URLs or dates.
  If something cannot be verified, leave it out.
- Paraphrase; quote at most one short sentence per source. No copied images: link to them instead.
- Slugs: lowercase-kebab-case, unique, never renamed once published.
- Never delete published chapters. Corrections are fine.
- `pipeline/backlog.yml` must stay valid YAML. Edit only your own idea's entry.
- Before finishing a run: `mkdocs build --strict` must pass.
- Commit with a clear message, e.g. `book: add chapter feedback-tunnels`.
