---
tags:
  - look
  - art-history
  - graphic-design
  - motion-graphics
---

# Saul Bass Cut-Paper Titles

> A flat poster that moves in snaps: paper bars slam in on the kick, one bold icon breaks along scissor cuts, and nothing is ever eased.

![Triangles of coloured paper laid flat as a background](https://images.unsplash.com/photo-1590412732788-d54badbebb53?w=1400)
*Similar look, not the original work. Photo: [engin akyurt](https://unsplash.com/photos/blue-yellow-and-black-triangle-illustration-G26eJNYZA24) on [Unsplash](https://unsplash.com/license).*

## The idea

Saul Bass was a Bronx-born American graphic designer and filmmaker, known for film posters, corporate logos and title sequences ([Wikipedia](https://en.wikipedia.org/wiki/Saul_Bass)). In the 1950s he turned opening credits from static cards into short films. His method was reduction: few shapes, two or three colours, one motion path per object, timing locked to the music.

This look keeps the rule set and drops the artefacts. It is a live layer of flat colour fields, rough hand-cut edges and angular icons. Bars slide in from set directions and stop dead. Motion is step-like and rhythmic, on or off, never physical ([Art of the Title](https://www.artofthetitle.com/title/psycho/)). One idea moves at a time; everything else holds still.

It suits almost any footage, because flat shapes work as mattes, masks and windows. The limited palette lets other material show through. It is boring on a big screen when it stays a literal title card: static, evenly timed, one icon, or a copy of the original films instead of their rules.

**Rule for this chapter: build original shapes, do not trace Bass works.** Study the rules, not the frames. All icons here are built from primitives, and the palettes are our own.

## Origins and references

- **Saul Bass** (1920-1996): posters, logos, title sequences; credited as one of the first to treat credits as a creative opportunity ([Wikipedia](https://en.wikipedia.org/wiki/Saul_Bass)).
- **The Man with the Golden Arm** (1955): white bars appear, vanish and form abstract patterns before settling into a jagged arm ([Art of the Title](https://www.artofthetitle.com/title/the-man-with-the-golden-arm/)). Bass said composer Elmer Bernstein gave him the counts and he designed to that beat (same source).
- **Anatomy of a Murder** (1959): paper cut-outs on uniform grey, a body taken apart into puzzle-like pieces, Duke Ellington score ([Art of the Title](https://www.artofthetitle.com/title/anatomy-of-a-murder)). Forms appear and disappear on musical cues ([Motion Design Archive](https://luckydumpling.github.io/motion-archive/entries/saul-bass-anatomy/)).
- **Vertigo** (1958): endlessly forming spirals, made with computer-animation pioneer John Whitney ([BFI](https://www.bfi.org.uk/lists/celebrating-saul-basss-centenary-10-essential-title-sequences)).
- **Psycho** (1960): white bars bring sans-serif type in and out, never intersecting; the rule set is on, off, left, right, up, down, black, white ([Art of the Title](https://www.artofthetitle.com/title/psycho/)). A new kind of kinetic typography ([Art of the Title](https://www.artofthetitle.com/designer/saul-bass/)).
- **West Side Story** (1961) and **Seconds** (1966) show the range ([BFI](https://www.bfi.org.uk/lists/celebrating-saul-basss-centenary-10-essential-title-sequences)).
- **Elaine Bass**: joined the studio in 1955, married Saul in 1961, collaborated for 30+ years, including the Oscar-winning short *Why Man Creates* (1968) ([Wikipedia](https://en.wikipedia.org/wiki/Saul_Bass)).

![Anatomy of a Murder album cover: fragmented black silhouette figure on flat colour fields](https://collections-images.lacma.org/images/165689/165689-1-primary.webp)
*Anatomy of a Murder (album cover), Saul Bass, 1959. © EMI Group Limited-Capitol Records, courtesy Saul Bass Estate. Shown as a quotation for reference; source: [LACMA](https://collections.lacma.org/object/165689).*

## Visual language

- **Colour:** four flat slots per palette: ground, near-black ink, off-white paper, one hot accent. No gradients, no shading, no glow. Palettes change by hard cut, never cross-fade.
- **Shape:** angular silhouettes, rough hand-cut edges, icons as puzzle pieces. Original icon set: eye, speaker cone, key, jagged zigzag, hand-cut star.
- **Bars:** thick paper bars slam in from the edges and stop dead. They never cross.
- **Type:** condensed sans-serif treated as shape. It is revealed by bars, knocked out of them, or sliced into strips that lock together on the beat.
- **Motion:** linear slides sampled at 8 to 12 fps, so edges "boil" like stop-motion paper. A 0.25 s slide shows three positions.
- **Rhythm:** kick moves a bar, snare breaks the icon or inverts the palette, drop snaps everything together and holds still for one bar. The stillness after the noise is the impact.

## See it

- [Anatomy of a Murder title page, Art of the Title](https://www.artofthetitle.com/title/anatomy-of-a-murder) — still and analysis of the cut-paper body on grey
- [Anatomy of a Murder, Motion Design Archive](https://luckydumpling.github.io/motion-archive/entries/saul-bass-anatomy/) — teaching entry with embedded video and analysis
- [Anatomy of a Murder (1959) title sequence, YouTube, channel MovieTitles](https://www.youtube.com/watch?v=QccJ2L-7DVk) — the sequence in motion
- [Why Man Creates, YouTube, channel DocumentarieS](https://www.youtube.com/watch?v=3Cip49iQYq4) — Oscar-winning short with Elaine Bass
- [The Man With the Golden Arm title re-creation, YouTube, channel UAT Digital Video](https://www.youtube.com/watch?v=oHU5K5B0leI) — a re-creation of the sequence, not the original
- [Psycho title page, Art of the Title](https://www.artofthetitle.com/title/psycho/) — bars and type
- [BFI: 10 essential Saul Bass title sequences](https://www.bfi.org.uk/lists/celebrating-saul-basss-centenary-10-essential-title-sequences) — overview of the main sequences

Bass's posters and titles are copyrighted. The Anatomy of a Murder cut-up figure is shown under Origins as a quotation from the LACMA collection; for the rest, follow the links.

## Code design

The look is built from three passes. P0 bakes an icon distance field (only when the icon changes). P1 composes ground, bars, notches, icon pieces, type and spiral wipe in painter's order and writes a palette index. P2 is an optional finish (grain, hard shadow, black floor, white cap). All state comes from counters and hashes, so the look is deterministic and restarts cleanly.

Core techniques:

- **Stepped boil clock:** time is quantised to 6, 8 or 12 fps; motion is linear, then sampled at the stepped time.
- **Rough edges:** shape distances displaced by 2-octave value noise ([The Book of Shaders](https://thebookofshaders.com/11/)), primitives from [Inigo Quilez](https://iquilezles.org/articles/distfunctions2d/), 1 px anti-aliased edge.
- **Bar sequencer:** lanes in units of wall height, bars hashed per trigger, never sharing a lane.
- **Scissor shatter:** 3 to 5 straight cut lines split the icon into up to 32 pieces. Each piece jumps outward in stepped levels (0, 0.25, 0.5, 1).
- **Type as matte:** reveal, knockout or slice, with the same edge displacement.
- **Spiral wipe:** a two-colour band that grows for one bar, used only to hide scene changes.

=== "Resolume Wire"
    - Beat-clock and counter nodes give beat phase, `trigCount` and `trigTime`. Node names differ between Wire versions.
    - P0 and P1 run as ISF nodes ([Resolume ISF support](https://resolume.com/support/en/isf)). P0 is a fixed 1024 x 1024 texture, re-rendered on icon change. P1 runs at composition resolution with palette resolve in the same pass.
    - Round `shatter` and `boil` in Wire math so faders snap to steps.
    - Wire has no vertex shader support; all passes are fragment-only, so this is fine.
    - In Arena: the patch as a source above live video in Alpha (matte mode) or Multiply. A Text Block on its own layer feeds the type texture. No-code fallback: cropped Solid Color clips with BPM-synced jump positions; the boil and shatter need the shader.

=== "Unreal Engine 5.7"
    - Approach A (recommended): a full-screen post-process or UI material reproducing P1, with a Custom HLSL node for the bar loop and piece loop, and render targets for the icon SDF and type.
    - Approach B: flat planes under an orthographic Cine Camera, animated in Sequencer with Constant (stepped) keys; torn edges via a noise opacity mask.
    - Optional Niagara burst of flat triangle mesh particles on the drop, with quantised positions.
    - Expose slam, shatter, rough, boil, bars, palette and invert through a Material Parameter Collection and a Remote Control Preset, mapped to OSC or MIDI ([Remote Control protocols](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-protocols-in-unreal-engine)). Send the output to Arena over Spout or NDI.

=== "GODOT"
    - A full-rect ColorRect with a `canvas_item` ShaderMaterial for P1 with palette resolve.
    - A 1024 x 1024 SubViewport (update mode Once, re-triggered on icon change) bakes the icon SDF; a second transparent SubViewport with a Label gives the type texture.
    - A GDScript keeps BPM and beat phase. An AudioEffectSpectrumAnalyzer reads the kick band and raises `trig_count` on an onset. MIDI arrives via `InputEventMIDI`.

## On stage

**Place in the chain.** Put it last, or on its own layer above video. Over video, use matte mode (ground alpha 0, Alpha blend) or the full poster in Multiply. Avoid Add and Screen, which wash out flat colour. No bloom, blur or grain after it.

**Audio reactivity.** Audio may only trigger steps or set targets. It never drives positions continuously, because a smoothly shaking bar breaks the style.

| band | drives |
|------|--------|
| 40-120 Hz (kick) | slam trigger, one bar moves |
| 150-400 Hz envelope | number of bars, rounded to an integer |
| 1-4 kHz (snare, clap) | shatter step or invert, one per section |
| 8-16 kHz (hats) | notches stepping along a bar edge |
| full-band RMS | small offset on `rough`, sampled at the boil clock |

On dense tracks, switch audio off and play the triggers by hand.

**MIDI.**

| control | does |
|---------|------|
| BeatStep Pro pads 1-4 | force a bar from left, right, up, down |
| BeatStep Pro knobs | `rough`, `bars` |
| KeyStep keys C-E | pick palette; F toggles invert |
| KeyStep pads or quantised knob | `shatter` in four steps |
| 4-position control | `boil`: 6, 8, 12 fps |

**Structure.**

- **Verse:** Grey Trial palette, 4 bars, boil 8.
- **Build:** over four bars, shatter steps up, rough rises from 0.3 to 0.8, boil goes to 12 fps, bars grow from 3 to 7.
- **Drop:** hard cut to Strict B/W, icon whole, four bars slam in from all sides, then hold still for one full bar.
- **Break:** one bar, one line of type in slice mode, boil 6, icon hidden or whole.
- **Phrase change (8 or 16 bars):** change palette and icon behind a spiral wipe or a full-frame bar slam.

**Tips.**

- Keep white under control. Cap white near #ECE5D5, black floor around #10-#15, run the wall at 50-70 % brightness, and keep paper bars under about 40 % of the screen in bright palettes.
- Run at native wall resolution. Scaled flat shapes turn soft.
- Size everything in output height or pixels, not UV, so 3:1 walls and 16:9 screens look alike.
- The boil costs bitrate on NDI or low-bitrate streams. Lower `rough` for streamed output.

## Variations

1. **Anatomy of a kick drum:** a figure or abstract symbol breaks into cut-paper pieces on each kick and reassembles on the drop.
2. **Bar sequencer:** bars driven by a step sequencer, BeatStep Pro pads set directions, type or logo revealed through the bars.
3. **Paper stop-motion clock:** 8 to 12 fps jitter on every edge so cut-outs wobble like real paper.
4. **Palette from the room:** three colours switched by MIDI notes, poster colours against strict black and white.
5. **Spiral hand-off:** a spiral as the transition layer between cut-paper scenes.
6. **Cut-out window:** show camera or clip only inside the icon, posterised to the same ink and paper colours.

## Did you know?

- Director Otto Preminger sent a note to theatres asking projectionists to start the first reel only after the curtains were drawn back, so the titles were seen properly ([Art of the Title](https://www.artofthetitle.com/title/the-man-with-the-golden-arm/)).
- Psycho's title was designed on a $21k budget ([Art of the Title](https://www.artofthetitle.com/title/psycho/)).
- *Why Man Creates* won the 1968 Oscar for Documentary Short Subject ([Wikipedia](https://en.wikipedia.org/wiki/Saul_Bass)).
- On West Side Story, Saul and Elaine Bass wrote their initials into the graffiti ([Criterion](https://www.criterion.com/current/posts/6937-credit-where-it-s-due-the-father-of-the-title-sequence)).

## Sources

- [Wikipedia: Saul Bass](https://en.wikipedia.org/wiki/Saul_Bass)
- [BFI: 10 essential Saul Bass title sequences](https://www.bfi.org.uk/lists/celebrating-saul-basss-centenary-10-essential-title-sequences)
- [Art of the Title: Saul Bass](https://www.artofthetitle.com/designer/saul-bass/)
- [Art of the Title: Anatomy of a Murder](https://www.artofthetitle.com/title/anatomy-of-a-murder)
- [Art of the Title: Psycho](https://www.artofthetitle.com/title/psycho/)
- [Art of the Title: The Man with the Golden Arm](https://www.artofthetitle.com/title/the-man-with-the-golden-arm/)
- [Criterion: Credit where it's due](https://www.criterion.com/current/posts/6937-credit-where-it-s-due-the-father-of-the-title-sequence)
- [Motion Design Archive: Anatomy of a Murder](https://luckydumpling.github.io/motion-archive/entries/saul-bass-anatomy/)
- [Inigo Quilez: 2D distance functions](https://iquilezles.org/articles/distfunctions2d/)
- [The Book of Shaders: Noise](https://thebookofshaders.com/11/)
- [Resolume: ISF support](https://resolume.com/support/en/isf)
- [Unreal Engine: Remote Control protocols](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-protocols-in-unreal-engine)
