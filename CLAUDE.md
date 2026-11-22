# VJ Art Book — 1000a

A living book of visual ideas for VJing and live audiovisual performance, grown
by a team of Claude agents. It is published as a MkDocs Material site.

Owner: Uli (1000a), VJ in Schaffhausen. Main tools: Resolume Arena + Wire,
Unreal Engine 5.7 (Niagara, Remote Control via OSC/MIDI), Spout, NDI,
Arturia KeyStep 37 / BeatStep Pro, Rust, TouchDesigner.

## Repository layout

```
docs/                     the published book (MkDocs source)
  index.md                landing page
  ideas/<slug>.md         one chapter per idea (written by book-editor)
  ideas/index.md          overview of the two parts
  ideas/looks.md          Part I: Looks and Styles, table of contents (auto-maintained)
  ideas/visuals.md        Part II: Generative Visuals, table of contents (auto-maintained)
  open-calls.md           current open calls (written by open-call-scout)
sketches/<slug>/          runnable code per idea (legacy; the design-sketcher writes no code)
pipeline/
  backlog.yml             idea queue and status, the single source of truth
  sources.md              where the scout looks for inspiration
  open-calls.yml          raw open-call records (incl. expired ones)
  work/<slug>/            intermediate notes: research.md, design.md
.claude/agents/           the agent team
.claude/commands/         /grow-book and /find-open-calls
```

## Parts

Every idea has a `part` in backlog.yml: `looks` (Part I, Looks and Styles: treatments and
styles applied to any footage) or `visuals` (Part II, Generative Visuals: systems that make
the image). Chapter files all stay in `docs/ideas/` so URLs never change; only the part
pages group them. A default run adds 2 looks and 2 visuals.

## Pipeline (see .claude/commands/grow-book.md)

1. **idea-scout**: finds new ideas on the web and appends them to `pipeline/backlog.yml` with status `new`
2. **idea-researcher**: explores one idea, writes `pipeline/work/<slug>/research.md`, sets status `researched`
3. **design-sketcher**: designs the effect (layers, techniques, shaders, colors, blending) in `pipeline/work/<slug>/design.md`, no code, sets status `designed`
4. **book-editor**: compiles the chapter `docs/ideas/<slug>.md`, updates the index, sets status `published`
5. **open-call-scout** (separate run): maintains `pipeline/open-calls.yml` and `docs/open-calls.md`

## Rules for all agents

- English for all book content. Short, concrete, practical. No filler.
- Always cite sources as Markdown links. Never invent artists, works, URLs or dates.
  If something cannot be verified, leave it out.
- Paraphrase; quote at most one short sentence per source.
- Images are always embedded by their URL (hotlinked). Never download or copy image files
  into the repo.
  - **Lead image** (directly under the hook): free images only. First choice, a free licence:
    public domain, CC0, CC BY, CC BY-SA (Wikimedia Commons first), checked on the file page,
    with author, licence and source credited. Second choice, a similar photo from Unsplash or
    Pexels that truly resembles the look, captioned
    `*Similar look, not the original work. Photo: [<photographer>](<photo page>) on [Unsplash](https://unsplash.com/license).*`
    (or Pexels with https://www.pexels.com/license/). No CC NC/ND, no unknown licences.
    If nothing free fits, no lead image.
  - **Quotation images** (in "Origins and references", next to the text that discusses the
    work): a copyrighted original, only from the rights holder's own site or an official
    archive (artist, estate, studio, label, publisher, museum, festival). Never from reposts,
    fan sites, Pinterest, wallpaper or image-search sites. Skip it (link only) if that site
    explicitly forbids reproduction. Use the exact image URL from the page's HTML. Caption:
    `*<Work>, <artist>, <year>. © <rights holder>. Shown as a quotation for reference; source: [<site>](<page URL>).*`
- Slugs: lowercase-kebab-case, unique, never renamed once published.
- Never delete published chapters. Corrections are fine.
- `pipeline/backlog.yml` must stay valid YAML. Edit only your own idea's entry.
- Before finishing a run: `mkdocs build --strict` must pass.
- Commit with a clear message, e.g. `book: add chapter feedback-tunnels`.
