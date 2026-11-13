---
tags:
  - math
  - tiling
  - geometry
  - ornament
---

# Penrose Aperiodic Tilings

> A floor of rhombi that looks ordered everywhere and repeats nowhere, shimmering as it re-decides itself on the beat.
<!-- LEAD-IMAGE -->

## The idea

A Penrose tiling covers the whole plane with just two shapes and never repeats. Roger Penrose worked it out in the 1970s. His first version (1974) used six pentagon-based tiles. He cut it down to two: kites and darts, or two rhombi. Robert Ammann found the rhombus version independently in 1976 ([Wikipedia: Penrose tiling](https://en.wikipedia.org/wiki/Penrose_tiling)). Martin Gardner brought it to a wide public in *Scientific American* in January 1977 (same source).

The tiling has local five-fold stars and decagons, but they never line up globally. At most one point of exact five-fold symmetry exists ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling)). Along any chain of tiles that share an edge direction, the ratio of wide to thin tiles is the golden ratio, which is irrational. That is why the pattern cannot repeat ([Minutephysics via Rosetta](https://rosetta.to/u/minutephysics/why-penrose-tiles-never-repeat)).

On a club screen this is useful. The eye keeps finding patterns that dissolve. A static tiling slowly zoomed gets boring fast, but this one has several natural handles for motion: growth by subdivision, sliding hidden offsets, rotation, colour by tile type. The tiling is also infinite and has no period, so any aspect ratio works without seams. That suits strips and odd LED shapes.

## Origins and references

- **Roger Penrose and Robert Ammann**: the P1 (1974), kite-and-dart and rhombus tilings; Gardner's 1977 article. Roots go back to Kepler (1619) and the 1960s aperiodic tile sets of Hao Wang and Robert Berger ([Wikipedia: Penrose tiling](https://en.wikipedia.org/wiki/Penrose_tiling)).
- **N. G. de Bruijn (1981)**: two constructions, the multigrid (pentagrid) method and cut-and-project from a 5D cubic lattice ([Wikipedia: Penrose tiling](https://en.wikipedia.org/wiki/Penrose_tiling), [Wikipedia: Quasicrystal](https://en.wikipedia.org/wiki/Quasicrystal), [Wikipedia: Pentagrid](https://en.wikipedia.org/wiki/Pentagrid)).
- **Dan Shechtman**: saw ten-fold electron diffraction in an Al-Mn alloy in 1982, published in 1984, and won the 2011 Nobel Prize in Chemistry. These are quasicrystals ([Wikipedia: Quasicrystal](https://en.wikipedia.org/wiki/Quasicrystal)).
- **Max Cooper, "Penrose Tiling"** (2019, album *Yearning for the Infinite*): visuals by Jessica In. A tiling grows in spinning trails that look random at first and slowly reveal their structure, with several scales layered. The track uses only prime-number loop lengths, so no bar repeats exactly. The visuals were made for a room's roof, walls and floor ([Creative Commission](https://creative-commission.com/project/max-cooper/music-video/jessin-0), [Partyflock](https://partyflock.nl/video/194348:muziekvideo-Penrose-Tiling-Max-Cooper)).
- **Girih and the Darb-i Imam shrine, Isfahan (1453)**: Peter Lu and Paul Steinhardt (2007) argued that Penrose-like tiling underlies medieval Islamic girih decoration ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling)).
- **Architecture**: a terrazzo Penrose floor at Miami University (1979), the Oxford Mathematics building entrance, a Helsinki pedestrian street (2014), the Salesforce Transit Center in San Francisco (2018) ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling)).
- **Clark Richert, Drop City (1970)**: used similar rhombs before Penrose published ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling)).

## Visual language

Two rhombus shapes, thick (72/108 degrees) and thin (36/144 degrees), or kites and darts ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling)). Picture the frame filled with thin cool-white outlines on near-black. The fills are dim: deep indigo for thick tiles, old gold for thin ones. Edges carry the image; fills only tint it.

The floor is never still. Hidden grid offsets drift, and small groups of tiles flip in place, so the pattern shimmers like a crystal re-deciding its structure. On snares a teal band sweeps along one ribbon, the wavy chain of tiles that share an edge direction. Hi-hats spark single vertices. On the kick each tile splits into its golden-ratio children for a beat, showing the self-similarity.

A second scene falls into the centre of a ten-fold sun, with spinning trails behind the edges. It follows the Cooper video: trails that look random, then reveal structure, with several scales stacked. The mood is crystalline and hypnotic: order you can feel but never pin down.

## See it

