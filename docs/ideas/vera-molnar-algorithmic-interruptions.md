---
tags:
  - art-history
  - generative
  - grid
  - geometry
  - computer-art
---

# Vera Molnar Algorithmic Interruptions

> A strict grid of thin lines, a few that disobey, and a kick that shatters the rule and lets it heal.

## The idea

Vera Molnár started with a rigid structure, such as a grid of lines or squares, and let a small algorithm decide the fate of each cell: its position, rotation, size, or whether it is drawn at all. Randomness was fed into a logical rule, not left to chance alone ([CMU IDeATe](https://courses.ideate.cmu.edu/15-104/f2018/2018/10/05/vera-molnars-generative-art)). Her stated aim was a "disturbed equilibrium": an abstract composition works when it mixes order and disorder ([Morgan](https://www.themorgan.org/morganmobile/order-and-disorder/molnar)).

This chapter rebuilds that method as a live generator. One field of lines fills the screen, one line per cell. Disorder amount, grid density and rule choice become controls. The eye hunts for the few deviations, so the image holds attention with very little on screen.

On a club screen the strength is contrast. Long stretches of perfect order make each rupture hit harder. A kick breaks the grid, the grid repairs itself over a few beats, and the repair is the payoff.

## Origins and references

- **Vera Molnár** (born Budapest, 1924, died Paris, 2023) co-founded the Groupe de Recherche d'Art Visuel (GRAV) in 1960 ([Wikipedia](https://en.wikipedia.org/wiki/Vera_Moln%C3%A1r)). From the late 1950s she worked by hand on grid paper with self-set rules, her "machine imaginaire" ([V&A](https://www.vam.ac.uk/blog/museum-life/vera-molnar-machine-imaginaire-the-dance-of-hands-and-machine-thinking), [Artnet](https://news.artnet.com/art-world/in-pictures-vera-molnar-pompidou-2446604)).
- In 1968 she gained access to a computer lab, taught herself Fortran and made plotter drawings. The Pompidou describes them as the first digital drawings with a plotter-linked computer by an artist in France ([Morgan](https://www.themorgan.org/morganmobile/order-and-disorder/molnar), [Toledo Museum](https://infiniteimages.toledomuseum.org/?p=811), [Centre Pompidou](https://www.centrepompidou.fr/en/program/calendar/event/PA7jRZ5)).
- ***Interruptions*, 1968**: black ink on plotter paper, a 25 x 25 grid of lines, each randomly rotated, with random interruptions that erase sections and leave voids ([Toledo Museum](https://infiniteimages.toledomuseum.org/?p=811), [Morgan](https://www.themorgan.org/morganmobile/order-and-disorder/molnar)).
- ***Interruptions*, 1969**: a plotter drawing in the National Gallery of Art, Washington ([NGA](https://www.nga.gov/artworks/216160-interruptions)). The V&A holds a 1969 *Interruptions* too, plus *(Des)Ordres* and *Structure of Squares* (both 1974) ([V&A](https://www.vam.ac.uk/blog/museum-life/vera-molnar-machine-imaginaire-the-dance-of-hands-and-machine-thinking)).
- ***Interruptions à recouvrements*, 1969**: a layered series reusing the logic with varied scale, shape and line thickness. *Structure de quadrilatères* (1987) is a related square series ([CMU IDeATe](https://courses.ideate.cmu.edu/15-104/f2018/2018/10/05/vera-molnars-generative-art)).
- ***Speak to the Eye*, Centre Pompidou, 2024**: the retrospective, with *Four Randomly Distributed Elements* (1959), *A Stroll Between Order and Chaos* (1975) and *160 Squares Pushed to the Limit* (1976) ([Centre Pompidou](https://www.centrepompidou.fr/en/program/calendar/event/PA7jRZ5), [Artnet](https://news.artnet.com/art-world/in-pictures-vera-molnar-pompidou-2446604)).

## Visual language

Thin lines or squares on a plain ground, in a dense, even grid. Most cells obey the rule. A few break it: a tilted line, a hole, an overlapping square ([CMU IDeATe](https://courses.ideate.cmu.edu/15-104/f2018/2018/10/05/vera-molnars-generative-art)).

In the calm state all lines point the same way and sit end to end, like hatching on graph paper. When the rule loosens, every line tilts a little, a few tilt a lot, and soft-edged holes open where whole patches are missing, as if the paper had worn away. Lines next to a hole lean further than the rest, an observation from re-codes of the piece ([Re-Code, Medium](https://medium.com/@xuetingzhang313/re-code-vera-moln%C3%A1rs-interruptions-b6dc9765eae5)).

A second rule set swaps lines for square outlines with jittered corners, after *Interruptions à recouvrements* and *Structure de quadrilatères*. The work is serial: many variations of one rule. Colour is a fixed pair per scene. For LED walls the plotter look is inverted to pale ink on near-black, with black-on-paper as a daylight preset. The mood is dry and exact.

## See it

- [Interruptions, 1968, Toledo Museum](https://infiniteimages.toledomuseum.org/?p=811) — museum page with the plotter drawing and method text
- [Interruptions, 1968, The Morgan](https://www.themorgan.org/morganmobile/order-and-disorder/molnar) — page on the 32 x 32 cm plotter drawing
- [Interruptions, 1969, National Gallery of Art](https://www.nga.gov/artworks/216160-interruptions) — collection record of the 1969 plotter drawing
- [Machine Imaginaire, V&A blog](https://www.vam.ac.uk/blog/museum-life/vera-molnar-machine-imaginaire-the-dance-of-hands-and-machine-thinking) — hand and plotter works including Interruptions and (Des)Ordres
- [Speak to the Eye in pictures, Artnet](https://news.artnet.com/art-world/in-pictures-vera-molnar-pompidou-2446604) — photos of the 2024 Pompidou retrospective works

## Code design

One generator draws the whole image in a single pass: per pixel, find the cell, run the cell's rule, test the 3 x 3 neighbourhood for anti-aliased segments or squares, and output ink plus an erasure mask. State lives in uniforms only, so no feedback buffers are needed.

- **Grid**: cells are square on any aspect ratio. Pitch is the short side divided by density (8 to 64, default 25). A 32:9 wall gets more columns, not stretched lines.
- **Rule per cell**: hashes from cell id and seed give an angle (biased so most cells stay near the rule), a displacement and a length. Integer hashes give identical results on every GPU; see [The Book of Shaders, ch. 10](https://thebookofshaders.com/10/).
- **Interruptions**: clustered erasure from low-frequency noise, not per-cell coin flips, as in the Perlin-noise re-code ([Medium](https://medium.com/@xuetingzhang313/re-code-vera-moln%C3%A1rs-interruptions-b6dc9765eae5)). A per-cell hash ragged-edges the holes. Cells near a hole get extra disorder.
- **Disorder field**: global, a sweeping band, a per-cell heal after each kick, or the luminance of live footage per cell. A small base disorder (0.05 to 0.15) keeps the grid alive.
- **Seed crossfade**: seeds A and B with a snap time from 0 (hard cut like a new plotter sheet) to one beat, so holes slide instead of popping.
- **Lines and squares**: segment and box distance functions ([Inigo Quilez](https://iquilezles.org/articles/distfunctions2d/)), 2 px line weight at 1080p, scaled with output height.
- **Layers**: ground, faint grain, ghost grid at 8 to 12 % (a memory of perfect order that makes deviations readable), main grid in Normal blend so crossings never glow, optional accent pen on 0 to 5 % of cells, optional footage window masked by the holes. No bloom, blur or feedback on the main ink.
- **Colour**: stage pair ground #0B0B0C and ink #ECE6D8; paper pair #F1ECE2 and #141414; accent pens red, blue, green. A salmon preset follows the white-on-salmon *Structure de quadrilatères* ([CMU IDeATe](https://courses.ideate.cmu.edu/15-104/f2018/2018/10/05/vera-molnars-generative-art)).

=== "Resolume Wire"
    Build the generator as an ISF node, with two passes if footage is used and one if not ([ISF multi-pass reference](https://docs.isf.video/ref_multipass.html)). Resolume documents ISF for Wire ([Resolume: ISF in Wire](https://resolume.com/support/en/isf)), so Wire hosts it and exports it as a source for Arena.

    - Derive `time` and `beatPhase` from the patch time and Arena's BPM. Convert beats to seconds with `seconds = beats * 60 / BPM`.
    - A `reseed` event drives a counter: the counter is seed B, the previous value is seed A. A ramp over the snap time feeds snapMix.
    - A `shatter` event (or an audio-threshold event) stores the current time as tKick.
    - A one-bar sawtooth from the BPM phase, scaled to grid width, drives the sweep position.
    - Exposed: disorder, interrupt, density, rule, heal, weight, fieldMode, paletteIndex, accentShare, and events reseed, shatter, invert. A second output carries the mask channel in grey.

    In Arena, put a solid ground on layer 1, the Wire source on layer 2 (Alpha blend), and an optional second instance with disorder 0 and interrupt 0 at 10 % opacity in Add as the ghost grid. Bind disorder to the Audio tab's low band and reseed to a one-bar BPM sync.
=== "Unreal Engine 5.7"
    Use an unlit material with one Custom node holding the neighbour loop, segment distance and hash. Feed it from ScalarParameter and VectorParameter nodes, and collect the scalars in a Material Parameter Collection so Blueprints and Remote Control reach them in one place.

    - A Blueprint actor handles seeds: on reseed, copy seed B to A, write a new B, and drive snapMix with a Timeline. On shatter, write the game time to tKick.
    - Draw the material into a Render Target with Draw Material to Render Target at output resolution.
    - Niagara alternative: one emitter spawning density squared particles on a grid, with a thin box mesh per line, angle from a hash in a Scratch Pad module, erased cells scaled to 0. Lines can then tilt in depth too.
    - Expose the scalars and the reseed and shatter functions in a Remote Control Preset, bound to OSC addresses such as `/molnar/disorder` and to MIDI CCs.
    - Send the render target out via Spout or NDI so Resolume can use it as a source or a mask.
=== "GODOT"
    A ColorRect fills the viewport with a `canvas_item` ShaderMaterial and works in pixel space with `FRAGCOORD`.

    - Uniforms: density, disorder, interrupt, gap_scale, gap_boost, length_ratio, weight, rule, theta0, seed_a, seed_b, snap_mix, field_mode, t_kick, heal, sweep_pos, colours with `source_color`, and an optional footage sampler.
    - A GDScript on the ColorRect tweens snap_mix on reseed and tracks time since the kick for heal.
    - An AudioEffectSpectrumAnalyzer on the music bus reads 40 to 120 Hz and fires shatter on threshold crossings.
    - For the mask, render the same shader in a SubViewport with a `mask_only` uniform and use its ViewportTexture to mask footage.

## On stage

**Audio reactivity**

| band | drives | notes |
|------|--------|-------|
| Kick, 40–120 Hz | shatter: sets tKick, kickAmount 0.6 | attack 0 ms, retrigger lockout 120 ms; onset above 1.4 x the 1 s running average |
| Bass, 60–250 Hz | base disorder +0 to +0.2 | attack 150 ms, release 800 ms; sustained bass loosens the grid |
| Mids, 500 Hz–2 kHz | interrupt +0 to +0.08 | attack 300 ms, release 2 s; holes grow, never jump |
| Hats, 6–12 kHz | erase or restore 1–3 random cells for a 1/16 note | cap at 3 cells so it stays an accent |
| Loudness | nothing | Molnar is about rules, not volume |

Without audio analysis, gate shatter from the BeatStep Pro kick track and reseed from Arena's BPM, one per bar.

**MIDI mapping**

- BeatStep Pro knobs: 1 disorder, 2 interrupt, 3 rule (0 lines, 0.5 overlap, 1 jittered squares), 4 heal (0.25 to 8 beats, synced to BPM), 5 weight (1 to 6 px at 1080p).
- KeyStep 37 keys: one octave steps density through 8, 13, 19, 25, 31, 37, 49, 64. Change on a phrase boundary.
- BeatStep Pro drum pads: reseed, shatter, invert (one-beat colour flip), orient (vertical or horizontal).

**Motion over a set**

- Base state: disorder 0.08, interrupt 0.12, only a slow drift of the hole noise, so holes creep like erosion.
- Per kick: heal mode, kickAmount 0.6, heal 1.5 beats, snap 60 ms.
- Per bar: new seed with a one-beat snap.
- Per phrase (8 or 16 bars): step density or flip orientation with a 2-beat snap.
- Build-up: ramp disorder 0.08 to 0.5 and interrupt 0.12 to 0.35 over 8 bars. Switch to sweep mode, crossing in 1 bar, then 2 beats, then 1 beat.
- Drop: snap 0, new seed, jump to squares or the overlap blend, invert for one beat, then let disorder fall to 0.15 over 2 bars. The return to order is the payoff.
- Break: disorder 0, interrupt 0, ghost off, a perfect grid. Let interrupt rise from 0 to 0.25 over 16 bars so holes appear one by one.

**Layering and tips**

- It is line art on a flat ground, so it keys cleanly over footage. Send the mask channel by Spout or NDI as a stencil for other layers or machines.
- Hold one fixed rule at constant disorder and it becomes a screensaver. Pure randomness loses the tension. Keep long order, short ruptures.
- Render at native output resolution; upscaling softens the plotter lines. Use 4 px weight at 2160p.
- Moiré risk: on low-resolution LED walls (3.9 mm pitch or more), keep cell pitch above 12 wall pixels, so `density ≤ shortSide / 12`.
- Thin saturated red lines at 2 px can break up on subpixel layouts. Use accent pens sparingly.
- Compute the hole noise per cell, not per pixel, when density is fixed. Do the ghost grid in the same pass rather than as a second Arena layer.

## Variations

- **Travelling wave**: a band of chaos sweeps across the grid on the beat and order returns behind it.
- **Healing grid**: broken cells return to their rule over a few bars, so order and disorder breathe with the track.
- **Rule chain on a fader**: crossfade between rotate, erase, overlap and size rule sets like clips.
- **Pen-plotter palette**: a new accent pen per hi-hat or per bar, advancing every 4 bars or on a pad.
- **Footage-driven disorder**: live camera or clip luminance sets each cell's disorder, so the image appears as a pattern of tilted lines and shows through the holes.
- **Salmon preset**: white ink on salmon ground (#E59A82, #FFF6EE) after *Structure de quadrilatères*.
- **3D grid**: the Niagara build lets lines tilt in depth as well as in the plane.

## Did you know?

- *Interruptions* (1968) starts from a 25 x 25 grid of randomly rotated lines, and the Morgan's version measures 32 x 32 cm ([Toledo Museum](https://infiniteimages.toledomuseum.org/?p=811), [Morgan](https://www.themorgan.org/morganmobile/order-and-disorder/molnar)).
- She had worked out her algorithmic process by hand for years before she had computer access ([Toledo Museum](https://infiniteimages.toledomuseum.org/?p=811), [V&A](https://www.vam.ac.uk/blog/museum-life/vera-molnar-machine-imaginaire-the-dance-of-hands-and-machine-thinking)).
- She learned both Fortran and BASIC and ran work via punch cards and a plotter ([Artnet](https://news.artnet.com/art-world/in-pictures-vera-molnar-pompidou-2446604), [Wikipedia](https://en.wikipedia.org/wiki/Vera_Moln%C3%A1r)).
- Her 22-volume "Diary" holds diagrams, photos and documents recording her works ([Centre Pompidou](https://www.centrepompidou.fr/en/program/calendar/event/PA7jRZ5)).

## Sources

- [V&A blog](https://www.vam.ac.uk/blog/museum-life/vera-molnar-machine-imaginaire-the-dance-of-hands-and-machine-thinking)
- [Toledo Museum, Interruptions](https://infiniteimages.toledomuseum.org/?p=811)
- [The Morgan, Interruptions](https://www.themorgan.org/morganmobile/order-and-disorder/molnar)
- [NGA, Interruptions](https://www.nga.gov/artworks/216160-interruptions)
- [CMU IDeATe, Vera Molnar's Generative Art](https://courses.ideate.cmu.edu/15-104/f2018/2018/10/05/vera-molnars-generative-art)
- [Centre Pompidou, Speak to the Eye](https://www.centrepompidou.fr/en/program/calendar/event/PA7jRZ5)
- [Artnet, in pictures](https://news.artnet.com/art-world/in-pictures-vera-molnar-pompidou-2446604)
- [Wikipedia, Vera Molnár](https://en.wikipedia.org/wiki/Vera_Moln%C3%A1r)
- [Re-Code: Vera Molnár's Interruptions (Medium)](https://medium.com/@xuetingzhang313/re-code-vera-moln%C3%A1rs-interruptions-b6dc9765eae5)
- [CMU 60-212 Interruptions gallery](https://ems.andrew.cmu.edu/2018_60212f/deliverables/due-9-7/interruptions-gallery)
- [The Book of Shaders, ch. 10](https://thebookofshaders.com/10/)
- [Inigo Quilez, 2D distance functions](https://iquilezles.org/articles/distfunctions2d/)
- [ISF Multi-Pass Reference](https://docs.isf.video/ref_multipass.html)
- [Resolume: ISF in Wire](https://resolume.com/support/en/isf)
