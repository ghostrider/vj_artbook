---
tags:
  - math
  - hyperbolic
  - tiling
  - shader
  - escher
---

# Hyperbolic Poincare Tilings

> An endless ornament inside a round portal: cells shrink toward a hard rim while the disk glides one tile per bar.

## The idea

The Poincare disk draws the whole infinite hyperbolic plane inside a circle. Tiles of equal hyperbolic size look ever smaller toward the rim, so the pattern never runs out. Edges are circle arcs that meet the rim at right angles ([Dunham](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)).

Eugenio Beltrami found the disk and half-plane models in the late 1860s. In 1957 M. C. Escher received a paper by H. S. M. Coxeter with a {6,4} triangle pattern, which gave him "quite a shock" ([Dunham](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)). The Circle Limit prints followed, drawn by hand.

On a club screen the disk works as an object and as a tunnel. A black surround makes it a portal. Constant motion toward the centre gives the sense of falling inward. Tiling type, glide and rotation all map to knobs and BPM, so the ornament can follow the music bar by bar.

## Origins and references

- [Dunham, Escher's use of the Poincare models](https://www.math-art.eu/Documents/pdfs/Dunham.pdf) — the disk model, conformality and Escher's Circle Limit history.
- [Sanderson, Bridges 2024](https://archive.bridgesmathart.org/2024/bridges2024-4.html) — retells how Escher's contact with mathematicians (Polya, the 1954 Congress, Coxeter) led to the Circle Limit series.
- [Circle Limit III (1959), National Gallery of Art](https://www.nga.gov/artworks/61280-circle-limit-iii) — woodcut printed from five blocks; interlocking fish whose mouths meet in three-pointed stars. Copyrighted, so link only.
- [Cook, Hyperbolic Poincare tiling of triangles](https://library.wolfram.com/infocenter/Demos/148) — Mathematica notebook that renders pictures and animations of a triangle tiling.
- [EscherSketch](https://www.github.com/looeee/hyperbolic-tiling) — open-source WebGL prototype that maps two fish triangle images onto a two-coloured {p,q} tiling. The computer algorithm was first described by Douglas Dunham and first implemented by his PhD student Ajit Datar.
- [Poincare meets Mandelbrot](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html) — Mandelbrot and Julia sets carried by a {7,3}-based tiling.
- [hypertiling](https://gitpages.physik.uni-wuerzburg.de/hypertiling/hyperweb/) — Python library for regular hyperbolic lattices with millions of vertices.
- [Fast layer-by-layer construction, arXiv 2508.04765](https://arxiv.org/html/2508.04765v1) — builds cells at nanoseconds per cell on the CPU.

## Visual language

Tilings are named {p,q}: regular p-gons, q meeting at each vertex. The plane is hyperbolic when (p-2)(q-2) > 4. Euclidean space fits only six triangles around a vertex; hyperbolic space allows seven, eight, 50 or infinitely many ([Scientific American](https://www.scientificamerican.com/blog/roots-of-unity/everything-looks-better-in-the-hyperbolic-plane/)).

Picture it: large cells at the centre, shrinking to a fine dust of arcs at the rim. Edges are curved, never straight. Symmetry is strict and radial. The look is a fish-eye view of a floor that has no end.

Colour gives rhythm. A two-tone checker in deep teal and warm brass echoes Escher's black and white fish. Thin bone-white edge lines draw the grid. A faint violet glow sits on the rim. Cells fade into darkness with depth. Outside the disk is true black.

Motion is the glide. The disk travels one cell per bar, so new cells pour in from the rim and old ones sink back into it. On the kick the edges flash. On the drop the tiling changes, for example {7,3} to {4,6}, and the cells fill with a kaleidoscope of live video.

## See it

- [Circle Limit III, NGA](https://www.nga.gov/artworks/61280-circle-limit-iii) — Escher's woodcut (copyrighted, link only).
- [EscherSketch](https://www.github.com/looeee/hyperbolic-tiling) — WebGL fish tiling in the Poincare disk.
- [Poincare meets Mandelbrot](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html) — fractal content in a {7,3}-based tiling.
- [Cook, Poincare tiling](https://library.wolfram.com/infocenter/Demos/148) — triangle tiling pictures and animations (notebook download).
- [hypertiling](https://gitpages.physik.uni-wuerzburg.de/hypertiling/hyperweb/) — lattice plots and animation in the disk.

## Code design

No drawing of polygons. One fragment shader walks every pixel back into a single fundamental triangle and colours it from there.

1. **Three mirrors.** Reflect a triangle with angles pi/p, pi/q and pi/2 across its edges. Two mirrors are straight lines through the centre. The third is a circle orthogonal to the unit circle, so reflecting in it is a circle inversion ([westy31.nl](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html)). Its centre and radius are solved once per tiling.
2. **Folding.** Per pixel, loop up to about 40 times: mirror in the x-axis, mirror in the second line, invert in the circle. Stop when nothing fires. Parity of flips gives the two-tone checker. The count of inversions is a rough distance from the centre and darkens far cells. Pixels that hit the cap get the rim colour.
3. **Moebius camera.** Rotation and translation of the disk are applied to the pixel before folding. One cell per bar loops seamlessly; for odd p, add a half-turn so the cut is invisible. A rotation of 2pi/p per bar or phrase is always a symmetry.
4. **Edges.** Lines have constant hyperbolic width, so they thin toward the rim like Escher's fish. Fade lines thinner than one pixel instead of drawing them aliased, which kills moire.
5. **Motif.** Every cell is a mirrored copy of the fundamental triangle. Sampling live video at the folded point paints a kaleidoscope into every cell. A Julia iteration on the folded point gives the fractal variant.

Layers, back to front: black background, rim glow (Add), tile fill (Normal), video-in tiles (Multiply, then Normal on the drop), edges (Add), vertex stars (Add), optional trails (Add, 0 to 40 %), optional disk matte over Spout, then post (trails mix, bloom, chromatic aberration, grain).

Palette: teal #0E3B43, brass #C8963E, bone #F2EBDD, rose #E0457B, violet rim #3A1D5C, black outside.

=== "Resolume Wire"
    - Run the tiling as one ISF node inside Wire. Resolume documents ISF only in Wire ([Resolume ISF support](https://resolume.com/support/en/isf)). Use the patch as a source in Arena, blend mode Alpha.
    - Clock: a BPM bar-phase node drives `glidePhase`; a second phase per 8-bar phrase drives `turnPhase`.
    - p and q come from a lookup indexed by a `tiling` parameter. The r, d, v maths is cheap enough to run per pixel inside the ISF.
    - Motif input from a Spout Receiver or NDI node. Float inputs with MIN/MAX for line width, depth fade, motif, edge gain, star gain, hue shift, texture zoom and disk size.
    - Trails: a second ISF node with a PERSISTENT, FLOAT buffer ([ISF multipass reference](https://docs.isf.video/ref_multipass.html)). Resolume's ISF page does not cover multipass, so test it and fall back to Wire's feedback node if needed.
    - Audio: low band into an envelope (attack 5 ms, release 120 ms), times `pulse`, into edge gain.
    - Published: tiling, glide, turn, depth, motif, pulse, lineWidth, hueShift. Optional Spout Sender for the disk mask. In Arena, add Bloom, a small Chromatic Aberration and 3 % grain; check names in your build. Parameters are reachable through the [Resolume REST API](https://resolume.com/docs/restapi/).

=== "Unreal Engine 5.7"
    - Unlit material on a full-screen plane or as a post-process material. The whole folding loop lives in one Custom HLSL node; the rest is plain material nodes.
    - Scalars from a Material Parameter Collection (`MPC_Hyper`): P, Q, glide phase, turn phase, line width, depth fade, motif, edge gain, hue shift.
    - Motif: a Media Texture or Spout receiver texture, UV from the Custom node.
    - Blueprint accumulates bar phase from BPM on tick and writes it to the MPC. A small array maps tiling index to P and Q.
    - Remote Control Preset exposes tiling, glide, turn, depth, motif, pulse and BPM to OSC (for example `/hyper/glide`) and MIDI CCs.
    - Post-process: bloom, film grain, fixed exposure so the black surround stays black. Output by Spout or NDI into Resolume.
    - Optional: a GPU sprite emitter in Niagara that spawns sparks where an edge-mask render target is above 0.5. A SceneCapture2D can give a fisheye dome output.

=== "GODOT"
    - A full-viewport ColorRect with a `canvas_item` shader that runs the fold pass.
    - Uniforms: int p and q, glide and turn phase, line width, depth fade, motif, edge gain, hue shift, five colours with the `source_color` hint, and a motif sampler from a VideoStreamPlayer or CameraTexture.
    - Trails: a SubViewport with Clear Mode Never that draws `max(edges, previous * decay)`. Ping-pong two SubViewports if Godot complains about a feedback loop.
    - Script: compute bar phase from BPM in `_process`. Map MIDI through `InputEventMIDI`, with CC to glide, turn, depth and motif, and notes to tiling.

## On stage

**Audio reactivity**

- Kick (40 to 120 Hz): edge gain 1.0 to 2.2 and line width +30 %, attack 5 ms, release 120 ms.
- Bass (60 to 250 Hz, RMS): rotation speed +/- 20 %, slow smoothing.
- Snare (1 to 3 kHz): optional half-cell push on beats 2 and 4, with a 200 ms retrigger lockout.
- Hats (6 to 12 kHz): light 5 to 15 % of the vertex stars through a hash of the cell ID.
- Loudness: depth fade, so louder music shows cells further toward the rim.

Keep tiling changes manual or on phrase boundaries. Audio-driven changes of {p,q} look random.

**MIDI mapping**

| control | parameter |
|---|---|
| BeatStep Pro pad row, or KeyStep keys C3 to G3 | `tiling`, one pad per pair, snapped |
| KeyStep knob | `glide`, 0 to 4 cells per bar |
| Knob with centre detent | `turn`, centre stops the rotation |
| Fader | `depth`, automate it rising over a build |
| Fader | `motif`, jump to 1 on the drop |
| Knob | `pulse`, scales how hard the kick flashes |

**Arc of a set**

- Intro: motif off, one cell per bar, teal and brass only.
- Build: depth fade sinks from 0.3 to 0.05 over 8 bars, so cells climb out of the dark toward the rim.
- Break: glide slows to one cell per 4 bars, trails up to 35 %, depth fade high. A small calm mandala remains.
- Drop: on the downbeat, cut or crossfade over one beat to the next {p,q} pair, motif to 100 %, glide back to one or two cells per bar.
- Every phrase, swap colours A and B over one beat. The checker inverts, a strong but cheap change.

**Layering**

- Send the disk or its parity channel over Spout as a matte, so another layer shows only inside the disk or only in odd cells.
- Use the rim as a vignette.
- Feed a camera or clip into the tiles.

**Tips**

- Keep cell fill under 70 % luminance on LED walls, so the edges carry the image.
- Keep the soft disk edge. A hard circle aliases on LED pixels.
- Enable mipmaps on the motif texture, or rim cells shimmer.
- Keep p and q at 8 or below. Larger values give thin, stretched cells; EscherSketch warns about the same above about 8 ([GitHub](https://www.github.com/looeee/hyperbolic-tiling)).
- On a wide LED wall the disk leaves black sides. Use two or three disks side by side, the half-plane model, or the disk as a mask for a wide second layer.
- A static centred disk gets boring. Keep it gliding, change tilings on drops and put texture in the tiles.
- To zoom, translate instead of enlarging the disk past the screen. Precision falls apart near the rim.

## Variations

1. Video-in tiles: fold a live feed through the tiling instead of flat colours.
2. Fractal tiles: Mandelbrot or Julia content inside the cells, as in [Poincare meets Mandelbrot](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html).
3. Tiling morph: step through {p,q} pairs on a MIDI fader with short crossfades.
4. Off-disk projection: map onto a dome, or use the disk as a mask for a second layer.
5. Poincare half-plane or Klein variants for a flatter, brick-like fly-through.

## Did you know?

- The {7,3} heptagon tiling has symmetry group (2,3,7), and Klein's quartic can be tiled by 24 regular heptagons ([AMS Visual Insight](https://blogs.ams.org/visualinsight/wp-json/wp/v2/posts/946)).
- Escher drew the hyperbolic patterns by hand, because motif sizes vary continuously ([Dunham](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)).
- Heptagon counts in the westy31 construction follow Fibonacci numbers by generation: 1 + 7(F(2N+1) - 1) ([westy31.nl](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html)).
- All fish in Circle Limit III have the same hyperbolic size; they only look smaller near the rim ([Dunham](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)).

## Sources

- [Bridges 2024, Sanderson](https://archive.bridgesmathart.org/2024/bridges2024-4.html)
- [Dunham, Escher's use of the Poincare models](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)
- [EscherSketch](https://www.github.com/looeee/hyperbolic-tiling)
- [NGA, Circle Limit III](https://www.nga.gov/artworks/61280-circle-limit-iii)
- [AMS Visual Insight, {7,3} tiling](https://blogs.ams.org/visualinsight/wp-json/wp/v2/posts/946)
- [Scientific American, Roots of Unity](https://www.scientificamerican.com/blog/roots-of-unity/everything-looks-better-in-the-hyperbolic-plane/)
- [westy31.nl, Poincare meets Mandelbrot](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html)
- [hypertiling](https://gitpages.physik.uni-wuerzburg.de/hypertiling/hyperweb/)
- [arXiv 2508.04765](https://arxiv.org/html/2508.04765v1)
- [Wolfram Library, Cook](https://library.wolfram.com/infocenter/Demos/148)
- [Resolume ISF support](https://resolume.com/support/en/isf)
- [Resolume REST API](https://resolume.com/docs/restapi/)
- [ISF multipass reference](https://docs.isf.video/ref_multipass.html)