- [Max Cooper, Penrose Tiling (Creative Commission)](https://creative-commission.com/project/max-cooper/music-video/jessin-0) — project page for the Jessica In music video: growing tiling with spinning trails (no video embedded).
- [The Infinite Pattern That Never Repeats (YouTube, Veritasium)](https://www.youtube.com/watch?v=48sCx-wBs34) — explainer with animated tilings and quasicrystals.
- [Why Penrose Tiles Never Repeat (YouTube, minutephysics)](https://www.youtube.com/watch?v=-eqdj63nEr4) — pentagrid and ribbon animation.
- [Penrose Tiling (Rhombi), Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Penrose_Tiling_(Rhombi).svg) — thick and thin rhombus tiling with five-fold symmetry.

## Code design

!!! warning "Status: designed, not run"
    Nothing here has been run or benchmarked. The designer derived three parts of the pentagrid maths itself: the grid-point estimate `z0` with its offset `c`, the claim that a 3x3 search window is enough, and the reading of the phason vector as a 5D slice shift. Treat them as a build plan. First check the shader for holes: the design outputs magenta where no tile is found, so any magenta pixel means the search window or the `z0` estimate is wrong.

The main scene is the de Bruijn pentagrid, computed per pixel in one fragment pass, stateless and with no CPU geometry. Five families of parallel lines, each rotated 72 degrees from the last, have offsets that sum to zero. Each crossing of two lines becomes one rhombus. The definitions follow [Le Bruyn's summary of de Bruijn's pentagrids](https://lievenlebruyn.github.io/neverendingbooks/de-bruijns-pentagrids). The shader guesses the grid point behind each pixel, searches nearby line pairs for the crossing, rebuilds the rhombus corners, and runs an inside test. It outputs edge distance, shape class, ribbon index and a vertex spark.

Animation comes from the offsets. Five slow LFOs at incommensurate rates (0.031 to 0.083 Hz) plus a phason vector make small groups of tiles flip; the mean is removed every frame so the result stays a Penrose tiling. Moving the phason vector is the cut-and-project idea of sliding a slice through 5D ([Wikipedia: Quasicrystal](https://en.wikipedia.org/wiki/Quasicrystal)). Never set all offsets equal: that is a singular grid and leaves holes. Changing the number of line families gives other symmetries (4 for 8-fold, 6 for 12-fold, 7 for 14-fold); 3 gives the periodic "stacked cubes" pattern as a contrast. The number cannot morph continuously, so switch on the bar line with a crossfade.

The growth scene uses Robinson-triangle subdivision per pixel, with rules from [Preshing](https://preshing.com/20110831/penrose-tiling-explained/), run the way the [shadcn Penrose shader](https://www.shadcn.io/shaders/penrose-tiling-shader) describes: each pixel keeps only the child triangle that contains it, for about 10 steps. Level L edges are bright and level L+1 edges are a ghost. Scaling the view by phi gives an endless zoom. Trails are a feedback buffer that rotates, zooms and decays each frame.

Render order: P1 pentagrid, P2 growth, P3 trail feedback, P4 composite, P5 post (bloom, grain, vignette). Everything above the background is additive.

=== "Resolume Wire"
    Resolume documents ISF only inside Wire ([Resolume ISF support](https://resolume.com/support/en/isf)), so build the shaders in Wire and bring them into Arena as patches.

    **Patch "Pentagrid"** (single-pass ISF):

    - `Time` into Math nodes for the five LFO sines, times `breath`, plus `base_j`, plus the phason term (`flow` integrated over time, dotted with the family directions), minus the mean. These feed five float inputs gamma0 to gamma4 on the ISF node.
    - `BPM` counter drives ribbonFamily (one step per bar) and ribbonPos (step on snare or every 2 beats).
    - `Audio Analysis` low band into an envelope (attack 10 ms, release 400 ms) adds to `flow`. Kick onset into an envelope (attack 1 ms, release 60 ms) drives `kickFlash`.
    - Exposed: scale, flow, breath, families, ribbonWidth, edgeWidth, palette index.

    **Patch "Penrose Growth"**: time and BPM drive the zoom phase (advance 1 per bar, eased) into the P2 ISF node. Do the trails with Wire's feedback loop and a Transform node (rotate, scale) times decay. Multipass and persistent buffers in ISF depend on the Wire version and the ISF page does not cover them. Exposed: ghost, levels, trails amount, rotation, decay.

    **In Arena**: layer 1 a solid #05060B; layer 2 the Pentagrid patch, Add, 100 %; layer 3 the Growth patch, Add, opacity on the crossfader or a dashboard knob as scene fader. Add Bloom, a light grain and a vignette as composition effects. For a matte, put a video layer above Pentagrid and use the white-edges-only output as a luma mask, so the video shows only along the edges.

    Wire has no vertex shaders, so the instanced-tile approach below does not apply here.

=== "Unreal Engine 5.7"
    Two routes.

    **A, fragment route.** A post-process material, or an unlit material on a screen plane, with a Custom node holding the P1 loop in HLSL. A Material Parameter Collection (`MPC_Penrose`) holds the offsets, scale, family count, ribbon values, edge width and kick flash. A Blueprint or Remote Control preset updates it every tick (LFOs, phason, mean removal on the game thread).

    **B, Niagara tile instances.** One GPU emitter spawns `pairs x (2M+1)^2` particles (M=20, N=5 gives 16,810). Each decodes its ID into a line pair and two line numbers. A Scratch Pad module computes the crossing, the other grid coordinates and the tile centre, and sets a target position, orientation and mesh (thick or thin). A critically damped spring of about 120 ms moves each particle to its target, so a phason flip becomes a visible slide and turn of three tiles, not a pop. Use a Mesh Renderer with two rhombus meshes and an unlit emissive material. The kick can lift tiles in Z for a 3D ripple. Set the scale of off-screen particles to 0 instead of killing them, to keep IDs stable.

    **Growth and trails.** P2 as a second post-process material blended by a Growth parameter. Trails use a render-target ping-pong in a Blueprint.

    **Control and output.** Expose Scale, Flow, Breath, N, RibbonWidth, Growth, Ghost and EdgeWidth in a Remote Control preset; bind OSC routes (for example `/penrose/flow`) and MIDI CCs. Post-process volume: Bloom 1.3 with threshold 0.7, manual exposure so flashes do not pump, film grain 0.02, vignette 0.2. Send out through Spout or NDI at the LED processor's native resolution.

=== "GODOT"
    - **Pentagrid:** a full-viewport `ColorRect` with a `canvas_item` shader holding P1. Uniforms include families, a gamma array, scale, rotation, pan, edge width in pixels, ribbon family, position and width, spark amount, kick flash and the palette. A GDScript `_process` computes LFOs, phason and mean removal, and sets the offsets.
    - **Smooth-flip variant:** a `MultiMeshInstance2D` with one instance per (j, k, n_j, n_k). GDScript computes the tile centres on the CPU (16,810 instances is fine) and lerps each instance transform toward its target. Use two MultiMeshes, thick and thin.
    - **Growth:** a second `ColorRect` with the P2 shader in its own `SubViewport`.
    - **Trails:** two `SubViewport`s ping-ponged. A shader samples the previous frame's `ViewportTexture` with rotation and zoom, multiplies by decay and adds the current edges.
    - **Post:** `WorldEnvironment` with glow on (intensity 1.3, HDR threshold 0.7).
    - **Control:** `InputEventMIDI` maps CCs and notes to the uniforms. For Spout or NDI output use a community addon if one exists for your Godot version.

## On stage

**Audio reactivity.** The kick gets the fast reactions; the tiling moves on slow envelopes. Never drive flow from a fast envelope, or the floor turns to static.

| band | drives | smoothing |
|------|--------|-----------|
| kick, 40–100 Hz onsets | edge flash to white for 60 ms; ghost level in the growth scene; every 4th onset advances the zoom phase | attack 1 ms, release 60 ms (flash), 1 beat (ghost) |
| bass, 60–250 Hz level | flow +0 to +0.3; fill opacity 0.2–0.45 | attack 80 ms, release 600 ms |
| snare, 1.5–4 kHz onsets | ribbon step | eased over 120 ms |
| hi-hat, 6–12 kHz | vertex spark amount 0–3 %, new seed per onset | attack 1 ms, release 80 ms |
| mids, 300 Hz–2 kHz | breath 0.05–0.25 | attack 200 ms, release 1.5 s |

**MIDI.** Suggested mapping for the Arturia gear:

| control | parameter |
|---------|-----------|
| KeyStep knob 1 (smooth 200 ms) | scale, 4–60 tile edges across the height |
| BeatStep Pro knob | flow (phason speed) |
| a fader | breath, automated up in builds |
| BeatStep Pro pads 1–6 | families 3, 4, 5, 6, 7, 9, quantised to the bar |
| snare trigger, a knob for width | ribbon step and width |
| crossfader and kick envelope | scene fader and ghost |

**Timing.** Rotate one full turn per 64 bars in the breathing scene. In the growth scene step 36 degrees per phrase, which keeps the ten-fold star aligned with itself. Ribbon family steps once per bar, so over five bars every edge direction gets a turn. Change palette on 16-bar boundaries, crossfading over a bar.

**Song shape.**

- **Verse:** five families, breath low, slow drift.
- **Build:** go 5 to 7 families, raise breath from 0.05 to 0.4 over 8 bars, ramp flow, bring trails up to 50 %. The floor dissolves into spinning noise.
- **Drop:** hard cut. Breath, flow and trails to 0; cut to the growth scene at a fresh zoom phase with a white edge flash. The tiling snaps sharp.
- **Break:** edges only at 40 %, no fills; scale drifts from 14 to 40 as a zoom out into density.

**Layering.** Use the edge channel as a luma matte: video shows only along the edges, like stained-glass leading. Or use the thin-tile class as an alpha matte; the video shows through about 38 % of the floor, since 10 of every 26.18 tiles are thin (from the 16.18 to 10 ratio in [Minutephysics via Rosetta](https://rosetta.to/u/minutephysics/why-penrose-tiles-never-repeat)).

**Tips.**

- Keep fills dim. Above about 45 % the image flattens into a quilt.
- Skip chromatic aberration: it doubles the thin edges.
- On LED walls keep the black at #05060B and cap edges at 85 % before bloom, so 1 px lines do not clip into blocky white.
- Below about 6 px per tile the edges alias into moire. Fade the edge mask and let fill carry the image.
- Feedback decay above 0.96 never clears; the tiling vanishes into a disc of light. Default is 0.90.
- If a one-frame hole flickers, nudge a base offset by 0.003. At large pan values float32 loses precision, so wrap the pan.
- For multi-screen mappings compute coordinates from the global canvas, so tiles continue across screens.
- Changing the family count mid-bar without a crossfade pops the whole floor.

## Variations

1. **Pentagrid shader:** compute tiles per pixel from line crossings and animate the five offsets with LFOs so the tiling breathes in real time.
2. **Beat-stepped inflation:** the kick advances the subdivision level in a Cooper-style growth with trails.
3. **n-fold switch:** a fader morphs from 5 to 7, 8 or 10 line families ([Minutephysics via Rosetta](https://rosetta.to/u/minutephysics/why-penrose-tiles-never-repeat)).
4. **Prime-loop rhythm:** five or more tile layers with prime-length loops, so the whole mix never repeats, echoing Cooper's track.
5. **Girih palette:** a tilework colourway (base #070A0C, edge #F2E6C9, thick #0E6E6E, thin #1A2A4F, accent #D9B26F), after the Isfahan link.
6. **Quasicrystal diffraction view:** show the ten-fold diffraction pattern next to the tiling, from the Shechtman story.

## Did you know?

- Penrose tilings are 2D slices of 5D hypercubic structures, as de Bruijn showed ([Wikipedia: Quasicrystal](https://en.wikipedia.org/wiki/Quasicrystal)).
- A Penrose-like tiling was found in girih patterns at the 1453 Darb-i Imam shrine ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling)).
- Shechtman waited about two years to publish, and Linus Pauling dismissed his result ([Wikipedia: Quasicrystal](https://en.wikipedia.org/wiki/Quasicrystal), [Veritasium summary](https://summify.io/discover/the-infinite-pattern-that-never-repeats-48sCx-)).
- Every loop length in Max Cooper's Penrose track is a prime number ([Creative Commission](https://creative-commission.com/project/max-cooper/music-video/jessin-0)).

## Sources

- [Wikipedia: Penrose tiling](https://en.wikipedia.org/wiki/Penrose_tiling)
- [Wikipedia: Pentagrid](https://en.wikipedia.org/wiki/Pentagrid)
- [Wikipedia: Quasicrystal](https://en.wikipedia.org/wiki/Quasicrystal)
- [Minutephysics, Why Penrose Tiles Never Repeat (transcript page)](https://rosetta.to/u/minutephysics/why-penrose-tiles-never-repeat)
- [Veritasium summary](https://summify.io/discover/the-infinite-pattern-that-never-repeats-48sCx-)
- [Creative Commission: Max Cooper, Penrose Tiling](https://creative-commission.com/project/max-cooper/music-video/jessin-0)
- [Partyflock: Penrose Tiling](https://partyflock.nl/video/194348:muziekvideo-Penrose-Tiling-Max-Cooper)
- [Wikimedia Commons: Category Penrose tilings](https://commons.wikimedia.org/wiki/Category:Penrose_tilings)
- [Lieven Le Bruyn: de Bruijn's pentagrids](https://lievenlebruyn.github.io/neverendingbooks/de-bruijns-pentagrids)
- [Jeff Preshing: Penrose Tiling Explained](https://preshing.com/20110831/penrose-tiling-explained/)
- [shadcn.io: Penrose tiling shader](https://www.shadcn.io/shaders/penrose-tiling-shader)
- [byewokko/penrose](https://github.com/byewokko/penrose) — Python multigrid implementation, a CPU reference for the instanced route
- [Grimm and Schreiber: Aperiodic Tilings on the Computer](https://arxiv.org/abs/cond-mat/9903010)
- [Resolume ISF support](https://resolume.com/support/en/isf)
