---
tags:
  - tiling
  - ornament
  - generative
  - shader
---

# Truchet Tile Weaving

> A grid of simple tiles, each turned at random, knots itself into endless paths, and a wave of rotation on every bar ties new knots.

## The idea

A Truchet tile is a square with a pattern that is not rotationally symmetric. Its orientation carries information ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)). The quarter-circle version has two orientations. Two arcs join the midpoints of adjacent sides. Because the arcs always meet at edge midpoints, lines run across cell borders and close into loops, knots and long paths.

The rule is tiny: hash each cell's coordinate, let the hash pick the orientation. A random square Truchet tiling is equivalent to bond percolation at p = 1/2 ([Chalkdust](https://chalkdustmagazine.com/features/when-truchet-met-chladni/)). That is why the paths are neither too short nor too long.

On a club screen this gives a strict grid rhythm with organic flow on top. Turn tiles in a wave and the paths break and close again in new loops. The break is the visible event. A flat static version gets boring on a big wall, so scale and probability must keep moving.

## Origins and references

- Sébastien Truchet, 1704: square tiles split along the diagonal into two contrasting triangles, in his "Mémoire sur les combinaisons" ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)).
- Cyril Stanley Smith, 1987: popularised the tiles and introduced the quarter-circle variant. It had already appeared in the games Trax and the Black Path Game ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)).
- Christopher Carlson, "Multi-Scale Truchet Patterns", Bridges 2018: a "winged" tile and an infinite set of tiles scaled by powers of 1/2 ([Bridges 2018](https://archive.bridgesmathart.org/2018/bridges2018-39.html)).
- Lionel Radisson (makio135), Observable notebook, 24 March 2021: implements the multi-scale idea ([Observable](https://observablehq.com/@makio135/multi-scale-truchet-patterns)).
- Nick Montfort calls the one-line Commodore 64 BASIC program that draws the maze version "a found poem" ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)).
- The Art of Code, "Shader Coding: Truchet Tiling Explained!" (2019): vine-like Truchet patterns in a shader ([YouTube](https://www.youtube.com/watch?v=2R7h76GoIJM)).

## Visual language

Over a near-black ground, warm bone-white ribbons run in quarter-circle arcs from edge to edge. They join into winding paths, closed rings and knots. Each ribbon is shaded like a soft tube: dark rim, bright spine. The two regions the lines cut the plane into are filled in two very dark tones, ink and deep indigo, so the tiling also reads as a two-colour maze. Smith's tilings can be coloured this way ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)).

In weave mode some cells hold a crossing tile. One strand passes over the other, alternating like a checkerboard, and the plane reads as woven cloth. Thin orange dashes travel along the paths like current in a circuit.

Rhythm comes from two sources. The grid is calm and exact. The random orientation is organic. Each bar a wave of 90° turns breaks the paths and closes them again. On the drop, cells split into halves and quarters, so the ornament suddenly gets finer. The palette is fixed: ground #07070B, indigo #141633, bone #E8E0CC, accent #FF5A1F, and cyan #1FC8FF swapped in on the drop.

## See it

- [Shader Coding: Truchet Tiling Explained!](https://www.youtube.com/watch?v=2R7h76GoIJM) — video tutorial by The Art of Code; builds vine-like Truchet patterns in a shader.
- [Multi-scale Truchet patterns](https://observablehq.com/@makio135/multi-scale-truchet-patterns) — Observable notebook by Lionel Radisson (makio135), multi-scale variant.
- [Truchet tile (Wikipedia)](https://en.wikipedia.org/wiki/Truchet_tile) — tile variants and example tilings.
- [Multi-Scale Truchet Patterns (Bridges 2018)](https://archive.bridgesmathart.org/2018/bridges2018-39.html) — paper by Christopher Carlson with figures.

## Code design

One stateless fragment shader does everything: fit the grid to the short side, hash the cell id, mirror the cell by the hash, measure the distance to the nearest quarter-circle arc and shade it as a tube. The path coordinate runs along the arc. A checker sign makes neighbouring cells run in opposite directions, so the dashes stay seamless. Region parity is checker XOR flip XOR inside. Weave mode swaps some cells for crossing tiles with alternating over/under and a soft shadow. The re-knot wave keeps two seeds. Only cells whose orientation differs between them rotate, each with a delay based on its distance from an origin. Subdivision splits cells into two more levels at most, with halved ribbon width per level. The plain version does not join perfectly at scale borders. For the exact Carlson look, follow the [paper](https://archive.bridgesmathart.org/2018/bridges2018-39.html) or [makio135's notebook](https://observablehq.com/@makio135/multi-scale-truchet-patterns).

Layer stack: solid ground, region fill, ribbons (one shader with alpha-over), flow dashes in Add, optional video through the ribbon matte, optional trail feedback (previous frame at 0.85, Lighten), then post: bloom, chromatic aberration on the drop only, grain.

=== "Resolume Wire"
    - Beat trigger into a bar `Counter`. On each bar: sample-and-hold a `Random` into seedB, another pair into the wave origin, and reset a ramp.
    - The ramp (time delta times wave speed, clamped 0..1) becomes `waveProgress`. At 1, copy seedB into seedA.
    - `Audio Analysis` low band into an `Envelope` (attack 10 ms, release 180 ms), scaled by 0.3, added to width. High band into a fast envelope (attack 2 ms, release 60 ms) drives dash gain.
    - An `ISF` node holds the Truchet shader, followed by an optional feedback loop and a bloom chain.
    - Exposed: density, flip, width, weave, split, flow, reknot, drop. Node names differ between Wire versions.

    In Arena, load the same shader as an ISF source ([Resolume ISF support](https://resolume.com/support/en/isf)) and animate `waveProgress` with BPM sync. A rough fallback with no custom shader is a single tile image run through a Tile effect. It gives Smith's regular pattern, not the random maze.

=== "Unreal Engine 5.7"
    - Material `M_Truchet`, Post Process domain or Unlit on a plane. One Custom (HLSL) node holds the whole pattern. Parameters sit in a Material Parameter Collection (Density, Flip, Width, Weave, Split, Flow, seeds, origin, wave speed, colours).
    - Without HLSL, the graph can do the core: floor and frac of screen position, a hash from `Dot`, `Sine` and `Frac`, an `If` to mirror, `Length`, `Abs` and `SmoothStep` for the ribbon. Rotation and subdivision are far easier in the Custom node.
    - 3D use: the ribbon mask as World Position Offset on a subdivided plane gives embossed metal ornament. Or render to a Render Target and use it as a Niagara mask, so beads run along the paths.
    - A Blueprint ticks time into the MPC and, on a bar event from OSC or MIDI clock, sets seedB, origin and start time.
    - Remote Control preset on the MPC scalars, mapped via the OSC and MIDI protocols ([Epic docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-protocols-in-unreal-engine)).
    - Post: bloom 0.8, threshold 0.75, manual exposure, film grain 0.03. Output via Spout plugin or NDI into Resolume.

=== "GODOT"
    - A full-viewport `ColorRect` with a `canvas_item` shader holds the pattern. Uniforms mirror the list above, colours with `source_color`. GDScript accumulates time and sets seeds on beat events.
    - Feedback: a `SubViewport` that is never cleared, plus a second `ColorRect` reading the previous frame at decay 0.85 with `max()`.
    - Post: `WorldEnvironment` Glow with 2D HDR, threshold 0.75.
    - Audio: `AudioEffectSpectrumAnalyzer`, 40–120 Hz for width, 6–12 kHz for dash flashes.
    - MIDI: `InputEventMIDI` maps BeatStep Pro CCs to the uniforms.

## On stage

**Audio reactivity**

| band | drives | smoothing |
|------|--------|-----------|
| kick 40–120 Hz | width +30 %, optional 5–10 % instant flips | attack 10 ms, release 180 ms |
| low-mid 200–800 Hz | dash speed ×1–1.5 | attack 80 ms, release 400 ms |
| hi-hat 6–12 kHz | dash gain ×2 flash | attack 2 ms, release 60 ms |
| full-band RMS, long average | split and density on the drop | attack 1 s, release 4 s |

Sync the re-knot to BPM, not to audio. A re-knot on every detected kick is too busy.

**MIDI mapping (BeatStep Pro knobs, KeyStep 37 mod strip)**

| control | parameter | note |
|---------|-----------|------|
| knob 1 or mod strip | density 4–40 | zoom from bold loops to fine lace |
| knob 2 | flip 0–1 | 0.5 is the endless maze, soft-takeover |
| knob 3 | width 0.02–0.30 | plus kick envelope |
| knob 4 | weave 0–0.6 | maze to woven cloth |
| knob 5 | split 0–0.6 | finer ornament |
| pad | reknot | fires the rotation wave and moves to the next seed |

**Timing.** Constant motion: dashes at 0.25–1.0 path lengths per beat and a slow diagonal pan of 0.02–0.1 cells per second. Re-knot every bar or two, crossing the screen in one beat. Each phrase, change the wave pattern, for example centre to corners or a horizontal wipe. Drop: split 0 to 0.5 over one beat, accent to cyan, chromatic aberration on, density up one step. Break: split back to 0 over two bars, fill fades out, feedback up to 35 %, re-knot every four bars.

**Layering.** Use the ribbon mask as a luma matte for video, so the strands become a window. Feedback at 20–40 % on breaks leaves arcs of light behind rotating tiles.

**Tips.**

- Keep ribbons at least 1.5 px wide and cells at least 24 px on the wall, or they shimmer when the grid pans.
- Avoid cell sizes near a multiple of the LED module size. The slow pan breaks up moiré.
- On wide strips, use 3–5 cells across the short side.
- Wrap the pan with `mod(pan, 1000)` or use an integer hash. Cheap sin hashes show patterns at large ids.
- Never re-roll all tiles in one frame on a big wall. The rotation wave is the softer way.
- Cap the ribbon near 90 % luminance on LED walls, and check the indigo on the wall, since LED gamma crushes it.

## Variations

- Flip probability away from 0.5: long diagonal rivers or tight closed rings.
- Multi-scale subdivision triggered by the kick: cells split in half on hits and merge back.
- Mix with the diagonal or labyrinth tile ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)).
- Hexagonal Truchet tiles for a softer, non-square look ([Chalkdust](https://chalkdustmagazine.com/features/when-truchet-met-chladni/)).
- Use the pattern as a displacement or texture source in Unreal via Spout.

## Did you know?

- Truchet described the tiles in 1704, and he died in 1729, long before Chladni was born in 1756 ([Chalkdust](https://chalkdustmagazine.com/features/when-truchet-met-chladni/)).
- In 2002 Bogomolny and Schmit conjectured that random Chladni figures statistically match random Truchet tilings at large scales ([Chalkdust](https://chalkdustmagazine.com/features/when-truchet-met-chladni/)).
- The classic 10 PRINT one-liner on the Commodore 64 draws the labyrinth version ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)).
- Carlson's multi-scale tiles use powers of 1/2 ([Bridges 2018](https://archive.bridgesmathart.org/2018/bridges2018-39.html)).

## Sources

- [Truchet tile, Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)
- [When Truchet met Chladni, Chalkdust](https://chalkdustmagazine.com/features/when-truchet-met-chladni/)
- [Multi-Scale Truchet Patterns, Bridges 2018](https://archive.bridgesmathart.org/2018/bridges2018-39.html)
- [Multi-scale Truchet patterns, Observable](https://observablehq.com/@makio135/multi-scale-truchet-patterns)
- [Shader Coding: Truchet Tiling Explained!, YouTube](https://www.youtube.com/watch?v=2R7h76GoIJM)
- [Resolume ISF support](https://resolume.com/support/en/isf)
- [Remote Control protocols, Epic docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-protocols-in-unreal-engine)
